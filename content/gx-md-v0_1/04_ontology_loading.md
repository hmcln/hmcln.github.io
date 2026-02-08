# 04_ontology_loading.md

## Table of Contents
- [1. Ontology File Discovery](#1-ontology-file-discovery)
- [2. serde Data Model](#2-serde-data-model)
- [3. Parsing TOML and YAML](#3-parsing-toml-and-yaml)
- [4. Self-Validation of Ontology](#4-self-validation-of-ontology)
- [5. Why this matters](#5-why-this-matters)

## 1. Ontology File Discovery
Support exactly one of `gx.toml` or `gx.yaml` at repository root.

```rust
pub fn find_ontology_file(root: &Path) -> Result<PathBuf, GxError> {
    let toml = root.join("gx.toml");
    let yaml = root.join("gx.yaml");

    match (toml.exists(), yaml.exists()) {
        (true, false) => Ok(toml),
        (false, true) => Ok(yaml),
        (false, false) => Err(GxError::OntologyMissing),
        (true, true) => Err(GxError::OntologyAmbiguous),
    }
}
```

## 2. serde Data Model
Prefer explicit structs over loose maps for schema stability.

```rust
use serde::Deserialize;
use std::collections::BTreeMap;

#[derive(Debug, Deserialize)]
pub struct Ontology {
    pub types: BTreeMap<String, TypeDef>,
}

#[derive(Debug, Deserialize)]
pub struct TypeDef {
    #[serde(default)]
    pub allowed_outgoing: Vec<RelationDef>,
}

#[derive(Debug, Deserialize)]
pub struct RelationDef {
    pub rel: String,
    pub target: String,
    #[serde(default)]
    pub required: bool,
    #[serde(default = "default_min")]
    pub min: usize,
    pub max: Option<usize>,
}

fn default_min() -> usize { 0 }
```

`BTreeMap` is chosen over `HashMap` for deterministic iteration order, which improves stable diagnostics and index output.

## 3. Parsing TOML and YAML
Use file extension dispatch:

```rust
pub fn load_ontology(root: &Path) -> Result<Ontology, GxError> {
    let path = find_ontology_file(root)?;
    let raw = std::fs::read_to_string(&path)?;

    let ontology = match path.extension().and_then(|s| s.to_str()) {
        Some("toml") => toml::from_str(&raw)?,
        Some("yaml") | Some("yml") => serde_yaml::from_str(&raw)?,
        _ => return Err(GxError::UnsupportedOntologyFormat(path)),
    };

    validate_ontology(&ontology)?;
    Ok(ontology)
}
```

## 4. Self-Validation of Ontology
Check ontology consistency before scanning content:

- every `target` type exists,
- min/max cardinality is valid,
- relation names are non-empty.

```rust
pub fn validate_ontology(onto: &Ontology) -> Result<(), GxError> {
    for (type_name, def) in &onto.types {
        for r in &def.allowed_outgoing {
            if r.rel.trim().is_empty() {
                return Err(GxError::InvalidOntology(format!("empty relation in {type_name}")));
            }
            if !onto.types.contains_key(&r.target) {
                return Err(GxError::InvalidOntology(format!(
                    "relation {} in {} points to unknown target {}",
                    r.rel, type_name, r.target
                )));
            }
            if let Some(max) = r.max {
                if r.min > max {
                    return Err(GxError::InvalidOntology(format!(
                        "invalid cardinality {}.{}: min {} > max {}",
                        type_name, r.rel, r.min, max
                    )));
                }
            }
        }
    }
    Ok(())
}
```

## 5. Why this matters
If ontology loading is weak, every later stage produces confusing errors. Early strict schema validation gives precise failures and a predictable compiler contract.

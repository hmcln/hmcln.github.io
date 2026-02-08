# 11_index_json.md

## Table of Contents
- [1. Designing the Output IR](#1-designing-the-output-ir)
- [2. Serialization with serde](#2-serialization-with-serde)
- [3. Lookup Table Design](#3-lookup-table-design)
- [4. Stability Guarantees](#4-stability-guarantees)
- [5. Why this matters](#5-why-this-matters)

## 1. Designing the Output IR
Define a canonical schema independent from internal structs.

```rust
#[derive(Debug, serde::Serialize)]
pub struct IndexJson {
    pub version: String,
    pub nodes: Vec<IndexNode>,
    pub edges: Vec<IndexEdge>,
    pub by_type: BTreeMap<String, Vec<String>>,
}

#[derive(Debug, serde::Serialize)]
pub struct IndexNode {
    pub id: String,
    #[serde(rename = "type")]
    pub type_name: String,
    pub path: String,
}

#[derive(Debug, serde::Serialize)]
pub struct IndexEdge {
    pub source: String,
    pub rel: String,
    pub target: String,
}
```

## 2. Serialization with serde
Transform graph to index IR and serialize with pretty JSON for reviewability.

```rust
pub fn write_index(path: &Path, index: &IndexJson) -> Result<(), GxError> {
    let bytes = serde_json::to_vec_pretty(index)?;
    std::fs::write(path, bytes)?;
    Ok(())
}
```

## 3. Lookup Table Design
`by_type` improves downstream query speed without recomputing grouping.

Build deterministically:

- iterate sorted nodes,
- push IDs in order,
- avoid hash-order instability.

## 4. Stability Guarantees
Canonical index guarantees:

- stable field names,
- sorted arrays,
- explicit `version` string (`"0.1"`).

When schema changes, bump version and document migration behavior.

## 5. Why this matters
`index.json` is the contract for downstream consumers. If unstable, every consumer becomes brittle.

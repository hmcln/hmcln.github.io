# 06_frontmatter_parsing.md

## Table of Contents
- [1. Frontmatter Contract](#1-frontmatter-contract)
- [2. Parsing Strategy](#2-parsing-strategy)
- [3. Structs vs Dynamic Maps](#3-structs-vs-dynamic-maps)
- [4. Friendly Error Design](#4-friendly-error-design)
- [5. Why this matters](#5-why-this-matters)

## 1. Frontmatter Contract
Each Markdown file must provide:

- `id`: globally unique node identifier
- `type`: ontology type name

Optional fields may be added later, but v0.1 should enforce required keys.

## 2. Parsing Strategy
Parse YAML frontmatter delimited by `---` at file start.

```rust
#[derive(Debug)]
pub struct ParsedDoc {
    pub header: Frontmatter,
    pub body: String,
}

#[derive(Debug, serde::Deserialize)]
pub struct Frontmatter {
    pub id: String,
    #[serde(rename = "type")]
    pub type_name: String,
}

pub fn parse_frontmatter(src: &SourceFile) -> Result<ParsedDoc, Diagnostic> {
    let text = src.text.as_str();
    if !text.starts_with("---
") {
        return Err(Diagnostic::error(src.path.clone(), "missing YAML frontmatter"));
    }

    let mut parts = text[4..].splitn(2, "
---
");
    let yaml = parts.next().unwrap_or("");
    let body = parts.next().unwrap_or("").to_string();

    let header: Frontmatter = serde_yaml::from_str(yaml)
        .map_err(|e| Diagnostic::error(src.path.clone(), format!("invalid frontmatter: {e}")))?;

    Ok(ParsedDoc { header, body })
}
```

## 3. Structs vs Dynamic Maps
Why not `HashMap<String, Value>`?

- Structs provide compiler-checked required fields.
- Renames and defaults are explicit through serde attributes.
- Validation and diagnostics are easier because types are known.

Use maps only for intentionally open metadata sections.

## 4. Friendly Error Design
Design diagnostics with context and possible fix:

- include file path,
- include missing field name,
- suggest expected shape.

```rust
Diagnostic::error(path, "frontmatter is missing required field `id`")
    .with_help("add: id: your-stable-node-id")
```

## 5. Why this matters
Frontmatter is where human-authored Markdown first becomes typed data. Strong parsing and clear diagnostics reduce author friction and prevent graph corruption.

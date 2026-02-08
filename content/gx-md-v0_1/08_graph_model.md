# 08_graph_model.md

## Table of Contents
- [1. Graph Type Design](#1-graph-type-design)
- [2. Ownership Strategy](#2-ownership-strategy)
- [3. Normalization Rules](#3-normalization-rules)
- [4. Avoiding Cycles with IDs](#4-avoiding-cycles-with-ids)
- [5. Why this matters](#5-why-this-matters)

## 1. Graph Type Design
Use stable IDs and explicit edge records.

```rust
use std::collections::BTreeMap;

pub type NodeId = String;

#[derive(Debug, Clone)]
pub struct Node {
    pub id: NodeId,
    pub type_name: String,
    pub source_path: PathBuf,
}

#[derive(Debug, Clone)]
pub struct Edge {
    pub source: NodeId,
    pub rel: String,
    pub target: NodeId,
}

#[derive(Debug, Default)]
pub struct Graph {
    pub nodes: BTreeMap<NodeId, Node>,
    pub edges: Vec<Edge>,
}
```

## 2. Ownership Strategy
Nodes and edges own strings to avoid lifetime coupling with parsed source buffers. This may duplicate some data, but keeps APIs simple and serialization direct.

When performance tuning later, intern frequently repeated strings (`rel`, `type_name`) with a symbol table.

## 3. Normalization Rules
Normalize during graph construction:

- trim IDs,
- reject empty IDs,
- canonicalize relation names,
- sort edges for deterministic output.

```rust
pub fn normalize_rel(rel: &str) -> String {
    rel.trim().to_ascii_lowercase()
}
```

## 4. Avoiding Cycles with IDs
Do not embed target `Node` inside `Edge` with references. Keep `Edge.target: NodeId` and resolve through map lookups.

Why not `Rc<Node>`?

- hard to serialize,
- encourages implicit graph ownership webs,
- can create cycles unless carefully managed with `Weak`.

IDs keep the data model explicit and compiler-like.

## 5. Why this matters
Graph design determines how easy validation, diagnostics, and index serialization will be. ID-based normalized graphs are robust and maintainable for v0.1.

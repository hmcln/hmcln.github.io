# 14_future_work.md

## Table of Contents
- [1. LSP Integration](#1-lsp-integration)
- [2. Incremental Indexing](#2-incremental-indexing)
- [3. Query Engine](#3-query-engine)
- [4. Export Formats](#4-export-formats)
- [5. Why this matters](#5-why-this-matters)

## 1. LSP Integration
Future `gx-lsp` could reuse `gx-core` for live diagnostics in editors.

Design requirement now: keep core free from terminal-specific formatting, and expose structured diagnostics with spans.

## 2. Incremental Indexing
Incremental mode would hash file content and ontology, then reuse cached parse/graph fragments.

Potential approach:

- content hash map keyed by path,
- dependency map keyed by node ID,
- invalidate affected nodes/edges only.

This demands deterministic node IDs and normalized graph shape, both already built in v0.1 architecture.

## 3. Query Engine
A read-only query layer could support semantic lookups:

- neighbors by relation,
- reverse relations,
- typed traversal.

Possible API in Rust:

```rust
pub fn outgoing<'a>(g: &'a Graph, node: &str, rel: &str) -> impl Iterator<Item = &'a Edge> {
    g.edges.iter().filter(move |e| e.source == node && e.rel == rel)
}
```

Borrowing is important here: iterator returns references to edges, avoiding allocations.

## 4. Export Formats
Beyond JSON, useful exports include:

- JSON Lines for streaming,
- GraphML for graph tooling,
- SQLite for large-scale queries.

Schema versioning must remain explicit across all exports.

## 5. Why this matters
Future features are easier when v0.1 has clear ownership boundaries, structured errors, normalized IR, and stable contracts. That is why each architectural choice in this manual is conservative and explicit.

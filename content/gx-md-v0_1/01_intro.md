# 01_intro.md

## Table of Contents
- [0. Build Path First](#0-build-path-first)
- [1. Purpose of gx.md](#1-purpose-of-gxmd)
- [2. Compiler Analogy](#2-compiler-analogy)
- [3. Mental Model](#3-mental-model)
- [4. Why Rust Fits This Problem](#4-why-rust-fits-this-problem)
- [5. Why this matters](#5-why-this-matters)

## 0. Build Path First
Before reading this chapter deeply, start with `00_build_workshop.md`.

That file gives the practical sequence:

- exact commands to run,
- milestone-by-milestone success checks,
- a path from learning to a working product.

## 1. Purpose of gx.md

gx.md v0.1 is a command-line compiler and type-checker for Markdown knowledge bases. It is not a publishing tool. It reads content and metadata, verifies semantic correctness against an ontology, emits diagnostics, and writes a canonical `index.json` artifact.

The scope is intentionally narrow:

- Input: repository-local ontology (`gx.toml` or `gx.yaml`) and Markdown files.
- Core transformation: Markdown documents become typed graph nodes and edges.
- Validation: ontology rules are enforced against graph content.
- Output: diagnostics and a stable machine-readable index.

This is the same structural shape as a compiler pipeline: parse input, build intermediate representation, run semantic checks, emit artifacts.

## 2. Compiler Analogy
Think in familiar compiler phases:

1. **Configuration load**: read ontology schema. Comparable to loading language rules.
2. **Lex/parse**: frontmatter + Markdown AST parse. Comparable to parsing source files.
3. **IR construction**: build a typed directed graph. Comparable to AST lowering into typed IR.
4. **Type/semantic checks**: unknown types, missing required relations, invalid targets, duplicate IDs.
5. **Diagnostics + emit**: emit structured errors and `index.json`.

The analogy is useful because it keeps implementation disciplined. If every stage receives and produces explicit types, the CLI stays maintainable and testable.

## 3. Mental Model
Design the system around immutable stage outputs.

```rust
pub struct PipelineOutput {
    pub graph: Graph,
    pub diagnostics: Vec<Diagnostic>,
}

pub fn run_pipeline(repo_root: &Path, mode: Mode) -> Result<PipelineOutput, GxError> {
    // 1. load ontology
    // 2. scan markdown files
    // 3. parse frontmatter + markdown ast
    // 4. extract links + build graph
    // 5. validate graph against ontology
    // 6. maybe serialize index
    unimplemented!()
}
```

Important ownership decision: stage outputs own their data (`Graph`, `Vec<Diagnostic>`), while functions borrow inputs where possible (`&Path`, `&Ontology`). This avoids accidental shared mutable state. Instead of retaining references between stages, pass IDs (for example `NodeId`) and lookup through maps.

Why IDs over `Rc`/`Arc` for graph relations?

- IDs serialize naturally into JSON.
- IDs avoid reference cycles and complicated lifetime graphs.
- IDs make validation deterministic and easier to unit test.

Use `Arc` only for shared read-only structures that are expensive to clone and truly cross-thread. In v0.1, single-threaded deterministic behavior usually makes plain ownership simpler.

## 4. Why Rust Fits This Problem
Rust is a strong fit for a compiler-like CLI because it supports:

- **Type-first architecture**: each phase can have explicit input/output structs.
- **Ownership clarity**: files, parsed documents, and graph nodes can be moved intentionally without hidden copies.
- **Reliable error modeling**: use domain error enums for expected failures; reserve `anyhow` for top-level glue where error context stacking is useful.
- **Performance predictability**: scanning and parsing many Markdown files benefits from low overhead.
- **Binary distribution**: one static CLI binary is practical for CI and developer workflows.

Error design guidance for this project:

- Use specific error enums (`enum GxError`) in core crates so callers can branch.
- Use structured diagnostics for user-facing issues (unknown type, duplicate ID).
- Avoid collapsing domain errors into opaque strings too early.

## 5. Why this matters
This chapter defines the contract: gx.md is a compiler pipeline with strict boundaries and typed intermediate data. If you hold this model, later decisions about module layout, ownership, and error handling become straightforward rather than reactive.

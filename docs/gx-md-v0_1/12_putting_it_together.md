# 12_putting_it_together.md

## Table of Contents
- [1. End-to-End Pipeline](#1-end-to-end-pipeline)
- [2. `gx check` Flow](#2-gx-check-flow)
- [3. `gx index` Flow](#3-gx-index-flow)
- [4. Performance Considerations](#4-performance-considerations)
- [5. Why this matters](#5-why-this-matters)

## 1. End-to-End Pipeline
Compose previously defined stages as pure functions where possible.

```rust
pub fn compile_repo(root: &Path) -> Result<CompileResult, GxError> {
    let ontology = load_ontology(root)?;
    let paths = collect_markdown_paths(root)?;
    let sources = read_sources(&paths)?;
    let docs = parse_docs(&sources);
    let graph = build_graph(&docs);
    let diagnostics = validate_all(&ontology, &graph);
    Ok(CompileResult { graph, diagnostics })
}
```

## 2. `gx check` Flow
1. Run `compile_repo`.
2. Emit diagnostics.
3. Return failing exit code if any `Error` severity exists.

No `index.json` is written in this command.

## 3. `gx index` Flow
1. Run `compile_repo`.
2. If no error diagnostics, transform graph to `IndexJson`.
3. Write output file.

Policy question: write partial index on errors? v0.1 should avoid it to preserve trust in artifact validity.

## 4. Performance Considerations
Early practical improvements:

- Avoid reparsing unchanged files in future versions (incremental cache).
- Reserve vector capacities when approximate sizes are known.
- Use borrowed parsing where possible; allocate owned strings only at graph boundary.

Parallelism note: parse stage could be parallelized later with Rayon, but deterministic diagnostics ordering must be preserved by sorting final results.

## 5. Why this matters
This chapter operationalizes architecture into command behavior. It is where compiler theory becomes practical CLI behavior.

# 02_project_layout.md

## Table of Contents
- [1. Workspace Structure](#1-workspace-structure)
- [2. Crate and Module Boundaries](#2-crate-and-module-boundaries)
- [3. `lib.rs` vs `main.rs`](#3-librs-vs-mainrs)
- [4. Visibility and API Design](#4-visibility-and-api-design)
- [5. Why this matters](#5-why-this-matters)

## 1. Workspace Structure
Use a Cargo workspace so compiler logic is reusable and testable independently of the CLI wrapper.

```text
gx/
├─ Cargo.toml                 # workspace manifest
├─ crates/
│  ├─ gx-core/                # parsing, graph, validation, diagnostics
│  │  └─ src/
│  │     ├─ lib.rs
│  │     ├─ ontology.rs
│  │     ├─ scan.rs
│  │     ├─ frontmatter.rs
│  │     ├─ markdown.rs
│  │     ├─ graph.rs
│  │     ├─ validate.rs
│  │     ├─ diagnostics.rs
│  │     └─ index.rs
│  └─ gx-cli/
│     └─ src/
│        └─ main.rs
└─ docs/
```

Workspace `Cargo.toml`:

```toml
[workspace]
members = ["crates/gx-core", "crates/gx-cli"]
resolver = "2"
```

## 2. Crate and Module Boundaries
Recommended ownership of responsibilities:

- `gx-core`: pure compiler engine. No terminal coloring assumptions, no process exit calls.
- `gx-cli`: argument parsing, command dispatch, printing diagnostics, exit codes.

Why strict boundary? It allows:

- integration tests against core logic without shell process overhead,
- future reuse from LSP service or build tooling,
- easier refactor when CLI behavior changes.

Rust concept introduced here: module visibility.

```rust
// crates/gx-core/src/lib.rs
pub mod diagnostics;
pub mod frontmatter;
pub mod graph;
pub mod index;
pub mod markdown;
pub mod ontology;
pub mod scan;
pub mod validate;

mod internal; // private helpers; not part of public API
```

Prefer exposing small stable APIs (`pub fn run_check(...)`) rather than exposing internal parser details.

## 3. `lib.rs` vs `main.rs`
`lib.rs` defines reusable library APIs.

```rust
// crates/gx-core/src/lib.rs
pub fn check_repo(root: &Path) -> Result<Vec<Diagnostic>, GxError> {
    unimplemented!()
}
```

`main.rs` should be thin:

```rust
fn main() {
    if let Err(err) = gx_cli::run() {
        eprintln!("{err}");
        std::process::exit(2);
    }
}
```

This split improves ownership boundaries: CLI owns process lifecycle; core owns domain behavior.

## 4. Visibility and API Design
Rules that prevent accidental complexity:

1. Keep most structs `pub(crate)` first. Make them public only when needed.
2. Hide concrete parser internals; expose typed outputs.
3. Return iterators when streaming is useful, but return owned collections where API simplicity is more important.

Iterator pattern example:

```rust
pub fn markdown_files(root: &Path) -> impl Iterator<Item = PathBuf> {
    // build and return iterator pipeline
    std::iter::empty()
}
```

For beginners, iterator-heavy signatures can be harder to debug. In v0.1, returning `Vec<PathBuf>` is acceptable if it keeps control flow explicit.

## 5. Why this matters
A clean workspace layout is the foundation for every later chapter. Without strict crate boundaries, error handling and ownership become tangled, and the compiler pipeline is harder to evolve safely.

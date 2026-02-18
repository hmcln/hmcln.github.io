# 00_build_workshop.md

## Table of Contents
- [1. What You Will Build](#1-what-you-will-build)
- [2. Prerequisites](#2-prerequisites)
- [3. Working Method](#3-working-method)
- [4. Milestone Plan](#4-milestone-plan)
- [5. Step-by-Step Build Guide](#5-step-by-step-build-guide)
- [6. Endgame and Demo](#6-endgame-and-demo)

## 1. What You Will Build
By the end of this guide you will have a working Rust CLI called `gx` with:

- `gx check` to validate Markdown knowledge files against ontology rules.
- `gx index --out index.json` to emit a canonical graph index.
- deterministic diagnostics for CI/editor use.

This file is a practical execution track that pairs with the conceptual chapters (`01` to `14`).

## 2. Prerequisites
Run these once before starting:

```bash
rustc --version
cargo --version
```

Expected: both commands print versions and exit successfully.

Create your project workspace:

```bash
mkdir -p gx && cd gx
```

## 3. Working Method
Use this loop for every milestone:

1. Read the linked chapter(s).
2. Create or edit the listed files.
3. Run the listed command(s).
4. Confirm the milestone checks.
5. Commit before moving to the next milestone.

Recommended commit style:

```bash
git add .
git commit -m "milestone N: <short summary>"
```

## 4. Milestone Plan
- Milestone 0: Scaffold workspace and crates (`02`, `03`).
- Milestone 1: Ontology loading and validation (`04`).
- Milestone 2: Markdown scanning and frontmatter parsing (`05`, `06`).
- Milestone 3: Markdown AST link extraction and graph model (`07`, `08`).
- Milestone 4: Validation engine and diagnostics (`09`, `10`).
- Milestone 5: Index JSON emission and command wiring (`11`, `12`).
- Milestone 6: Next.js integration and CI flow (`13`).

## 5. Step-by-Step Build Guide

### Milestone 0: Workspace + CLI skeleton
Read: `02_project_layout.md`, `03_cli_entrypoint.md`

Commands:

```bash
mkdir -p crates/gx-core/src crates/gx-cli/src
cat > Cargo.toml <<'TOML'
[workspace]
members = ["crates/gx-core", "crates/gx-cli"]
resolver = "2"
TOML

cat > crates/gx-core/Cargo.toml <<'TOML'
[package]
name = "gx-core"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"
serde_yaml = "0.9"
toml = "0.8"
thiserror = "1"
walkdir = "2"
pulldown-cmark = "0.10"
TOML

cat > crates/gx-cli/Cargo.toml <<'TOML'
[package]
name = "gx-cli"
version = "0.1.0"
edition = "2021"

[dependencies]
clap = { version = "4", features = ["derive"] }
gx-core = { path = "../gx-core" }
thiserror = "1"
serde_json = "1"
TOML
```

Create `crates/gx-cli/src/main.rs`:

```rust
fn main() {
    println!("gx bootstrap");
}
```

Checks:

```bash
cargo check
cargo run -p gx-cli
```

Success criteria:

- `cargo check` succeeds.
- CLI runs and prints `gx bootstrap`.

### Milestone 1: Ontology loader
Read: `04_ontology_loading.md`

Implement in `crates/gx-core/src/ontology.rs`:

- `find_ontology_file`
- `load_ontology`
- `validate_ontology`
- ontology data structs

Wire module in `crates/gx-core/src/lib.rs`:

```rust
pub mod ontology;
```

Add a sample ontology at repo root (`gx.toml`):

```toml
[types.topic]
allowed_outgoing = [
  { rel = "depends_on", target = "topic", required = false, min = 0 }
]
```

Checks:

```bash
cargo check
```

Success criteria:

- Missing ontology produces a typed error.
- Invalid target type in ontology is caught by self-validation.

### Milestone 2: Scan and parse docs
Read: `05_markdown_scanning.md`, `06_frontmatter_parsing.md`

Implement:

- `scan.rs`: discover Markdown files.
- `frontmatter.rs`: parse YAML header and body.

Create sample content:

```bash
mkdir -p content
cat > content/a.md <<'MD'
---
id: topic-a
type: topic
---
# Topic A
MD
```

Checks:

```bash
cargo check
```

Success criteria:

- scanner finds `content/a.md`.
- parser returns `id=topic-a`, `type=topic`, and body text.

### Milestone 3: AST links + graph build
Read: `07_markdown_ast.md`, `08_graph_model.md`

Implement:

- Markdown event traversal with `pulldown-cmark`.
- gx link parser for `gx://rel/<relation>/<target-id>`.
- `graph.rs` with `Node`, `Edge`, `Graph`.

Update sample markdown:

```bash
cat > content/a.md <<'MD'
---
id: topic-a
type: topic
---
# Topic A

See [Topic B](gx://rel/depends_on/topic-b).
MD

cat > content/b.md <<'MD'
---
id: topic-b
type: topic
---
# Topic B
MD
```

Checks:

```bash
cargo check
```

Success criteria:

- graph has 2 nodes.
- graph has 1 edge (`topic-a -> depends_on -> topic-b`).

### Milestone 4: Validation + diagnostics
Read: `09_validation_engine.md`, `10_diagnostics.md`

Implement validation rules:

- unknown type
- duplicate ID
- invalid target
- required relation
- cardinality

Implement deterministic diagnostic sorting before emit.

Checks:

```bash
cargo check
```

Break one file intentionally to test diagnostics:

```bash
cat > content/b.md <<'MD'
---
id: topic-a
type: topic
---
# Duplicate id
MD
```

Success criteria:

- you receive a stable duplicate-ID diagnostic code.
- fixing the file removes the error.

### Milestone 5: `gx check` and `gx index`
Read: `11_index_json.md`, `12_putting_it_together.md`

Wire pipeline in `gx-core` and commands in `gx-cli`.

Required commands:

```bash
cargo run -p gx-cli -- check --root .
cargo run -p gx-cli -- index --root . --out index.json
```

Success criteria:

- `check` exits non-zero when error diagnostics exist.
- `index` writes `index.json` only when no error diagnostics exist.
- JSON is deterministic (sorted records, stable field names).

### Milestone 6: Product integration
Read: `13_nextjs_integration.md`

In your web app repository, add a prebuild script that runs gx:

```bash
# example CI/build sequence
cargo run -p gx-cli -- check --root .
cargo run -p gx-cli -- index --root . --out content/index.json
npm run build
```

Success criteria:

- build fails early on semantic content errors.
- Next.js can load `content/index.json` reliably.

## 6. Endgame and Demo
When all milestones pass, run:

```bash
cargo run -p gx-cli -- check --root .
cargo run -p gx-cli -- index --root . --out index.json
cat index.json
```

Demo checklist:

- add a new markdown node and relation,
- rerun `gx index`,
- verify new node/edge appears in `index.json`,
- intentionally break ontology or frontmatter and confirm diagnostics catch it.

That loop proves both educational understanding and product usefulness.

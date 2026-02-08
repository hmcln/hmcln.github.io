# gx.md — Product Requirements Document (PRD)

**Version:** 0.1
**Status:** Draft
**Audience:** Individual developers, terminal-native knowledge workers, PKM hackers
**Primary Implementation Language:** Rust

---

## 1. Product Overview

### 1.1 What is gx.md?

**gx.md** is a low-level, extensible standard and CLI toolchain for representing personal knowledge bases as **typed, ontology-driven graphs** over plain Markdown files.

gx.md operates strictly at the **plain-text + CLI layer**, enabling users to define domain-specific knowledge ontologies (e.g. PRDs, Features, User Stories, Feedback) and enforce semantic correctness through deterministic validation and indexing.

gx.md does **not** render content, host content, or provide a UI.
It exists to make knowledge bases _structurally meaningful_, _machine-verifiable_, and _editor-native_.

---

### 1.2 Target Users

**Primary users**

- Terminal-native developers
- Neovim / VS Code users
- Individuals maintaining long-lived personal or professional knowledge bases
- Hobbyists interested in PKM, systems thinking, or structured documentation

**Secondary users**

- Developers building higher-level tools (SSGs, UIs, dashboards) on top of gx.md
- Teams experimenting with ontology-driven documentation (non-collaborative)

**Explicit non-users**

- Non-technical users
- Teams requiring real-time collaboration
- Users seeking WYSIWYG editors
- Users seeking hosted or cloud-first solutions

---

## 2. Problem Statement

Most Markdown-based knowledge systems treat documents as:

- files with frontmatter
- loosely connected via links or tags

This approach fails to:

- enforce semantic correctness
- express nuanced relationships
- detect structural incompleteness
- scale beyond ad-hoc note linking

As knowledge bases mature, they require:

- **typed entities**
- **explicit relationships**
- **validation rules**
- **stable identities**
- **tooling support (LSP, CI, SSG integration)**

gx.md addresses this gap by introducing a **compiler-like layer** between raw Markdown and any presentation or UI.

---

## 3. Goals and Non-Goals

### 3.1 Goals

gx.md aims to:

1. Enable **ontology-driven knowledge bases** using plain Markdown
2. Provide **deterministic validation** of semantic structure
3. Produce a **canonical, machine-readable index** of a knowledge base
4. Support **editor tooling** (via LSP) without duplicating logic
5. Serve as a **semantic substrate** for static site generators and UIs
6. Remain **storage-agnostic** and **sync-agnostic**

---

### 3.2 Non-Goals

gx.md explicitly does **not**:

- Provide a hosted service
- Render Markdown to HTML or React
- Act as a static site generator
- Include AI-driven features by default
- Support multi-user collaboration
- Enforce opinionated writing style or formatting
- Replace existing Markdown editors

---

## 4. Core Concepts & Mental Model

### 4.1 Conceptual Model

gx.md treats a knowledge base as a **typed directed graph**.

| Concept  | Description                                                |
| -------- | ---------------------------------------------------------- |
| Node     | A Markdown file with a stable identity and declared type   |
| Type     | A schema-defined category (e.g. `prd`, `feature`)          |
| Edge     | A typed, directed relationship between nodes               |
| Ontology | Repo-local rules defining valid types and relationships    |
| Index    | The compiled semantic representation of the knowledge base |

Markdown files are **source code**.
The gx index is the **intermediate representation (IR)**.

---

### 4.2 Identity Model

Each node has:

- a **globally unique ID** (within the repo)
- a **stable slug** (for URLs and routing)
- a **type** declared in frontmatter

File paths and filenames are _not_ identity.

---

## 5. Ontology System

### 5.1 Ontology Definition File

Each repository contains a single ontology definition file:

- `gx.toml` or `gx.yaml`

This file defines:

- Allowed content types
- Allowed relationships between types
- Required relationships
- Cardinality constraints
- Optional per-type metadata expectations

#### Example (conceptual)

```toml
[type.feature]
requires = [
  { rel = "backed_by_story", target = "user_story", min = 1 }
]

[type.prd]
allows = [
  { rel = "has_feature", target = "feature" }
]
```

gx.md does **not** prescribe a domain ontology — only the mechanism.

---

### 5.2 Frontmatter Requirements

Each Markdown file must contain frontmatter with:

```yaml
id: feature-abc
type: feature
```

Additional metadata is permitted but not interpreted unless referenced by the ontology.

---

## 6. Linking Semantics

gx.md defines a **single canonical semantic link syntax** (exact syntax TBD) that:

- references nodes by ID
- optionally declares relationship type
- is unambiguous and machine-parseable

All semantic relationships must be expressible via this syntax.

Normal Markdown links are permitted but are **semantically ignored** by gx.md.

---

## 7. CLI Interface

### 7.1 Core Commands (v0.1)

#### `gx check <path>`

- Loads ontology
- Scans Markdown files
- Parses frontmatter and semantic links
- Builds knowledge graph
- Validates:
  - unknown types
  - duplicate IDs
  - missing required relationships
  - invalid relationship targets
  - cardinality violations

- Emits diagnostics
- Returns non-zero exit code on error

Used for:

- CI
- editor integration
- confidence in correctness

---

#### `gx index <path> --out <file>`

- Performs all steps of `gx check`
- Writes a **canonical index.json**
- Index contains:
  - nodes
  - edges
  - lookup tables
  - diagnostics
  - resolved slugs and metadata

This index is the **sole integration surface** for downstream tools.

---

### 7.2 Explicitly Deferred Commands

- `gx fmt`
- `gx query`
- `gx new`
- `gx export`
- `gx serve`

These are intentionally out of scope for v0.1.

---

## 8. index.json (Semantic IR)

### 8.1 Purpose

`index.json` is the **compiled semantic model** of a gx.md knowledge base.

It enables:

- deterministic routing
- typed navigation
- relationship-aware rendering
- dashboards and views
- LSP and editor tooling
- SSG integration without re-parsing Markdown

---

### 8.2 Required Contents (v0.1)

- Version metadata
- Node list (id, type, slug, source path, metadata)
- Edge list (from, to, relationship type)
- Lookup tables:
  - by ID
  - by slug
  - outgoing edges
  - incoming edges

- Diagnostics

---

## 9. Integration with Next.js / SSGs

### 9.1 Design Principle

**Next.js renders content; gx.md defines meaning.**

Next.js:

- renders Markdown with React
- builds pages from gx index
- displays relationships

Next.js does **not**:

- walk the repo
- infer types
- build link graphs
- validate correctness

---

### 9.2 Typical Pipeline

1. `gx check` (fail fast)
2. `gx index --out .gx/index.json`
3. Next.js build reads `.gx/index.json`
4. Routes and pages generated from index
5. Markdown rendered per node

This preserves separation of concerns and avoids “split-brain” logic.

---

## 10. Architecture Overview

### 10.1 High-Level Flow

1. Load ontology definition
2. Walk Markdown files
3. Parse frontmatter
4. Parse Markdown AST
5. Extract semantic links
6. Build graph
7. Validate ontology constraints
8. Emit diagnostics
9. Serialize index

---

### 10.2 Core Modules (Rust)

- Ontology loader
- Markdown parser (AST)
- Node registry
- Graph builder
- Validator
- Diagnostics engine
- Index serializer
- CLI interface

Each module must be usable by:

- CLI
- LSP (future)
- external tooling

---

## 11. Success Criteria (v0.1)

gx.md v0.1 is successful if:

- A user can define a custom ontology
- gx.md detects structural errors deterministically
- A Next.js app can render a rich KB using only `index.json` + Markdown
- The CLI feels like a compiler, not a script
- The system scales to hundreds/thousands of files without degradation

---

## 12. Open Questions

- Canonical semantic link syntax
- Ontology file syntax details
- Multi-ontology repos
- Incremental indexing
- Cross-repo references

These are explicitly deferred.

---

## 13. One-Sentence Positioning

> **gx.md is a compiler and type-checker for knowledge bases, turning plain Markdown into a validated, ontology-aware graph that other tools can safely build upon.**

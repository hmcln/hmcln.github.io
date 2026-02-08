# 10_diagnostics.md

## Table of Contents
- [1. Diagnostic Data Model](#1-diagnostic-data-model)
- [2. Severity and Codes](#2-severity-and-codes)
- [3. File and Span Reporting](#3-file-and-span-reporting)
- [4. Editor-Friendly Output](#4-editor-friendly-output)
- [5. Why this matters](#5-why-this-matters)

## 1. Diagnostic Data Model
Use structured diagnostics, not plain strings.

```rust
#[derive(Debug, Clone, serde::Serialize)]
pub struct Diagnostic {
    pub severity: Severity,
    pub code: &'static str,
    pub message: String,
    pub file: PathBuf,
    pub span: Option<Span>,
    pub help: Option<String>,
}

#[derive(Debug, Clone, Copy, serde::Serialize)]
pub enum Severity { Error, Warning, Note }

#[derive(Debug, Clone, Copy, serde::Serialize)]
pub struct Span { pub start: usize, pub end: usize }
```

## 2. Severity and Codes
Assign stable error codes like:

- `GX001` unknown type
- `GX002` duplicate ID
- `GX003` invalid target
- `GX004` required relation missing

Stable codes support automation and editor filtering.

## 3. File and Span Reporting
Store byte spans from parsers when available. If precise span is unavailable, report file-level diagnostic with `span: None` rather than fake ranges.

## 4. Editor-Friendly Output
Provide at least two output formats:

- human-readable text for terminal,
- JSON for tooling.

```rust
pub fn emit_human(diags: &[Diagnostic]) {
    for d in diags {
        eprintln!("{} {}: {} ({})", d.code, format_severity(d.severity), d.message, d.file.display());
    }
}
```

Prefer deterministic ordering by `(file, code, span.start)` before emit.

## 5. Why this matters
Diagnostics are the primary user interface of a compiler. Precise, stable, structured diagnostics make gx.md practical in CI and editors.

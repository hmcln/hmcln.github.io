# 05_markdown_scanning.md

## Table of Contents
- [1. Directory Walking](#1-directory-walking)
- [2. Markdown File Filtering](#2-markdown-file-filtering)
- [3. File I/O Patterns](#3-file-io-patterns)
- [4. Error Handling Strategy](#4-error-handling-strategy)
- [5. Why this matters](#5-why-this-matters)

## 1. Directory Walking
Use `walkdir` for robust traversal.

```rust
use walkdir::WalkDir;

pub fn collect_markdown_paths(root: &Path) -> Result<Vec<PathBuf>, GxError> {
    let mut files = Vec::new();

    for entry in WalkDir::new(root)
        .into_iter()
        .filter_entry(|e| !is_ignored_dir(e.path()))
    {
        let entry = entry.map_err(GxError::Walk)?;
        let path = entry.path();
        if entry.file_type().is_file() && is_markdown(path) {
            files.push(path.to_path_buf());
        }
    }

    files.sort();
    Ok(files)
}
```

Borrowing detail: `path` is borrowed from walker; clone into `PathBuf` only for retained outputs.

## 2. Markdown File Filtering
Simple extension-based filter:

```rust
fn is_markdown(path: &Path) -> bool {
    matches!(
        path.extension().and_then(|s| s.to_str()),
        Some("md") | Some("markdown")
    )
}
```

Ignored directory strategy should include `.git`, `node_modules`, build outputs.

## 3. File I/O Patterns
Represent source file as owned struct:

```rust
#[derive(Debug, Clone)]
pub struct SourceFile {
    pub path: PathBuf,
    pub text: String,
}

pub fn read_sources(paths: &[PathBuf]) -> Result<Vec<SourceFile>, GxError> {
    paths.iter()
        .map(|p| {
            let text = std::fs::read_to_string(p)?;
            Ok(SourceFile { path: p.clone(), text })
        })
        .collect()
}
```

Iterator `map + collect` keeps code concise while preserving typed error propagation.

## 4. Error Handling Strategy
Treat IO failures as hard errors when files are explicitly discovered. Alternatives:

- continue-on-error and emit warning diagnostics,
- fail-fast.

For compiler consistency, v0.1 should fail-fast on unreadable files because index completeness cannot be guaranteed.

## 5. Why this matters
Scanning and loading files is the bridge between repository state and compiler pipeline. Deterministic traversal and strict IO behavior ensure reproducible outputs.

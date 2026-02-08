# 03_cli_entrypoint.md

## Table of Contents
- [1. CLI Goals](#1-cli-goals)
- [2. Defining Commands with clap](#2-defining-commands-with-clap)
- [3. Ownership of CLI State](#3-ownership-of-cli-state)
- [4. Error Propagation and Exit Codes](#4-error-propagation-and-exit-codes)
- [5. Why this matters](#5-why-this-matters)

## 1. CLI Goals
v0.1 requires two primary flows:

- `gx check`: run analysis and diagnostics only.
- `gx index`: run analysis and emit canonical `index.json`.

The CLI should not implement compiler logic directly.

## 2. Defining Commands with clap
`Cargo.toml` for CLI:

```toml
[dependencies]
clap = { version = "4", features = ["derive"] }
gx-core = { path = "../gx-core" }
```

CLI types:

```rust
use clap::{Parser, Subcommand};
use std::path::PathBuf;

#[derive(Debug, Parser)]
#[command(name = "gx", about = "Compiler/type-checker for Markdown knowledge bases")]
pub struct Cli {
    #[arg(long, default_value = ".")]
    pub root: PathBuf,

    #[command(subcommand)]
    pub command: Command,
}

#[derive(Debug, Subcommand)]
pub enum Command {
    Check,
    Index {
        #[arg(long, default_value = "index.json")]
        out: PathBuf,
    },
}
```

## 3. Ownership of CLI State
`Cli` owns parsed values (`PathBuf`, enum variant data). Hand borrowed references to core APIs:

```rust
pub fn run() -> Result<(), CliError> {
    let cli = Cli::parse();

    match cli.command {
        Command::Check => {
            let diagnostics = gx_core::check_repo(&cli.root)?;
            print_diagnostics(&diagnostics);
            if diagnostics.iter().any(|d| d.is_error()) {
                return Err(CliError::ValidationFailed);
            }
        }
        Command::Index { out } => {
            let result = gx_core::build_index(&cli.root)?;
            std::fs::write(&out, serde_json::to_vec_pretty(&result)?)?;
        }
    }

    Ok(())
}
```

Key point: core receives `&Path` and returns owned results. This minimizes copies and keeps lifetimes local to the function.

## 4. Error Propagation and Exit Codes
Use two layers:

- Domain errors from `gx-core` as `GxError` enum.
- CLI wrapper error as `CliError` for IO/serialization/exit semantics.

Example:

```rust
#[derive(thiserror::Error, Debug)]
pub enum CliError {
    #[error("compiler failure: {0}")]
    Core(#[from] gx_core::GxError),

    #[error("io failure: {0}")]
    Io(#[from] std::io::Error),

    #[error("json failure: {0}")]
    Json(#[from] serde_json::Error),

    #[error("validation failed")]
    ValidationFailed,
}
```

Main function sets exit codes:

- `0`: success, no errors.
- `1`: user-facing validation errors.
- `2`: internal/IO/configuration failure.

## 5. Why this matters
A disciplined CLI entrypoint keeps command handling understandable while preserving separation from compiler internals. This is crucial when adding future commands or editor integration.

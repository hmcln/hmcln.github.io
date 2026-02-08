# 07_markdown_ast.md

## Table of Contents
- [1. Why Parse Markdown AST](#1-why-parse-markdown-ast)
- [2. Parser Selection](#2-parser-selection)
- [3. AST Traversal for Links](#3-ast-traversal-for-links)
- [4. Link Syntax to Semantic Edges](#4-link-syntax-to-semantic-edges)
- [5. Why this matters](#5-why-this-matters)

## 1. Why Parse Markdown AST
gx does not render Markdown, but still needs AST parsing to:

- reliably extract links from structured nodes,
- avoid false positives from plain text regex,
- capture spans for diagnostics.

## 2. Parser Selection
`pulldown-cmark` is practical for v0.1 due to speed and broad ecosystem use.

```rust
use pulldown_cmark::{Event, Options, Parser, Tag};

pub fn markdown_events(body: &str) -> impl Iterator<Item = Event<'_>> {
    let options = Options::ENABLE_TABLES | Options::ENABLE_STRIKETHROUGH;
    Parser::new_ext(body, options)
}
```

## 3. AST Traversal for Links
Extract links while iterating events:

```rust
#[derive(Debug, Clone)]
pub struct RawLink {
    pub rel: String,
    pub target_id: String,
}

pub fn extract_raw_links(body: &str) -> Vec<RawLink> {
    let mut out = Vec::new();

    for ev in markdown_events(body) {
        if let Event::Start(Tag::Link { dest_url, .. }) = ev {
            if let Some(link) = parse_gx_link(dest_url.as_ref()) {
                out.push(link);
            }
        }
    }

    out
}
```

## 4. Link Syntax to Semantic Edges
Define gx-specific syntax in URL target, for example:

`gx://rel/depends_on/target-id`

```rust
fn parse_gx_link(raw: &str) -> Option<RawLink> {
    let prefix = "gx://rel/";
    if !raw.starts_with(prefix) {
        return None;
    }
    let rest = &raw[prefix.len()..];
    let (rel, target) = rest.split_once('/')?;
    Some(RawLink { rel: rel.to_string(), target_id: target.to_string() })
}
```

Alternatives considered:

- custom inline syntax (`[[rel:target]]`): simpler to read, but requires custom parser or token pass.
- regex over raw text: rejected due to correctness issues with code blocks and escapes.

## 5. Why this matters
AST extraction guarantees semantic links come from real Markdown structure, which is necessary for reliable validation and editor-grade diagnostics.

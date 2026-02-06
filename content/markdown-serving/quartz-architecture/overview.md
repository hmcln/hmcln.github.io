---
title: Quartz Architecture Overview
---

# Quartz Architecture Overview

This document explains Quartz as a **markdown-first static site compiler** for knowledge bases.

---

## 1) High-level mental model

Think of Quartz as a pipeline:

```text
Markdown files
   -> Parse + enrich (frontmatter, links, syntax trees)
   -> Transform with plugins
   -> Generate static pages + assets
   -> Deploy to static hosting
```

A practical way to reason about Quartz:

- Your `content/` directory is the data source.
- Quartz builds an in-memory representation of your notes.
- Plugins transform both content and metadata.
- Emitters create HTML, RSS, sitemap, indexes, and related assets.

---

## 2) Repository-level structure

In this repo, key files/folders include:

```text
content/              # Your markdown knowledge base
quartz/               # Core rendering, plugins, processing utilities
quartz.config.ts      # Site config + plugin pipeline
quartz.layout.ts      # Layout/component composition
docs/                 # Quartz docs for reference
```

### Why this split works

- **Content stays content**: pure markdown in `content/`.
- **Behavior is code**: rendering logic and plugins in `quartz/`.
- **Policy is config**: `quartz.config.ts` and `quartz.layout.ts` capture decisions.

---

## 3) Content model and linking

Quartz supports wiki-style authoring patterns:

- Standard markdown links: `[Label](./path.md)`
- Wiki links (depending on configuration): `[[Some Note]]`
- Backlinks generated from graph/link analysis
- Tags and taxonomy pages generated through metadata

Example note:

```md
---
title: Caching Strategies
tags: [architecture, performance]
---

# Caching Strategies

Related:
- [[CDN Invalidation]]
- [[HTTP Caching Headers]]
```

During build, links are resolved and normalized into site paths.

---

## 4) Build pipeline internals

Quartz architecture can be seen in three plugin stages:

1. **Transformers**: modify/annotate source content and AST
2. **Filters**: decide what passes through
3. **Emitters**: produce final output artifacts

Conceptual pseudocode:

```ts
for (const file of contentFiles) {
  const parsed = parseMarkdown(file)
  const transformed = runTransformers(parsed)
  if (runFilters(transformed)) {
    buildGraphNode(transformed)
  }
}

runEmitters(siteGraph, transformedFiles)
```

### Typical responsibilities by stage

- **Transformers**
  - parse frontmatter
  - syntax highlighting
  - heading extraction / TOC
  - link normalization
- **Filters**
  - draft exclusion
  - private path removal
- **Emitters**
  - page HTML
  - folder/tag indexes
  - RSS, sitemap, search index

---

## 5) Config architecture

Quartz is intentionally code-configurable.

### `quartz.config.ts`

Defines site metadata and plugin behavior (what gets processed, how links behave, what gets emitted).

Example shape:

```ts
export default {
  configuration: {
    pageTitle: "My Knowledge Base",
    baseUrl: "example.com",
    enableSPA: true,
  },
  plugins: {
    transformers: [/* ... */],
    filters: [/* ... */],
    emitters: [/* ... */],
  },
}
```

### `quartz.layout.ts`

Defines the composition of visible page regions and components.

Example shape:

```ts
export default {
  sharedPageComponents: {
    head: /* SEO + metadata */,
    header: /* breadcrumbs / title */,
    footer: /* nav / credits */,
  },
  defaultContentPageLayout: {
    left: [/* explorer, graph */],
    right: [/* toc, backlinks */],
  },
}
```

This separation keeps **content transformation concerns** apart from **visual composition concerns**.

---

## 6) Rendering and output

Quartz builds static artifacts suitable for CDN hosting:

```text
public/
  index.html
  notes/.../index.html
  assets/*.js
  assets/*.css
  sitemap.xml
  index.xml  (rss)
```

That means:

- excellent cacheability
- no server runtime required for core docs delivery
- low-cost, high-reliability hosting (GitHub Pages, Netlify, Vercel static)

---

## 7) Search architecture (conceptual)

Quartz commonly ships with local search via a generated index.

```text
Content -> tokenization/index generation -> static search index asset
Browser -> query index client-side -> ranked matches
```

Advantages:

- no external search service required
- privacy-friendly (no outbound query logging by default)
- predictable behavior in offline-ish/static contexts

Tradeoff:

- larger client payload as content grows significantly

---

## 8) Performance and scalability considerations

As your knowledge base grows, key considerations are:

- **Build time**
  - AST transforms and cross-link graph analysis dominate cost.
- **Bundle weight**
  - choose lightweight components and limit heavy client scripts.
- **Content organization**
  - deeply nested structures are fine, but index pages improve discoverability.

Operational recommendations:

```text
- Keep notes granular and linked.
- Use tags intentionally (avoid tag explosion).
- Add CI checks for broken links and frontmatter validity.
- Deploy immutable build artifacts via CDN.
```

---

## 9) End-to-end request lifecycle

### Authoring time

1. You create/edit markdown in `content/`.
2. Git tracks changes.
3. CI or local command triggers Quartz build.

### Build time

1. Quartz parses markdown into syntax trees.
2. Plugins transform and enrich.
3. Emitters output static files.

### Runtime (user browsing)

1. Browser requests static HTML from CDN.
2. CSS/JS assets hydrate interactive components.
3. Internal links and search drive knowledge exploration.

---

## 10) Extension strategy

When extending Quartz for a team knowledge base:

- Add conventions in frontmatter:

```yaml
owner: platform-team
status: draft
last_reviewed: 2026-02-01
```

- Add filters to hide drafts from production.
- Add custom transformers for org-specific metadata.
- Add emitters for machine-readable exports (JSON for integrations).

This keeps markdown authoring simple while enabling enterprise workflows.

---

## 11) Architecture summary

Quartz is strongest when treated as:

- a **content compiler** (markdown -> linked static knowledge site)
- a **plugin platform** (transform/filter/emit)
- a **static-first deployment model** with excellent operational simplicity

If your priority is authoring speed, wiki linking, and static hosting reliability, Quartz is a very strong architecture choice.

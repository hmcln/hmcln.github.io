---
title: Building a Quartz-Like Knowledge Base with Next.js + Vercel
---

# Building a Quartz-Like Knowledge Base with Next.js + Vercel

This guide describes how I would implement a markdown-driven knowledge base with a **Next.js + Vercel** stack, while preserving many of the strengths of Quartz.

---

## 1) Target architecture goals

We want:

- Markdown as source of truth
- Fast static delivery
- Wiki-style linking and backlinks
- Search
- CI-driven deployment
- Room for custom React UI/logic

---

## 2) Recommended stack

```text
Framework:      Next.js (App Router)
Hosting:        Vercel
Content source: /content/**/*.md (or .mdx)
Parsing:        unified + remark + rehype
Metadata:       gray-matter (frontmatter)
Syntax hl:      rehype-pretty-code (or shiki)
Search:         generated static JSON/Lunr/FlexSearch index
Links graph:    prebuild script for backlinks and related notes
```

---

## 3) Project layout

```text
app/
  (kb)/
    [...slug]/page.tsx       # note pages
    layout.tsx               # kb shell
  page.tsx                   # home
components/
  kb/
lib/
  content/
    loader.ts                # read markdown + frontmatter
    parser.ts                # remark/rehype pipeline
    graph.ts                 # backlinks graph builder
    search.ts                # search index generation
content/
  architecture/
  guides/
public/
  search-index.json          # generated at build time
scripts/
  build-content.ts           # optional prebuild pipeline
```

---

## 4) Content ingestion pipeline

### Step A: Read all markdown files

```ts
// lib/content/loader.ts
import fs from "node:fs"
import path from "node:path"
import matter from "gray-matter"

export function loadAllNotes(contentDir = "content") {
  // walk files, parse frontmatter, return structured note objects
}
```

### Step B: Parse markdown to HTML/React

```ts
// lib/content/parser.ts
import { unified } from "unified"
import remarkParse from "remark-parse"
import remarkGfm from "remark-gfm"
import remarkRehype from "remark-rehype"
import rehypeStringify from "rehype-stringify"

export async function renderMarkdown(markdown: string) {
  return unified()
    .use(remarkParse)
    .use(remarkGfm)
    .use(remarkRehype)
    .use(rehypeStringify)
    .process(markdown)
}
```

### Step C: Build link graph/backlinks

- Extract internal links from notes.
- Build adjacency list.
- Persist JSON map keyed by slug.

```ts
type Graph = Record<string, { out: string[]; in: string[] }>
```

This gives Quartz-like backlink experiences.

---

## 5) Routing and page generation (App Router)

Use dynamic catch-all routes:

```ts
// app/(kb)/[...slug]/page.tsx
export async function generateStaticParams() {
  // return all note slugs so pages are statically generated
}

export default async function NotePage({ params }: { params: { slug: string[] } }) {
  // load note by slug, render content, include TOC/backlinks
}
```

Use `generateMetadata` for SEO from frontmatter:

```ts
export async function generateMetadata({ params }) {
  // title, description, openGraph, alternates
}
```

---

## 6) Search implementation on Vercel

### Static client-side search

Precompute `public/search-index.json` at build time.

```bash
node scripts/build-content.ts
next build
```

At runtime:

- load index lazily
- query in browser
- return top N results ranked by title/body/token hits

### Alternative: server-backed search

For very large corpora:

- index into Algolia/Meilisearch/OpenSearch
- query via API route
- add rate limiting/caching

For most docs sites, static client search is enough and simpler.

---

## 7) Deployment model on Vercel

### CI/CD flow

```text
git push -> Vercel build
         -> run prebuild content scripts
         -> next build (SSG pages)
         -> deploy immutable assets/pages to edge network
```

Recommended `package.json` script order:

```json
{
  "scripts": {
    "prebuild": "node scripts/build-content.ts",
    "build": "next build"
  }
}
```

### Caching

- Static pages/assets are edge-cached by default.
- Revalidate only if using ISR/dynamic sources.
- Prefer full SSG for deterministic docs behavior.

---

## 8) Authoring ergonomics (Quartz-like)

To emulate Quartz’s wiki feel:

- Add support for `[[Wiki Links]]` in remark plugin
- Auto-resolve slug aliases
- Show backlinks panel and related notes
- Generate folder/tag landing pages

Pseudo plugin idea:

```ts
function remarkWikiLinks() {
  // find [[...]] patterns and convert to normal md links
}
```

---

## 9) Governance, quality, and observability

Add guardrails:

- link checker in CI
- frontmatter schema validation (zod/yup)
- lint markdown headings and required fields
- analytics (privacy-aware) for top viewed notes and dead-end pages

Example validation schema:

```ts
const NoteSchema = z.object({
  title: z.string(),
  tags: z.array(z.string()).default([]),
  draft: z.boolean().default(false),
  updated: z.string().optional(),
})
```

---

## 10) When to choose this over Quartz

Use **Next.js + Vercel** when you need:

- heavy custom interactivity
- unified product + docs app in one framework
- complex authenticated knowledge portals
- tighter control over rendering and data fetching strategy

Use **Quartz** when you need:

- fast setup for markdown-native knowledge gardens
- strong wiki/linking ergonomics out of the box
- static-first simplicity with less custom code

---

## 11) Implementation plan (practical)

```text
Phase 1: Static markdown rendering + routing
Phase 2: TOC + backlinks + tags
Phase 3: Search index + UI
Phase 4: CI validation + content governance
Phase 5: Visual polish + analytics
```

This phased approach reaches production quickly while keeping architecture clean.

---

## 12) Final recommendation

If your current Quartz knowledge base is working well, keep it as the default.

Build a Next.js + Vercel alternative only if you need capabilities that justify the extra engineering surface area (custom app behaviors, authenticated flows, deeper React integration).

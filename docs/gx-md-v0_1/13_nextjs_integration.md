# 13_nextjs_integration.md

## Table of Contents
- [1. Consumption Model](#1-consumption-model)
- [2. Example Next.js Usage](#2-example-nextjs-usage)
- [3. Boundary: Compiler vs Renderer](#3-boundary-compiler-vs-renderer)
- [4. SSG Design Considerations](#4-ssg-design-considerations)
- [5. Why this matters](#5-why-this-matters)

## 1. Consumption Model
Next.js should treat `index.json` as precomputed semantic metadata. gx remains responsible for correctness; Next.js remains responsible for rendering/UI.

## 2. Example Next.js Usage
Minimal server-side loading:

```ts
import fs from "node:fs/promises";

export async function loadIndex() {
  const raw = await fs.readFile("./content/index.json", "utf-8");
  return JSON.parse(raw) as {
    version: string;
    nodes: Array<{ id: string; type: string; path: string }>;
    edges: Array<{ source: string; rel: string; target: string }>;
  };
}
```

Then build route params by node type, or relation-driven pages by filtering edges.

## 3. Boundary: Compiler vs Renderer
Do not move Markdown rendering into gx:

- it mixes concerns,
- ties compiler release cycle to web framework changes,
- complicates portability to non-web consumers.

Likewise, Next.js should not re-implement ontology checks because that duplicates logic and may diverge.

## 4. SSG Design Considerations
Use `index.json` for build-time querying:

- generate static params from `nodes`,
- derive related content from `edges`,
- surface gx diagnostics in CI before `next build`.

Preferred pipeline in CI:

1. `gx check`
2. `gx index --out content/index.json`
3. `next build`

## 5. Why this matters
Clear system boundaries prevent architecture drift. gx guarantees semantic correctness; Next.js delivers presentation.

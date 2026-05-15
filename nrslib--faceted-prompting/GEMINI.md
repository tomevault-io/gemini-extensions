## faceted-prompting

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build          # TypeScript compile (tsc) → dist/
npm run test           # Run all tests (vitest)
npm run test -- src/__tests__/compose.test.ts  # Run a single test file
npm run test:watch     # Watch mode
npm run lint           # ESLint on src/
```

## Architecture

This is a **TypeScript ESM library** (`"type": "module"`) that implements the Faceted Prompting pattern — structured prompt composition for LLMs. It is designed to be consumed as an npm package (`faceted-prompting`). All public API is re-exported from `src/index.ts`.

### Core Concept: Facets

Prompts are decomposed into four facet kinds (`FacetKind`), each with a defined role and placement:

| Facet | Placement | Purpose |
|-------|-----------|---------|
| Persona | System prompt | WHO — agent identity |
| Policy | User message | HOW — rules/standards |
| Knowledge | User message | WHAT TO KNOW — domain context |
| Instruction | User message | WHAT TO DO — the task |

There is also an `output-contracts` kind defined in `FacetKind` but not yet used in `FacetSet`.

### Key Modules

- **`compose.ts`** — Core composition: takes a `FacetSet` + `ComposeOptions` → `ComposedPrompt` (systemPrompt + userMessage). Policy and knowledge are truncated via `truncation.ts` when exceeding `contextMaxChars`.
- **`data-engine.ts`** — `DataEngine` interface for facet retrieval. `FileDataEngine` resolves `{root}/{kind}/{key}.md`. `CompositeDataEngine` chains engines with first-match-wins.
- **`resolve.ts`** — Facet reference resolution: resolves names/paths/inline content from candidate directories and section maps. Supports `~`, `./`, `../`, `/` path prefixes and `.md` extension detection.
- **`scope.ts`** — `@{owner}/{repo}/{facet-name}` scope reference parsing/validation for repertoire packages.
- **`template.ts`** — Minimal template engine: `{{variable}}` substitution and `{{#if var}}...{{else}}...{{/if}}` conditionals (no nesting).
- **`truncation.ts`** — Trims content to char limit, appends truncation markers and source-path metadata.
- **`escape.ts`** — Escapes `{}` to full-width Unicode equivalents to prevent template injection.

### Design Constraints

- Modules annotated "ZERO dependencies on TAKT internals" must stay framework-independent. This library is extracted from TAKT and must remain a standalone package.
- All imports use `.js` extensions (NodeNext module resolution).
- Tests live in `src/__tests__/` and are excluded from compilation (`tsconfig.json` excludes them).
- Strict TypeScript: `noUncheckedIndexedAccess`, `strictNullChecks`, `noImplicitAny` are all enabled.

---
> Source: [nrslib/faceted-prompting](https://github.com/nrslib/faceted-prompting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-09 -->

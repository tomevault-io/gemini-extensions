## sentry-javascript

> Monorepo with 40+ packages in `@sentry/*`, managed with Yarn workspaces and Nx.

# Sentry JavaScript SDK

Monorepo with 40+ packages in `@sentry/*`, managed with Yarn workspaces and Nx.

## Setup

- [Volta](https://volta.sh/) for Node.js/Yarn/PNPM version management
- Requires `VOLTA_FEATURE_PNPM=1`
- After cloning: `yarn install && yarn build`
- Never change Volta, Yarn, or package manager versions unless explicitly asked

### Code Intelligence

Prefer LSP over Grep/Read for code navigation — it's faster, precise, and avoids reading entire files:

- `workspaceSymbol` to find where something is defined
- `findReferences` to see all usages across the codebase
- `goToDefinition` / `goToImplementation` to jump to source
- `hover` for type info without reading the file

Use Grep only when LSP isn't available or for text/pattern searches (comments, strings, config).

After writing or editing code, check LSP diagnostics and fix errors before proceeding.

## Package Manager

Use **yarn**: `yarn install`, `yarn build:dev`, `yarn test`, `yarn lint`

| Command                               | Purpose                       |
| ------------------------------------- | ----------------------------- |
| `yarn build`                          | Full production build         |
| `yarn build:dev`                      | Dev build (transpile + types) |
| `yarn build:dev:filter @sentry/<pkg>` | Build one package + deps      |
| `yarn build:bundle`                   | Browser bundles only          |
| `yarn test`                           | All unit tests                |
| `yarn verify`                         | Lint + format check           |
| `yarn fix`                            | Format + lint fix             |
| `yarn lint`                           | Lint (Oxlint)                 |
| `yarn lint:fix`                       | Lint + auto-fix (Oxlint)      |
| `yarn format`                         | Format files (Oxfmt)          |
| `yarn format:check`                   | Check formatting (Oxfmt)      |

Single package: `cd packages/<name> && yarn test`

## Commit Attribution

AI commits MUST include a `Co-Authored-By` line with the appropriate committer email when known:

```
Co-Authored-By: <Claude model name> <noreply@anthropic.com>
Co-Authored-By: <OpenAI/ChatGPT model name> <codex@openai.com>
Co-Authored-By: <Cursor agent name> <cursoragent@cursor.com>
```

Use the Cursor email for Cursor, even when it runs a Claude or OpenAI model. Omit the line only when there is no known committer email address for the agent.

## Git Workflow

Uses **Git Flow** (see `docs/gitflow.md`).

- **All PRs target `develop`** (NOT `master`)
- `master` = last released state — never merge directly
- Feature branches: `feat/descriptive-name`
- Never update dependencies, `package.json`, or build scripts unless explicitly asked

## Before Every Commit

1. `yarn format`
2. `yarn lint`
3. `yarn test`
4. `yarn build:dev`
5. NEVER push on `develop`

## Pull Requests

- **Do NOT add a "Test plan" / "Testing" checklist to PR bodies.** CI runs the full test suite on every PR — a hand-rolled checklist duplicates that signal and rots fast. Write the summary content directly and add a _Root cause_ section only if relevant.
- **Omit the "Summary" heading** in PR bodies — lead with the summary text itself, no `## Summary` header.
- Include `Fixes #<issue-number>` somewhere in the PR body so the merge auto-closes the linked issue.
- Always open PRs as draft.
- Include reasoning of changes in the PR description, as well as decisions that were taken during implementation. Do not explain the implementation that can be viewed in the code.

## Architecture

### Core

- `packages/core/` — Base SDK: interfaces, types, core functionality
- `packages/types/` — Shared types (**deprecated, never modify – instead find types in packages/core**)
- `packages/browser-utils/` — Browser utilities and instrumentation
- `packages/node-core/` — Node core logic (excludes OTel instrumentation)

### Platform SDKs

- `packages/browser/` — Browser SDK + CDN bundles
- `packages/node/` — Node.js SDK (OTel instrumentation on top of node-core)
- `packages/bun/`, `packages/deno/`, `packages/cloudflare/`

### Framework Integrations

- `packages/{framework}/` — React, Vue, Angular, Next.js, Nuxt, SvelteKit, Remix, etc.
- Some have client/server entry points (nextjs, nuxt, sveltekit)

### AI Integrations

- `packages/core/src/tracing/{provider}/` — Core instrumentation
- `packages/node/src/integrations/tracing/{provider}/` — Node.js integration + OTel
- `packages/cloudflare/src/integrations/tracing/{provider}.ts` — Edge runtime
- Use `/add-ai-integration` skill when adding or modifying integrations

### User Experience

- `packages/replay-internal/`, `packages/replay-canvas/`, `packages/replay-worker/` — Session replay
- `packages/feedback/` — User feedback

### Dev Packages (`dev-packages/`)

- `browser-integration-tests/` — Playwright browser tests
- `e2e-tests/` — E2E tests (70+ framework combos)
- `node-integration-tests/` — Node.js integration tests
- `test-utils/` — Shared test utilities
- `rollup-utils/` — Build utilities

## Linting & Formatting

- This project uses **Oxlint** and **Oxfmt** — NOT ESLint or Prettier
- Never run `eslint`, `npx eslint`, or any ESLint CLI — use `yarn lint` (Oxlint) instead
- Never run `prettier` — use `yarn format` (Oxfmt) instead
- ESLint packages in the repo are legacy/e2e test app dependencies — ignore them
- Do not create, modify, or suggest `.eslintrc`, `eslint.config.*`, or `.prettierrc` files

## Coding Standards

- Follow existing conventions — check neighboring files
- Reach for existing utils before writing a new one. Most shared helpers live in `@sentry/core` (`packages/core/src/utils/`), with browser helpers in `packages/browser-utils/`. Search first (LSP `workspaceSymbol` or grep) for common needs (type guards in `is.ts`, object/array helpers, `normalize`, `dsn`, `merge`, string/url helpers). Reuse or extend the existing util rather than adding a near-duplicate; only introduce a new util when nothing fits.
- Only use libraries already in the codebase
- Never expose secrets or keys
- When modifying files, cover all occurrences (including `src/` and `test/`)
- Comments explain **why**, never **what** — never add a comment that restates what the code does or describes the change being made; only comment when the reasoning isn't obvious from the code itself

## Reference Documentation

- [Span Attributes](https://develop.sentry.dev/sdk/telemetry/attributes.md)
- [Scopes (global, isolation, current)](https://develop.sentry.dev/sdk/telemetry/scopes.md)

## Skills

### E2E Testing

Use `/e2e` skill to run E2E tests. See `.claude/skills/e2e/SKILL.md`

### Security Vulnerabilities

Use `/fix-security-vulnerability` skill for Dependabot alerts. See `.claude/skills/fix-security-vulnerability/SKILL.md`

### Issue Triage

Use `/triage-issue` skill. See `.claude/skills/triage-issue/SKILL.md`

### CDN Bundles

Use `/add-cdn-bundle` skill. See `.claude/skills/add-cdn-bundle/SKILL.md`

### Publishing a Release

Use `/release` skill. See `.claude/skills/release/SKILL.md`

### Dependency Upgrades

Use `/upgrade-dep` skill. See `.claude/skills/upgrade-dep/SKILL.md`

### Vendor OpenTelemetry Instrumentation

Use `/vendor-otel` skill. See `.claude/skills/vendor-otel/SKILL.md`

### AI Integration

Use `/add-ai-integration` skill. See `.claude/skills/add-ai-integration/SKILL.md`

---
> Source: [getsentry/sentry-javascript](https://github.com/getsentry/sentry-javascript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-07-21 -->

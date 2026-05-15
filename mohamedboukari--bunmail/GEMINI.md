## bunmail

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Communication Style

- **Always plan before coding**: Use EnterPlanMode for any non-trivial task. Present the plan, explain trade-offs, and get approval before writing code.
- **Explain the "why" before each edit**: Before making any code change, state the goal — what problem it solves and why this approach. The user is a senior JS/TS developer (5+ years) who wants to collaborate and co-decide.
- **Keep the user in the loop**: Share potential impacts, edge cases, and alternatives. Don't make silent assumptions.
- **No `any` types**: Use proper TypeScript types. Avoid unsafe casts (`as SomeType`) — prefer type narrowing, generics, or extending interfaces.

## Project Overview

BunMail is a self-hosted email API for developers — a free alternative to SendGrid/Resend. REST API for sending transactional emails with direct SMTP delivery, DKIM/SPF/DMARC signing, email queue with retries, templates, and a web dashboard.

## Tech Stack

- **Runtime:** Bun
- **Backend:** Elysia
- **SMTP Sending:** Nodemailer (direct mode, no provider)
- **SMTP Receiving:** smtp-server
- **Email Auth:** DKIM signing, SPF/DMARC DNS verification
- **Database:** SQLite (default) or PostgreSQL
- **Queue:** Custom with retries (3 attempts)
- **Dashboard:** React or Svelte frontend
- **Deploy:** Docker

## Development Commands

```bash
bun install                # install dependencies
bun run dev                # start dev server
bun test                   # run all tests
bun test <file>            # run a single test file
bun run build              # build for production
bunx tsc --noEmit          # type-check without emitting
docker compose up          # run full stack with Docker
```

## Architecture

```
Elysia API (routes/) → Services (services/) → Database (db/)
                            ↓
                       Queue (retries) → SMTP Send (Nodemailer + DKIM)
                                             ↓
                                       Webhooks fired on delivery/bounce
```

- **Routes** (`src/routes/`) — REST API endpoints under `/api/v1/`. Auth via Bearer API key.
- **Services** (`src/services/`) — Core business logic: mailer, DKIM signing, email queue, DNS verification, webhook dispatch.
- **Middleware** (`src/middleware/`) — API key authentication and rate limiting.
- **Database** (`src/db/`) — Schema, migrations, and connection setup.
- **Dashboard** (`dashboard/`) — Separate frontend app for managing emails, templates, domains, and API keys.

## Code Conventions

### General

- Use module-per-feature under `src/modules/<feature>/` when organizing domain logic.
- Keep route handlers thin; put business logic in services.
- Use kebab-case for filenames; PascalCase for classes; camelCase for methods/variables.
- Prefer editing existing files over creating new ones.
- Follow existing patterns in the codebase — match the style of surrounding code.
- Keep changes minimal and focused — don't refactor unrelated code.

### Module Layout

Each feature module follows this pattern:
```
src/modules/<feature>/
  ├── <feature>.plugin.ts     ← Elysia plugin (route group)
  ├── services/               ← Business logic
  ├── dtos/                   ← Request/response validation schemas (Elysia t.Object)
  ├── models/                 ← Database schemas
  ├── serializations/         ← Response mappers/serializers
  └── types/                  ← Shared types for this module only
```

- Never introduce new top-level folders under `src/` except: `modules/`, `db/`, `utils/`, `email-templates/`.

### Elysia Specifics

- Define route groups as Elysia plugins (`.use()` pattern) — one plugin per feature module.
- Use Elysia's built-in validation with `t.Object()` schemas for request body/params/query.
- Use Elysia's `onBeforeHandle` for guards and middleware (auth, rate limiting).
- Route handlers call services. No DB or cross-cutting logic in route handlers.
- Route prefix: the module's feature name in kebab-case (e.g., `api-keys` → `/api/v1/api-keys`).

### DTOs and Serialization

- Place validation schemas in the module's `dtos/` directory. File names: `<action>-<entity>.dto.ts`.
- Responses should be mapped via `serializations/` when shaping output or hiding internals.
- Do not import DTOs or serializers across modules; keep them feature-local.

### Database

- Define schemas in `models/` with file name: `<entity>.schema.ts`.
- Only services may access the database — never from route handlers directly.

### Error Handling

- Throw Elysia-compatible errors from services and let Elysia's error handler format responses.
- Do not return raw errors or stack traces from route handlers.

### Email

- Email templates live under `src/email-templates/`. Place email send logic in `src/services/mailer.ts`. Do not send mail from route handlers directly.

### Tests

- Place unit tests alongside source or under `test/unit/` matching the module structure.
- New endpoints must include or update tests for both route and service logic.

### Cross-Module Types

- Keep types local in `src/modules/<feature>/types/`. Promote to a shared place only if used across 3+ modules.

## Changes Checklist

When adding an endpoint:
1. Add/adjust validation schemas in `dtos/`.
2. Add route handler in the feature plugin.
3. Implement service logic under `services/`.
4. If response shape differs from raw model, add/update a serializer.
5. Ensure the plugin is registered in the main app.
6. Add/extend tests.

When adding a data model:
1. Create `<entity>.schema.ts` under `models/`.
2. Register/migrate the schema.
3. Inject and use the model in the service only.
4. Update serializers if response shape changes.
5. Add/extend tests.

## Workflow

- Read files before editing — understand existing code first.
- Run `bunx tsc --noEmit` after changes to catch type errors.
- Run tests after implementation to verify nothing breaks.
- When exploring the codebase, use the Explore agent for broad searches.
- For multi-file changes, create a task list to track progress.

## Documentation

- After any code change, check if `docs/`, `ARCHITECTURE.md`, or `README.md` need updating.
- Keep docs concise: update only what changed.
- Every module should have its own `docs/<module-name>.md` documenting schema, types, service methods, and module layout.
- Every module's endpoints must be listed in `docs/api.md`.
- **Every PR must update the relevant `.md` docs in the same commit** — `CHANGELOG.md` (always, under `[Unreleased]`), and any of `README.md` / `ARCHITECTURE.md` / `docs/api.md` / `docs/<module>.md` whose content the PR makes outdated. Mention the doc updates in the PR description's "Changes" section. Don't merge a PR that adds/removes/changes a public API surface or env var without the matching doc edit.
- **Before every commit, audit every `.md` file the change touches.** Don't assume "I only changed code, the docs are fine" — sweep across `README.md`, `ARCHITECTURE.md`, `THREAT_MODEL.md`, `SECURITY.md`, `CHANGELOG.md`, and everything under `docs/`. Specific things to re-check on every PR:
  - **Schema tables** in `ARCHITECTURE.md` and per-module docs — new columns, dropped columns, new indexes, FK-on-delete behaviour.
  - **API endpoints** in `docs/api.md` and the table in `ARCHITECTURE.md` — added/removed routes, changed status codes, new error body fields.
  - **Env var lists** in `.env.example`, `docs/self-hosting.md`, `ARCHITECTURE.md` (Deployment), `SECURITY.md`.
  - **Webhook events** — `docs/webhooks.md` event list and the `README.md` features bullet.
  - **Status enums** (`EmailStatus`, suppression `reason`, etc.) referenced in any doc.
  - **"Tracked in #N"** references in `THREAT_MODEL.md` and `SECURITY.md` — once an issue ships, flip the residual-risk row from "tracked" to "mitigated".
  - **"Future / roadmap" / "v2+" sections** — drop items as they ship.
  - **Historical / planning docs** (e.g. `BunMail-Plan.md`) — should carry a "historical, see X for current state" header so readers don't mistake them for current.
- **Update `CHANGELOG.md` on every release.** When `bumpp` cuts a new version, add a corresponding entry summarizing user-facing changes (added / changed / fixed) under the new version heading, following Keep a Changelog format.

## Collaboration

- Be proactive: share honest opinions, suggest improvements, or flag concerns before proceeding.
- Think like a co-developer: challenge ideas constructively, propose alternatives, and plan together before executing.

## Git

- Don't commit unless explicitly asked.
- Don't push unless explicitly asked.
- Use descriptive commit messages focused on "why".

## Boundaries

- Do not create global helpers unless the same logic is needed in 3+ modules and fits under `src/utils/`.
- Do not introduce new frameworks or adapters; stick with Bun + Elysia + Nodemailer.
- Preserve existing formatting and indentation.
- Prefer explicit types for public APIs and service method parameters/returns.

---
> Source: [mohamedboukari/bunmail](https://github.com/mohamedboukari/bunmail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-10 -->

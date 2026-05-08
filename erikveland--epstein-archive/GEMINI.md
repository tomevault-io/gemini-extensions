## epstein-archive

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Epstein Archive is an investigative research platform for analyzing documents, entities, and relationships from the Epstein Files corpus. It is a monolith with a React 18 frontend, Express.js API server, and a PostgreSQL database.

## Commands

All commands run from `epstein-archive/` (the main app directory, not the repo root).

```bash
# Install dependencies
pnpm install

# Development (Vite on :3002, Express API on :3012)
pnpm dev          # frontend only
pnpm server       # API server only (run in separate terminal)

# Type checking and linting
pnpm type-check
pnpm lint
pnpm lint:fix
pnpm format:check

# Production build (runs CI gates first via prebuild:prod)
pnpm build:prod

# Boundary check (catch client importing server-only code)
pnpm check:boundaries

# Deploy to production
./deploy.sh

# Tests
pnpm test                               # all Playwright e2e tests
pnpm test:contracts                     # API DTO contract tests only
pnpm test:route-sync                    # route/UI sync test only
pnpm test:smoke                         # smoke tests (no browser)
playwright test tests/<file>.spec.ts    # run a single test file

# Database (Postgres)
pnpm db:migrate:pg                      # run migrations
pnpm schema:hash:check                  # verify schema hash
pnpm schema:hash:update                 # update schema hash after changes
pnpm verify                             # verify ops integrity

Pool selection for scripts:
- getIngestPool() — ingestion pipeline scripts (heavy workloads, 8 connections)
- getMaintenancePool() — maintenance/backfill/repair scripts (long timeouts, 256MB work_mem)
- getApiPool() — read-only analysis/debug scripts

# Workspace package
pnpm --filter @epstein/db build         # rebuild the @epstein/db package
```

## Architecture

### Repository layout

```
epstein-archive/         # Main application (has its own git repo)
├── src/
│   ├── client/          # React 18 SPA (@client/* alias)
│   │   ├── App.tsx      # Root component with lazy-loaded page routes
│   │   ├── components/  # UI components (Lucide icons, Tailwind, no emoji UI)
│   │   ├── contexts/    # AuthContext, FilterContext, SensitiveSettingsContext, InvestigationsContext
│   │   ├── pages/       # Route-level page components (lazy-imported)
│   │   ├── services/    # API client, navigation service
│   │   └── hooks/       # Custom React hooks
│   ├── server/          # Express API (@server/* alias)
│   │   ├── db/          # Repository pattern; one file per entity (Postgres via pg pool)
│   │   │   ├── connection.ts  # Re-exports from runtime.ts (pool singletons)
│   │   │   ├── runtime.ts     # Pool init, slow-query logging (threshold: PG_SLOW_QUERY_LOG_MS)
│   │   │   └── migrations/    # DB migration files
│   │   ├── routes/      # Express route handlers (one file per domain)
│   │   ├── auth/        # JWT auth middleware and routes
│   │   ├── middleware/  # Validate, requestId, rate-limit wrappers
│   │   └── utils/       # Startup validation, audit logger, error handler
│   ├── shared/          # Cross-environment code (browser + server safe); no Node-only modules
│   │   ├── contracts/   # Zod schemas for API contracts
│   │   └── dto/         # DTO types
│   ├── types.ts         # Root-level shared types
│   └── server.ts        # Express app entry point
├── packages/
│   └── db/              # @epstein/db workspace package (pgtyped-generated typed queries)
├── tests/               # Playwright tests (e2e + API contract + performance)
├── scripts/             # Maintenance, ingest pipeline, CI gate scripts
├── data/                # Raw corpus (gitignored; 300GB+ for full dataset)
└── public/              # Static assets
```

### Key architectural patterns

**Client/Server boundary**: The `check:boundaries` script enforces that `src/client` never imports from `src/server`. The `@client`, `@server`, and `@shared` path aliases map to these three lanes. Only code in `src/shared` is safe in both environments.

**Database layer**: Production uses PostgreSQL via the `pg` pool (three pools: `apiPool`, `maintenancePool`, `ingressPool`). The `@epstein/db` workspace package provides pgtyped-generated strongly-typed SQL queries.

**Repository pattern**: Each domain area has a dedicated repository file in `src/server/db/` (e.g. `entitiesRepository.ts`, `documentsRepository.ts`). Route handlers import from repositories, not directly from the pool.

**API contract enforcement**: DTOs and Zod schemas live in `src/shared/dto` and `src/shared/schemas`. The `tests/api-dto-contract.spec.ts` test validates that API responses match the declared contracts.

**Frontend routing**: React Router v6 with lazy-loaded page components. All pages are lazy-imported in `App.tsx` using `React.lazy()`.

**Vite dev proxy**: In development, Vite (:3002) proxies `/api` and `/files` to the Express server (:3012).

**Build chunking**: `vite.config.ts` splits vendor libs into `vendor-pdf`, `vendor-charts`, `vendor-icons`, and `vendor`. Feature-specific code splits into `feature-investigation`, `feature-media`, `feature-email`, `feature-documents`, `feature-network`.

### Data flow (ingestion)

Raw corpus (PDF/email/media) → ingest pipeline scripts (`scripts/ingest_pipeline.ts`) → OCR (Tesseract) / text extraction → entity/relationship extraction → PostgreSQL. An optional agentic intelligence stage (`scripts/ingest_intelligence.ts`) applies LLM-based contextual repair and entity linking.

## Environment setup

Copy `.env.example` to `.env` and set at minimum:

- `DATABASE_URL` — Postgres connection string
- `JWT_SECRET` / `SESSION_SECRET`

---
> Source: [ErikVeland/epstein-archive](https://github.com/ErikVeland/epstein-archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-24 -->

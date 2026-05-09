## hovod

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hovod is a self-hosted, open-source VOD (Video on Demand) platform. It handles video upload, transcoding to multi-quality HLS streams (360p/720p/1080p), and playback delivery via S3-compatible storage.

## Commands

### Full stack (Docker)
```bash
cp .env.example .env          # first time setup
docker compose up -d --build   # start all services
```

### Per-app development (requires local MySQL, Redis, S3)
```bash
npm run build                  # build all workspaces
npm run typecheck              # typecheck all workspaces
npm run lint                   # lint all workspaces (not yet configured)

# API (port 3000)
npm run dev -w @hovod/api     # watch mode with tsx
npm run build -w @hovod/api
npm run typecheck -w @hovod/api

# Worker
npm run dev -w @hovod/worker  # watch mode with tsx
npm run build -w @hovod/worker

# Dashboard (port 3001)
npm run dev -w @hovod/dashboard  # vite dev server
npm run build -w @hovod/dashboard

# DB package (must build before api/worker)
npm run build -w @hovod/db
```

### Build order
`@hovod/db` must be built first — both `@hovod/api` and `@hovod/worker` depend on it. The root `npm run build` handles this via workspace ordering.

## Architecture

```
Client → API (Fastify :3000) → MySQL (state) + Redis (queue) + S3 (storage)
                                        ↓ BullMQ job
                              Worker (FFmpeg) → S3 (HLS output)
Dashboard (React :3001) → API
```

**Monorepo** using npm workspaces with 4 packages:

- **`apps/api`** — Fastify REST server with modular route/service/middleware architecture. Env validated with Zod (`src/env.ts`). Enqueues transcode jobs to BullMQ.
  - `src/index.ts` — Entry point: Fastify setup, plugin registration, graceful shutdown
  - `src/db.ts` — Database connection, raw SQL migrations with FK constraints and indexes
  - `src/env.ts` — Zod-validated environment variables
  - `src/routes/assets.ts` — CRUD + upload/import/process endpoints for assets
  - `src/routes/playback.ts` — HLS playback URL resolution
  - `src/routes/health.ts` — Health check with DB connectivity verification
  - `src/services/asset.ts` — Shared business logic (findAssetOrFail, URL builders)
  - `src/middleware/error-handler.ts` — Centralized error handling (AppError, NotFoundError, ZodError)

- **`apps/worker`** — BullMQ consumer with modular FFmpeg/S3/transcoding modules. Env validated with Zod. Hardware-adaptive: auto-detects CPU/RAM at startup to set concurrency, FFmpeg threads, and DB pool size.
  - `src/index.ts` — Entry point: BullMQ worker setup, hardware-adaptive config, job orchestration, graceful shutdown
  - `src/env.ts` — Zod-validated environment variables (includes optional scaling overrides)
  - `src/ffmpeg.ts` — FFmpeg/ffprobe wrappers with timeout and error handling
  - `src/s3.ts` — S3 client singleton, streaming upload with createReadStream
  - `src/transcoding.ts` — Rendition profiles (TRANSCODING_LADDER), HLS transcoding, master playlist generation
  - `src/thumbnails.ts` — Thumbnail sprite generation and VTT output

- **`apps/dashboard`** — React SPA (Vite + Tailwind CSS v4) with component-based architecture. Polls API every 5s. Embeddable HLS player at `/embed/:playbackId` using hls.js.
  - `src/App.tsx` — Entry point with routing and layout
  - `src/components/` — AssetCard, AssetModal, EmbedPlayer, ImportForm, Player, StatusBadge, UploadZone
  - `src/lib/types.ts` — TypeScript interfaces (Asset, Rendition, AssetDetail, etc.)
  - `src/lib/api.ts` — API client helper with typed responses
  - `src/lib/helpers.ts` — Utility functions (timeAgo, formatDuration, STATUS_CFG, etc.)

- **`packages/db`** — Shared Drizzle ORM schemas, MySQL connection factory (`createDb()` accepts optional `DbPoolConfig`), and constants.
  - `src/schema.ts` — `assets`, `renditions`, `jobs` tables with FK constraints (CASCADE DELETE)
  - `src/client.ts` — `createDb(url, poolConfig?)` — MySQL pool factory with configurable `connectionLimit`, `queueLimit`, `idleTimeout`
  - `src/constants.ts` — Shared enums (ASSET_STATUS, JOB_STATUS, SOURCE_TYPE, S3_PATHS, ID_LENGTH)
  - `src/index.ts` — Exports `createDb()`, schemas, and constants

### Asset Lifecycle

`created` → (upload or import) → `uploaded` → (process) → `queued` → `processing` → `ready` | `error`

Delete is a soft delete (sets status to `deleted`).

### Key Conventions

- All API responses wrap data in `{ data: {...} }` or return `{ error: "..." }`
- Status strings use shared constants from `@hovod/db` (ASSET_STATUS, JOB_STATUS, etc.)
- IDs generated with `nanoid(12)`, playback IDs with `nanoid(16)` — lengths defined in `ID_LENGTH`
- DB column names use `snake_case`, Drizzle schema fields use `camelCase`
- S3 paths: sources at `sources/{assetId}/input.mp4`, HLS output at `playback/{assetId}/` — prefixes defined in `S3_PATHS`
- ESM throughout (`"type": "module"` in all packages), imports use `.js` extensions
- Error handling: API uses centralized error handler (AppError/NotFoundError), Worker wraps DB updates in try/catch for resilience
- No linter or test framework configured yet
- No API authentication (MVP stage)

### Database

MySQL 8.4 with Drizzle ORM (mysql2 driver). Three tables: `assets`, `renditions`, `jobs`. Schema defined in `packages/db/src/schema.ts` with foreign key references and CASCADE DELETE. Migrations are raw SQL `CREATE TABLE IF NOT EXISTS` in `apps/api/src/db.ts` with FK constraints and indexes on status columns. Connection pool is hardware-adaptive (auto-sized from RAM, overridable via `DB_POOL_SIZE`).

### Transcoding Profiles

Defined in `apps/worker/src/transcoding.ts` as `TRANSCODING_LADDER`:

| Quality | Resolution | Bitrate | Codec |
|---------|-----------|---------|-------|
| 360p    | 640x360   | 800k    | H.264 |
| 720p    | 1280x720  | 3000k   | H.264 |
| 1080p   | 1920x1080 | 6000k   | H.264 |

HLS segments are 6 seconds, VOD playlist type, AAC audio at 128k.

### Scaling & Hardware Adaptation

Worker and API auto-detect CPU cores and RAM at startup to configure concurrency, FFmpeg threading, and DB pool sizes. All values can be overridden via environment variables:

| Variable | Used by | Default | Description |
|----------|---------|---------|-------------|
| `WORKER_CONCURRENCY` | Worker | `min(RAM-based, CPU-based)` | Concurrent transcode jobs |
| `FFMPEG_THREADS` | Worker | `floor(CPU cores / concurrency)` | Threads per FFmpeg process |
| `DB_POOL_SIZE` | API, Worker | API: `min(50, max(10, RAM×3))` / Worker: `concurrency×2+2` | MySQL connection pool size |

Auto-detection logic (worker):
- **Concurrency**: `max(1, min(floor((RAM-1GB) / 1.5GB), floor(cores / 4)))` — constrained by the lesser of memory or CPU
- **FFmpeg threads**: `max(1, floor(cores / concurrency))` — distributes available cores across jobs (x264 sweet spot ~4)
- **DB pool**: `max(5, concurrency × 2 + 2)` — enough connections for concurrent jobs

### Environment Variables

Defined in `.env.example`. Both API and Worker validate all env vars at startup via Zod schemas in their respective `src/env.ts` files. Dashboard uses `VITE_API_BASE_URL` (build-time via Vite).

---
> Source: [Synapsr/Hovod](https://github.com/Synapsr/Hovod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-26 -->

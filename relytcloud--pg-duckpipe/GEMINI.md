## pg-duckpipe

> pg_duckpipe: PostgreSQL CDC extension — syncs heap tables to pg_ducklake columnar tables for HTAP.

# CLAUDE.md

## Project

pg_duckpipe: PostgreSQL CDC extension — syncs heap tables to pg_ducklake columnar tables for HTAP.

## Build & Test

```bash
make installcheck               # Build + install + run all regression tests
make check-regression TEST=api  # Run a single regression test
make check-daemon               # Run daemon E2E tests
make installcheck-all           # Both regression + daemon tests
make format                     # cargo fmt (CI checks all workspace crates)
```

Tests use a temporary PG instance on port 5555 (wal_level=logical). CI tests against PG 17 + 18.

## Workspace

```
duckpipe-core/     # Shared engine: decoder, DuckDB flush, streaming replication, metadata, connstr
duckpipe-pg/       # PG extension (pgrx): GUCs, SQL API, bgworker, remote group DDL
duckpipe-daemon/   # Standalone daemon (TCP, clap CLI)
test/regression/   # SQL regression tests
test/daemon/       # Daemon E2E tests
docker/            # Dockerfile, compose files, ducklake extension build script
```

## Architecture

```
Heap Tables → WAL → Replication Slot (pgoutput) → Decoder → FlushCoordinator → DuckLake
```

- **WAL consumer** (main thread): streaming replication via `START_REPLICATION`, pushes decoded changes to per-table shared queues
- **Flush threads** (per-table OS threads): self-trigger on queue size or time interval, own DuckDB connection + tokio runtime, handle PG metadata updates independently
- **Backpressure**: per-queue byte tracking pauses WAL consumer when total queued bytes exceed `max_queued_bytes`
- **Crash safety**: `confirmed_lsn = min(applied_lsn)` — slot never advances past durably flushed data
- **State machine** (per table): PENDING → SNAPSHOT → CATCHUP → STREAMING (or ERRORED with auto-retry)

## Key Rules

- Source tables must have PRIMARY KEY (upsert mode; append mode works without PK)
- Target auto-created as `{table}_ducklake` via explicit column definitions with PG→DuckDB type mapping + `USING ducklake`
- `add_table()` auto-starts the bgworker
- TRUNCATE uses per-table drain before DELETE (DuckLake ignores TRUNCATE)

## Documentation

User-facing docs live in `doc/` (QUICKSTART, USAGE, FAN_IN, REMOTE_SYNC, DAEMON, ACCESS_CONTROL, DATA_TYPES). Key developer docs:

- `doc/CODE_WALKTHROUGH.md` — detailed code walkthrough (read before major changes)
- `doc/PARALLELISM.md` — threading model, async tasks, backpressure
- `PROGRESS.md` — done/todo checklist + phase history

## Dev Guidelines

- **TDD**: failing test first → fix → `make installcheck` (all must pass)
- **Format before commit**: `make format` before committing (CI enforces `cargo fmt --all --check`)
- **Docs**: update relevant docs (see table above) after major changes
- **Diagrams**: follow `doc/img/STYLE.md` when creating or updating Excalidraw diagrams; export as PNG to `doc/img/`
- **Side effects**: always consider the side effects of code changes — deps, correctness, perf, overhead, stability

---
> Source: [relytcloud/pg_duckpipe](https://github.com/relytcloud/pg_duckpipe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-03 -->

## gig

> Lightweight task management system — Go CLI + SDK backed by SQLite.

# gig

Lightweight task management system — Go CLI + SDK backed by SQLite.

## Layout

```
gig/
├── gig.go              # Core types: Task, Comment, Checkpoint, Status, Priority, enums
├── store.go            # Store: Open/Close, ID generation, WAL+FK pragmas
├── task.go             # Task mutations: Create, Get, Update, Close, Cancel, Reopen, Claim, autoUnblock
├── query.go            # Task queries: List, Search, Ready(parentID), Blocked, Children, GetTree
├── checkpoint.go       # AddCheckpoint, ListCheckpoints, LatestCheckpoint
├── comment.go          # AddComment, ListComments
├── dependency.go       # AddDependency, RemoveDependency, ListDeps, DepTree, DetectCycles
├── attribute.go        # Custom attributes: DefineAttr, SetAttr, GetAttr, Attrs, DeleteAttr
├── event.go            # Events, EventsSince (audit log)
├── hook.go             # Shell hook execution from gig.yaml config
├── hooks.go            # //go:embed hooks + MaterializeHooks (gig install hooks)
├── config.go           # LoadConfig, SaveConfig, DefaultGigHome, gig.yaml parsing
├── export.go           # ExportJSONL, ImportJSONL, ExportEvents
├── util.go             # Time helpers, JSON label marshaling
├── doctor.go           # Doctor(): health checks
├── hooks/              # Embedded hook scripts (agent/ and git/)
├── ui/                 # Embedded web UI (kanban board)
├── cmd/gig/            # CLI (cobra) — thin wrapper over SDK
├── internal/migrate/   # Versioned SQLite schema migrations (v1: core, v2: attrs, v3: checkpoints)
├── examples/           # SDK usage examples
├── docs/               # Detailed guides (testing, adding commands, architecture)
└── *_test.go           # SDK unit tests
```

## Build & Test

```bash
go build -o gig ./cmd/gig/                       # Build binary
go test ./...                                      # SDK unit tests
go test -tags=e2e -v -count=1 ./cmd/gig/           # E2E CLI tests
go vet ./...                                       # Static analysis
```

## Conventions

- **SDK-first**: All logic in root package. CLI (`cmd/gig/`) is a thin cobra wrapper. Never put business logic in `cmd/`.
- **Module path**: `github.com/NeerajG03/gig` — must match GitHub repo URL.
- All public SDK functions return `(*Type, error)` or `error`.
- Use `UpdateParams` with pointer fields (nil = don't change) for partial updates.
- `GIG_HOME` env var overrides `~/.gig/` as the central storage location.
- Error wrapping: `fmt.Errorf("context: %w", err)`. Never swallow errors.
- Parameterized SQL: all queries use `?` placeholders. Never `fmt.Sprintf` for SQL.
- Tests use `tempDB(t)` helper for isolated in-memory stores.

## Status Model

```
open → in_progress → closed
  ↓         ↓          ↓
blocked   cancelled   (reopen → open)
  ↓
  (auto-unblock → open when all blockers terminal)
```

- **`IsTerminal()`**: returns true for `closed` and `cancelled`. Always use this instead of `== StatusClosed`.
- **Auto-unblock**: closing/cancelling a task auto-transitions `blocked` dependents to `open` if all their blockers are terminal.
- **`Ready(parentID)`**: returns only `open` tasks with no unresolved blockers. `in_progress` = already claimed, not "ready".
- **Icons**: `[ ]` open, `[>]` in_progress, `[!]` blocked, `[~]` deferred, `[x]` closed, `[-]` cancelled.

## What NOT to Do

- Don't add CGO dependencies — pure-Go SQLite is intentional.
- Don't put business logic in `cmd/gig/`.
- Don't scan `time.Time` directly from SQLite — scan as string, parse with `strToTime()`.
- Don't check `== StatusClosed` — use `IsTerminal()`.
- Don't break JSONL format without a major version bump.
- Don't remove columns from migrations — only add.
- Don't re-tag released versions — Go module proxy caches checksums.

## Key Types

- **Checkpoints** vs **Comments**: Checkpoints are structured snapshots (done, decisions, next, blockers, files) in their own table. Comments are freeform. `gig show` displays latest checkpoint.
- **Custom attributes**: Two-layer — definitions registry (key + type) → per-task values. Types: string, boolean, object.
- **Events**: Every mutation auto-records an event. DB write → event insert → SDK callbacks → shell hooks.

## JEFF Integration

gig is the state layer for [JEFF](https://github.com/NeerajG03/JEFF). JEFF imports gig as a Go SDK — never shells out to CLI. JEFF defines custom attrs: `repos` (object) and `worktree_setup` (string).

## Further Reading

- `docs/testing.md` — test infrastructure, how to write tests, existing test files
- `docs/adding-commands.md` — step-by-step guide for new CLI commands
- `docs/architecture.md` — sync model, schema, design rationale
- `docs/sdk-reference.md` — full SDK API reference

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NeerajG03) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-09 -->

## watchfire

> Watchfire orchestrates coding agents sessions based on specs (project definitions and tasks). Read `ARCHITECTURE.md` for the full design.

# Watchfire - Development Guide

## What is Watchfire?

Watchfire orchestrates coding agents sessions based on specs (project definitions and tasks). Read `ARCHITECTURE.md` for the full design.

## Source of Truth

`ARCHITECTURE.md` is the single source of truth for:
- Component responsibilities
- Data structures
- Directory layout
- Build phases
- Tech stack

If this file and `ARCHITECTURE.md` conflict, `ARCHITECTURE.md` wins.

For all decisions, use architecture document as reference. If you need to do something different, please update atchitecture after checking with user. 

## Task-completion lifecycle (v7.0 Relay)

When an agent finishes a task, the daemon picks one of two merge paths via the
on-task-done hook (`internal/daemon/agent/taskdone.go:HandleTaskDone`):

- **Silent merge (default).** `git merge --no-ff watchfire/<n>` lands the work
  on the project's default branch and the worktree is removed. This matches
  the v6.x behaviour and is the path every project gets unless they opt in
  to auto-PR.
- **GitHub auto-PR.** If `~/.watchfire/integrations.yaml` enables
  `github.auto_pr` for the project (either globally or via `project_scopes`),
  the daemon pushes `watchfire/<n>` to GitHub and opens a PR via `gh api`
  with body rendered from the task metadata + v6.0 diff stats. The local
  merge is suppressed; the user reviews and merges in GitHub. The worktree
  is still cleaned up after PR creation.

Auto-PR requires the `gh` CLI on PATH and `gh auth status` returning 0. If
either fails, the daemon logs a single WARN per project lifetime and falls
back to silent merge — no task ever strands inside an unmerged worktree.
Push failures and `gh api` errors fall back the same way but log per failure.

To enable, set `github.enabled: true` in `~/.watchfire/integrations.yaml` and
optionally restrict to specific projects via `github.project_scopes: [<id>...]`.

## Repository Structure

```
watchfire/
├── ARCHITECTURE.md         # Single source of truth
├── CLAUDE.md               # This file - development guide
├── assets/                 # Images, logos, brand references (shared across components)
├── proto/                  # gRPC protobuf definitions
│   └── watchfire.proto
├── daemon/                 # watchfired - Go daemon
│   ├── cmd/
│   │   └── watchfired/
│   │       └── main.go
│   ├── internal/
│   └── go.mod
├── cli/                    # watchfire - Go CLI/TUI
│   ├── cmd/
│   │   └── watchfire/
│   │       └── main.go
│   ├── internal/
│   └── go.mod
└── gui/                    # Electron GUI (future)
```

---
> Source: [watchfire-io/watchfire](https://github.com/watchfire-io/watchfire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-03 -->

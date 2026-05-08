## wt

> `wt` is a Git worktree management CLI written as a single Zsh script (`wt.zsh`, ~4400 lines). It's distributed as one file for easy `curl` installation. All code lives in `wt.zsh` — do not split it into multiple files.

# CLAUDE.md

## What this project is

`wt` is a Git worktree management CLI written as a single Zsh script (`wt.zsh`, ~4400 lines). It's distributed as one file for easy `curl` installation. All code lives in `wt.zsh` — do not split it into multiple files.

## Architecture

The script is organized into logical sections (top to bottom):

1. **Pattern collections & constants** (lines 1–123) — lockfile mappings, dependency dirs, sync categories, color constants
2. **UI helpers** (125–487) — banner, spinners, arrow-key pickers, formatting
3. **Config management** (488–602) — `~/.config/wt/<repo>.conf` key=value files
4. **`init` command** (604–1058) — interactive setup wizard
5. **Repo utilities** (1059–1130) — repo root, main worktree, default branch detection
6. **Theme management** (1130–1255) — 9-theme rotation pool, IDE settings.json manipulation
7. **File syncing** (1257–1314) — rsync-based gitignored file sync across worktrees
8. **Workspace handling** (1316–1567) — `.code-workspace` file detection and navigation
9. **Open/cd navigation** (1568–1691)
10. **Worktree initialization** (1693–2556) — parallel background tasks (sync, theme, deps), status rendering, post-creation flow
11. **`create` command** (2557–3064) — PR support, stash handling, branch creation
12. **Deletion & list helpers** (3065–3542)
13. **`sync`, `rename`, `clean`, `open` commands** (3543–3920)
14. **Help & main dispatcher** (4161–4295)
15. **Zsh completion** (4296–4398)

## Conventions

- **All functions** are prefixed `_wt_` (private). The only public function is `wt()`.
- **Global constants** use `_WT_` uppercase. Mutable state uses `_wt_` lowercase.
- **Per-invocation caches** are reset at the top of `_wt_main()` to prevent staleness.
- **Commands** have long and short forms (e.g. `create`/`c`, `delete`/`d`, `list`/`ls`/`l`).
- **Return codes**: 0=success, 1=error, 255=cancelled.
- **Python** is used for three things only: banner rendering, JSON/JSONC parsing, and workspace file detection. Don't add new Python dependencies.
- **Parallel tasks** use `&!` (disown) with PID tracking via temp files for status signaling.

## Commands to know

```sh
make lint    # Zsh syntax check + ShellCheck (uses bash approximation)
source wt.zsh  # Load into current shell for manual testing
```

There are no automated tests beyond linting. Test changes manually by sourcing the file and running commands.

## Common pitfalls

- ShellCheck runs in `--shell=bash` mode since it doesn't support Zsh. Many Zsh-isms are suppressed in `.shellcheckrc`. Don't "fix" valid Zsh syntax to satisfy ShellCheck.
- The interactive pickers (`_wt_pick`, `_wt_multi_pick`) use raw terminal escape sequences — be careful with changes there.
- Background task orchestration in `_wt_worktree_init` is subtle — tasks signal status through marker files in a temp directory. Understand the flow before modifying.

---
> Source: [neefrehman/wt](https://github.com/neefrehman/wt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-24 -->

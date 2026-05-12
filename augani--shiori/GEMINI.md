## shiori

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Shiori?

A lightweight, GPU-accelerated code editor for macOS built in Rust using GPUI (custom fork `adabraka-gpui`). Designed as a terminal-first editor for AI coding agent workflows.

## Build Commands

```bash
cargo +nightly build --release    # Build (requires Rust nightly)
cargo +nightly build              # Debug build
cargo +nightly clippy             # Lint
bash macos/bundle.sh              # Create macOS .app bundle
```

No test suite exists yet. The project has no `cargo test` targets.

## Architecture

Single-binary macOS app. All source is in `src/`.

### Core Files

- **`main.rs`** — Entry point: asset loading (checks `.app` bundle vs dev paths), window creation, CLI arg parsing
- **`app.rs`** (~4,800 lines) — Central state machine (`AppState`): tab management, file explorer tree, keybindings/actions, sidebar views, command palette. This is the largest file and the hub of the application.
- **`ide_theme.rs`** — 6 built-in themes synced via `adabraka-ui`

### Terminal System

- **`terminal_view.rs`** — Terminal UI rendering (ANSI colors, styles, mouse events)
- **`terminal_state.rs`** — Terminal state management, PTY lifecycle, input handling
- **`ansi_parser.rs`** — Full ANSI escape sequence parser (256 colors, OSC 8 hyperlinks)
- **`pty_service.rs`** — PTY creation/management via `portable-pty`

### Git Integration

- **`git_service.rs`** — Git operations via `libgit2` (git2 crate)
- **`git_state.rs`** — Git state machine with 3-second polling
- **`git_view.rs`** — Split/unified diff viewer, staging, commit UI
- **`diff_highlighter.rs`** — Diff syntax highlighting

### LSP (Optional)

- **`lsp/client.rs`** — LSP client: completions, diagnostics, hover, go-to-definition
- **`lsp/transport.rs`** — Async IPC transport via channels
- **`lsp/registry.rs`** — Server lifecycle management
- **`lsp/config.rs`** — Pre-configured servers: rust-analyzer, typescript-language-server, pyright, gopls, clangd, lua-language-server, zls

### Completion & Search

- **`completion/`** — Autocomplete: tree-sitter symbol extraction (24 languages), prefix filtering, anchor-positioned popup
- **`search_bar.rs`** — Find/replace with regex support

### Other

- **`settings.rs`** — Configuration management, language server setup
- **`autosave.rs`** — 2-second debounced auto-save

## Key Dependencies

- `adabraka-gpui` (v0.5) — GPU-accelerated UI framework (custom GPUI fork)
- `adabraka-ui` (local path `../adabraka-ui`) — Shared UI component library with editor language support
- `portable-pty` — PTY support for terminal
- `tree-sitter` — Syntax highlighting and symbol extraction
- `git2` — Git operations via libgit2
- `lsp-types` — LSP protocol types
- `thiserror` — Custom error types

## Patterns

- **GPUI Entity model**: State is managed via `Entity<T>` with `cx.notify()` for reactive UI updates
- **Error handling**: Uses `thiserror` for typed errors (e.g., `PtyError`, `TransportError`), `?` propagation
- **Asset loading**: Runtime detection of `.app` bundle (`Contents/MacOS/` path check) vs `CARGO_MANIFEST_DIR` for dev
- **Async channels**: `flume` for cross-thread communication (terminal I/O, LSP transport)

## macOS Specifics

- Bundle ID: `com.augani.shiori`
- Minimum macOS: 13.0
- Metal-accelerated rendering
- CI builds universal binaries (arm64 + x86_64) via `lipo`
- Signing, notarization, and DMG creation in `macos/sign.sh`

---
> Source: [Augani/shiori](https://github.com/Augani/shiori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-03 -->

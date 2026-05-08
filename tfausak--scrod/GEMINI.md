## scrod

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Scrod?

Scrod is a Haskell documentation tool that parses Haskell source code using the GHC API and outputs documentation as HTML or JSON. It has a CLI tool (reads from stdin) and a WASM-based web app deployed to GitHub Pages with a live split-pane editor.

**Pipeline:** Haskell Source → GHC Parser → GHC AST → Scrod Core Types → HTML or JSON

## Building, Testing, and Conventions

See [CONTRIBUTING.md](CONTRIBUTING.md) for build commands, test filtering, linting, code style, and git conventions.

## Architecture

### Source layout

- `source/library/` — all library code under `Scrod.*` modules
- `source/executables/` — CLI (`scrod`), WASM (`scrod-wasm`), and WASI (`scrod-wasi`) entry points
- `source/test-suite/` — test suite entry point (uses Tasty/HUnit via custom `Scrod.Spec` abstraction)
- `extra/github-pages/` — web app frontend source (HTML, JS, CSS) deployed to GitHub Pages
- `extra/wasm/` — WASM cross-compilation build script and cabal project file
- `extra/vscode/` — VSCode extension (TypeScript + esbuild) for live documentation preview

### Key module groups

- **`Scrod.Core.*`** — Data types for the intermediate representation (Module, Item, Doc, ItemKind, Visibility, etc.). Each type lives in its own module with a `Mk`-prefixed constructor (e.g., `MkModule`, `MkItem`). Items carry a `Visibility` field (`Exported`, `Implicit`, or `Unexported`) and are ordered in the `Module.items` list to reflect export-list order.
- **`Scrod.Convert.FromGhc`** — Converts GHC AST into Scrod Core types. Uses a `ConvertM` state monad that assigns auto-incrementing `ItemKey`s. Has many submodules (`ExportOrdering`, `Visibility`, `ItemKind`, `Exports`, `Names`, etc.) that handle specific conversion concerns.
- **`Scrod.Convert.FromHaddock`** — Converts Haddock doc strings into Scrod's `Doc` type.
- **`Scrod.Convert.ToHtml`** — Renders Core types as self-contained HTML (Bootstrap 5 + KaTeX).
- **`Scrod.Convert.ToJson`** — Renders Core types to JSON output.
- **`Scrod.Ghc.*`** — Wrappers around GHC API internals to set up parsing without a full GHC session.
- **`Scrod.Json.*`** — Custom implementation for JSON generation (no external libraries).
- **`Scrod.Xml.*`** — Custom XML generation library used by `ToHtml`.

### Testing

Tests use `Scrod.Spec`, a custom test specification DSL (`MkSpec` record with `assertFailure`/`describe`/`it`) that abstracts over the test framework. The test-suite entry point (`source/test-suite/Main.hs`) wires the DSL into Tasty/HUnit.

There are two kinds of tests:

- **Unit tests** — defined inline in library modules via a `spec` function (e.g., `Scrod.Decimal.spec`). These test individual functions directly.
- **Integration tests** — in `Scrod.TestSuite.Integration`. These run the full pipeline (parse → convert → JSON) and assert on JSON output using JSON Pointer paths. The `check` helper takes a Haskell source string and a list of `(pointer, expected JSON)` pairs.

`Scrod.TestSuite.All` aggregates all module specs (both unit and integration) in one place. When adding a new module with a `spec` function, it must be registered here.

### WASM web app

The WASM build exports a `scrod` function via JavaScript FFI that supports both HTML and JSON output via `--format`. The web frontend source lives in `extra/github-pages/` and runs it in a Web Worker (`worker.js`) to avoid blocking the UI. Shareable URLs use base64-encoded hash fragments.

---
> Source: [tfausak/scrod](https://github.com/tfausak/scrod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-24 -->

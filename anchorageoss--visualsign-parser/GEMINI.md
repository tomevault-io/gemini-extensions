## visualsign-parser

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

All commands run from `src/`:

```bash
make -C src build          # Build all workspace crates
make -C src test           # Build all, then run all tests (integration tests need binaries)
make -C src lint           # cargo clippy --all-targets -- -D warnings
make -C src fmt            # cargo fmt
make -C src generated      # Regenerate protobuf types (tonic_build), then fmt
make -C src grpc-server    # Run the gRPC server locally
```

Run a single test:
```bash
cargo test -p visualsign-ethereum test_name
```

Parse a transaction locally:
```bash
cargo run --bin parser_cli -- --chain ethereum --network ETHEREUM_MAINNET --output human -t <hex>
```

CI requires: codegen produces no diff, clippy passes with `-D warnings`, all tests pass. Protoc v21.4.

## Architecture

**Multi-chain transaction parser** — converts raw blockchain transactions (hex/base64) into structured VisualSign JSON payloads for human-readable display in wallets.

### Core Flow

```
Raw tx bytes → ChainPlugin (CLI) or gRPC request
  → TransactionConverterRegistry (dispatches by chain)
    → VisualSignConverter<T> (chain-specific conversion)
      → SignablePayload (deterministic JSON output)
```

### Workspace Layout (src/)

- **`visualsign`** — Core library: `SignablePayload` types, field builders, `Transaction`/`VisualSignConverter` traits, `DeterministicOrdering` trait, error types
- **`chain_parsers/visualsign-{ethereum,solana,sui,tron,unspecified}`** — Per-chain converter crates. Ethereum and Solana are feature-gated (both on by default)
- **`parser/cli`** — CLI binary with `ChainPlugin` trait for per-chain args/metadata/registration
- **`parser/app`** — Enclave/VM binary using vsock + protobuf IPC (links qos_* modules)
- **`parser/grpc-server`** — tonic gRPC server wrapping parser_app
- **`generated`** — Protobuf codegen output (do not edit; run `make generated`)
- **`codegen`** — tonic_build script that generates protobuf types with serde+borsh derives
- **`integration`** — gRPC integration tests against parser_app

### Key Traits

- **`Transaction`** — Parse from string, identify transaction type
- **`VisualSignConverter<T>`** — Convert a `Transaction` into `SignablePayload`
- **`VisualSignConverterAny`** — Type-erased version for polymorphic registry storage
- **`ChainPlugin`** — CLI-only: register converter + build chain metadata from args
- **`DeterministicOrdering`** — Alphabetical field ordering for stable metadata hashing

### Ethereum-Specific Patterns

- **`VisualizerContext`** — Carries chain_id, sender, contract, calldata, registries; cloned with incremented depth for nested calls
- **`ContractRegistry`** — Maps `(chain_id, Address) → TokenMetadata` for token resolution
- **`LayeredRegistry<T>`** — Composes wallet-provided + compiled-in data
- **Protocol decoders** — Use `sol!` macro for type-safe ABI decoding; follow 4-step pattern: decode params → resolve tokens → format amounts → return field
- **Field builders** (`visualsign::field_builders`) — Always use `create_text_field`, `create_amount_field`, `create_number_field`, `create_address_field`, `create_raw_data_field` instead of constructing field structs directly
- **ASCII only** — Use `>=` not `≥`, `->` not `→` (terminal compatibility)

### Testing Patterns

- Fixture-based snapshot tests: `tests/fixtures/{name}.input` + `{name}.expected` pairs per chain crate
- Integration tests in `integration/tests/` use gRPC client against built binaries
- `test_utils` module in `visualsign` provides shared test helpers
- Place all `use` imports at the top of the test module, not inside individual test functions

### Local Dev Container

A unified Docker container (see `images/parser_app/Containerfile`) bundles parser_app + simulator_enclave + parser_host + Go gateway into a single image for non-TEE local development. Same API as production TDX deployment, only difference is no attestation. REST at `:8080`, gRPC at `:44020`. Build with `make non-oci-docker-images` from repo root.

### Workspace Lint Policy

Workspace-level clippy lints are enforced in `src/Cargo.toml`:
- **`unwrap_used = "deny"`** — Use `?` operator or explicit error handling instead of `.unwrap()`
- **`expect_used = "deny"`** — Same; use `?` or `.ok_or_else(|| ...)?`
- **`panic = "deny"`** — Return `Err(...)` instead of `panic!()`
- **`unsafe_code = "forbid"`** — No `unsafe` blocks

Exceptions: test modules use `#[allow(clippy::unwrap_used, clippy::expect_used, clippy::panic)]`. Build scripts allow `unwrap_used`. Some crates have temporary crate-level exemptions with `TODO(#231)` pending cleanup.

### Design Decisions

- **Deterministic serialization everywhere** — BTreeMap for proto maps, `DeterministicOrdering` trait, alphabetical field ordering for stable metadata hashing (borsh encoding)
- **Bounded readers** — File loading capped at 10MB to prevent DoS
- **Type-erased converters** — `VisualSignConverterAny` trait objects for polymorphic registry without generics overhead
- **Feature gates for chains** — Ethereum/Solana gated, extensible to new chains
- **Rust edition 2024** on nightly channel 1.88

---
> Source: [anchorageoss/visualsign-parser](https://github.com/anchorageoss/visualsign-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-06 -->

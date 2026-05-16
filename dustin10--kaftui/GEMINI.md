## kaftui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

kaftui is a Rust TUI application for interacting with Apache Kafka. It uses ratatui for rendering, tokio for async, crossterm for terminal events, and rdkafka (dynamically linked with SASL/SSL) for Kafka connectivity. It supports JSON Schema, Avro, and Protobuf deserialization via Schema Registry integration.

## Build & Development Commands

```bash
cargo build                  # Debug build
cargo build --release        # Release build
cargo clippy                 # Lint
cargo test                   # Run tests
cargo run -- --bootstrap-servers localhost:9092  # Run with args
```

Requires Rust 1.89+ and system librdkafka (rdkafka uses dynamic-linking feature).

Enable debug logging: `KAFTUI_LOGS_ENABLED=true KAFTUI_LOGS_DIR=./logs`

## Architecture

**Async event-driven architecture** — the main loop uses `tokio::select!` to multiplex terminal events, Kafka consumer events, application events (EventBus), log events, and a tick timer. Events are spawned as independent tasks rather than blocking.

### Key Modules

- **`main.rs`** — CLI arg parsing (clap), config loading, deserializer setup, launches `App::run()`
- **`app/mod.rs`** — Central `App` struct orchestrating the event loop, component management, consumer lifecycle, and key binding routing
- **`app/config.rs`** — `Config` merges CLI args + persisted config (`$HOME/.kaftui.json`) + profile defaults. `PersistedConfig` handles profiles, themes, and settings
- **`app/export.rs`** — Record/schema/topic export to disk
- **`event.rs`** — `EventBus` (unbounded channel) and `Event` enum (30+ variants for all app actions)
- **`kafka/mod.rs`** — Kafka `Consumer` wrapping rdkafka `StreamConsumer`, `Record` struct, `Format` enum, `ConsumerMode`, `SeekTo`
- **`kafka/admin.rs`** — Topic browsing and metadata retrieval
- **`kafka/de.rs`** — Deserializers: String, JsonString, JsonSchema, AvroSchema, ProtobufSchema
- **`kafka/schema.rs`** — Schema Registry client wrapper
- **`ui/mod.rs`** — `Component` trait: `name()`, `render()`, `map_key_event()`, `update()`
- **`ui/records.rs`, `topics.rs`, `schemas.rs`, `settings.rs`, `stats.rs`, `logs.rs`** — Screen components implementing the Component trait
- **`trace.rs`** — Custom tracing `CaptureLayer` that buffers logs for in-app display + file output

### Data Flow

```
CLI Args → Config (merged with persisted profile) → App::new()
Kafka Consumer → Records Channel → EventBus → App::handle_event() → Component Updates → UI Render
Schema Registry (optional) feeds deserializers for Avro/JSON Schema/Protobuf payloads
```

### UI Component Pattern

All screens implement the `Component` trait. The `App` routes keyboard events to the active component's `map_key_event()`, which returns an `Event`. The app then calls `update()` on relevant components. Rendering happens each loop iteration via `render()`.

---
> Source: [dustin10/kaftui](https://github.com/dustin10/kaftui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-13 -->

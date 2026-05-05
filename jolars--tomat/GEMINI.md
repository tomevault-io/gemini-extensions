## tomat

> **tomat** is a Pomodoro timer with daemon support designed for waybar and other

# LLM Agent Instructions for tomat

## Repository Overview

**tomat** is a Pomodoro timer with daemon support designed for waybar and other
status bars. It's a Rust project (~4,300 lines across multiple modules) that
implements a server/client architecture using Unix sockets for inter-process
communication.

**Key Details:**

- **Language:** Rust (2024 edition)
- **Architecture:** Client/server with Unix socket communication
- **Target:** Linux systems with systemd user services
- **Purpose:** Lightweight Pomodoro timer for waybar integration
- **Dependencies:** Standard Rust ecosystem (tokio, clap, serde, chrono,
  notify-rust, fs2, rodio)
- **Testing:** Comprehensive integration tests (37 tests across 7 modules
  covering all functionality)

## Build & Development Environment

### Prerequisites

- Rust stable toolchain (specified in `rust-toolchain.toml`)
- Cargo for building and dependency management
- Optional: Task runner (`go-task`) for development workflows
- Optional: Nix/devenv for reproducible development environment

### Essential Build Commands

**Always run commands from the repository root (`/home/jola/projects/tomat`).**

1. **Quick development check:**

   ```bash
   task dev
   ```

   This runs: `cargo check` → `cargo test` →
   `cargo clippy --all-targets --all-features -- -D warnings`

2. **Individual commands:**

   ```bash
   # Check compilation without building
   cargo check

   # Run tests (comprehensive integration test suite)
   cargo test

   # Run specific test categories by module
   cargo test --test cli integration::timer      # Timer behavior tests
   cargo test --test cli integration::daemon     # Daemon management tests
   cargo test --test cli integration::formats    # Output format tests
   cargo test --test cli integration::commands   # Command validation tests
   cargo test --test cli integration::hooks      # Hook execution tests

   # Lint with clippy - MUST pass with zero warnings
   cargo clippy --all-targets --all-features -- -D warnings

   # Check code formatting
   cargo fmt -- --check

   # Format code
   cargo fmt
   ```

3. **Build commands:**

   ```bash
   # Development build (fast)
   cargo build

   # Release build (optimized, ~1.2s from clean)
   cargo build --release

   # Clean build (from clean state takes ~10s for dependencies)
   cargo clean && cargo build
   ```

4. **Installation:**

   ```bash
   # Quick install with systemd service setup
   ./install.sh

   # Manual install
   cargo install --path .
   ```

### Pre-commit Validation

**CRITICAL:** All code changes MUST pass these checks before commit:

1. **Formatting:** `cargo fmt -- --check` (MUST exit with code 0)
2. **Linting:** `cargo clippy --all-targets --all-features -- -D warnings` (MUST
   exit with code 0, no warnings allowed)
3. **Compilation:** `cargo check` (MUST pass)
4. **Tests:** `cargo test` (37 integration tests must pass)

**Pre-commit hooks are configured** in `.pre-commit-config.yaml` and will run
clippy and rustfmt automatically if using the Nix devenv.

## Project Layout & Architecture

### File Structure

```
/
├── src/
│   ├── main.rs               # Entry point and command dispatching
│   ├── cli.rs                # CLI argument parsing with clap (command definitions)
│   ├── config.rs             # Configuration system (timer, sound, notification settings)
│   ├── server.rs             # Unix socket server, daemon logic, and process management
│   ├── timer.rs              # Timer state management, phase transitions, and notification system
│   └── audio.rs              # Sound playback system with embedded audio files
├── tests/
│   ├── cli.rs                # Integration test entry point
│   └── integration/          # Modular integration test modules
│       ├── mod.rs           # Module declarations
│       ├── common.rs        # Shared test utilities (TestDaemon helper)
│       ├── daemon.rs        # Daemon lifecycle tests
│       ├── timer.rs         # Timer behavior and auto-advance tests
│       ├── commands.rs      # Command validation tests
│       ├── formats.rs       # Output format tests
│       └── hooks.rs         # Hook execution tests
├── docs/
│   ├── book.toml             # mdbook configuration
│   └── src/                  # Documentation source (markdown)
│       ├── SUMMARY.md       # Navigation structure
│       ├── index.md         # Documentation index
│       ├── overview.md      # Architecture, quick start, examples
│       ├── configuration.md # Configuration guide
│       ├── integration.md   # Waybar, systemd, hooks
│       ├── troubleshooting.md # Common issues
│       └── cli-reference.md # Auto-generated from clap (DO NOT EDIT)
├── assets/
│   ├── icon.png              # Embedded notification icon
│   └── sounds/               # Embedded audio files
├── images/
│   ├── logo.svg              # Source logo (visual identity)
│   ├── logo.png              # Generated logo for GitHub/docs (256x256)
│   └── og.png                # Generated social media image (1280x640)
├── build.rs                  # Build script for man pages, mdbook, icons, completions
├── Cargo.toml               # Dependencies and metadata, includes cargo-deb config
├── Cargo.lock               # Dependency lockfile
├── Taskfile.yml             # Task runner commands (dev, lint, build-release, test-*)
├── rust-toolchain.toml      # Rust version specification (stable)
├── install.sh               # Installation script with systemd setup
├── .github/
│   ├── workflows/
│   │   ├── build-and-test.yml    # CI: tests on Ubuntu/Windows/macOS + security audit
│   │   └── release.yml           # Semantic release workflow
│   └── dependabot.yml            # Dependency updates
├── .pre-commit-config.yaml  # Pre-commit hooks for formatting and linting
├── .releaserc.json          # Semantic release configuration
└── devenv.*                 # Nix development environment files
```

### Code Architecture

The project is organized into six main modules:

- **`main.rs`**: Entry point, command dispatching to server/client functions, and
  client-side formatting logic (applies text templates to timer status)
- **`cli.rs`**: CLI argument parsing with clap, defines all commands and their
  arguments using derive macros
- **`config.rs`**: Configuration system with timer, sound, notification, and display
  settings loaded from TOML
- **`server.rs`**: Unix socket server implementation, client communication
  handling, daemon process management (PID files, graceful shutdown), timer
  event loop, and configuration loading. Returns raw `TimerStatus` data.
- **`timer.rs`**: Timer state management (`TimerState`), phase transitions, 
  notification system, and client-side formatting. Contains `TimerStatus` struct
  (pure state) and `format_status()` method (presentation logic).
- **`audio.rs`**: Sound playback system with embedded audio files (compiled with
  `audio` feature flag), handles phase transition sounds via rodio
- **`tests/`**: Modular integration test suite with 37 tests across 7 modules
  - **`cli.rs`**: Integration test entry point
  - **`integration/common.rs`**: Shared TestDaemon helper and utilities
  - **`integration/timer.rs`**: Timer behavior and auto-advance tests
  - **`integration/daemon.rs`**: Daemon lifecycle tests
  - **`integration/formats.rs`**: Output format tests
  - **`integration/commands.rs`**: Command validation tests
  - **`integration/hooks.rs`**: Hook execution tests

**Communication flow:**

- **Single binary** with subcommands: `daemon start|stop|status|run`, `start`,
  `stop`, `status`, `skip`, `toggle`
- **Daemon mode:** Runs continuously, listens on Unix socket at
  `$XDG_RUNTIME_DIR/tomat.sock`
- **Client mode:** All other commands send requests to daemon via socket
- **Timer state:** Manages work/break/long-break phases with configurable
  auto-advance behavior
- **Data flow:** Server returns `TimerStatus` (pure state: phase, remaining_seconds, 
  is_paused, etc.) → Client applies formatting (icons, state symbols, tooltips) → 
  Output in requested format (waybar JSON, plain text, i3status-rs JSON)
- **Separation of concerns:** Server has no knowledge of presentation; all icons,
  symbols, CSS classes, and tooltips are generated client-side

### Key Dependencies

- `tokio`: Async runtime for socket handling and timers
- `clap`: Command-line argument parsing with derive macros
- `serde`/`serde_json`: Serialization for client/server communication
- `chrono`: Time handling with serialization support
- `dirs`: Standard directory discovery
- `libc`: Unix user ID access and process management
- `notify-rust`: Desktop notifications for phase transitions
- `fs2`: File locking for daemon instance prevention (prevents race conditions)
- `toml`: Configuration file parsing
- `tempfile` (dev-dependency): Temporary directories for integration tests

### Documentation Architecture

The project uses a **single source of truth** approach with automatic generation:

**Source:**
1. **`src/cli.rs`** - Clap command definitions (separated from main.rs)
   - Auto-generates → Section 1 man pages via `clap_mangen` (17 command pages)
   - Auto-generates → `docs/src/cli-reference.md` via `clap-markdown`

2. **`docs/src/*.md`** - Hand-written guides in clean markdown
   - `overview.md` - Architecture, quick start, examples
   - `configuration.md` - Configuration guide
   - `integration.md` - Waybar, systemd, hooks
   - `troubleshooting.md` - Common issues
   - `cli-reference.md` - **DO NOT EDIT** (auto-generated)

**Output:**
- **Man pages**: `target/man/*.1` - CLI command reference
- **HTML docs**: `docs/book/html/` - mdbook with sidebar, search, themes

**Build process:** `cargo build` runs:
1. `clap_mangen` - Generates man pages from clap definitions
2. `clap-markdown` - Generates CLI reference markdown
3. `mdbook` - Builds HTML documentation

**Important:**
- Never edit `docs/src/cli-reference.md` - it's regenerated on every build
- To update CLI docs, modify the clap definitions in `src/cli.rs`
- Write guides in normal markdown (no man page YAML frontmatter)
- Only section 1 man pages are generated (commands), not sections 5/7

## Continuous Integration

### GitHub Actions Workflows

1. **build-and-test.yml** (runs on PR, push to main):
   - **Multi-platform:** Ubuntu, Windows, macOS
   - **Steps:** Build → Test → Clippy → Format check
   - **Security:** RustSec security audit
   - **Caching:** Cargo registry and target directory

2. **release.yml** (manual trigger):
   - **Semantic release** with conventional commits
   - **Automated:** Version bumping, changelog, GitHub releases

### Validation Pipeline

Your changes will be validated against:

1. **Compilation** on all three platforms
2. **Zero clippy warnings** (with `-D warnings` flag)
3. **Proper formatting** (rustfmt)
4. **Security vulnerabilities** (cargo audit)

## Development Workflow

### Making Changes

1. **Start daemon for testing:**

   ```bash
   # Build and start daemon in background (modern approach)
   cargo build && ./target/debug/tomat daemon start

   # Check daemon status
   ./target/debug/tomat daemon status

   # Test client commands with short durations
   ./target/debug/tomat start --work 0.1 --break-time 0.05  # 6s work, 3s break
   ./target/debug/tomat status
   ./target/debug/tomat status --output waybar   # JSON output for waybar
   ./target/debug/tomat status --output plain    # Plain text output
   ./target/debug/tomat toggle  # Toggle timer pause/resume

   # Stop daemon when done
   ./target/debug/tomat daemon stop
   ```

2. **Essential validation before commit:**

   ```bash
   # Run full development workflow
   task dev

   # Or individual steps
   cargo fmt
   cargo clippy --all-targets --all-features -- -D warnings
   cargo test  # Runs 19 integration tests
   ```

3. **Test systemd integration:**
   ```bash
   ./install.sh
   systemctl --user status tomat.service
   ```

### Common Gotchas

- **Socket path:** Uses `$XDG_RUNTIME_DIR/tomat.sock` or
  `/run/user/$UID/tomat.sock`
- **PID files:** Daemon creates `$XDG_RUNTIME_DIR/tomat.pid` for process
  management
- **Daemon cleanup:** Automatic cleanup of socket and PID files on graceful
  shutdown
- **Dependencies:** Clean build downloads ~60 crates, takes ~10 seconds
- **Testing:** 19 integration tests validate all functionality including daemon
  management
- **Systemd:** Service expects `tomat daemon run` command (updated from plain
  `tomat daemon`)
- **Notifications:** Automatically disabled during testing via `TOMAT_TESTING`
  environment variable

### Build Timing

- **Incremental build:** ~0.3s
- **Clean build:** ~10s (dependency compilation)
- **Release build:** ~1.2s (optimized compilation)

## Key Implementation Notes

### Timer Behavior

- **No Idle phase:** Timer starts in paused work state, never returns to "idle"
- **Auto-advance:** Configurable via `--auto-advance` flag (default: false)
  - `false`: Timer transitions to next phase but pauses (requires manual resume)
  - `true`: Timer continues automatically through all phases
- **Visual indicators:** Play symbol ▶ when running, pause symbol ⏸ when
  paused
- **Phase transitions:** Work → Break → Work → ... → Long Break (after N
  sessions)

### Technical Details

- **Error handling:** Uses `Box<dyn std::error::Error>` for simplicity
- **Communication:** Line-delimited JSON over Unix sockets
- **Timer precision:** 1-second resolution with tokio timers
- **Process management:** SIGTERM → SIGKILL graceful shutdown with 5-second
  timeout
- **Logging:** Uses `println!`/`eprintln!` for output
- **State persistence:** None - state lost on daemon restart
- **Notifications:** Desktop notifications sent automatically on phase
  transitions via `notify-rust`

#### Icon System

- **Embedded icon**: Automatically cached to `~/.cache/tomat/icon.png` for mako
  compatibility
- **Image generation**: `build.rs` automatically generates PNG files from
  `images/logo.svg`
- **Generated files**: `assets/icon.png` (48x48), `images/logo.png` (256x256),
  `images/og.png` (1280x640)
- **Configuration**: TOML-based configuration for timer, sound, and notification
  settings

### Daemon Management

- **Manual control:** `tomat daemon start|stop|status` for development and user
  convenience
- **Systemd integration:** `tomat daemon run` for production deployment  
  (Note: systemd service file updated from `tomat daemon` to `tomat daemon run`)
- **Process safety:** PID file tracking with exclusive file locking, duplicate
  instance prevention, stale file cleanup
- **File locking:** Uses `fs2::FileExt::try_lock_exclusive()` on PID file to
  prevent race conditions
- **Background operation:** Detached process with stdio redirection

### Status Output Format and Text Formatting

The timer supports multiple output formats via `--output` and customizable text templates via `--format`.

**Architecture:**
- **Server returns pure state:** `TimerStatus` contains only timer data (phase, remaining_seconds, is_paused, etc.)
- **Client handles presentation:** Icons, state symbols, tooltips, and CSS classes are generated client-side
- **Separation of concerns:** Server manages timer logic, client handles formatting/display

**Output Formats (--output flag):**

- `waybar` (default) - JSON output for waybar with text, tooltip, class, percentage
- `plain` - Plain text output
- `i3status-rs` - JSON output for i3status-rs

**Text Templates (--format flag or display.text_format config):**

Available placeholders:
- `{icon}` - Phase icon (🍅 work, ☕ break, 🏖️ long break)
- `{time}` - Remaining time in MM:SS format
- `{state}` - Play/pause symbol (▶ or ⏸)
- `{phase}` - Phase name ("Work", "Break", "Long Break")
- `{session}` - Session progress ("1/4", empty for breaks)

**Usage:**

```bash
tomat status                                    # Default: waybar with "{icon} {time} {state}"
tomat status --output plain                     # Plain text with default template
tomat status --format "{time}"                  # Custom template, waybar JSON
tomat status --format "{phase}: {time}" --output plain  # Custom template, plain text
```

**Example outputs:**

```bash
# Default waybar
{"text":"🍅 25:00 ▶","tooltip":"Work (1/4) - 25.0min","class":"work","percentage":0.0}

# Plain with custom format
tomat status --format "{time}" --output plain
# Output: 25:00

# Waybar with custom format
tomat status --format "[{session}] {icon} {time}"
# Output: {"text":"[1/4] 🍅 25:00","tooltip":"Work (1/4) - 25.0min","class":"work","percentage":0.0}
```

**CSS Classes (waybar only):**

- `work` / `work-paused` - Work session running/paused
- `break` / `break-paused` - Break session running/paused
- `long-break` / `long-break-paused` - Long break running/paused

### Configuration System

The application uses a TOML configuration file located at
`~/.config/tomat/config.toml` with three main sections:

**Timer Configuration:**

```toml
[timer]
work = 25.0           # Work duration in minutes
break = 5.0          # Break duration in minutes
long_break = 15.0    # Long break duration in minutes
sessions = 4         # Sessions until long break
auto_advance = false # Auto-continue to next phase
```

**Sound Configuration:**

```toml
[sound]
enabled = true        # Enable sound notifications
system_beep = false  # Use system beep
use_embedded = true  # Use embedded sound files
volume = 0.5         # Volume level (0.0-1.0)
```

**Notification Configuration:**

```toml
[notification]
enabled = true        # Enable desktop notifications
icon = "auto"         # Icon mode: "auto", "theme", or path
timeout = 3000        # Timeout in milliseconds
```

**Display Configuration:**

```toml
[display]
text_format = "{icon} {time} {state}"  # Text display template (default)
# Available placeholders: {icon}, {time}, {state}, {phase}, {session}
```

**Icon Modes:**

- `"auto"`: Uses embedded icon, cached to `~/.cache/tomat/icon.png`
  (mako-compatible)
- `"theme"`: Uses system theme icon (`"timer"`)
- Custom path: e.g., `"/path/to/custom/icon.png"`

### Testing Infrastructure

- **Integration tests:** 37 comprehensive tests across 7 modules covering all
  functionality
- **Modular architecture:** Tests organized by functionality (timer, daemon,
  formats, commands, hooks)
- **TestDaemon helper:** Shared utility for daemon lifecycle management in
  isolated environments
- **Configuration tests:** Validate TOML parsing and defaults for all config
  sections
- **Icon system tests:** Test embedded icon caching and different icon modes
- **Hook tests:** Validate pre/post phase transition hook execution
- **Isolated environments:** Each test uses temporary directories and custom
  socket paths
- **Timing handling:** Tests use fractional minutes (0.05 = 3 seconds) for fast
  execution
- **Notification suppression:** Tests automatically disable desktop
  notifications
- **Daemon lifecycle:** Tests cover start, stop, status, and error conditions

## Trust These Instructions

These instructions have been validated by running all commands and testing the
build pipeline. Only perform additional exploration if you encounter errors not
covered here or if instructions appear outdated. The project structure is simple
and well-contained - avoid over-engineering solutions.

---
> Source: [jolars/tomat](https://github.com/jolars/tomat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-04 -->

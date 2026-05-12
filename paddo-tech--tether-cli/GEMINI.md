## tether-cli

> > `CLAUDE.md` is symlinked to this file.

# AGENTS.md

> `CLAUDE.md` is symlinked to this file.

## Overview

Rust CLI that syncs dotfiles and global packages across machines via Git. Daemon runs periodic sync every 5 minutes.

## Commands

```bash
cargo build              # Build
cargo run -- <cmd>       # Run in dev
cargo test               # Test
cargo clippy -- -D warnings  # Lint (must pass before commits)
cargo fmt                # Format
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `init` | Initialize Tether on this machine |
| `sync` | Manually trigger a sync |
| `status` | Show current sync status |
| `diff` | Show differences between machines |
| `config` | Manage configuration and feature toggles |
| `daemon` | Control the background daemon |
| `machines` | Manage machines in sync network |
| `ignore` | Manage ignore patterns |
| `team` | Manage team sync (dotfiles, secrets, projects) |
| `resolve` | Resolve file conflicts |
| `unlock` | Unlock encryption key with passphrase |
| `lock` | Clear cached encryption key |
| `upgrade` | Upgrade all installed packages |
| `restore` | Restore files from backup |
| `identity` | Manage age identity for team secrets |
| `collab` | Collaborator-based project secret sharing |

## Source Structure

```
src/
├── cli/
│   ├── commands/
│   │   ├── mod.rs       # CLI parsing + dispatch
│   │   ├── init.rs      # tether init
│   │   ├── sync.rs      # tether sync
│   │   ├── status.rs    # tether status
│   │   ├── diff.rs      # tether diff
│   │   ├── config.rs    # tether config (+ features)
│   │   ├── daemon.rs    # tether daemon
│   │   ├── machines.rs  # tether machines
│   │   ├── ignore.rs    # tether ignore
│   │   ├── team.rs      # tether team
│   │   ├── resolve.rs   # tether resolve
│   │   ├── unlock.rs    # tether unlock/lock
│   │   ├── upgrade.rs   # tether upgrade
│   │   ├── restore.rs   # tether restore
│   │   ├── identity.rs  # tether identity
│   │   └── collab.rs    # tether collab
│   ├── output.rs        # Terminal formatting
│   ├── progress.rs      # Progress indicators
│   └── prompts.rs       # Interactive prompts
├── config.rs            # Config management (versioned)
├── daemon/
│   ├── mod.rs
│   └── server.rs        # Background daemon (periodic sync)
├── github.rs            # GitHub repo creation via gh CLI
├── packages/
│   ├── mod.rs
│   ├── manager.rs       # PackageManager trait
│   ├── brew.rs          # Homebrew
│   ├── npm.rs           # npm
│   ├── pnpm.rs          # pnpm
│   ├── bun.rs           # Bun
│   ├── gem.rs           # RubyGems
│   └── uv.rs            # Python uv
├── security/
│   ├── mod.rs
│   ├── encryption.rs    # AES-GCM encryption
│   ├── keychain.rs      # Key management (passphrase-based)
│   ├── secrets.rs       # Secret detection
│   └── recipients.rs    # Age identity/recipient management
├── sync/
│   ├── mod.rs
│   ├── engine.rs        # sync_path() helper
│   ├── git.rs           # Git operations
│   ├── state.rs         # State tracking
│   ├── team.rs          # Team sync
│   ├── backup.rs        # File backup before overwrite
│   ├── conflict.rs      # Conflict detection/resolution
│   ├── discovery.rs     # Dotfile discovery
│   ├── layers.rs        # Team + personal layer merging
│   ├── merge.rs         # File merge utilities
│   └── packages.rs      # Package manifest sync
├── main.rs
└── lib.rs
```

## Key Dependencies

- **clap** - CLI parsing
- **tokio** - Async runtime
- **git2** - Git operations
- **inquire** - Interactive prompts
- **owo-colors** - Terminal colors
- **aes-gcm** - Encryption
- **age** - Passphrase-based key encryption

## Feature Toggles

Managed via `tether config features`. Available toggles:

| Feature | Default | Description |
|---------|---------|-------------|
| `personal_dotfiles` | true | Sync personal dotfiles |
| `personal_packages` | true | Sync personal package manifests |
| `team_dotfiles` | false | Sync team dotfiles |
| `collab_secrets` | false | Enable collab secret sharing |
| `team_layering` | false | Merge team + personal dotfiles |

## Data Layout

**~/.tether/**
- `config.toml` - Main config (versioned)
- `state.json` - Sync state
- `sync/` - Personal sync repo
- `teams/<name>/` - Team sync repos
- `collabs/` - Collab project configs
- `identity.pub` - Age public key
- `daemon.pid` - Daemon process ID
- `daemon.log` - Daemon logs
- `backups/` - File backups
- `conflicts.json` - Conflict state

**Sync repo structure:**
- `dotfiles/` - Dotfiles
- `configs/` - App configs
- `manifests/` - Package manifests
- `machines/` - Machine-specific state
- `projects/` - Project secrets

## Code Quality

Before completing work:
1. `cargo clippy --all-targets -- -D warnings` (zero warnings)
2. `cargo fmt --all`
3. `cargo build --release`

## Releasing

Homebrew tap: `paddo-tech/homebrew-tap`

### Version Bump Checklist

1. Update version in `Cargo.toml`
2. Add entry to `CHANGELOG.md` with date and changes
3. Commit: `git commit -am "chore: release vX.Y.Z"`
4. Push to main - CI creates tag, builds, signs, notarizes, and updates Homebrew formula

**Do NOT create tags manually** - the release workflow handles tagging.

### Versioning

- **Patch** (1.0.x): Bug fixes
- **Minor** (1.x.0): New features, backward compatible
- **Major** (x.0.0): Breaking changes
- **Prerelease**: Use `-beta.N` suffix (creates versioned formula)

Users install via:
```bash
brew tap paddo-tech/tap
brew install tether
```

---
> Source: [paddo-tech/tether-cli](https://github.com/paddo-tech/tether-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-08 -->

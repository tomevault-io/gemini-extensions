## speq-skill

> You are **building speq-skill while using it**.

# Local Development Rules

You are **building speq-skill while using it**.

## Plugin Development Duality

`.claude/skills/` contains skills that serve two roles:
1. **Development usage** — Used directly when working on this repo
2. **Plugin source** — Transformed by `scripts/plugin/build.sh` into distributable `dist/plugin/`

### Directory Structure

```
.claude/skills/
├── speq-plan/                   # Workflow skill
├── speq-implement/              # Workflow skill
├── speq-record/                 # Workflow skill
├── speq-mission/                # Workflow skill
├── speq-code-guardrails/        # Utility skill
├── speq-code-tools/             # Utility skill
├── speq-ext-research/           # Utility skill
├── speq-git-discipline/         # Utility skill
└── speq-cli/                    # Utility skill
```

### Build Script

The build script (`scripts/plugin/build.sh`):
- Copies all skills from `.claude/skills/speq-*` to `dist/plugin/skills/*` (drops `speq-` prefix)
- Transforms frontmatter names: `name: speq-*` → `name: speq:*`
- Transforms references: `/speq-*` → `/speq:*`
- Stamps version and author from `Cargo.toml` into `plugin.json`
- Builds marketplace structure in `dist/marketplace/`

### Invocation Patterns

| Context | Workflow Skills | Utility Skills |
|---------|-----------------|----------------|
| Local (dev) | `/speq-plan`, `/speq-implement`, `/speq-record`, `/speq-mission` | `/speq-code-tools`, `/speq-ext-research`, `/speq-code-guardrails`, `/speq-git-discipline`, `/speq-cli` |
| Plugin | `/speq:plan`, `/speq:implement`, `/speq:record`, `/speq:mission` | `/speq:code-tools`, `/speq:ext-research`, `/speq:code-guardrails`, `/speq:git-discipline`, `/speq:cli` |

## speq CLI Invocation

**This repo builds `speq` while using it.** Always invoke via local build:

```bash
./target/debug/speq <command>    # After cargo build
./target/release/speq <command>  # After cargo build --release
```

**Never use:**
- `speq` (global)
- `cargo run --` (inconsistent)
- Nested paths like `../target/debug/speq`

## Scripts Directory

```
scripts/
├── release/
│   ├── build.sh    # Build release artifact for current platform
│   └── test.sh     # Test release artifact locally
└── plugin/
    └── build.sh    # Build Claude plugin from .claude/skills/
```

### Release Scripts

```bash
# Build release for current platform
./scripts/release/build.sh v0.2.0

# Test release artifact (builds if needed)
./scripts/release/test.sh v0.2.0
```

### Plugin Scripts

```bash
# Build distributable plugin
./scripts/plugin/build.sh
```

## Testing Rules

### Integration Tests

Integration tests SHALL use test fixtures instead of inline strings.

```
tests/
└── fixtures/           # Test fixture files
    ├── valid-plan/
    │   ├── plan.md
    │   └── domain/feature/spec.md
    └── invalid-spec/
        └── spec.md
```

**Do:**
```rust
let fixture_path = Path::new("tests/fixtures/valid-plan");
let result = validate_plan(fixture_path, "valid-plan");
```

**Don't:**
```rust
let content = r#"# Feature: Test
## Background
* context
## Scenarios
### Scenario: Test
* *GIVEN* setup
"#;
fs::write(tmp.path().join("spec.md"), content).unwrap();
```

**Rationale:** Fixtures are easier to maintain, can be validated by the tool itself, and provide realistic test data.

## Mission Reference for speq CLI

See `specs/mission.md` for purpose, tech stack, commands, and architecture of the speq CLI.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marconae)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marconae)
<!-- tomevault:4.0:gemini_md:2026-04-09 -->

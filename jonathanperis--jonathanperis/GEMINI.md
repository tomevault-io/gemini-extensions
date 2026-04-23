## jonathanperis

> GitHub profile README repository. Terminal-themed developer portfolio showcasing microservices, game dev, and polyglot performance engineering.

# jonathanperis — Claude Code Guide

GitHub profile README repository. Terminal-themed developer portfolio showcasing microservices, game dev, and polyglot performance engineering.

---

## Purpose

This is a **special GitHub repository** — its README.md is rendered as the profile page at `github.com/jonathanperis`.

---

## Structure

```
jonathanperis/
├── README.md                   # Profile README (169 lines, terminal aesthetic)
├── LICENSE                     # MIT License
├── .editorconfig               # UTF-8, LF, 4-space indent
├── .gitignore                  # .DS_Store only
├── .github/
│   ├── dependabot.yml          # Weekly GitHub Actions updates
│   └── workflows/
│       └── snake.yml           # Daily snake animation generation
└── jonathanperis.wiki/         # GitHub Wiki (separate git repo)
    ├── Home.md                 # Wiki index
    ├── About.md                # Extended bio
    ├── cpnucleo.md             # Project deep dive
    ├── rinha-de-backend.md     # Challenge documentation
    ├── super-mango-editor.md     # Game dev docs
    ├── portfolio-site.md       # Portfolio site docs
    ├── blazor-mudblazor-starter.md
    ├── tech-stack.md           # Complete tech inventory
    ├── architecture-patterns.md
    └── performance-engineering.md
```

---

## README Sections

| Section | Command Header | Content |
|---------|---------------|---------|
| Profile | `$ whoami` | Developer bio + languages |
| Projects | `$ ls ~/projects --sort=stars` | 10 projects table with stars |
| Tech Stack | `$ cat /proc/tech_stack` | ASCII table of technologies |
| Benchmarks | `$ cat benchmarks.md` | Rinha implementations comparison |
| System Info | `$ neofetch` | Contact info, system emulation |
| Contribution Graph | — | Animated snake SVG (light/dark) |

---

## Automated Workflows

- **snake.yml**: Runs daily (`0 0 * * *`) via `Platane/snk` action to generate contribution graph snake animation SVGs. Pushes to `output` branch.

---

## Content Guidelines

- Terminal aesthetic: section headers use shell commands (`$ whoami`, `$ ls`, `$ cat`)
- Code-first design: tech stack as ASCII art, profile as C# struct
- Polyglot philosophy: always showcase multiple languages
- Keep project table updated when adding/removing repos
- Wiki documents each project's architecture and decisions

---

## Git Workflow

- **All changes must go through a branch + PR strategy** — never commit directly to `main`
- **PRs are rebase-only** — always use `--rebase` merge strategy
- **Use `gh` CLI** for all GitHub operations (PRs, releases, checks, etc.)
- Branch naming: `feat/`, `fix/`, `chore/`, `docs/` prefixes

---

## Repo-Wide Files

The `.github` organization repo (`jonathanperis/.github`) already provides:

- `CODE_OF_CONDUCT.md`
- `CONTRIBUTING.md`
- `SECURITY.md`
- `FUNDING.yml`

**Do not recreate these files in this repo.** They are inherited from the org-level `.github` repo.

---

## README Badges & Links

The README must include these four badges (top of file, before the quote block):

```markdown
[![Main Release](https://github.com/jonathanperis/jonathanperis/actions/workflows/main-release.yml/badge.svg)](...)
[![CodeQL](https://github.com/jonathanperis/jonathanperis/actions/workflows/codeql.yml/badge.svg)](...)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
```

The README must include these bold links (before the snake animation):

```markdown
**[Live demo →](https://jonathanperis.github.io/)** | **[Documentation →](https://github.com/jonathanperis/jonathanperis/wiki)**
```

---

## Workflows

| Workflow | File | Trigger | Purpose |
|----------|------|---------|---------|
| Main Release | `main-release.yml` | push to main + manual | Markdown lint (badge source) |
| CodeQL | `codeql.yml` | push to main + manual + weekly | Security analysis of Actions |
| Snake | `snake.yml` | daily + manual | Contribution graph animation |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonathanperis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-10 -->

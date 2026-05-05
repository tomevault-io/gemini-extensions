## skill-structure

> Skill folder layout, SKILL.md frontmatter, and naming conventions


Every skill under `skills/uipath-<kebab-case>/` must have a `SKILL.md` with valid YAML frontmatter: `name` (matching folder name exactly) and `description` (under 1024 characters — repo cap; Claude Code's hard truncation for `description` + `when_to_use` is 1,536 chars).

The `description` must front-load skill identity and unique file/domain signals (e.g. `.cs`, `.xaml`, `.flow`, `servo`) within the first ~100 characters. Use compact `→` redirects for commonly confused sibling skills (e.g. `For XAML→uipath-rpa`). Start with `[PREVIEW]` for unstable skills. Do NOT use verbose `TRIGGER when:` / `DO NOT TRIGGER when:` clauses or nest fields under `metadata:` — both cause Claude Code to drop the field.

SKILL.md body order: Title → When to Use → Critical Rules (numbered) → Workflow/Quick Start → Reference Navigation → Anti-patterns. Reference files use kebab-case; guides end with `-guide.md`; templates end with `-template.md` or `-template.<ext>`. Skills must be self-contained — no cross-skill references.

Full reference: `.claude/rules/skill-structure.md`.

---
> Source: [UiPath/skills](https://github.com/UiPath/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-28 -->

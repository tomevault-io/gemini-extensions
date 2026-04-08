## ai-app-platform

> Work PR-native with small, test-first changes (TDD).

# Repo Copilot Instructions

Work PR-native with small, test-first changes (TDD).
Use contract-first APIs (OpenAPI), Conventional Commits, and atomic PRs.
Attach CI evidence on every PR: coverage, SARIF (no high/critical), SBOM (CycloneDX), SLSA provenance, Playwright report/traces.
Wire OTel traces; link trace IDs in PR summary.
Use ReAct/Reflexion; avoid placeholders/mocks; production-grade only.

You must:

- Work in small branches and open DRAFT PRs with a checklist tied to acceptance criteria.
- Keep CI green; if you break a check, fix it before requesting review.
- Never add secrets to code or CI. Use env placeholders and document them.
- Prefer minimal, auditable changes. Explain root cause in `docs/*`.
- When a task mentions “evidence”, attach links to logs, artifacts, or screenshots.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Baragji)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Baragji)
<!-- tomevault:4.0:gemini_md:2026-04-07 -->

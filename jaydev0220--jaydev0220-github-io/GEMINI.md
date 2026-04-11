## jaydev0220-github-io

> - **Project Name:** personal-website

## Project Overview

- **Project Name:** personal-website
- **Objective:** Digital business card and simple resume.
- **Architecture:** Svelte static site.

## Technical Stack and Environment

- **Language:** TypeScript 5.9
- **Frameworks & Libraries:** Svelte v5, SvelteKit v2, Tailwind CSS v4
- **Environment:** GitHub Pages

## Directory Structure

```text
personal-website/
├── e2e/                  # Playwright test files
├── src/
│   ├── lib/
│   │   ├── components/
│   │   │   ├── AboutSection.svelte
│   │   │   ├── CertificateModal.svelte
│   │   │   ├── CertificateSection.svelte
│   │   │   ├── Footer.svelte
│   │   │   ├── Hero.svelte
│   │   │   ├── Navigation.svelte
│   │   │   ├── PortfolioSection.svelte
│   │   │   └── SkillsSection.svelte
│   │   ├── data.ts
│   │   ├── iconMap.ts
│   │   └── utils.ts
│   ├── routes/
│   │   ├── [...404]/
│   │   │   └── +page.svelte
│   │   ├── credits/
│   │   │   └── +page.svelte
│   │   ├── +page.svelte
│   │   ├── +layout.svelte
│   │   ├── +layout.ts
│   │   └── layout.css
│   ├── app.d.ts
│   └── app.html
├── static/
│   ├── robot.txt
│   └── .nojekyll
├── package.json
├── svelte.config.js
├── vite.config.ts
├── playwright.config.ts
├── tsconfig.json
├── eslint.config.js
├── .prettierrc
├── .prettierignore
├── .npmrc
├── .gitignore
├── README.md
└── PLAN.md               # AI Agent planning document
```

## Coding Standards and Style

- **Naming Conventions:** camelCase
- **Formatting & Linting:** Prettier, ESLint
- **Documentation:** All comments must be written in English.

## Development and Testing Workflow

- **Testing Framework:** Playwright
- **Testing Scope:** UI interactions and standard user flows.
- **Local Development:** `pnpm dev`
- **Version Control:** Strict adherence to Conventional Commits specification.

## Error Handling and Logging

- **Exception Handling:** Display UI pop-up messages for errors. Silent errors are prohibited.
- **Logging:** Use native browser console APIs (`console.log`, `console.info`, `console.warn`, `console.error`).

## Agent-Specific Directives

- **Task Execution:** Divide development into distinct phases. Commit code using git commands upon completion of each phase.
- **Pre-commit Requirement:** Execute `pnpm format && pnpm lint` before every commit to ensure code quality and consistency.
- **Restricted Operations:** Strict prohibition on modifying or removing any files outside of the project directory and session directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaydev0220)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jaydev0220)
<!-- tomevault:4.0:gemini_md:2026-04-08 -->

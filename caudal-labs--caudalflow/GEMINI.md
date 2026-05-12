## caudalflow

> Visual canvas for AI conversations — branch, explore in parallel, merge insights.

# CaudalFlow

Visual canvas for AI conversations — branch, explore in parallel, merge insights.

## Build & Run

```bash
npm run dev        # Dev server at http://localhost:5173
npm run build      # Type-check + production build (tsc -b && vite build)
npm run lint       # ESLint
npm test           # Vitest — run all unit tests
npm run test:watch # Vitest in watch mode
```

## Tech Stack

- React 19, TypeScript 5.9, Vite 8
- @xyflow/react 12 for the canvas (nodes, edges, selection, viewport)
- Zustand 5 for state (flowStore, chatStore, settingsStore, workspaceStore)
- Tailwind CSS 4 for styling
- Vitest for testing

## Project Structure

```
src/
├── components/
│   ├── canvas/        # Canvas, CanvasControls, MergeSelectionPopup
│   ├── nodes/         # ChatNode, ChatMessage, ChatInput, SelectionPopup
│   ├── edges/         # TopicEdge (custom edge renderer)
│   └── ui/            # SettingsPanel, HelpGuide, WorkspaceSelector
├── hooks/             # useChatNode (core chat logic), usePersistence
├── stores/            # flowStore, chatStore, settingsStore, workspaceStore
├── services/
│   ├── llm.ts         # Streaming orchestrator
│   └── providers/     # LLM providers: mock, openai, anthropic
├── types/             # chat.ts, flow.ts, workspace.ts
└── utils/             # systemPrompts.ts, nodeLayout.ts
```

## Architecture

### State Management — 4 Zustand stores

- **flowStore** — nodes, edges, graph mutations (addChatNode, removeNode, addEdge)
- **chatStore** — messages per node, streaming state
- **settingsStore** — LLM config, UI preferences (persisted to localStorage)
- **workspaceStore** — multi-workspace management

### LLM Provider System

Every provider implements `LLMProvider` interface (`services/providers/types.ts`):
- `id`, `name`, `streamChat(messages, config, callbacks, signal)`
- Registered in `services/providers/registry.ts`
- Providers use raw `fetch` with SSE streaming — no SDKs

### Key Patterns

- Branching: select text in a conversation, creates child node with parent context
- Merging: Shift+drag to select 2+ nodes, creates merge node synthesizing all parent contexts
- System prompts built in `utils/systemPrompts.ts` (root, branch, merge)
- Node positioning calculated in `utils/nodeLayout.ts`
- All workspace data persisted to localStorage with debounced auto-save

## Code Style

- Functional components only, no class components
- Explicit TypeScript types for function params and return values
- `interface` over `type` for object shapes
- One component per file, named after the component
- Hooks in `hooks/`, stores in `stores/`, types in `types/`, utilities in `utils/`
- Tailwind classes using `surface-*`, `neutral-*`, `accent-*` color tokens
- Keep components under ~200 lines — split when they grow

## Testing

- Vitest with `globals: true`, `environment: 'node'`
- Test files in `__tests__/` directories next to the module: `<module>.test.ts`
- Reset Zustand store state in `beforeEach` to avoid test interference
- Focus on behavior (inputs/outputs), not implementation details
- All stores, utils, and services should have unit tests
- Bug fixes should include a regression test

## Adding a New LLM Provider

4 files, zero changes to branching/merging/streaming logic:

1. `src/services/providers/yourprovider.ts` — implement `LLMProvider`
2. `src/services/providers/registry.ts` — register it
3. `src/components/ui/SettingsPanel.tsx` — add config UI section
4. `src/stores/settingsStore.ts` — add default endpoint/model

## Git Conventions

- Branch naming: `feature/short-description`, `fix/issue-short-description`, `docs/what-changed`
- Concise commit messages focused on "why" not "what"
- PRs must pass: `npm run build`, `npm run lint`, `npm test`

## Releasing

Tag-based release flow — CI handles everything:

```bash
npm version patch   # or minor / major — bumps package.json + creates git tag
git push origin main --tags   # triggers release workflow → npm publish + GitHub Release
```

**First publish (manual, one-time):**

```bash
npm run build
npm login
npm publish --access public
```

**Then configure Trusted Publishing:** go to `https://www.npmjs.com/package/caudalflow/access` → Trusted Publishers → GitHub Actions → set org: `caudal-labs`, repo: `caudalflow`, workflow: `release.yml`.

All subsequent releases use OIDC — no tokens, no secrets. The release workflow runs lint + test + build before publishing with `--provenance` (links the package to the exact commit and Actions run).

---
> Source: [caudal-labs/caudalflow](https://github.com/caudal-labs/caudalflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-04 -->

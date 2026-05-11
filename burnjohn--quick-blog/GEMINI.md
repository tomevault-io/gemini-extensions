## react-frontend

> React Frontend Development Conventions


# React Frontend Rules

## Code Organization

- Move ALL helper functions out of components to `utils/` or top of file
- Extract constants to a separate constants file when there are 3+
- Keep helper files colocated with components when specific to one feature
- One component per file; max 200 lines per component — split if larger
- File order: imports → constants → helpers → component → exports

## Component Architecture

- **Container components**: fetch data via hooks, pass props down, handle loading/error/empty states
- **Presentational components**: receive props, render UI, no data fetching
- Compose small, reusable components over large monolithic ones
- Components must be pure — same inputs = same outputs
- NEVER mutate props or state — treat as immutable
- Side effects in `useEffect` only, never during render
- Use early returns for loading and error states

## State Management

- PREFER local state (`useState`) over global state
- PREFER props over Context — prop drilling is clearer for small trees
- Use Context ONLY for truly global concerns (auth in `AppContext`)
- Colocate state with the components that use it
- URL-dependent state (filters, search, pagination) belongs in URL search params

## Hooks Rules

- Hooks at top level only — never in loops, conditions, or nested functions
- Custom hooks start with `use`; data hooks live in `hooks/api/`
- ALL data fetching in custom hooks (`hooks/api/queries/`, `hooks/api/mutations/`)
- Use the core `useApiQuery` / `useApiMutation` patterns from `hooks/core/`
- `useEffect` must synchronize with an external system — if not, it's misused
- NEVER use `useEffect` for derived state — compute during render
- NEVER chain `useEffect`s that trigger each other

## Derive, Don't Store

The #1 React anti-pattern to avoid:
- NEVER store derived values in `useState` — compute during render
- NEVER use `useState` + `useEffect` to sync a computed value
- If a value can be calculated from existing props/state, calculate it inline

## Tailwind CSS Conventions

- Use Tailwind utility classes for ALL styling — no inline `style={}` objects
- Use responsive prefixes (`sm:`, `md:`, `lg:`) for responsive design
- Use dark mode variant (`dark:`) when supporting themes
- Extract repeated utility combinations into component abstractions, not CSS files
- Use the project's `ui/` components (Button, Card, Input, Badge, etc.) before creating new ones

## API Layer

- Centralized in `api/` directory using Axios with `axiosConfig.js`
- API endpoint paths defined in `constants/apiEndpoints.js`
- User-facing messages in `constants/messages.js`
- Route paths in `constants/routes.js`

## Routing

- React Router v7 with pages under `pages/admin/` and `pages/public/`
- Use barrel exports via `index.js` in each directory

## Error Handling

- Handle loading, error, and empty states in container components
- Use `react-hot-toast` for user notifications
- Try-catch in async functions within hooks

---
> Source: [burnjohn/quick-blog](https://github.com/burnjohn/quick-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-07 -->

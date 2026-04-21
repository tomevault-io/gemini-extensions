## anygpt

> - ✅ Always `npx nx` (never global)


# Nx Rules

## Execution
- ✅ Always `npx nx` (never global)
- ✅ `npx nx build`, `npx nx run-many -t build`, `npx nx affected -t test`

## Custom Plugins
- **Required**: `nx-tsdown`, `nx-vitest`, `nx-tsgo`, `@nx/eslint/plugin`
- ✅ Plugins infer targets automatically from config files
- ✅ Configured centrally in `nx.json` plugins array

## Package Configuration
- ❌ **No npm scripts** in package.json
- ✅ Use Nx targets: `npx nx build my-package`
- ✅ Let plugins create targets automatically

## Target Configuration
- ✅ Define in `nx.json` `targetDefaults` (central)
- ❌ No `project.json` unless unique requirements
- ✅ Dependencies: `"dependsOn": ["^build"]` (build deps first)

## Incremental Builds
- ✅ **Dependencies build automatically** via `^build` in targetDefaults
- ❌ **Never manually build dependencies** - Nx handles it
- ✅ Just run target on final package: `npx nx build my-package`
- ✅ Nx builds dependency tree automatically with caching

```bash
# ❌ Bad - Unnecessary manual dependency builds
npx nx build @anygpt/config
npx nx build @anygpt/cli

# ✅ Good - Nx builds dependencies automatically
npx nx build @anygpt/cli  # Builds @anygpt/config first if needed
```

## ESLint
- ✅ Include `@nx/eslint-plugin` configs
- ✅ Enable `@nx/enforce-module-boundaries` rule
- ✅ Flat config format (ESLint 9+)

```javascript
import nx from '@nx/eslint-plugin';
export default [
  ...nx.configs['flat/base'],
  ...nx.configs['flat/typescript'],
  { files: ['**/*.ts'], rules: { '@nx/enforce-module-boundaries': 'error' } }
];
```

## Release
- ✅ Use custom `./tools/nx-release:release` executor
- ❌ Never standard `nx release`
- ✅ Independent versioning, conventional commits, AI changelogs

## Task Execution
```bash
npx nx build @anygpt/cli              # Single (deps auto-built)
npx nx run-many -t build test lint    # Multiple
npx nx affected -t test               # Affected only
npx nx graph --json                   # Graph as JSON
```

## Caching
- ✅ Local + Nx Cloud distributed caching
- ✅ Cached builds skip execution entirely
- ✅ `npx nx reset` to clear cache
- ✅ Use `affected` in CI for efficiency

## Checklist
- [ ] Using `npx nx` (not global)
- [ ] No npm scripts in packages
- [ ] No devDependencies in packages (use workspace root)
- [ ] No manual dependency builds (Nx does it)
- [ ] Plugins in nx.json, no project.json
- [ ] ESLint with Nx rules (`@nx/enforce-module-boundaries`, `@nx/dependency-checks`)
- [ ] Custom nx-release executor
- [ ] targetDefaults for shared config
## Dependencies
- ❌ **No devDependencies in packages** - manage at workspace root
- ✅ Only runtime `dependencies` in package.json if needed
- ✅ Dev tools (tsdown, vitest, typescript) in root package.json
- 🎯 **Benefits**: Single version across packages, smaller package.json

```json
// ❌ Bad - package has devDependencies
{
  "name": "@anygpt/my-package",
  "devDependencies": {
    "tsdown": "^0.2.15",
    "vitest": "^2.1.8"
  }
}

// ✅ Good - clean package.json
{
  "name": "@anygpt/my-package",
  "dependencies": {
    "@anygpt/other-package": "1.0.0"  // Only if needed at runtime
  }
}
```

## Dependency Checks (ESLint)
- ✅ **`@nx/enforce-module-boundaries`** - Required in root eslint.config
- ✅ **`@nx/dependency-checks`** - Validates package.json dependencies
- ✅ Configured at workspace root, applies to all packages
- ✅ Prevents circular dependencies and unused deps

```javascript
// Root eslint.config.mjs
export default [
  ...nx.configs['flat/base'],
  ...nx.configs['flat/typescript'],
  {
    files: ['**/*.ts', '**/*.tsx', '**/*.js', '**/*.jsx'],
    rules: {
      '@nx/enforce-module-boundaries': [
        'error',
        {
          enforceBuildableLibDependency: true,
          depConstraints: [
            { sourceTag: '*', onlyDependOnLibsWithTags: ['*'] }
          ]
        }
      ]
    }
  },
  {
    files: ['**/*.json'],
    rules: {
      '@nx/dependency-checks': ['error', {
        ignoredFiles: [
          '{projectRoot}/**/*.config.{js,ts}',
          '{projectRoot}/**/*.{spec,test}.{js,ts}'
        ]
      }]
    }
  }
];
```

## Testing
```bash
# ❌ Bad - Interactive watch mode
npx nx test my-package

# ✅ Good - Non-interactive, exits on completion
npx nx test my-package --run --reporter=verbose
npx nx test my-package -- --run --reporter=json

# ✅ Run multiple packages
npx nx run-many -t test --all --run

# ✅ Test affected only
npx nx affected -t test --run

# ✅ With coverage
npx nx test my-package --coverage --run
```

### Test File Structure
- **Location**: `test/` directory (NOT `src/`)
- **Fixtures**: `test/fixtures/` for JSON/mock data
- **Config**: `vitest.config.ts` in package root
- **Pattern**: `test/**/*.test.ts` or `test/**/*.spec.ts`

### Vitest Configuration
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['test/**/*.{test,spec}.ts'],  // NOT src/**
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['**/*.test.ts', '**/*.spec.ts', 'test/**'],
    },
  },
});
```

### Key Points
- ✅ Tests in `test/` directory (excluded from build)
- ✅ Fixtures in `test/fixtures/` (JSON format)
- ✅ Always use `--run` flag for non-interactive execution
- ✅ Use `--reporter=verbose` or `--reporter=json` for output
- ❌ Never put test files in `src/` (they'll be bundled)
- ❌ Never run tests without `--run` flag (opens watch mode)

## Workspace Dependencies
- ✅ **Use npm workspaces** - NOT pnpm workspaces
- ❌ **Never use `workspace:*`** - This is pnpm syntax
- ✅ Use exact versions or version ranges in dependencies
- ✅ Nx handles the monorepo, npm handles package linking

```json
// ❌ Bad - pnpm workspace protocol
{
  "dependencies": {
    "@anygpt/types": "workspace:*"
  }
}

// ✅ Good - npm workspace (let npm link it)
{
  "dependencies": {
    "@anygpt/types": "1.2.0"
  }
}

// ✅ Also good - version range
{
  "dependencies": {
    "@anygpt/types": "^1.0.0"
  }
}
```

**Why**: This workspace uses npm, not pnpm. The `workspace:*` protocol is pnpm-specific and won't work with npm workspaces.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/genai-tools) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-13 -->

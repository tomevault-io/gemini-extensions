## html2pptx-pro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 1. Project Overview

html2pptx-pro is a library that converts HTML elements to PowerPoint presentations using the pptxgenjs library.

- **Type**: ESM module (`"type": "module"`)
- **Node Version**: >=16.0.0 (build tooling only; library is browser-only at runtime)
- **Purpose**: Convert HTML elements to PowerPoint slides while preserving CSS styling

## 2. Build & Dev Commands

| Command                  | Description                                       |
| ------------------------ | ------------------------------------------------- |
| `npm run build`          | Full build (tsc → rollup → reftest list → uglify) |
| `npm run watch`          | Dev mode (rollup -w)                              |
| `npm run lint`           | ESLint check                                      |
| `npm run format`         | Prettier format                                   |
| `npm run unittest`       | Jest unit tests                                   |
| `npm run test`           | lint + unittest + karma (full suite)              |
| `npm run karma`          | Browser regression tests                          |
| `npm run start`          | Test server (port 8080)                           |
| `npm run watch:unittest` | Watch mode unit tests                             |
| `npm run docs:dev`       | VitePress docs dev server                         |
| `npm run docs:build`     | Build docs                                        |
| `npm run release`        | Version release (standard-version)                |

## 3. Code Style & Conventions

### Formatting

- **Indent**: 4 spaces (2 spaces for yml/package.json)
- **Quotes**: Single quotes
- **Trailing commas**: Disabled
- **Line width**: 120 characters
- **Semicolons**: Enabled

### ESLint Rules

- **No console.log**: Allowed only warn/error
- **Class members**: Use `no-public` (don't write `public`)

### Commits

- **Format**: Conventional Commits
- **Types**: feat/fix/docs/style/refactor/perf/test/chore/revert/build/ci

### Automation

- **Pre-commit**: husky + lint-staged (auto prettier + eslint)

## 4. Architecture

### 4.1 Core Flow (10 Steps)

1. **Input validation** (Validator) — SSRF/XSS protection
2. **Performance monitoring init** (PerformanceMonitor)
3. **DOM cloning** (DocumentCloner → iframe isolation)
4. **DOM normalization** (DOMNormalizer — disable animations, reset transforms with identity values)
5. **DOM parsing** (node-parser → ElementContainer tree)
6. **CSS parsing** (CSSParsedDeclaration — 50+ property descriptors)
7. **Stacking context construction** (parseStackingContexts)
8. **PPTX rendering** (PptxRenderer → background/border/text/image)
9. **DOM restoration** (restoreTree — restore modified styles)
10. **Cleanup** (destroy cloned iframe)

### 4.2 Key Files (Grouped by Function)

#### Entry & Config

- `src/index.ts` — Main entry, `html2pptx()` function
- `src/config.ts` — `PptxConfig` configuration class
- `src/global.d.ts` — Global type declarations (CSSStyleDeclaration extensions)

#### Core Infrastructure (`src/core/`)

- `context.ts` — Runtime context (Logger + Cache + OriginChecker)
- `validator.ts` — Input validation (SSRF/XSS/injection protection, ~594 lines)
- `performance-monitor.ts` — Pipeline stage performance tracking
- `cache-storage.ts` — Resource cache with LRU eviction
- `logger.ts` — Logging system
- `origin-checker.ts` — Same-origin checking

#### DOM Processing (`src/dom/`)

- `document-cloner.ts` — Clone DOM into iframe for isolated processing
- `node-parser.ts` — Parse DOM tree into ElementContainer tree
- `element-container.ts` — Core data structure (element + styles + bounds)
- `dom-normalizer.ts` — Pre-render DOM normalization
- `text-container.ts` — Text node container

#### Vendored Libraries (`src/lib/`)

- `unicode.ts` — Shared `toCodePoints` / `fromCodePoint` utilities
- `line-break.ts` — CSS line breaking (UAX #14), provides `LineBreaker`
- `line-break-trie.ts` — Base64-encoded Unicode line-break trie data
- `grapheme-break.ts` — Grapheme cluster breaking (UAX #29), provides `splitGraphemes`
- `grapheme-break-trie.ts` — Base64-encoded Unicode grapheme-break trie data
- `utrie.ts` — Unicode Trie data structure (`createTrieFromBase64`, `Trie` class)
- `index.ts` — Re-exports: `toCodePoints`, `fromCodePoint`, `LineBreaker`, `splitGraphemes`

#### CSS Parsing (`src/css/`)

- `index.ts` — CSS declaration parser entry
- `property-descriptors/` — 50+ CSS property parsers
- `syntax/` — CSS tokenizer and parser
- `types/` — CSS value types (color, length, gradient, etc.)
- `layout/` — Bounds and text layout calculation

#### Rendering (`src/render/`)

- `stacking-context.ts` — CSS stacking context (z-index ordering)
- `renderer.ts` — Base renderer class
- `renderer-interface.ts` — Renderer interface definition

#### PPTX Renderer (`src/render/pptx/`)

- `pptx-renderer.ts` — Main PPTX renderer (orchestrates sub-renderers)
- `background-renderer.ts` — Background rendering (solid color, linear gradient, rounded rect)
- `border-renderer.ts` — Border rendering (4-side independent borders)
- `text-renderer.ts` — Text rendering (font mapping, multi-line, list items/bullets, text decoration)
- `utils.ts` — Utility functions (`pxToInches`, `colorToPptx`, `parseFontFamily`)

### 4.3 Rendering Pipeline (CSS Painting Order)

PPTX rendering follows the CSS painting order (7 layers):

1. **Background and borders** — `background-renderer.ts`, `border-renderer.ts`
2. **Negative z-index children** — Stacking contexts with z-index < 0
3. **Block-level content** — Block elements in normal flow
4. **Floated content** — Float elements
5. **Inline-level content** — Inline elements, text
6. **Zero/auto z-index or transformed elements** — Positioned elements
7. **Positive z-index children** — Stacking contexts with z-index > 0

### 4.4 Coordinate Conversion

- **Internal**: CSS pixels
- **PPTX**: Inches (96 DPI)
- **Conversion**: `pxToInches()` function
- **Scale**: Adjustable via `options.scale`

## 5. Public API

### Main Export

```typescript
html2pptx(element: HTMLElement | HTMLElement[], options?, config?) → Promise<PptxGenJS>
```

- **Single element**: Creates one slide
- **Element array**: Creates multiple slides

### Options Interface

```typescript
interface Options {
    slideLayout?: string;
    title?: string;
    author?: string;
    company?: string;
    scale?: number;
    x?: number;
    y?: number;
    pptx?: PptxGenJS;
    skipValidation?: boolean;
    enablePerformanceMonitoring?: boolean;
    onclone?: (document: Document) => void;
    ignoreElements?: (element: Element) => boolean;
}
```

### Other Exports

- `PptxConfig` — Configuration class
- `ConfigOptions` — Configuration interface
- `Validator` — Input validator
- `ValidationResult` — Validation result
- `createDefaultValidator` — Validator factory
- `PerformanceMonitor` — Performance monitoring
- `PptxRenderer` — PPTX renderer
- `PptxRenderOptions` — Render options
- `PptxRenderConfigurations` — Render configurations

### Output Methods

```typescript
// Via PptxGenJS instance
pptx.writeFile('presentation.pptx'); // Save to file
pptx.write({ outputType: 'blob' }); // Browser Blob
pptx.write({ outputType: 'dataUrl' }); // Base64 data URL
pptx.write({ outputType: 'arraybuffer' }); // ArrayBuffer
```

## 6. Dependencies

- `pptxgenjs` ^4.0.1 — PowerPoint generation (external in rollup, not bundled)
- `css-line-break` — Vendored in `src/lib/` (originally by Niklas von Hertzen, MIT)
- `text-segmentation` — Vendored in `src/lib/` (originally by Niklas von Hertzen, MIT)
- `utrie` — Vendored in `src/lib/` (originally by Niklas von Hertzen, MIT)

## 7. Testing

### Jest Unit Tests

- **Location**: `src/**/__tests__/` directories
- **Runner**: ts-jest with jsdom
- **Commands**:
    - `npm run unittest`
    - `npm run watch:unittest`

### Karma Browser Tests

- **Framework**: mocha
- **Commands**: `npm run karma`
- **Browsers**: Chrome, Firefox, Safari via `TARGET_BROWSER` env var

### Visual Reference Tests

- **Location**: `tests/reftests/`
- **Count**: 100+ HTML test cases

### Full Test Suite

- **Command**: `npm run test` = lint + unittest + karma

## 8. Build Output

- `dist/html2pptx-pro.js` — UMD (global: `window.html2pptx`)
- `dist/html2pptx-pro.esm.js` — ESM
- `dist/html2pptx-pro.min.js` — Minified UMD
- `dist/types/index.d.ts` — TypeScript declarations
- **External**: `pptxgenjs` (not bundled)
- **UMD Global**: `PptxGenJS`

## 10. PPTX Rendering Capabilities

### ✅ Supported

- **Backgrounds**: Solid colors, linear gradients, rounded rectangles
- **Borders**: Four-side independent borders
- **Text**: Font family, size, color, bold, italic, underline, strikethrough
- **Text Layout**: Multi-line wrapping, list items (ordered/unordered with bullets)
- **Images**: object-fit (contain, cover, scale-down)
- **Slides**: Multiple slides from element array
- **Stacking**: Full CSS stacking context (7-layer painting order)

### ❌ Not Yet Supported

- Radial gradients
- URL background images
- box-shadow effects
- opacity blending
- CSS filters
- SVG/Canvas element rendering to PPTX
- CSS transforms in PPTX output

---
> Source: [dxsun97/html2pptx-pro](https://github.com/dxsun97/html2pptx-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-24 -->

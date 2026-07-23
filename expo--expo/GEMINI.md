## expo

> A universal error overlay for Expo apps that displays runtime errors, warnings, and build errors with symbolicated stack traces.

# @expo/log-box

A universal error overlay for Expo apps that displays runtime errors, warnings, and build errors with symbolicated stack traces.

## Package Structure

```
src/
├── Data/                    # State management and data parsing
│   ├── LogBoxData.tsx       # Central state store (observer pattern)
│   ├── LogBoxLog.ts         # Log object + React context
│   ├── parseLogBoxLog.ts    # Error parsing and categorization
│   └── Types.ts             # TypeScript definitions
├── overlay/                 # Full-screen error inspector
│   ├── Overlay.tsx          # Main inspector component
│   ├── Header.tsx           # Top bar with controls
│   ├── StackTraceList.tsx   # Animated stack trace display
│   ├── CodeFrame.tsx        # Code snippet with syntax highlighting
│   ├── Message.tsx          # Error message rendering
│   ├── AnsiHighlight.tsx    # ANSI terminal color parsing
│   └── *.module.css         # CSS modules for each component
├── toast/                   # Minimal error notification
│   ├── ErrorToast.tsx       # Bottom-left toast
│   └── ErrorToast.module.css
├── utils/                   # Shared utilities
│   ├── renderInShadowRoot.ts    # Shadow DOM isolation
│   ├── devServerEndpoints.ts    # Dev server communication
│   └── parseErrorStack.ts       # Stack trace parsing
├── LogBox.ts                # Main API (install, middleware)
├── index.tsx                # Web entry point
├── index.native.tsx         # Native entry point (no-op)
├── logbox-dom-polyfill.tsx  # DOM wrapper component
├── logbox-rn-polyfill.tsx   # React Native Modal wrapper
└── logbox-web-polyfill.tsx  # Static build error display
```

## How It Works

### 1. Installation & Entry Points

**Web (`src/index.tsx`):**

- Auto-installs in development mode when `EXPO_OS === 'web'`
- Renders ErrorToast into a Shadow DOM to isolate styles
- `LogBox.install()` patches `console.error` to capture errors

**Native (`src/index.native.tsx`):**

- No-op - native platforms use React Native's built-in LogBox
- The `logbox-rn-polyfill.tsx` provides native Modal presentation when needed

### 2. Error Capture Flow

```
console.error() called
    ↓
LogBox middleware intercepts (LogBox.ts)
    ↓
parseLogBoxLog() parses into structured data
    ↓
LogBoxData.addLog() stores in Set<LogBoxLog>
    ↓
Observers notified → UI re-renders
    ↓
Symbolication requested from dev server
    ↓
Stack traces resolved → UI updates with source locations
```

### 3. Data Layer

**LogBoxData.tsx** - Central state store using observer pattern:

- `logs: Set<LogBoxLog>` - All captured logs
- `selectedLogIndex` - Currently viewed log in overlay
- `isDisabled` - Whether LogBox is suppressed
- Implements rollup: duplicate errors increment count instead of adding new entries

**LogBoxLog** - Individual log object:

```typescript
{
  level: 'error' | 'fatal' | 'syntax' | 'static',
  message: { content: string, substitutions: [] },
  category: string,           // Used for deduplication
  stack: MetroStackFrame[],   // JS call stack
  componentStack: MetroStackFrame[],  // React component tree
  codeFrame: { content, fileName, location },
  symbolicated: { stack, component }  // Resolved source maps
}
```

### 4. UI Components

**ErrorToast** - Minimal notification:

- Shows error count badge and truncated message
- Click to open full overlay
- Dismiss button clears errors

**Overlay** - Full inspector:

- Header with navigation (prev/next error), reload, minimize
- Error message with syntax highlighting
- CodeFrame showing source code context
- StackTraceList with collapsible frames
- "Open in editor" functionality

### 5. Symbolication

Stack traces start as raw JS locations and get resolved to source code:

1. Raw stack captured from error
2. Request sent to dev server `/symbolicate` endpoint
3. Server uses source maps to resolve original locations
4. UI updates with file names, line numbers, code preview
5. Results cached to avoid duplicate requests

States: `PENDING` → `COMPLETE` or `FAILED`

### 6. Shadow DOM Isolation

Web builds render into Shadow DOM (`renderInShadowRoot.ts`):

- Prevents LogBox styles from affecting the app
- Prevents app styles from affecting LogBox
- Copies only necessary stylesheets into shadow root

## Build Process

```bash
pnpm build:lib     # TypeScript → build/ (types + JS)
pnpm build:bundle  # Metro bundle → dist/ExpoLogBox.bundle/
pnpm build         # TypeScript library build only
pnpm build:publish # TypeScript library + Metro bundle
```

**Output:**

- `build/` - TypeScript compilation output
- `dist/ExpoLogBox.bundle/` - Bundled web assets (HTML + JS + CSS)

## CSS Architecture

All styles use CSS Modules (`.module.css`) with CSS custom properties:

```css
--expo-log-color-background: #111113;
--expo-log-color-border: #313538;
--expo-log-color-label: #edeef0;
--expo-log-color-danger: rgb(220, 103, 99);
--expo-log-font-family: system-ui, -apple-system, ...;
--expo-log-font-mono: SFMono-Regular, Menlo, ...;
```

## Key Files for Common Tasks

| Task                        | Files                                           |
| --------------------------- | ----------------------------------------------- |
| Change error parsing        | `Data/parseLogBoxLog.ts`                        |
| Modify toast appearance     | `toast/ErrorToast.tsx`, `ErrorToast.module.css` |
| Update overlay UI           | `overlay/Overlay.tsx`, `Overlay.module.css`     |
| Change stack trace display  | `overlay/StackTraceList.tsx`                    |
| Modify code frame           | `overlay/CodeFrame.tsx`                         |
| Update state management     | `Data/LogBoxData.tsx`                           |
| Change dev server endpoints | `utils/devServerEndpoints.ts`                   |

## Platform Differences

| Feature       | Web             | Native                     |
| ------------- | --------------- | -------------------------- |
| Entry point   | `index.tsx`     | `index.native.tsx` (no-op) |
| Rendering     | Shadow DOM      | React Native Modal         |
| Styling       | CSS Modules     | React Native StyleSheet    |
| Error capture | Patched console | RN's built-in LogBox       |

## Recent Migration (RN → DOM)

The following files were migrated from React Native components to plain React DOM:

- `AnsiHighlight.tsx` - `View`/`Text` → `div`/`span`
- `CodeFrame.tsx` - `ScrollView` → `div` with overflow
- `StackTraceList.tsx` - `Pressable` → `button` with CSS hover
- `LogBoxInspectorSourceMapStatus.tsx` - `Animated` → CSS animations, images → inline SVGs
- `ErrorToast.tsx` - `Pressable`/`Text`/`View` → `div`/`span`/`button`

---
> Source: [expo/expo](https://github.com/expo/expo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-07-23 -->

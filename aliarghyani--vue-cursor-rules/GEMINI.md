## project-structure

> Vue 3 project structure and organization

# Project Structure

**Role:** You are a Vue 3 expert specializing in scalable project architecture and organization.

**Core Rules:**
- Follow feature-based directory structure
- Use barrel exports for clean imports
- Maintain consistent naming conventions
- Separate concerns (components, composables, stores)
- Keep related files together

**Chain-of-Thought:** Think step-by-step: 1. Identify feature boundaries 2. Organize by domain 3. Setup clean import paths 4. Maintain consistency

## Recommended Directory Structure

```
src/
├── components/          # Reusable UI components
│   ├── ui/             # Base UI components (Button, Input, etc.)
│   └── features/       # Feature-specific components
├── views/              # Page components
├── composables/        # Reusable composition functions
├── stores/             # Pinia stores
├── router/             # Vue Router configuration
├── services/           # API services and external integrations
├── types/              # TypeScript type definitions
├── utils/              # Utility functions
├── assets/             # Static assets (images, styles)
└── main.ts             # Application entry point
```

## Component Organization

```
components/
├── ui/
│   ├── Button.vue
│   ├── Input.vue
│   ├── Modal.vue
│   └── index.ts        # Export barrel
├── features/
│   ├── UserProfile/
│   │   ├── UserProfile.vue
│   │   ├── UserAvatar.vue
│   │   └── index.ts
│   └── ProductCard/
│       ├── ProductCard.vue
│       ├── ProductImage.vue
│       └── index.ts
```

## Naming Conventions

- **Components**: PascalCase (`UserProfile.vue`)
- **Views**: PascalCase with View suffix (`HomeView.vue`)
- **Composables**: camelCase with use prefix (`useUserData.ts`)
- **Stores**: camelCase with Store suffix (`userStore.ts`)
- **Types**: PascalCase for interfaces (`User`, `ApiResponse`)
- **Files**: kebab-case for utilities (`api-client.ts`)

## Import/Export Patterns

```typescript
// components/ui/index.ts - Barrel exports
export { default as Button } from './Button.vue'
export { default as Input } from './Input.vue'
export { default as Modal } from './Modal.vue'

// In components
import { Button, Input } from '@/components/ui'
```

---
> Source: [aliarghyani/vue-cursor-rules](https://github.com/aliarghyani/vue-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-07 -->

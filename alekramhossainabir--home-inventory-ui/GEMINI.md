## home-inventory-ui

> This is a Next.js 15+ application using React 19, TypeScript, and modern web development best practices. All code generation must follow these guidelines strictly.

# GitHub Copilot Development Guidelines

## Project Overview
This is a Next.js 15+ application using React 19, TypeScript, and modern web development best practices. All code generation must follow these guidelines strictly.

## 1. React 19 Latest Features

### Required Usage
- **Server Components by default**: Use `"use client"` directive only when necessary (interactivity, hooks, browser APIs)
- **Server Actions**: Use Server Actions for form submissions and mutations
- **useActionState**: Replace useFormState with the new useActionState hook
- **use() hook**: For reading promises and context in components
- **useOptimistic**: For optimistic UI updates
- **useFormStatus**: For form submission states
- **ref as prop**: Pass refs directly as props instead of forwardRef
- **Enhanced Context**: Use new context improvements

### Example Pattern
```tsx
// Server Component (default)
export default async function Page() {
  const data = await fetchData();
  return <ClientComponent data={data} />;
}

// Client Component (when needed)
"use client";
export function ClientComponent({ data }) {
  const [state, dispatch] = useActionState(serverAction, initialState);
  return <form action={dispatch}>...</form>;
}
```

## 2. UX States - MANDATORY

Every data-fetching component MUST handle all three states:

### Required State Handling
```tsx
// ✅ CORRECT - All states handled
function Component() {
  const { data, isLoading, error } = useQuery(...);
  
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data || data.length === 0) return <EmptyState />;
  
  return <DataDisplay data={data} />;
}

// ❌ WRONG - Missing states
function Component() {
  const { data } = useQuery(...);
  return <DataDisplay data={data} />; // No loading/error/empty handling
}
```

### State Components
- **Loading**: Skeletons, spinners, or loading indicators
- **Empty**: Meaningful empty states with actions (e.g., "No items yet. Add your first item")
- **Error**: User-friendly error messages with retry options

## 3. Code Quality & Structure Standards

### File Organization
```
app/
├── (routes)/              # Route groups
│   ├── dashboard/
│   │   ├── page.tsx       # Server Component
│   │   └── loading.tsx    # Loading UI
│   └── items/
│       ├── page.tsx
│       ├── [id]/
│       │   └── page.tsx
│       └── error.tsx      # Error boundary
├── components/            # Reusable components
│   ├── ui/               # Base UI components
│   ├── forms/            # Form components
│   └── layout/           # Layout components
├── hooks/                # Custom React hooks
├── lib/                  # Utilities and configuration
│   ├── api/             # API functions
│   ├── utils/           # Helper functions
│   └── validations/     # Zod schemas
└── types/               # TypeScript type definitions
```

### Naming Conventions
- **Components**: PascalCase (e.g., `UserProfile.tsx`)
- **Hooks**: camelCase with `use` prefix (e.g., `useUserData.ts`)
- **Utilities**: camelCase (e.g., `formatDate.ts`)
- **Types**: PascalCase (e.g., `User`, `ApiResponse`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`)

### Code Style
- Max function length: 50 lines
- Max file length: 300 lines
- Use early returns to reduce nesting
- Prefer composition over inheritance
- Write self-documenting code with clear variable names
- Add JSDoc comments for complex functions

## 4. Responsiveness & Accessibility

### Responsive Design (Tailwind CSS)
```tsx
// ✅ Mobile-first responsive design
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <Card className="p-4 sm:p-6" />
</div>

// Use responsive utilities: sm: md: lg: xl: 2xl:
```

### Accessibility Requirements
```tsx
// ✅ Proper semantic HTML and ARIA
<button 
  aria-label="Close dialog"
  aria-pressed={isOpen}
  disabled={isLoading}
>
  Close
</button>

<img src={url} alt="Descriptive text" />

<form aria-labelledby="form-title">
  <label htmlFor="email">Email</label>
  <input id="email" type="email" required aria-required="true" />
</form>
```

### Checklist
- [ ] Semantic HTML elements (`<nav>`, `<main>`, `<article>`, etc.)
- [ ] Proper heading hierarchy (h1 → h2 → h3)
- [ ] Alt text for images
- [ ] ARIA labels for icon buttons
- [ ] Keyboard navigation support (focus states)
- [ ] Color contrast ratio ≥ 4.5:1
- [ ] Form labels and validation messages

## 5. TypeScript - STRICT MODE

### Absolutely NO `any` Type
```tsx
// ❌ FORBIDDEN
const data: any = await fetch(...);
function process(item: any) { }

// ✅ CORRECT - Use proper types
interface Item {
  id: string;
  name: string;
  quantity: number;
}

const data: Item[] = await fetch(...);
function process(item: Item): void { }

// ✅ For unknown data, use unknown and narrow
const data: unknown = await fetch(...);
if (isItem(data)) {
  process(data); // TypeScript knows data is Item
}
```

### Type Best Practices
- Define interfaces for all data structures
- Use `type` for unions/intersections, `interface` for objects
- Leverage utility types: `Partial<T>`, `Pick<T>`, `Omit<T>`, `Required<T>`
- Use `as const` for literal types
- Define return types explicitly for functions
- Use discriminated unions for variants

```tsx
// ✅ Proper typing example
interface User {
  id: string;
  name: string;
  email: string;
}

type UserInput = Omit<User, 'id'>;

async function createUser(input: UserInput): Promise<User> {
  // Implementation
}
```

## 6. TanStack Query (React Query) Guidelines

### Installation
```bash
npm install @tanstack/react-query
```

### Setup Pattern
```tsx
// app/providers.tsx
"use client";
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
      retry: 1,
    },
  },
});

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

### Query Patterns
```tsx
// lib/api/items.ts
export async function fetchItems(): Promise<Item[]> {
  const res = await fetch('/api/items');
  if (!res.ok) throw new Error('Failed to fetch items');
  return res.json();
}

// hooks/useItems.ts
import { useQuery } from '@tanstack/react-query';

export function useItems() {
  return useQuery({
    queryKey: ['items'],
    queryFn: fetchItems,
  });
}

// components/ItemList.tsx
"use client";
export function ItemList() {
  const { data, isLoading, error, refetch } = useItems();
  
  if (isLoading) return <LoadingSkeleton />;
  if (error) return <ErrorDisplay error={error} onRetry={refetch} />;
  if (!data?.length) return <EmptyState />;
  
  return <div>{data.map(item => <ItemCard key={item.id} item={item} />)}</div>;
}
```

### Mutation Patterns
```tsx
// hooks/useCreateItem.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

export function useCreateItem() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: createItem,
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['items'] });
    },
  });
}

// Usage in component
const createMutation = useCreateItem();

async function handleSubmit(data: ItemInput) {
  try {
    await createMutation.mutateAsync(data);
    toast.success('Item created');
  } catch (error) {
    toast.error('Failed to create item');
  }
}
```

### Query Key Conventions
```tsx
// ✅ Structured query keys
['items']                      // List all items
['items', itemId]             // Single item
['items', { status: 'active' }] // Filtered items
['user', userId, 'items']     // User's items
```

## 7. Component Structure

### Directory Structure
```
components/
├── ui/                    # Shadcn/ui components
│   ├── button.tsx
│   ├── card.tsx
│   └── dialog.tsx
├── forms/                 # Form-specific components
│   ├── ItemForm.tsx
│   └── LoginForm.tsx
├── layout/               # Layout components
│   ├── Header.tsx
│   ├── Sidebar.tsx
│   └── Footer.tsx
└── features/             # Feature-specific components
    ├── items/
    │   ├── ItemList.tsx
    │   ├── ItemCard.tsx
    │   └── ItemDetails.tsx
    └── dashboard/
        └── StatsCard.tsx
```

### Component Template
```tsx
// components/features/items/ItemCard.tsx
import { type FC } from 'react';
import { Card } from '@/components/ui/card';
import { type Item } from '@/types/item';

interface ItemCardProps {
  item: Item;
  onEdit?: (item: Item) => void;
  onDelete?: (id: string) => void;
}

export const ItemCard: FC<ItemCardProps> = ({ item, onEdit, onDelete }) => {
  return (
    <Card className="p-4">
      <h3 className="text-lg font-semibold">{item.name}</h3>
      <p className="text-sm text-muted-foreground">Qty: {item.quantity}</p>
      {/* Component content */}
    </Card>
  );
};
```

## 8. Error Handling

### Error Boundary (React 19)
```tsx
// app/error.tsx (Next.js 15 convention)
"use client";
import { useEffect } from 'react';
import { Button } from '@/components/ui/button';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log to error reporting service
    console.error(error);
  }, [error]);

  return (
    <div className="flex flex-col items-center justify-center min-h-[400px] gap-4">
      <h2 className="text-2xl font-bold">Something went wrong!</h2>
      <p className="text-muted-foreground">{error.message}</p>
      <Button onClick={reset}>Try again</Button>
    </div>
  );
}
```

### Centralized Error Handler
```tsx
// lib/error-handler.ts
export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export function handleApiError(error: unknown): AppError {
  if (error instanceof AppError) return error;
  
  if (error instanceof Error) {
    return new AppError(error.message);
  }
  
  return new AppError('An unexpected error occurred');
}

// Usage
try {
  await createItem(data);
} catch (error) {
  const appError = handleApiError(error);
  toast.error(appError.message);
}
```

### API Error Responses
```tsx
// app/api/items/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  try {
    const body = await request.json();
    // Process request
    return NextResponse.json({ success: true, data });
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Failed to create item' },
      { status: 500 }
    );
  }
}
```

## Additional Best Practices

### Environment Variables
```tsx
// ✅ Type-safe env variables
const API_URL = process.env.NEXT_PUBLIC_API_URL;
if (!API_URL) throw new Error('NEXT_PUBLIC_API_URL is not defined');
```

### Performance
- Use `React.memo()` for expensive components
- Implement proper code splitting
- Optimize images with Next.js Image component
- Use `loading.tsx` for route-level loading states

### Testing Considerations
- Write components to be testable (pure functions, dependency injection)
- Keep business logic separate from UI
- Use meaningful test IDs: `data-testid="item-card"`

---

## Quick Checklist for Every Feature

- [ ] React 19 features used appropriately
- [ ] All UX states handled (loading/empty/error)
- [ ] No `any` types used
- [ ] TanStack Query for data fetching
- [ ] Proper TypeScript interfaces
- [ ] Responsive design (mobile-first)
- [ ] Accessibility attributes
- [ ] Error boundaries in place
- [ ] Proper file structure
- [ ] Code is readable and maintainable

**Remember**: Quality over speed. Write code that future you will thank you for.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlEkramHossainAbir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-10 -->

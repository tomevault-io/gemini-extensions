## ai-prospect-agent

> This is the Project Genesis sandbox - a development environment for building SaaS applications using AI-assisted development with Linear task management.

# Project Genesis - Cursor AI Rules

# Last Updated: October 3, 2025

# Project: Sandbox (Development Environment)

# Owner: Farid Kheloco

## Project Context

This is the Project Genesis sandbox - a development environment for building SaaS applications using AI-assisted development with Linear task management.

Repository: https://github.com/purelyapp/projects
Vercel Team: Purely Startup (team_8lsIg2OsJ2ORppOIGFUZt55M)
Supabase Project: sandbox (grqlsdaapzqrsbaepiuj)

## Tech Stack

- **Framework**: Next.js 15.5.4 (App Router)
- **Language**: TypeScript 5 (strict mode)
- **Styling**: Tailwind CSS 4
- **UI Components**: Shadcn/ui (to be added)
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth
- **Deployment**: Vercel
- **State Management**: React hooks (useState, useContext)
- **Data Fetching**: React Server Components where possible

## Code Standards

### TypeScript Rules

- **ALWAYS** use TypeScript strict mode
- **NEVER** use `any` - use `unknown` if type is truly dynamic
- Define interfaces for all props and function parameters
- Use type inference when obvious
- Export types alongside components

Example:

```typescript
// ✅ Good
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
  disabled?: boolean;
}

export function Button({
  label,
  onClick,
  variant = "primary",
  disabled = false,
}: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled} className={`btn-${variant}`}>
      {label}
    </button>
  );
}

// ❌ Bad
export function Button(props: any) {
  return <button>{props.label}</button>;
}
```

### React Component Standards

- Use functional components only (no class components)
- Use named exports (not default exports)
- Keep components small and focused (< 200 lines)
- Extract complex logic into custom hooks
- Use Server Components by default (add 'use client' only when needed)
- One component per file

Component structure:

```typescript
"use client"; // Only if you need client-side features

import { useState } from "react"; // Imports first
// Types second
interface ComponentProps {
  title: string;
  onSave?: () => void;
}

// Component third
export function ComponentName({ title, onSave }: ComponentProps) {
  // 1. Hooks
  const [state, setState] = useState<string>("");

  // 2. Effects
  useEffect(() => {
    // effect logic
  }, []);

  // 3. Handlers
  const handleClick = () => {
    setState("clicked");
    onSave?.();
  };

  // 4. Early returns
  if (!title) return null;

  // 5. Render
  return (
    <div>
      <h1>{title}</h1>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```

### File Naming Conventions

- **Components**: `PascalCase.tsx` (e.g., `UserProfile.tsx`, `LoginForm.tsx`)
- **Utilities**: `camelCase.ts` (e.g., `formatDate.ts`, `validateEmail.ts`)
- **API routes**: `route.ts` (Next.js App Router convention)
- **Types**: `types.ts` or `ComponentName.types.ts`
- **Hooks**: `useCamelCase.ts` (e.g., `useAuth.ts`, `useLocalStorage.ts`)

### Directory Structure

```
app/
  ├── (auth)/              # Auth-related pages (route groups)
  │   ├── login/
  │   └── signup/
  ├── (dashboard)/         # Protected pages
  │   ├── profile/
  │   └── settings/
  ├── api/                 # API routes
  │   ├── auth/
  │   └── users/
  ├── layout.tsx           # Root layout
  └── page.tsx             # Home page
components/
  ├── ui/                  # Shadcn components
  ├── forms/               # Form components
  ├── layouts/             # Layout components
  └── ...                  # Feature components
lib/
  ├── supabase.ts          # Supabase client
  ├── utils.ts             # Utility functions
  ├── constants.ts         # App constants
  └── ...                  # Other libraries
hooks/
  ├── useAuth.ts           # Authentication hook
  └── ...                  # Custom hooks
types/
  ├── database.ts          # Database types
  └── ...                  # Type definitions
```

### API Route Standards

- **Always** validate input with Zod
- **Always** handle errors gracefully
- Return consistent JSON response format
- Use appropriate HTTP status codes
- Include error messages that are helpful but not revealing

API Route Template:

```typescript
import { NextResponse } from "next/server";
import { z } from "zod";
import { supabase } from "@/lib/supabase";

// Define request schema
const createUserSchema = z.object({
  email: z.string().email("Invalid email address"),
  full_name: z.string().min(1, "Name is required").max(100),
  avatar_url: z.string().url().optional(),
});

export async function POST(req: Request) {
  try {
    // 1. Parse and validate request body
    const body = await req.json();
    const validatedData = createUserSchema.parse(body);

    // 2. Business logic
    const { data, error } = await supabase
      .from("profiles")
      .insert([validatedData])
      .select()
      .single();

    if (error) {
      throw new Error(`Database error: ${error.message}`);
    }

    // 3. Return success response
    return NextResponse.json(
      { data, message: "User created successfully" },
      { status: 201 }
    );
  } catch (error) {
    // 4. Handle specific error types
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: "Validation failed", details: error.errors },
        { status: 400 }
      );
    }

    // 5. Log error for debugging (never expose to user)
    console.error("API Error:", error);

    // 6. Return generic error to user
    return NextResponse.json(
      { error: "An error occurred. Please try again." },
      { status: 500 }
    );
  }
}
```

### Database Query Standards

- **Always** use Supabase RLS (Row Level Security)
- **Always** handle errors explicitly
- Use TypeScript types generated from Supabase
- Use transactions for multi-step operations
- Add comments for complex queries

Query Template:

```typescript
// ✅ Good - Explicit error handling
const { data, error } = await supabase
  .from("profiles")
  .select("id, email, full_name, avatar_url")
  .eq("id", userId)
  .single();

if (error) {
  console.error("Failed to fetch profile:", error);
  throw new Error("Could not load profile");
}

if (!data) {
  throw new Error("Profile not found");
}

return data;

// ❌ Bad - No error handling
const { data } = await supabase.from("profiles").select("*").eq("id", userId);

return data;
```

### Styling Standards

- Use Tailwind utility classes (no custom CSS unless absolutely necessary)
- Use Shadcn/ui components as base
- Follow mobile-first responsive design
- Use semantic color names from Tailwind config
- Group related classes (layout, spacing, colors, typography)

Example:

```tsx
// ✅ Good - Clear, organized Tailwind classes
<button
  className="
    flex items-center gap-2
    rounded-lg px-4 py-2
    bg-blue-600 hover:bg-blue-700
    text-white font-medium
    transition-colors
    disabled:opacity-50 disabled:cursor-not-allowed
  "
>
  Save Changes
</button>

// ❌ Bad - Inline styles
<button style={{ backgroundColor: 'blue', padding: '8px 16px' }}>
  Save
</button>

// ❌ Bad - Custom CSS
<button className="custom-button">
  Save
</button>
```

### Error Handling Standards

- Use try-catch for all async operations
- Log errors with context for debugging
- Show user-friendly messages (never show stack traces)
- Have fallback UI for errors
- Handle loading and error states in components

Example:

```typescript
// ✅ Good - Comprehensive error handling
try {
  const result = await saveUserProfile(data);
  toast.success("Profile updated successfully");
  return result;
} catch (error) {
  // Log for debugging (never show to user)
  console.error("Profile update failed:", {
    userId: data.id,
    error: error instanceof Error ? error.message : "Unknown error",
    timestamp: new Date().toISOString(),
  });

  // Show user-friendly message
  toast.error("Could not save profile. Please try again.");

  // Re-throw if caller needs to handle it
  throw error;
}

// ❌ Bad - No error handling
const result = await saveUserProfile(data);
toast.success("Saved!");
```

## Git Commit Standards

Follow Conventional Commits:

- `feat: Add user authentication flow`
- `fix: Resolve login redirect loop`
- `docs: Update README with deployment steps`
- `refactor: Simplify database query logic`
- `style: Format code with Prettier`
- `test: Add tests for user profile API`
- `chore: Update dependencies to latest versions`
- `perf: Optimize image loading performance`

Include Linear task reference:

- `feat(auth): [LIN-42] Implement OAuth login`
- `fix(ui): [LIN-43] Fix responsive nav on mobile`

## Linear Task Integration

When working on a Linear task:

1. ✅ **Read** the full task description in Linear
2. ✅ **Review** all acceptance criteria
3. ✅ **Follow** implementation details provided
4. ✅ **Update** Linear status when starting work
5. ✅ **Comment** on task with questions or progress updates
6. ✅ **Reference** task in commits: `feat: [LIN-42] Add login form`
7. ✅ **Link** PR to Linear task in PR description
8. ✅ **Test** against acceptance criteria before marking complete

## Pull Request Checklist

Before creating a PR, verify:

- [ ] No TypeScript errors (`npm run build`)
- [ ] No ESLint warnings (`npm run lint`)
- [ ] No console.log() or console.error() statements (use proper logging)
- [ ] Code is formatted (Prettier runs automatically)
- [ ] All acceptance criteria from Linear task are met
- [ ] Error handling is implemented for all async operations
- [ ] Loading states are implemented for async UI
- [ ] Mobile responsive (if UI change)
- [ ] Tested locally in development mode
- [ ] Tested in production mode (`npm run build && npm start`)
- [ ] Environment variables not committed (.env.local not in Git)
- [ ] No hardcoded values (use environment variables)
- [ ] Comments added for complex logic
- [ ] Linear task is referenced in PR title and description

## Performance Guidelines

- ✅ Use `next/image` for all images (automatic optimization)
- ✅ Implement loading states for async operations (Suspense or manual)
- ✅ Use React Server Components by default (faster initial load)
- ✅ Lazy load components when appropriate (`React.lazy` or `next/dynamic`)
- ✅ Minimize client-side JavaScript (keep 'use client' minimal)
- ✅ Use proper caching headers for static assets
- ✅ Optimize database queries (select only needed columns)
- ✅ Use database indexes for frequently queried columns

## Security Guidelines

- ⚠️ **NEVER** commit `.env.local` or any file with secrets
- ⚠️ **NEVER** expose `SUPABASE_SERVICE_ROLE_KEY` to client
- ⚠️ **ALWAYS** validate user input (use Zod schemas)
- ⚠️ **ALWAYS** use Supabase RLS for database security
- ⚠️ **ALWAYS** sanitize user-generated content before display
- ⚠️ Use HTTPS only (Vercel provides this automatically)
- ⚠️ Implement rate limiting for public APIs
- ⚠️ Use CSRF protection for forms (Next.js provides this)
- ⚠️ Never trust client-side validation alone

## Testing Requirements

- Write unit tests for utility functions
- Test API routes with edge cases
- Test components with user interactions
- Test error scenarios
- Aim for 80%+ coverage on critical paths
- Use meaningful test descriptions

## AI Assistant Guidelines (For You, Cursor)

When helping Farid:

- ✅ **Ask** clarifying questions if the task is ambiguous
- ✅ **Provide** code that follows ALL standards in this file
- ✅ **Explain** your reasoning for architectural decisions
- ✅ **Point out** potential issues, edge cases, or improvements
- ✅ **Suggest** testing approaches for the code you generate
- ✅ **Reference** relevant documentation when appropriate
- ✅ **Break down** complex tasks into smaller, manageable steps
- ✅ **Validate** that generated code meets the acceptance criteria
- ✅ **Check** for common mistakes (missing error handling, any types, etc.)

## Common Mistakes to Avoid

- ❌ Using `any` type instead of proper TypeScript types
- ❌ Default exports (use named exports)
- ❌ Forgetting error handling in async operations
- ❌ Not validating API input
- ❌ Exposing sensitive data in client code
- ❌ Hardcoding values instead of using environment variables
- ❌ Forgetting to add 'use client' when using hooks/browser APIs
- ❌ Not testing responsive design on mobile
- ❌ Leaving console.log statements in production code
- ❌ Not referencing Linear tasks in commits

## Resources

- **Next.js Docs**: https://nextjs.org/docs
- **Supabase Docs**: https://supabase.com/docs
- **Tailwind CSS Docs**: https://tailwindcss.com/docs
- **Shadcn/ui**: https://ui.shadcn.com/
- **TypeScript Handbook**: https://www.typescriptlang.org/docs
- **React Docs**: https://react.dev/
- **Zod Documentation**: https://zod.dev/

## Project-Specific Notes

**Vercel Configuration:**

- Team: Purely Startup
- Team ID: team_wpRuz7DhDS5hvKSyDFKapBhy
- Project: sandbox
- Auto-deploy: main branch → production, PRs → preview
- Production URL: https://sandbox-purely-startup.vercel.app

**Supabase Configuration:**

- Project: sandbox
- Project ID: grqlsdaapzqrsbaepiuj
- URL: https://grqlsdaapzqrsbaepiuj.supabase.co
- Region: West US (North California)
- RLS enabled on all tables
- Initial schema includes: profiles table with triggers

**Current Schema:**

- `profiles` table with id, email, full_name, avatar_url, created_at, updated_at
- RLS policies: users can view/update own profile
- Trigger: auto-create profile on user signup
- Trigger: auto-update updated_at timestamp

**Environment Variables:**

- `NEXT_PUBLIC_SUPABASE_URL`: https://grqlsdaapzqrsbaepiuj.supabase.co
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: [configured]
- `SUPABASE_SERVICE_ROLE_KEY`: [configured]
- `DATABASE_URL`: [configured]

**Current Dependencies:**

- Next.js: 15.5.4
- React: 19.1.0
- TypeScript: 5
- Tailwind CSS: 4
- Supabase: 2.58.0
- ESLint: 9

---

**Remember:** Quality over speed. Write code that you'd be proud to review in 6 months.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/purelyworks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-09 -->

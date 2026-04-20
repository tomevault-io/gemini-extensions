## liftronic-elevator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Start development server:**
```bash
pnpm dev
```
Launches Next.js with Turbopack at http://localhost:3000 with hot reload enabled.

**Production build:**
```bash
pnpm build
```
Creates optimized production bundle. Run before committing significant changes.

**Serve production build locally:**
```bash
pnpm start
```
Use to validate production behavior and deployment fixes.

**Lint code:**
```bash
pnpm lint
```
Runs ESLint with Next.js and TypeScript rules. Must pass before opening PRs.

## Architecture Overview

### Tech Stack
- **Framework:** Next.js 15.5.2 (React 19.1.0) with App Router
- **CMS:** Sanity v4.9.0 (headless CMS for content management)
- **Styling:** Tailwind CSS v4 with semantic tokens
- **Animation:** Motion v12.23.12 (motion/react)
- **Forms:** react-hook-form v7.62.0 + Zod v4.1.5 validation
- **Progress Indicators:** nextjs-toploader v3.9.17
- **Icons:** @sanity/icons v3.7.4 + react-icons v5.5.0
- **Utilities:** clsx v2.1.1 + tailwind-merge v3.3.1
- **Content Rendering:** @portabletext/react v4.0.3
- **Language:** TypeScript v5 (strict mode)
- **Deployment:** Cloudflare Pages (@opennextjs/cloudflare v1.9.0)

### Directory Structure
```
src/
├── app/                  # Next.js App Router
│   ├── (main)/          # Main site route group
│   │   ├── aboutus/     # About Us page
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── blogs/       # Blog listing & posts
│   │   │   ├── [slug]/
│   │   │   │   ├── BlogPostClient.tsx
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── sitemap.ts
│   │   ├── media/       # Media gallery
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── products/    # Products section
│   │   │   ├── [slug]/
│   │   │   │   ├── [city]/      # Location-based product pages
│   │   │   │   │   └── page.tsx
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── sitemap.ts
│   │   ├── services/    # Services section
│   │   │   ├── [slug]/
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── layout.tsx   # Main site layout (Navbar + Footer)
│   │   └── page.tsx     # Homepage
│   ├── studio/          # Sanity Studio admin interface
│   │   └── [[...tool]]/page.tsx
│   ├── layout.tsx       # Root layout with fonts & metadata
│   ├── globals.css      # Global styles & Tailwind config
│   ├── not-found.tsx    # 404 page
│   ├── robots.ts        # robots.txt generation
│   └── sitemap.ts       # XML sitemap generation
├── components/          # Reusable React components (PascalCase)
│   ├── aboutus/
│   │   ├── CertificateModal.tsx
│   │   ├── CertificatesSection.tsx
│   │   ├── TeamMemberModal.tsx
│   │   ├── TeamSection.tsx
│   │   ├── VisionMissionValues.tsx
│   │   └── WhyUsSection.tsx
│   ├── blog/
│   │   ├── BlogCard.tsx
│   │   ├── FeaturedBlogCard.tsx
│   │   └── PortableTextRenderer.tsx
│   ├── homepage/
│   │   ├── AboutUs.tsx
│   │   ├── BlogSection.tsx
│   │   ├── ClientsMarquee.tsx
│   │   ├── ContactSection.tsx
│   │   ├── FAQSection.tsx
│   │   ├── Hero.tsx
│   │   ├── MediaPreview.tsx
│   │   ├── SEOContentSection.tsx
│   │   ├── Services.tsx
│   │   └── Testimonials.tsx
│   ├── layout/
│   │   ├── Footer.tsx
│   │   ├── FooterSitemapLinks.tsx
│   │   └── Navbar.tsx
│   ├── media/
│   │   ├── MediaCard.tsx
│   │   ├── MediaFilters.tsx
│   │   ├── MediaPageClient.tsx
│   │   └── MediaPreview.tsx
│   ├── products/
│   │   ├── ProductCard.tsx
│   │   ├── ProductCarousel.tsx
│   │   ├── ProductGalleryModal.tsx
│   │   └── ProductsInteractive.tsx
│   ├── services/
│   │   ├── ServiceCard.tsx
│   │   └── ServicesGrid.tsx
│   └── [Shared Components]
│       ├── AnimatedNumbers.tsx
│       ├── Breadcrumb.tsx
│       ├── CallToActionSection.tsx
│       ├── CatalogModal.tsx
│       ├── DownloadCatalogButton.tsx
│       ├── FAQ.tsx
│       ├── Features.tsx
│       ├── Quote.tsx
│       ├── QuoteCTA.tsx
│       └── WhatsAppButton.tsx
├── hooks/               # Custom React hooks
│   ├── useSmoothScroll.ts     # Smooth scrolling navigation
│   └── useViewTransition.ts   # View transitions API
└── sanity/              # Sanity CMS integration
    ├── lib/             # Type definitions & client config
    │   ├── aboutTypes.ts
    │   ├── blogTypes.ts
    │   ├── certificateTypes.ts
    │   ├── client.ts           # Sanity client configuration
    │   ├── clientTypes.ts
    │   ├── image.ts            # Image URL builder
    │   ├── live.ts             # Live preview support
    │   ├── mediaTypes.ts
    │   ├── productTypes.ts
    │   ├── serviceTypes.ts
    │   └── testimonialTypes.ts
    ├── schemaTypes/     # Content type schemas (26 schemas)
    │   ├── authorType.ts
    │   ├── blockContentType.ts
    │   ├── categoryType.ts
    │   ├── certificateType.ts
    │   ├── clients.ts
    │   ├── companyInfo.ts
    │   ├── contactInfo.ts
    │   ├── faq.ts
    │   ├── homePageSeo.ts
    │   ├── homePageSettings.ts
    │   ├── keyFeatures.ts
    │   ├── media.ts
    │   ├── postType.ts
    │   ├── products.ts          # Product schema with location pages
    │   ├── serviceOffered.ts    # Service schema
    │   ├── servicePerformanceNumbers.ts
    │   ├── socials.ts
    │   ├── tags.ts
    │   ├── teamMember.ts
    │   ├── testimonial.ts
    │   ├── timeline.ts
    │   ├── visionMissionValues.ts
    │   ├── whyChooseUs.ts
    │   └── index.ts             # Schema registry
    ├── utils/           # Data fetching utilities
    │   ├── getAboutUs.ts
    │   ├── getCertificates.ts
    │   ├── getClients.ts
    │   ├── getContactInfo.ts
    │   ├── getFAQs.ts
    │   ├── getHomePageSeo.ts
    │   ├── getKeyFeatures.ts
    │   ├── getPerformances.ts
    │   ├── getPost.ts
    │   ├── getProducts.ts
    │   ├── getSocials.ts
    │   ├── getTags.ts
    │   ├── getTestimonials.ts
    │   └── iconMapper.ts        # Icon mapping utility
    ├── env.ts           # Environment validation
    └── structure.ts     # Studio structure config
```

### Key Architectural Patterns

**Server Components by Default:**
- Build React Server Components (RSC) unless interactivity or browser APIs are required
- Only add `"use client"` when necessary (forms, animations, browser APIs, event handlers)
- Server components can import server-side utilities directly
- Use async/await in Server Components to fetch data at build/request time

**Motion Library Usage:**
```tsx
// Client components
import { motion } from "motion/react"

// Server components
import * as motion from "motion/react-client"
```

**Path Aliases:**
Use `~/` for all imports from `src/`:
```tsx
import { Hero } from "~/components/Hero"
import { client } from "~/sanity/lib/client"
```
Never use relative paths like `../../components/Hero`.

**Sanity CMS Integration:**
- Content types defined in `src/sanity/schemaTypes/` (26 schema types total)
- Sanity Studio available at `/studio` route
- Client configured in `src/sanity/lib/client.ts`
- All Sanity queries use GROQ syntax
- Type definitions in `src/sanity/lib/*Types.ts` for type safety
- Data fetching utilities in `src/sanity/utils/get*.ts`
- Use `~/sanity/utils/iconMapper.ts` for all backend-integrated icons instead of local imports
- Image optimization via `~/sanity/lib/image.ts` with next-sanity

**Location-Based SEO Pages:**
- Products support location-specific pages (`/products/[slug]/[city]`)
- Each location page requires:
  - Minimum 1500 words unique content
  - Unique meta title and description
  - City-specific keywords
  - Published and enableIndexing flags for control
- Prevents duplicate content penalties through unique content validation

**Navigation & Smooth Scrolling:**
- Custom `useSmoothScroll` hook for anchor navigation
- Navbar detects active routes and highlights current section
- Hash links (`#contact`) work across pages with automatic navigation
- Featured products highlighted with special styling

**Performance Optimizations:**
- Next.js Turbopack for fast development builds
- nextjs-toploader for page transition feedback
- Image optimization with next/image + Sanity image URLs
- DM Sans font with display: swap and preload
- DNS prefetch and preconnect for Sanity CDN

**Environment Variables:**
Required in `.env.local`:
```
NEXT_PUBLIC_SANITY_PROJECT_ID=your_project_id
NEXT_PUBLIC_SANITY_DATASET=production
NEXT_PUBLIC_SANITY_API_VERSION=2025-09-20
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

**Important Notes:**
- **DO NOT use View Transition API** - This project doesn't utilize view transitions
- Navbar uses glass morphism effect that changes on scroll
- WhatsApp and catalog download buttons are persistent across pages
- Custom forms built with react-hook-form and Zod validation

## Coding Standards

**TypeScript:**
- Strict mode enabled (`strict: true` in tsconfig.json)
- Explicit types for props, hooks, utility returns
- No `any` types unless absolutely necessary. Add comment `// @ts-ignore` if using `any`
- Use TypeScript interfaces for component props
- Leverage type definitions from `~/sanity/lib/*Types.ts`

**React Components:**
- Functional components only (no class components)
- PascalCase file naming (e.g., `HeroBanner.tsx`, `ProductCard.tsx`)
- Keep hooks near their usage site
- Prefer composition over prop drilling
- Use React 19 features appropriately
- Mark client components with `"use client"` directive only when needed

**Styling:**
- Tailwind CSS v4 utilities preferred
- Use semantic color tokens:
  - `bg-accent` - Primary brand color (#2ae394)
  - `text-charcoal` - Primary text color
  - `bg-soft` - Background color
  - `bg-brand` - Brand color for CTAs
  - `text-brand` - Brand color for text
- Mobile-first responsive design (use `md:`, `lg:` breakpoints)
- Avoid custom inline CSS variables (use Tailwind config)
- Use `clsx` and `tailwind-merge` for conditional classes
- Glass morphism effects: `glass-solid`, `glass-transparent`

**Component Organization:**
- Domain-specific components in subdirectories:
  - `components/blog/` - Blog-related components
  - `components/products/` - Product-related components
  - `components/services/` - Service-related components
  - `components/homepage/` - Homepage sections
  - `components/aboutus/` - About page sections
  - `components/media/` - Media gallery components
  - `components/layout/` - Layout components (Navbar, Footer)
- Shared components at root of `components/` directory
- Keep route-specific components colocated with routes when only used there
- One component per file

**Validation:**
- Use `react-hook-form` for form state management
- Use `zod` schemas for input validation
- Validate all user inputs and external data
- Sanity schemas include built-in validation rules

## Commit Guidelines

**Commit Message Format:**
```
feat: Add passenger elevator specs
fix: Correct animation timing on Hero component
chore: Update dependencies
```

Use imperative mood with optional conventional commit prefixes (`feat:`, `fix:`, `chore:`).

**Pull Request Requirements:**
- Single-topic diffs
- Reference issues using `#123`
- Include before/after screenshots for UI changes
- Document manual verification steps
- Confirm `pnpm lint` and `pnpm build` succeed locally

## SEO & Metadata

**JSON-LD Structured Data:**
Root layout includes four JSON-LD blocks:
1. Organization with aggregate rating
2. Website with sitelinks searchbox
3. LocalBusiness (HomeAndConstructionBusiness) with service areas
4. BreadcrumbList for navigation

When adding new routes, maintain consistent metadata patterns using Next.js `metadata` export.

**Viewport & Theme:**
- Theme color: `#2ae394` (accent color)
- Viewport configured in root layout export
- Mobile-optimized with proper scaling

## Testing

- No automated test suite currently configured
- Manual testing required for all changes
- Use `pnpm lint` as primary quality gate
- Validate responsive behavior manually across breakpoints

## Important Notes

- **Never commit secrets** to the repository; use `.env.local`
- Sanity Studio configuration uses `"use client"` directive at top level
- ViewTransition wrapper in root layout enables native view transitions
- Route groups like `(main)` don't affect URL structure
- All media assets go in `public/` directory
- Optimize images before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dev-rocketdevelopers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-09 -->

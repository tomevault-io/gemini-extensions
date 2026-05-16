## masdik-iqis

> **MASDIK IQIS Digital Hub** is the official web application for **Masjid Pendidikan Ibnul Qayyim Makassar (MASDIK IQIS)** — an Islamic educational mosque in Makassar, Indonesia. The site serves as the mosque's digital presence and management platform.

# CLAUDE.md — MASDIK IQIS Digital Hub

## 1. Project Overview

**MASDIK IQIS Digital Hub** is the official web application for **Masjid Pendidikan Ibnul Qayyim Makassar (MASDIK IQIS)** — an Islamic educational mosque in Makassar, Indonesia. The site serves as the mosque's digital presence and management platform.

**Production URL:** `https://masdik.iqis.sch.id`

### Public-facing features:
- Live prayer times (from aladhan.com API, location: Makassar)
- Mosque activity calendar
- Facility reservation/booking form
- Financial transparency (income/expense ledger + QRIS donation QR)
- DKM organizational structure
- Pustaka (library of documents, videos, audio)
- e-Taklim (online learning — currently "coming soon")

### Admin features (behind auth):
- Manage activities (add/edit/delete)
- Manage reservations (approve/reject)
- Manage financial transactions
- Manage Pustaka items
- QR-code-based attendance system for kajian/rapat/daurah activities

---

## 2. Tech Stack

| Category | Technology |
|---|---|
| Framework | React 18 with TypeScript |
| Build tool | Vite 5 (with `@vitejs/plugin-react-swc`) |
| Routing | React Router DOM v6 |
| Backend/DB | Supabase (PostgreSQL + Auth + RLS) |
| State/data | TanStack React Query v5 |
| UI components | shadcn/ui (Radix UI primitives) |
| Styling | Tailwind CSS v3 + `tailwindcss-animate` |
| Fonts | Plus Jakarta Sans (body), Amiri (Arabic) |
| Forms | react-hook-form + zod + @hookform/resolvers |
| Animations | Framer Motion |
| Date handling | date-fns v3 |
| Charts | Recharts |
| QR codes | qrcode.react |
| Notifications | sonner + shadcn toast |
| Deployment | FTP via GitHub Actions → `masdik.iqis.sch.id` |
| Lint | ESLint 9 + typescript-eslint |

---

## 3. Project Structure

```
masdik-iqis-digital-hub/
├── .env                          # Supabase credentials (VITE_SUPABASE_URL, VITE_SUPABASE_PUBLISHABLE_KEY)
├── .github/workflows/main.yml    # CI/CD: build + FTP deploy on push to main
├── vercel.json                   # SPA rewrite rules (fallback to index.html)
├── vite.config.ts
├── tailwind.config.ts            # Custom colors (gold, emerald), fonts, animations
├── components.json               # shadcn/ui config
├── tsconfig.json / tsconfig.app.json / tsconfig.node.json
├── supabase/
│   ├── config.toml               # project_id = ozwshtyhnbayieximrkh
│   └── migrations/               # 14 SQL migration files (schema history)
├── public/
│   ├── favicon.png
│   ├── robots.txt
│   └── sitemap.xml
└── src/
    ├── main.tsx                  # React root mount
    ├── App.tsx                   # Router + providers + AdminRoute guard
    ├── index.css                 # Tailwind + CSS vars + Islamic custom utilities
    ├── App.css
    ├── vite-env.d.ts
    ├── assets/
    │   ├── logo-masjid.png       # Color logo (used in Navbar)
    │   ├── logo-masjid-white.png # White logo variant
    │   └── qris-donasi.jpg       # QRIS donation QR image (BSI 7301136287)
    ├── types/
    │   └── index.ts              # Shared TS interfaces: PrayerTime, Event, BookingRequest, DKMMember, Transaction
    ├── hooks/
    │   ├── useAuth.tsx           # AuthContext: user, session, isAdmin, signIn/signUp/signOut
    │   ├── use-toast.ts          # Toast hook (shadcn)
    │   └── use-mobile.tsx        # Mobile breakpoint hook
    ├── lib/
    │   └── utils.ts              # cn() helper (clsx + tailwind-merge)
    ├── integrations/supabase/
    │   ├── client.ts             # createClient<Database>(...) singleton
    │   └── types.ts              # Auto-generated Supabase types (full DB schema)
    ├── pages/
    │   ├── Index.tsx             # Home: ArabicGreeting + PrayerTimes + QuickLinks + About
    │   ├── Kegiatan.tsx          # Activity calendar page
    │   ├── ETaklim.tsx           # E-learning page (Coming Soon)
    │   ├── Pustaka.tsx           # Library page (public browsing)
    │   ├── Layanan.tsx           # Reservation page (public form)
    │   ├── Struktur.tsx          # DKM org structure page
    │   ├── Saldo.tsx             # Financial info page
    │   ├── Admin.tsx             # Full admin dashboard (tabs: Kegiatan, Reservasi, Keuangan, Pustaka, Absensi)
    │   ├── AdminLogin.tsx        # Email/password login form
    │   ├── AttendanceManagement.tsx  # Per-activity QR + attendance records (admin)
    │   ├── AttendanceForm.tsx    # Public QR-scan attendance form (/absen/:token)
    │   └── NotFound.tsx          # 404 page
    ├── components/
    │   ├── Navbar.tsx            # Fixed nav with mobile menu, active link highlighting
    │   ├── Footer.tsx            # Footer with links/info
    │   ├── ArabicGreeting.tsx    # Bismillah / Assalamu'alaikum display
    │   ├── PrayerTimes.tsx       # Live clock + prayer times card (aladhan API)
    │   ├── EventCalendar.tsx     # Calendar + event list for Kegiatan page
    │   ├── BookingForm.tsx       # Reservation form with conflict detection
    │   ├── SaldoSection.tsx      # Balance display + QRIS card + transaction list
    │   ├── DKMStructure.tsx      # Hardcoded org chart (static data)
    │   ├── NavLink.tsx           # Navigation link helper
    │   ├── admin/
    │   │   └── PustakaManager.tsx  # Admin CRUD for pustaka items
    │   └── ui/                   # ~40 shadcn/ui components (accordion, button, card, dialog, etc.)
```

---

## 4. Key Files

| File | Purpose |
|---|---|
| `src/App.tsx` | App root: wraps everything in QueryClient, AuthProvider, TooltipProvider, BrowserRouter. Defines all routes and the `AdminRoute` protected wrapper. |
| `src/hooks/useAuth.tsx` | Auth context. Checks `user_roles` table for `admin` role after login. Exposes `isAdmin` boolean. |
| `src/integrations/supabase/client.ts` | Supabase client singleton. Auth persisted in localStorage. |
| `src/integrations/supabase/types.ts` | Full auto-generated DB types. Source of truth for table schemas. |
| `src/pages/Admin.tsx` | Largest file — full admin dashboard with tabs for all management tasks. ~500+ lines. |
| `src/pages/AttendanceForm.tsx` | Public attendance form accessed via QR token URL. Validates GPS proximity (500m radius). |
| `src/pages/AttendanceManagement.tsx` | Admin page to generate QR sessions, view attendance records and charts per activity. |
| `src/components/PrayerTimes.tsx` | Fetches live prayer times from `api.aladhan.com` for Makassar coords. Shows running activities. |
| `src/components/DKMStructure.tsx` | **Hardcoded** list of 17 DKM members — not from database. |
| `tailwind.config.ts` | Defines `gold` and `emerald` custom colors; `gradient-islamic`, `gradient-gold`, `shadow-islamic`, `islamic-pattern` utilities in CSS. |
| `src/index.css` | CSS custom properties for all theme colors (light/dark), font-arabic utility, islamic-pattern background. |
| `.github/workflows/main.yml` | On push to `main`: installs deps, builds, deploys `/dist/` to FTP server. |

---

## 5. Architecture & Patterns

### Routing
- React Router DOM v6 with `<BrowserRouter>` + `<Routes>`
- `vercel.json` and server must rewrite all paths to `index.html` for SPA routing
- `AdminRoute` component guards `/admin` — redirects to `/admin-login` if not authenticated or not admin

### Authentication
- Supabase Auth (email/password)
- Session persisted in localStorage (`persistSession: true`)
- Admin check: query `user_roles` table for `role = 'admin'` after auth state change
- `AuthProvider` wraps entire app; `useAuth()` hook used throughout
- `setTimeout(0)` pattern used in auth listener to avoid Supabase deadlock

### Data Fetching
- Direct Supabase client calls (`.from('table').select(...)`) in `useEffect` hooks
- TanStack React Query is set up (QueryClientProvider in App.tsx) but components primarily use direct Supabase calls with local state
- No centralized query key management

### State Management
- Local `useState` in each page/component
- No global state beyond AuthContext

### UI
- shadcn/ui components built on Radix UI primitives
- Islamic-themed design system: emerald green primary, gold accent
- Custom CSS: `.gradient-islamic`, `.gradient-gold`, `.shadow-islamic`, `.islamic-pattern`, `.font-arabic`
- Responsive with Tailwind breakpoints (mobile-first, `lg:` variants common)
- Framer Motion for page entrance animations (ETaklim, Pustaka)

---

## 6. Database Schema (Supabase)

### Tables

**`activities`** — Mosque events/activities
- `id`, `title`, `description`, `event_date` (DATE), `event_time` (TEXT HH:mm), `event_end_time`, `type` (kajian|pengajian|shalat|acara|sosial|reservasi), `is_active`, `speaker_name`, `topic`, `total_sessions`, `created_by`, timestamps

**`reservations`** — Public facility booking requests
- `id`, `name`, `phone`, `email`, `activity_type`, `reservation_date`, `reservation_time`, `reservation_end_time`, `description`, `status` (pending|approved|rejected), `reviewed_by`, `reviewed_at`, `created_at`

**`transactions`** — Financial ledger (income/expense)
- `id`, `type` (income|expense), `amount`, `description`, `category`, `created_by`, `created_at`

**`dkm_members`** — Organizational members (DB version, currently not used by DKMStructure component which is hardcoded)
- `id`, `name`, `position`, `role`, `photo_url`, `order_index`, `is_active`, timestamps

**`profiles`** — User profile data (auto-created on signup via trigger)
- `id` (FK → auth.users), `full_name`, `phone`, `email`, `avatar_url`, timestamps

**`user_roles`** — RBAC
- `id`, `user_id` (FK → auth.users), `role` (app_role enum: admin|user), `created_at`
- Unique constraint: `(user_id, role)`

**`pustaka`** — Library items (documents, videos, audio)
- `id`, `title`, `description`, `type` (document|video|audio), `file_url`, `thumbnail_url`, `category`, `is_active`, `created_by`, `created_by_name`, timestamps

**`attendance_sessions`** — QR sessions per activity
- `id`, `activity_id` (FK → activities), `session_number`, `session_label`, `qr_token` (UNIQUE), `scan_type` (arrival|completion), `is_active`, timestamps

**`attendance_records`** — Individual attendance submissions
- `id`, `session_id` (FK), `activity_id` (FK), `participant_name`, `feedback`, `device_fingerprint`, `latitude`, `longitude`, `created_at`
- Unique constraint: `(session_id, device_fingerprint)` — prevents duplicate submissions per device

### RLS Policies (Row Level Security)
- Most tables: **SELECT public, INSERT/UPDATE/DELETE admin only**
- Exceptions:
  - `reservations`: anyone can INSERT (public booking)
  - `attendance_records`: anyone can INSERT (public attendance), SELECT admin only
  - `transactions`: SELECT public

### Database Functions
- `has_role(_user_id, _role)` — SECURITY DEFINER — checks user_roles table
- `handle_new_user()` — trigger on auth.users INSERT, auto-creates profiles row
- `update_updated_at_column()` — trigger to keep updated_at current

---

## 7. Environment & Config

### Required Environment Variables
```
VITE_SUPABASE_URL=https://ozwshtyhnbayieximrkh.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=<anon key>
VITE_SUPABASE_PROJECT_ID=ozwshtyhnbayieximrkh
```

The `.env` file is committed (contains public anon key, not secret key — this is intentional for Supabase public clients).

### GitHub Actions Secrets Required
For CI/CD (`main.yml`):
- `VITE_SUPABASE_PROJECT_ID`
- `VITE_SUPABASE_PUBLISHABLE_KEY`
- `VITE_SUPABASE_URL`
- `FTP_SERVER`
- `FTP_USERNAME`
- `FTP_PASSWORD`

### Supabase Project
- Project ID: `ozwshtyhnbayieximrkh`
- Config: `supabase/config.toml`

---

## 8. Scripts & Commands

```bash
npm run dev        # Start dev server (Vite HMR)
npm run build      # Production build → /dist
npm run build:dev  # Dev-mode build (for debugging)
npm run preview    # Preview production build locally
npm run lint       # ESLint check
```

### Deployment
Push to `main` branch triggers GitHub Actions which:
1. Creates `.env` from secrets
2. Runs `npm install && npm run build`
3. Uploads `./dist/` to FTP server root via `SamKirkland/FTP-Deploy-Action@v4.3.5`

---

## 9. Business Logic

### Prayer Times
- Fetched from `https://api.aladhan.com/v1/timings/{day}-{month}-{year}?latitude=-5.1477&longitude=119.4327&method=20`
- Falls back to hardcoded defaults if API fails
- A prayer is "active" for 20 minutes after its start time
- Display shows either active prayer OR currently running activity (prayer takes priority)
- Time zone: Asia/Makassar (WITA, UTC+8)

### Reservation/Booking Conflict Detection
- `BookingForm` fetches approved reservations + active activities
- Detects time slot overlap using date-fns `isWithinInterval` + `addMinutes`
- Prevents submitting if selected date+time overlaps existing booking
- Time options: 05:00–23:00 in 30-minute intervals

### Attendance System (QR-Based)
- Admin creates `attendance_sessions` with a unique `qr_token` (crypto.randomUUID)
- QR code encodes URL: `https://masdik.iqis.sch.id/absen/{qr_token}`
- Two scan types: `arrival` (kedatangan) and `completion` (selesai)
- **GPS validation**: participant must be within **500 meters** of mosque (lat: -5.0930015, lng: 119.5287606) — uses Haversine formula
- **Duplicate prevention**: 
  1. DB unique constraint on `(session_id, device_fingerprint)`
  2. Cookie `attendance_{sessionId}=1` set for 30 days
  3. Device fingerprint stored in localStorage (`attendance_device_id`)
- **Completion scan requirement**: for `kajian`/`rapat` activities, must have completed arrival scan first (checked by device fingerprint)
- **Expiry**: arrival scans are rejected after `event_end_time` has passed
- Feedback is required (minimum 10 characters) for all submissions
- Name is saved to localStorage for convenience

### Financial Transparency
- `SaldoSection` computes balance as `sum(income) - sum(expenses)` client-side
- All transactions are publicly readable (no auth required)
- Donation: BSI account 7301136287, a.n. Msjd Pendidikan Ibnul Qayyim
- QRIS image displayed as static asset (`/src/assets/qris-donasi.jpg`)

### Admin Dashboard (Admin.tsx)
Tabs:
1. **Kegiatan** — Add/delete activities; link to AttendanceManagement per activity
2. **Reservasi** — View/approve/reject reservations with conflict check; filter by status/date
3. **Keuangan** — Add income/expense transactions; displays total balance
4. **Pustaka** — Delegates to `PustakaManager` component (add/delete library items by URL)
5. **Absensi** — Quick access to attendance management (redirect to `/admin/absensi/:activityId`)

### DKM Structure
- **Currently hardcoded** in `DKMStructure.tsx` with 17 members
- A `dkm_members` table exists in the DB but is NOT used by this component
- Groups: Pembina, Ketua, Sekretaris, Bendahara, Bidang Keagamaan, Bidang Humas, Bidang Sarana, Bidang Perawatan, Remaja Masjid

---

## 10. Current State

### Implemented & Working
- Full public site (homepage, calendar, layanan, struktur, saldo, pustaka)
- Prayer times with live clock (Asia/Makassar timezone)
- Reservation system (public submit → admin approve/reject)
- Financial ledger with public transparency
- Admin auth (email/password via Supabase)
- Full CRUD for activities, reservations, transactions in admin
- Pustaka library (document/video/audio with external URLs)
- QR attendance system with GPS validation, duplicate prevention, arrival/completion flow
- Charts in AttendanceManagement (BarChart, PieChart via Recharts)
- Automated FTP deploy on push to main

### In Progress / Incomplete
- **e-Taklim** page: shows "Coming Soon" — feature not yet built
- **DKM Structure**: hardcoded data, `dkm_members` table exists but is unused — eventual goal is DB-driven

### Known Issues / Notes
- `DKMStructure.tsx` data is static — any org changes require code edits
- The `.env` file is committed to the repo (contains public anon key which is safe for Supabase clients, but the file is still tracked by git)
- `package-lock.json` has uncommitted changes (shown in git status)
- Supabase types file has `PostgrestVersion: "14.1"` indicating the DB version
- TanStack React Query is imported but components bypass it in favor of direct Supabase calls — if refactoring, consider migrating to use Query for caching
- `activities.type` CHECK constraint in original migration only allowed: `kajian|pengajian|shalat|acara|sosial|reservasi` — later migrations added `rapat` and `daurah` types via ALTER TABLE (check latest migrations for current valid types)

---
> Source: [kdm-developers/masdik-iqis](https://github.com/kdm-developers/masdik-iqis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-15 -->

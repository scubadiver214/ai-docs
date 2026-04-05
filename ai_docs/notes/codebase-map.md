# LCE Menu Admin UI — Codebase Map

**Generated**: 2026-02-25  
**Project**: lce-menu-admin-ui v0.0.1  
**Purpose**: Authoritative reference for the constitution and all speckit agents. Describes every significant file, its role, and how it connects to other parts of the system.

---

## Repository Root

```
lce-menu-admin-ui/
├── app/                          # Next.js App Router entry points
├── src/                          # Application source (components, api, hooks, theme)
├── ai_docs/                      # AI-generated analysis and trace notes
├── .specify/                     # Speckit SDD tooling (templates, memory, scripts)
├── .github/                      # GitHub Copilot agents + prompts
├── .changes/                     # Changie changelog entries
├── .husky/                       # Git hooks (pre-commit)
├── .trunk/                       # Trunk.io linter config
├── middleware.ts                  # NextAuth route protection (redirects to /login)
├── next.config.ts                # Next.js config (standalone output)
├── tsconfig.json                 # TypeScript strict config (@/ alias → src/)
├── eslint.config.mjs             # ESLint 9 flat config + Prettier
├── .prettierrc                   # Prettier: singleQuote, 140 printWidth
├── package.json                  # Scripts, deps (see below)
└── .env.local.example            # Required env vars template
```

---

## `app/` — Next.js App Router

### Root Layout (`app/layout.tsx`)

- Wraps entire app in `<Providers>` (MUI + Theme + Snackbar)
- Sets `<html lang="en" suppressHydrationWarning>`
- Loads Inter font via `next/font/google`
- Defines `metadata` title/description

### Root Page (`app/page.tsx`)

- Immediately redirects to `/admin`
- `export const dynamic = 'force-dynamic'`

### Global Error (`app/global-error.tsx`)

- Top-level error boundary for catastrophic failures

### 404 (`app/not-found.tsx`)

- Custom 404 page

### Login Page (`app/login/page.tsx`)

- Client Component (`'use client'`) — needs form interactivity
- Supports Credentials (username/password) and Azure AD sign-in via `next-auth/react` `signIn()`
- Lives outside `(admin)` route group — no Dashboard shell
- Reads `callbackUrl` from search params to redirect after successful login

### Route Protection (`middleware.ts`)

- Uses NextAuth v4 `withAuth` to protect all `(admin)` routes
- Unauthenticated requests redirect to `/login`
- Matcher: `/(admin)/:path*`

### Admin Route Group (`app/(admin)/`)

The `(admin)` route group segments all admin pages without affecting the URL path.

**`app/(admin)/layout.tsx`**

- Wraps all admin routes with `<Dashboard>` shell (sidebar + AppBar)
- `export const dynamic = 'force-dynamic'`
- Entry point for the entire admin shell

**Feature Routes** (all Server Components by default):

| Route File                              | URL                      | Status                     |
| --------------------------------------- | ------------------------ | -------------------------- |
| `app/(admin)/admin/page.tsx`            | `/admin`                 | Stub — placeholder content |
| `app/(admin)/admin/loading.tsx`         | `/admin` loading         | Suspense fallback          |
| `app/(admin)/admin/error.tsx`           | `/admin` error           | Error boundary             |
| `app/(admin)/organizations/page.tsx`    | `/organizations`         | Stub — placeholder content |
| `app/(admin)/organizations/loading.tsx` | `/organizations` loading | Suspense fallback          |
| `app/(admin)/organizations/error.tsx`   | `/organizations` error   | Error boundary             |
| `app/(admin)/stores/page.tsx`           | `/stores`                | Stub — placeholder content |
| `app/(admin)/stores/loading.tsx`        | `/stores` loading        | Suspense fallback          |
| `app/(admin)/stores/error.tsx`          | `/stores` error          | Error boundary             |

### Auth API Route (`app/api/auth/[...nextauth]/route.ts`)

- NextAuth.js v4 catch-all handler
- Imports `authOptions` from `src/api/auth/authOptions.ts`
- Exports `GET` and `POST` handlers

---

## `src/api/` — Data Fetching Layer

All API requests use the native `fetch` API. Server Components call pure fetch functions directly.
Client Components use TanStack Query hooks that wrap the same pure fetch functions for automatic
caching, deduplication, and background refetching.

### `src/api/shared/api.ts`

```typescript
export const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL ?? 'http://localhost:5000';
```

- **Single source of truth** for the backend API base URL
- Used by all fetch functions (server-side and client-side via TanStack Query hooks)
- Default env: `http://localhost:8081` (per `.env.local.example`)

### `src/api/auth/authOptions.ts`

- NextAuth `NextAuthOptions` configuration
- **Providers**: `CredentialsProvider` (username/password — TODO: wire to API) + `AzureADProvider`
- **Session strategy**: JWT
- **Sign-in page**: `/login`
- **Required env vars**: `AZURE_AD_CLIENT_ID`, `AZURE_AD_CLIENT_SECRET`, `AZURE_AD_TENANT_ID`, `NEXTAUTH_SECRET`

> ⚠️ `CredentialsProvider.authorize()` is a stub — returns `null`. Needs real API call.

---

## `src/components/` — UI Components

### `src/components/dashboard/Dashboard.tsx`

**The main application shell.** Client Component (`'use client'`).

- Permanent sidebar (desktop) / temporary drawer (mobile)
- `navigationItems` array — **update this when adding routes**:
  ```typescript
  { key: 'admin', label: 'Administration', route: '/admin', icon: <ShieldIcon /> }
  { key: 'organizations', label: 'Organizations', route: '/organizations', icon: <BusinessIcon /> }
  { key: 'stores', label: 'Stores', route: '/stores', icon: <StoreIcon /> }
  ```
- AppBar with breadcrumbs, dark/light mode toggle, settings icon
- Profile avatar menu (hardcoded "John Doe" — needs real auth integration)
- Uses `useResponsive`, `useThemeMode`, `usePathname`, `useRouter`
- Layout constants from `@/constants/layout` (`LAYOUT.sidebar.*`, `LAYOUT.appBar.*`)

### `src/components/providers/Providers.tsx`

Root provider wrapper (`'use client'`). Stack:

1. `QueryClientProvider` (TanStack Query — client created via `makeQueryClient()` in `src/lib/query-client.ts`)
2. `AppRouterCacheProvider` (MUI v7 Next.js integration — emotion cache)
3. `ThemeProvider` (custom MUI theme + dark/light)
4. `MuiSnackbarProvider` (global snackbar context)
5. `ReactQueryDevtools` (dev only — rendered outside the provider tree inside `QueryClientProvider`)

---

## `src/lib/` — Shared Utilities

### `src/lib/query-client.ts`

```typescript
export function makeQueryClient(): QueryClient;
```

- Factory function for `QueryClient` with project-wide defaults:
  - `staleTime: 60s`, `gcTime: 5min`, `retry: 1`, `refetchOnWindowFocus: false`
- Called inside `useState(() => makeQueryClient())` in `Providers.tsx` so each render tree gets its own instance (SSR-safe)

---

## `src/hooks/` — Custom Hooks

### `src/hooks/useMuiSnackbar.tsx`

- `MuiSnackbarProvider`: context provider — wraps app in `Providers.tsx`
- `useMuiSnackbar()`: hook for client components
  ```typescript
  const { showSnackbar } = useMuiSnackbar();
  showSnackbar('Message', 'success' | 'error' | 'warning' | 'info');
  ```
- Auto-hides after 4000ms
- **This is the ONLY approved notification mechanism**

### `src/hooks/useResponsive.ts`

- `useResponsive()` → `{ isMobile, isTablet, isDesktop, breakpoint }`
- MUI breakpoints: xs(0) / sm(600) / md(900) / lg(1200) / xl(1536)
- `isMobile`: < 900px | `isTablet`: 900–1199px | `isDesktop`: ≥ 1200px

---

## `src/theme/` — MUI Theme System

### `src/theme/theme.ts`

- `createAppTheme(mode: PaletteMode)` → MUI theme
- **Brand color**: `#FF6B00` (orange) — used as primary + info
- Custom `sidebar` palette extension (`main`, `contrastText`, `border`)
- Component overrides: Button, Card, Paper, Table, TextField, Drawer, Menu, Chip, Tabs, Switch, etc.
- Typography: Inter font, responsive heading sizes
- Shape: `borderRadius: 12`

### `src/theme/ThemeProvider.tsx`

- Stores mode in `localStorage` under key `pizza-box-admin-theme-mode`
- Uses `useSyncExternalStore` for cross-tab sync
- Server snapshot defaults to `'light'`
- Provides `ThemeContext` with `{ mode, toggleTheme, setMode }`

### `src/theme/themeContext.ts`

- `ThemeContext` React context definition

### `src/theme/useThemeMode.ts`

- `useThemeMode()` → `{ mode, toggleTheme, setMode }` (consumes ThemeContext)

---

## `src/constants/layout.ts`

```typescript
LAYOUT.sidebar.expandedWidth = 280;
LAYOUT.sidebar.collapsedWidth = 88;
LAYOUT.appBar.height = 64;
LAYOUT.content.maxWidth = 1600;
LAYOUT.content.padding = { xs: 2, sm: 3, md: 4 };
LAYOUT.modal.maxWidth = { xs: '95vw', sm: 600, md: 900, lg: 1200 };
LAYOUT.table.minWidth = 650;
LAYOUT.touchTarget.minSize = 44;
```

---

## `.specify/` — Speckit SDD Tooling

| Path                                          | Purpose                                   |
| --------------------------------------------- | ----------------------------------------- |
| `.specify/memory/constitution.md`             | **This project's governing constitution** |
| `.specify/templates/constitution-template.md` | Source template for constitution          |
| `.specify/templates/spec-template.md`         | Feature spec template                     |
| `.specify/templates/plan-template.md`         | Implementation plan template              |
| `.specify/templates/tasks-template.md`        | Task list template                        |
| `.specify/templates/checklist-template.md`    | Checklist template                        |
| `.specify/scripts/bash/create-new-feature.sh` | Scaffold new feature structure            |
| `.specify/scripts/bash/setup-plan.sh`         | Initialize plan for a spec                |

---

`.github/` — AI Agent Configuration

| Path                                  | Purpose                           |
| ------------------------------------- | --------------------------------- |
| `.github/copilot-instructions.md`     | GitHub Copilot enforced rules     |
| `.github/agents/speckit.*.agent.md`   | All Speckit SDD agent definitions |
| `.github/prompts/speckit.*.prompt.md` | All Speckit SDD prompt files      |

---

## Configuration Files

### `next.config.ts`

```typescript
const nextConfig: NextConfig = { output: 'standalone' };
```

- Standalone output for containerized deployments

### `tsconfig.json`

- `strict: true`, `target: ES2017`
- Path alias: `@/*` → `./src/*`
- Module resolution: `bundler` (Turbopack compatible)

### `eslint.config.mjs`

- ESLint 9 flat config
- Rules: `no-console: warn`, `@typescript-eslint/no-explicit-any: off` (constitution overrides this — no `any` is enforced by convention)
- Prettier integration via `eslint-plugin-prettier`

### `.prettierrc`

```json
{ "singleQuote": true, "tabWidth": 2, "printWidth": 140, "trailingComma": "es5" }
```

### Pre-commit Hook (`.husky/pre-commit`)

- Runs `lint-staged` → Prettier formats all staged `*.{js,jsx,ts,tsx,mjs,mts,json,css,md}`

---

## Environment Variables

| Variable                          | Required | Description                                             |
| --------------------------------- | -------- | ------------------------------------------------------- |
| `NEXT_PUBLIC_API_BASE_URL`        | Yes      | Backend API base URL (default: `http://localhost:8081`) |
| `NEXT_PUBLIC_ENVIRONMENT_VERSION` | No       | App version label                                       |
| `AZURE_AD_CLIENT_ID`              | Auth     | Azure AD app registration client ID                     |
| `AZURE_AD_CLIENT_SECRET`          | Auth     | Azure AD client secret                                  |
| `AZURE_AD_TENANT_ID`              | Auth     | Azure AD tenant ID                                      |
| `NEXTAUTH_SECRET`                 | Auth     | JWT signing secret                                      |

---

## Key Dependencies

| Package                              | Version  | Role                                    |
| ------------------------------------ | -------- | --------------------------------------- |
| `next`                               | ^16.1.6  | Framework (App Router, Turbopack)       |
| `react` / `react-dom`                | ^19.2.4  | UI runtime (React Compiler enabled)     |
| `@tanstack/react-query`              | ^5.90    | Data fetching, caching, server state    |
| `zustand`                            | ^5.0     | Lightweight client state management     |
| `@mui/material`                      | ^7.3.8   | Primary UI library                      |
| `@mui/icons-material`                | ^7.3.8   | Icon set                                |
| `@mui/material-nextjs`               | ^7.3.8   | MUI + Next.js SSR integration           |
| `@mui/x-date-pickers`                | ^8.27.2  | Date/time picker components             |
| `@emotion/react` + `@emotion/styled` | ^11      | MUI styling engine                      |
| `next-auth`                          | ^4.24.13 | Authentication (Azure AD + Credentials) |
| `@monaco-editor/react`               | ^4.7.0   | Code editor component                   |
| `tailwindcss`                        | ^4 (dev) | Supplementary CSS utility               |
| `typescript`                         | ^5 (dev) | Language                                |

---

## Data Flow

### Server Component → Client Component (SSR path)

```
Browser Request
    ↓
app/(admin)/layout.tsx           [Server Component]
    → <Dashboard>                [Client Component — shell/nav]
    ↓
app/(admin)/<feature>/page.tsx   [Server Component — async data fetch]
    → await fetchXxx()           [src/api/<feature>/api.ts — pure fetch]
    → <XxxClientComponent>       [src/components/dashboard/<feature>/]
    ↓
Mutation via Server Action       [src/api/<feature>/actions.ts]
    → revalidateTag / revalidatePath
    → useMuiSnackbar feedback    [src/hooks/useMuiSnackbar.tsx]
```

### Client Component → TanStack Query (CSR path)

```
Client Component
    → useFetchXxx() hook         [src/api/<feature>/hooks/useFetchXxx.ts]
    → TanStack Query (useQuery)  [caching, deduplication, background refetch]
    → fetchXxx()                 [src/api/<feature>/api.ts — pure fetch]
    → Backend API
    ↓
Mutation via useMutation         [TanStack Query]
    → mutateXxx()                [src/api/<feature>/api.ts — pure fetch]
    → invalidateQueries          [TanStack Query cache invalidation]
    → useMuiSnackbar feedback    [src/hooks/useMuiSnackbar.tsx]
```

---

## Authentication Flow

```
User visits /admin
    ↓
middleware.ts                     [NextAuth withAuth — redirects to /login if no session]
    ↓
NextAuth.js session check        [app/api/auth/[...nextauth]/route.ts]
    → authOptions                [src/api/auth/authOptions.ts]
    → AzureAD provider OR
    → Credentials provider       [TODO: wire to backend API]
    ↓
JWT session stored in cookie
    → session.strategy: 'jwt'
    → signIn page: /login        [app/login/page.tsx]
```

---

_This map is the source of truth for all speckit planning and agent context. Update when adding routes, components, or architectural changes._

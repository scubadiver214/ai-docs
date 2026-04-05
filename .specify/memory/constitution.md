<!--
SYNC IMPACT REPORT
==================
Version change: 1.0.0 → 1.1.0
Added sections:
  - Core Principle VII: Responsive Design (NON-NEGOTIABLE)
Modified principles: None
Removed sections: None
Templates reviewed:
  ✅ .specify/templates/constitution-template.md — generic template, no update needed
  ✅ .specify/templates/plan-template.md — Constitution Check is dynamically generated, no update needed
  ✅ .specify/templates/spec-template.md — no conflicts found
  ✅ .specify/templates/tasks-template.md — no conflicts found
  ✅ .specify/templates/checklist-template.md — no conflicts found
  ✅ .specify/templates/agent-file-template.md — no conflicts found
  ✅ .specify/scripts/ — no principle references found, no update needed
Codebase map written to: ai_docs/notes/codebase-map.md
Deferred TODOs:
  - NEXTAUTH_SECRET and Azure AD env vars must be set before auth is wired
  - CredentialsProvider.authorize() is a stub — requires real API call
  - ✅ middleware.ts route protection — implemented (withAuth, matcher: /(admin)/:path*)
  - ✅ /login page — implemented (app/login/page.tsx, Credentials + Azure AD)
-->

# LCE Menu Admin UI Constitution

> **Codebase Map**: [`ai_docs/notes/codebase-map.md`](../../ai_docs/notes/codebase-map.md)  
> **Runtime Guidance**: [`.github/skills/frontend-dev-guidelines/SKILL.md`](../../.github/skills/frontend-dev-guidelines/SKILL.md)  
> **Copilot Rules**: [`.github/copilot-instructions.md`](../../.github/copilot-instructions.md)

---

## Core Principles

### I. Server Components First (NON-NEGOTIABLE)

All components in `app/` are Server Components by default — no directive required.
`'use client'` MUST only be added when a component genuinely requires interactivity:
state (`useState`/`useReducer`), effects (`useEffect`), event handlers, or browser APIs.
Client components MUST remain as small and leaf-level as possible.

**Forbidden patterns**:

- `useEffect` + `fetch` for data loading inside client components
- Fetching data inside components that are not Server Components
- Passing `'use client'` to a component just to avoid an async boundary

**Rationale**: Server Components eliminate client-side JavaScript for data display, reduce First
Contentful Paint, and keep sensitive logic server-side. This project's data fetching architecture
depends on this boundary being respected — violating it forces unnecessary client bundles and
breaks streaming/Suspense behavior.

**Reference**: [`app/(admin)/`](<../../app/(admin)/>) for all route entry points.

---

### II. API Service Layer (NON-NEGOTIABLE)

All data access MUST be routed through a dedicated service layer — never inline in components.

- **Pure fetch functions** MUST live in `src/api/<feature>/api.ts` (no React dependencies)
- **TanStack Query hooks** MUST live in `src/api/<feature>/hooks/use*.ts` and wrap the pure fetch functions
- **Server Actions** (mutations) MUST live in `src/api/<feature>/actions.ts` with `'use server'` directive
- **TypeScript types** for API responses MUST live in `src/api/<feature>/types.ts`
- **API base URL** MUST come exclusively from `src/api/shared/api.ts` → `API_BASE_URL`
- **Hardcoded URLs** in any component, action, or fetch function are FORBIDDEN
- **Cache invalidation** after mutations MUST use `revalidateTag()` or `revalidatePath()` (server-side) and TanStack Query `invalidateQueries` (client-side)
- **All HTTP requests** MUST use the native `fetch` API (no axios, got, or other HTTP clients)

**Approved data fetching patterns**:

- **Server Components**: Direct `await` of pure fetch functions from `src/api/<feature>/api.ts`
- **Client Components**: TanStack Query hooks (`useQuery`, `useMutation`) that wrap pure fetch functions
- **Mutations**: Server Actions for server-side mutations; TanStack Query `useMutation` for client-side mutations

**Forbidden patterns**:

- Direct `fetch()` calls inside React components (server or client) — use the API service layer
- `process.env.NEXT_PUBLIC_API_BASE_URL` used directly — always import from `src/api/shared/api.ts`
- `useEffect` + `fetch` for data loading inside client components — use TanStack Query hooks
- SWR, Apollo, or any data fetching library other than TanStack Query

**State management**:

- **Server state**: TanStack Query (caching, background refetch, deduplication)
- **Client state**: Zustand for shared/global UI state; `useState`/`useReducer` for local component state

**Rationale**: TanStack Query with native `fetch` provides automatic caching, deduplication,
background refetching, and optimistic updates. Zustand provides lightweight, scalable client-side
state management. Centralizing API access through a service layer enables type safety, consistent
error handling, and API middleware (auth headers, retries) in a single place.

**Reference**: [`src/api/shared/api.ts`](../../src/api/shared/api.ts)

---

### III. TypeScript Strict (NON-NEGOTIABLE)

This project uses TypeScript 5 with `strict: true`. The following MUST be observed:

- **No `any` type** — use `unknown`, proper interfaces, or generics instead
- **Explicit return types** on all exported functions
- **`import type`** for type-only imports: `import type { Store } from '@/api/stores/types'`
- **`params` and `searchParams`** in page components MUST be typed as `Promise<{...}>` and awaited:
  ```typescript
  export default async function StorePage({ params }: { params: Promise<{ id: string }> }) {
    const { id } = await params;
  }
  ```
- **`@/` import alias** MUST be used for all `src/` imports — relative paths are forbidden

**Rationale**: Strict TypeScript prevents runtime errors, documents intent at call sites, and
enables safe refactoring. The `params`/`searchParams` async change is a Next.js 16 breaking
change — violating it causes runtime failures.

**Reference**: [`tsconfig.json`](../../tsconfig.json)

---

### IV. MUI-First Styling

Material UI v7 is the primary and authoritative UI library for this project.

- All styling MUST use the MUI `sx` prop with `SxProps<Theme>` typing
- **Inline styles**: `const styles: Record<string, SxProps<Theme>>` when total style lines < 100
- **Separate file**: `ComponentName.styles.ts` when total style lines ≥ 100
- **MUI Grid v7 syntax**: `<Grid size={{ xs: 12, md: 6 }}>` — the old `xs={12}` prop is FORBIDDEN
- **Theme tokens** MUST be used for colors, not hardcoded hex values inside component files
  (brand tokens are already defined in `src/theme/theme.ts`)
- **`useMuiSnackbar`** MUST be used for all client-side user notifications
  — `react-toastify`, `window.alert()`, and other notification libraries are FORBIDDEN
- **Tailwind CSS v4** is available as a supplement for utility classes when MUI `sx` is insufficient,
  but MUST NOT be mixed with `sx` in the same component

**Rationale**: Consistency in styling prevents component drift and reduces review overhead.
Centralizing on MUI's theme system means dark/light mode, brand colors, and spacing are
maintained from one source of truth (`src/theme/theme.ts`).

**References**: [`src/theme/theme.ts`](../../src/theme/theme.ts), [`src/hooks/useMuiSnackbar.tsx`](../../src/hooks/useMuiSnackbar.tsx)

---

### V. Mutation Test Coverage

Server Actions and utility functions containing business logic MUST have test coverage
before a PR is merged.

- **In scope**: Server Actions (`src/api/<feature>/actions.ts`), pure utility functions,
  data transformation logic
- **Out of scope**: Presentational/rendering components, layout code, theme configuration
- Tests MUST colocate with source files (e.g., `actions.test.ts` next to `actions.ts`)
- Tests MUST be written before implementation is considered complete

**Rationale**: Server Actions are the mutation layer of this application — they directly write to
the backend API and invalidate caches. Untested mutations are a correctness risk that cannot be
caught purely through UI testing.

---

### VI. Simplicity (YAGNI)

Features MUST be built to their minimum viable scope.

- Abstractions MUST NOT be introduced until 2+ concrete use cases require them
- `useCallback` and `useMemo` MUST NOT be added preemptively — React Compiler (stable in
  React 19 / Next.js 16) handles automatic memoization; add manually only if CPU profiling
  shows a specific bottleneck
- New dependencies MUST be justified — no package SHOULD be added to `package.json` without
  a concrete, immediate use case
- `next/dynamic` (lazy loading) SHOULD only be used for heavy client components confirmed by
  bundle analysis

**Rationale**: Every abstraction layer and dependency adds cognitive overhead and maintenance cost.
Given this is an early-stage internal tool, complexity MUST be earned, not anticipated.

---

### VII. Responsive Design (NON-NEGOTIABLE)

All interfaces MUST work seamlessly across desktop, tablet, and mobile devices.

- Components MUST adapt gracefully to different screen sizes using MUI responsive props
  and/or Tailwind CSS utilities
- Touch targets MUST be appropriately sized for mobile interaction (minimum 44×44 px)
- Content MUST remain accessible and readable at all viewport sizes
- Use the existing `useResponsive` hook (`src/hooks/useResponsive.ts`) for breakpoint-aware
  logic — do not create ad-hoc `window.innerWidth` checks
- MUI responsive props (`size={{ xs: 12, md: 6 }}`) and Tailwind breakpoint prefixes
  (`sm:`, `md:`, `lg:`) are the approved mechanisms for responsive layout

**Forbidden patterns**:

- Fixed-width containers that do not reflow on smaller screens
- Hover-only interactions with no touch/keyboard equivalent
- Text or controls that overflow or become unreadable below 375 px viewport width

**Rationale**: This is an operational tool used across locations and devices. Field users may
access it on tablets or phones. A non-responsive interface blocks adoption and creates
support burden.

**Reference**: [`src/hooks/useResponsive.ts`](../../src/hooks/useResponsive.ts)

---

## Technology Stack Constraints

The following stack is locked. Changes require a MAJOR or MINOR constitution amendment.

| Layer             | Technology                      | Version  | Notes                              |
| ----------------- | ------------------------------- | -------- | ---------------------------------- |
| Framework         | Next.js (App Router, Turbopack) | ^16.1.6  | `output: 'standalone'`             |
| Runtime           | React                           | ^19.2.4  | React Compiler enabled             |
| UI Library        | MUI Material + Icons            | ^7.3.8   | Primary — see Principle IV         |
| MUI Next.js       | `@mui/material-nextjs`          | ^7.3.8   | Emotion cache for SSR              |
| MUI Date          | `@mui/x-date-pickers`           | ^8.27.2  | Date/time inputs                   |
| Data Fetching     | TanStack Query                  | ^5.90    | Server state, caching, hooks       |
| Client State      | Zustand                         | ^5.0     | Shared/global UI state             |
| Auth              | NextAuth.js                     | ^4.24.13 | Azure AD + Credentials             |
| Language          | TypeScript                      | ^5       | `strict: true` — see Principle III |
| Styling (primary) | MUI `sx` prop                   | —        | See Principle IV                   |
| Styling (utility) | Tailwind CSS                    | ^4 (dev) | Supplement only                    |
| Linting           | ESLint 9 + Prettier             | —        | Flat config, pre-commit enforced   |
| Git Hooks         | Husky + lint-staged             | —        | Prettier on staged files           |
| Changelog         | Changie                         | —        | `npm run changelog:new`            |
| Dead code         | Knip                            | —        | `npm run knip`                     |

**Forbidden additions** (MUST NOT be added without a constitution amendment):

- SWR, Apollo, or any data fetching library other than TanStack Query
- axios, got, or any HTTP client other than the native `fetch` API
- `react-toastify` or any alternative notification library
- A second component library (shadcn/ui, Chakra, Ant Design, etc.)

---

## Development Workflow

Every new feature route MUST follow this standard structure (see codebase map for full context):

```
app/(admin)/<feature>/
  page.tsx           # Server Component — async data fetch entry point
  loading.tsx        # Suspense fallback (always create)
  error.tsx          # Error boundary ('use client', always create)
  [id]/
    page.tsx         # Dynamic route (if needed)

src/api/<feature>/
  api.ts             # Pure fetch functions (no React dependencies)
  hooks/
    use*.ts          # TanStack Query hooks wrapping fetch functions
    index.ts         # Re-exports
  actions.ts         # Server Actions ('use server', mutations only)
  types.ts           # TypeScript types for API responses

src/components/dashboard/<feature>/
  <Feature>List.tsx  # Client component (uses TanStack Query hooks)
  <Feature>Card.tsx  # Presentation component (props-driven)
```

**When adding a new route**, also:

1. Add to `navigationItems` in [`src/components/dashboard/Dashboard.tsx`](../../src/components/dashboard/Dashboard.tsx)
2. Add `routeToKey` mapping in the same file
3. Add `generateMetadata` export to `page.tsx` for page title/SEO

**Existing hooks** (use these — do not reinvent):

| Hook             | File                           | Use                                 |
| ---------------- | ------------------------------ | ----------------------------------- |
| `useResponsive`  | `src/hooks/useResponsive.ts`   | `isMobile`, `isTablet`, `isDesktop` |
| `useThemeMode`   | `src/theme/useThemeMode.ts`    | Dark/light toggle                   |
| `useMuiSnackbar` | `src/hooks/useMuiSnackbar.tsx` | User notifications                  |

**Layout constants** (use these — do not hardcode pixel values):

- `LAYOUT.sidebar.*`, `LAYOUT.appBar.*`, `LAYOUT.content.*`, `LAYOUT.modal.*`
- Source: [`src/constants/layout.ts`](../../src/constants/layout.ts)

---

## Governance

This constitution supersedes all other project documentation. In any conflict between this file,
README, `.github/copilot-instructions.md`, or agent files, this constitution takes precedence.

**Amendment versioning** (semantic):

- **MAJOR** (x.0.0): Principle removed, redefined, or locked stack item changed
- **MINOR** (x.y.0): New principle added, new mandatory section, new locked stack item
- **PATCH** (x.y.z): Clarifications, wording improvements, typo fixes, example updates

**Amendment process**:

1. Open a PR with the proposed change and a description of the rationale
2. Increment the version per the rules above
3. Update `LAST_AMENDED_DATE` to today
4. Run the speckit constitution command to propagate to dependent templates

**Compliance**:

- All PRs MUST verify that changes comply with all Core Principles before merging
- The Review Agent (`.cursor/agents.md`) enforces these rules during code review
- Violations that cannot be avoided MUST be documented in a `# Complexity Justification` comment

**Runtime guidance files** (consulted by agents, do not supersede constitution):

- `.github/skills/frontend-dev-guidelines/SKILL.md` — detailed implementation patterns
- `.github/copilot-instructions.md` — Copilot-specific enforcement rules
- `ai_docs/notes/codebase-map.md` — authoritative file-level reference

---

**Version**: 1.1.0 | **Ratified**: 2026-02-25 | **Last Amended**: 2026-02-25

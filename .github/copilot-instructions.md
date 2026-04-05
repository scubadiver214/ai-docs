# Copilot Instructions — LCE Menu Admin UI

This file defines **solution-specific, enforceable rules** for Copilot when generating or editing code in this repository. Prefer existing patterns over inventing new ones.

## Tech stack (current)

- **Framework**: Next.js 16 (App Router, Turbopack — default bundler)
- **UI**: React 19.2, Material UI (MUI) v7
- **Data Fetching**: TanStack Query (`@tanstack/react-query`) with native `fetch` API for all API requests; Server Components + Server Actions for SSR
- **Client State**: Zustand (shared/global UI state)
- **Auth**: NextAuth.js v4
- **Styling**: MUI `sx` prop + Tailwind CSS v4
- **Language**: TypeScript (strict)
- **Optimization**: React Compiler (stable — automatic memoization)

---

## Non-negotiable rules

- **No secrets** in code, config, or commits. Use environment variables (`NEXT_PUBLIC_*` for client, server-only env vars for Server Components/Actions).
- **No hardcoded API base URLs**. Use `process.env.NEXT_PUBLIC_API_BASE_URL` (see `src/api/shared/api.ts`).
- **Use TanStack Query + `fetch` for all API requests**. Server Components call pure fetch functions directly; client components use TanStack Query hooks wrapping the same fetch functions. Never use `useEffect` + `fetch` for data loading.
- **No "smart" presentation components**. Keep data-fetching in Server Components or dedicated server-side functions; keep client UI components prop-driven.
- **Do not introduce breaking UI navigation changes** without updating `app/` routes and `src/components/dashboard/Dashboard.tsx` consistently.
- **Prefer minimal, localized changes**. Do not refactor broadly unless explicitly asked.
- **Do not bypass shared utilities**: use the `@/` alias and existing helpers/components instead of duplicating logic.
- **Always `await` `params` and `searchParams`** — they are async `Promise<>` in Next.js 16 (breaking change from v15).
- **No `any` type**. Use proper TypeScript types; use `import type` for type-only imports.

---

## Next.js 16 patterns

### Async route params (breaking change)

`params` and `searchParams` are now `Promise<>` and must be awaited:

```typescript
// app/(admin)/stores/[id]/page.tsx
interface StorePageProps {
  params: Promise<{ id: string }>;
}

export default async function StorePage({ params }: StorePageProps) {
  const { id } = await params;
  const store = await fetchStore(id);
  return <StoreDetail store={store} />;
}
```

### Cache Components (`use cache`)

Use the `use cache` directive for components that benefit from Partial Pre-Rendering (PPR) and instant navigation:

```typescript
'use cache';

export async function ProductList() {
  const products = await fetchProducts();
  return /* ... */;
}
```

### Advanced caching APIs

- `next: { tags: ['tag'] }` on fetch for tag-based revalidation
- `next: { revalidate: 60 }` for time-based revalidation
- `cache: 'no-store'` to opt out of caching
- `revalidateTag()` / `revalidatePath()` in Server Actions after mutations
- `revalidatePath()` for path-based cache invalidation

### React Compiler

React Compiler is stable in this stack. Manual `useMemo`/`useCallback` is usually unnecessary -- the compiler handles optimization automatically. Only add manual memoization if profiling shows a specific need.

### Metadata API

Use `generateMetadata` for dynamic page titles and SEO:

```typescript
export async function generateMetadata({ params }: StorePageProps) {
  const { id } = await params;
  const store = await fetchStore(id);
  return { title: store?.name ?? 'Store Not Found' };
}
```

---

## Architecture & folder boundaries (follow this shape)

### Data Fetching Layer (`src/api/`)

- Put **pure fetch functions** in `src/api/<feature>/api.ts` (no React dependencies, uses native `fetch`).
- Put **TanStack Query hooks** in `src/api/<feature>/hooks/use*.ts` (wrap fetch functions with `useQuery`/`useMutation`).
- Put **Server Actions** in `src/api/<feature>/actions.ts` (with `'use server'` directive).
- Put feature types in `src/api/<feature>/types.ts`; shared types in `src/api/types.ts`.
- **Rules**:
  - Pure fetch functions are framework-agnostic (no React imports). Used by both Server Components and TanStack Query hooks.
  - TanStack Query hooks provide caching, deduplication, and background refetching for client components.
  - Server Actions handle server-side mutations (create, update, delete) with `revalidatePath`/`revalidateTag`.
  - All HTTP requests MUST use the native `fetch` API (no axios or other HTTP clients).
  - Throw on non-OK responses; use `error.tsx` boundaries or try/catch in Server Actions.

### App Router (`app/`)

- **Layouts**: `app/(admin)/layout.tsx` wraps all admin routes with the Dashboard shell.
- **Pages**: `app/(admin)/<feature>/page.tsx` — Server Components by default, fetch data directly.
- **Loading UI**: `app/(admin)/<feature>/loading.tsx` for streaming/Suspense fallbacks (create for every new feature route).
- **Error UI**: `app/(admin)/<feature>/error.tsx` for error boundaries (create for every new feature route).

### UI components (`src/components/`)

- **Server Components** (default): Fetch data, render HTML, pass data to client components.
- **Client Components** (`'use client'`): Interactive UI only — forms, click handlers, local state.
- **Presentation components**: `src/components/dashboard/<feature>/*` — Prop-driven UI only.
- **Shared UI**: `src/components/shared/` — reusable building blocks.

### Theme & global providers (`src/theme/`)

- App is wrapped by `Providers` in `app/layout.tsx` (MUI + Theme).
- Dark/light mode managed by `useThemeMode` hook (`src/theme/useThemeMode.ts`).
- Prefer using theme tokens/colors (`theme.palette.*`, `theme.spacing()`) rather than hardcoding hex values.

---

## Routing & navigation conventions

- Routes use Next.js App Router file-based routing in `app/`.
- The main app shell and navigation live in `src/components/dashboard/Dashboard.tsx`.
- If you add a new page/section:
  - Create `app/(admin)/<feature>/page.tsx` (Server Component).
  - Add `loading.tsx` (Suspense fallback) and `error.tsx` (error boundary, `'use client'`).
  - Update `navigationItems` in `src/components/dashboard/Dashboard.tsx`.
  - Add `generateMetadata` export for page title/SEO.

---

## Middleware & authentication

- NextAuth.js v4 handles authentication (config in `src/api/auth/`).
- Use `middleware.ts` at the project root for route protection, redirects, and request modification.
- Protect admin routes by checking session/token in middleware.

---

## UI library guidance (MUI vs Tailwind)

- **Follow the file's existing style**:
  - If a component is using **MUI**, continue using MUI (`sx`, theme tokens, MUI layout components).
  - If a component is using **Tailwind**, continue using Tailwind utility classes.
- Avoid mixing MUI and Tailwind in the same component unless there's a clear reason.
- **MUI v7 Grid syntax**: `<Grid size={{ xs: 12, md: 6 }}>` (not the old `xs={12} md={6}` props).
- **Inline vs separate styles**: `<100` lines of styles → inline `const styles: Record<string, SxProps<Theme>>`; `>100` lines → separate `.styles.ts` file.

---

## Existing hooks (use these, don't reinvent)

| Hook             | Location                       | Purpose                                                                           |
| ---------------- | ------------------------------ | --------------------------------------------------------------------------------- |
| `useResponsive`  | `src/hooks/useResponsive.ts`   | Responsive breakpoint checks (`isMobile`, `isTablet`, `isDesktop`)                |
| `useThemeMode`   | `src/theme/useThemeMode.ts`    | Dark/light mode toggle                                                            |
| `useMuiSnackbar` | `src/hooks/useMuiSnackbar.tsx` | User notifications in client components (never use `react-toastify` or `alert()`) |

---

## State management rules

- **Server state**: TanStack Query for all client-side data fetching (caching, background refetch, deduplication). Server Components call fetch functions directly.
- **Client state (shared/global)**: Zustand stores for cross-component UI state (e.g., sidebar state, filters, user preferences).
- **Local UI state**: `useState` for simple state; `useReducer` for complex workflows.
- **Forms**: Use `useActionState` with Server Actions for form submissions. Use `react-hook-form` + `zod` only if/when those packages are added to `package.json`.
- **Notifications**: Use `useMuiSnackbar` for client-side user feedback after mutations.

---

## Environment & local setup reminders (do not bake into code)

- Runtime configuration should be provided via environment variables (e.g., `NEXT_PUBLIC_API_BASE_URL`), not hardcoded.
- See `.env.local.example` for required variables.

---

## Code style expectations (match repo)

- Use **TypeScript** (strict) and existing path alias: prefer `@/...` imports.
- Use `import type` for type-only imports.
- Keep formatting consistent with ESLint config:
  - 2-space indentation
  - curly braces required
  - avoid `console.*` (warned by lint)
- Prefer small, readable functions and typed props/interfaces.
- Explicit return types on exported functions.

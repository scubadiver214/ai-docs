# Copilot Instructions

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
- **Private `@lce` feed credentials must stay local-only**. Keep auth only in user-level `~/.npmrc` (or pnpm user config), never in repo files. Use the `pkgs.dev.azure.com/LCE-DIGITAL/CVtoCloud` `lce-cvcloud-npm` feed, and ensure registry `_password` values are base64-encoded PAT values with Azure DevOps `Packaging (Read)` scope.
- **No hardcoded API base URLs**. Use `process.env.NEXT_PUBLIC_API_BASE_URL` (see `src/api/shared/api.ts`).
- **No hardcoded theme or font values**. Use project tokens and theme helpers
  for colors, typography, spacing, radii, shadows, and contrast values unless
  the user or design explicitly requests a one-off override. See "Typography
  and opacity tokens in `sx`" under UI library guidance for the concrete
  `fontWeight`/`fontSize`/opacity token mappings and a nested-selector caveat.
- **No ad-hoc `fontFamily` overrides**. Do not inline a font stack (e.g.
  `'ui-monospace, "SF Mono", Menlo, monospace'` or the bare literal
  `'monospace'`) in `sx`/`styled()`, and do not redeclare a local
  `MONO_FONT_FAMILY`-style constant per file. Body/UI text uses the theme
  default font (no override needed). If a monospace treatment is genuinely
  required (e.g. displaying an ID/code value), import the single shared
  `TYPOGRAPHY.monoFontFamily` token from `src/constants/typography.ts`.
- **Use TanStack Query + the shared HTTP wrappers for all client-side API requests**. Implement pure functions in `src/api/<feature>/api.ts` using **`apiJsonFetch`** (JSON APIs) or **`apiFetch`** (custom requests) from `@/api/shared/api` — **never** raw `fetch()` to `/api/menu-admin/...` or app-resolved API paths from feature code (see `src/api/README.md`). Server Components call those same pure functions; client components use TanStack Query hooks wrapping them. Never use `useEffect` + `fetch` for data loading.
- **No "smart" presentation components**. Keep data-fetching in Server Components or dedicated server-side functions; keep client UI components prop-driven.
- **Do not introduce breaking UI navigation changes** without updating `app/` routes and `src/components/app/Dashboard.tsx` consistently.
- **Body headers are mandatory and centralized**. Every visible page must have exactly one body-header title. Admin routes use the shared shell `PageHeader` in `src/components/app/Dashboard.tsx`, sourced from `ADMIN_DASHBOARD_ROUTE_TITLES`. Non-dashboard pages use `BodyHeader` from `src/components/layout/BodyHeader.tsx`, sourced from `SITE_PAGE_TITLES` / `STANDALONE_PAGE_TITLES` in `src/constants/sitePageTitles.ts`. Do not add duplicate page-level `h1`/`h4` titles directly under the body header.
- **Prefer minimal, localized changes**. Do not refactor broadly unless explicitly asked.
- **Colocate everything a feature owns in that feature's folder**, using the same kebab-case feature name in every layer. See "Feature folder structure" below. Do not scatter feature files into generic buckets.
- **Do not bypass shared utilities**: use the `@/` alias and existing helpers/components instead of duplicating logic.
- **Always `await` `params` and `searchParams`** — they are async `Promise<>` in Next.js 16 (breaking change from v15).
- **No `any` type**. Use proper TypeScript types; use `import type` for type-only imports.
- **Server-side sort and search for all list views**. Every list view grid with sortable columns or a search/filter input must delegate sorting and searching to the API — no client-side sort or filter pipeline on top of server data. Sort field, sort direction, and search term must be part of the TanStack Query key. Changing sort or search must reset pagination to page 1. Sort field mapping (UI column id → API field name) and search normalization (digit-only term → ID filter, text → name filter) belong in the API layer (`src/api/<feature>/api.ts`), not in the component.
- **Consistent rows-per-page options across the app**. Every paginated list/table must use the app-wide rows-per-page options `[10, 25, 50, 100]` with a default of `25`, wired through `usePersistedRowsPerPage` (`src/hooks/usePersistedRowsPerPage.ts`). Import these defaults from `src/constants/pagination.ts` (`DEFAULT_ROWS_PER_PAGE_OPTIONS`, `DEFAULT_ROWS_PER_PAGE`) instead of redefining arrays in feature files. Do not define a feature-specific options array with different values (for example `[10, 20, 50, 100]`) or a different default page size unless explicitly required for that feature. When reviewing or adding a paginated view, verify its rows-per-page options/default match this standard.
- **React component file size limit**: `*.tsx` files that define components must stay at or below 400 lines of code (blank lines and comments are excluded), enforced by ESLint.
- **No magic strings for shared code contracts**. Put query keys, route
  segments, event names, role names, storage keys, feature flags, and repeated
  API path fragments in named constants or feature-level factories (for
  example, `src/api/<feature>/query-keys.ts`) instead of inlining them across
  the codebase.
- **Do not run UI runtime diagnostics unless explicitly requested.** Do not start dev servers, open browser automation, run Playwright/UI smoke tests, capture screenshots, or otherwise test the rendered UI just to gather diagnostics unless the user asks for that specific verification. Static checks that do not launch the UI, such as code diagnostics, lint, typecheck, or focused unit tests, are allowed when appropriate.

## Optional Prompt Modes (Ponytail)

- If a contributor explicitly wants a "caveman" or ultra-minimal implementation style, use Ponytail ultra mode.
- Install commands:
  - `/plugin marketplace add DietrichGebert/ponytail`
  - `/plugin install ponytail@ponytail`
- Use commands:
  - `/ponytail:ponytail ultra` for minimal implementation/refactoring.
  - `/ponytail:ponytail-review` for over-engineering-focused review.

---

## Prompt caching & agent context hygiene

Use prompt caching where supported by the active agent/model and keep reusable context stable so repeated agent turns do not resend the same high-token instructions unnecessarily.

- **Keep stable instructions stable**: repository-wide rules belong in long-lived documents such as this file, `AGENTS.md`, `.cursor/rules/*`, and reusable Skills. Do not restate large rule blocks in every prompt.
- **Order context from stable to volatile**: place durable system/project guidance before task-specific details, diffs, terminal output, timestamps, or user-specific observations. Volatile content should be the smallest possible suffix.
- **Prefer file references over pasted content**: reference paths, symbols, or focused line ranges and let the agent read the file when needed. Paste large files only when the tool cannot access them directly.
- **Use Skills for repeatable workflows**: move recurring agent procedures, review checklists, setup flows, and MCP/tool usage patterns into Skills or rules. Keep ad hoc prompts limited to the current decision or delta.
- **Keep subagent prompts compact**: give subagents the task, relevant file paths, acceptance criteria, and desired output shape. Avoid copying this entire instruction file unless the subagent lacks repository context.
- **Request concise subagent output**: ask for findings, changed files, blockers, or a short implementation summary instead of full transcripts. Preserve detailed logs in tool output rather than chat.
- **Avoid duplicating external docs**: when library/API details are needed, query the relevant documentation tool and summarize only the applicable behavior. Do not paste long vendor docs into prompts or repo instructions.
- **Cacheable examples should be generic**: examples in shared instructions should avoid run-specific IDs, dates, branch names, terminal paths, local URLs, and one-off debugging output.
- **Refresh cached context intentionally**: update stable instruction files when a convention changes, then keep task prompts focused on the new work. Do not layer temporary exceptions into permanent rules.

Good practices for token utilization:

- **Keep only active files open**: close files that are not relevant to the current task so editor context stays focused.
- **Start a fresh chat when switching tasks**: new goals should begin with a small, relevant context window instead of inheriting stale history.
- **Use `/compact` on long conversations**: prune accumulated history before continuing work that still belongs in the same thread.
- **Use `/fork` for alternatives**: explore competing approaches in a fork instead of cluttering the main thread with abandoned paths.
- **Ask for `code only` when explanations are not needed**: skip narrative output for straightforward edits or snippets.
- **Ask for `show only changed lines` when reviewing edits**: avoid regenerating entire files when the delta is enough.
- **Convert large Office documents to Markdown before sharing**: Markdown removes heavy formatting noise and gives agents cleaner structure to reason over.

To preserve cache hits and reduce token churn, avoid unnecessary context resets:

- **Do not switch models halfway through a conversation** unless the task clearly requires different capabilities; model changes usually invalidate cached prompt context.
- **Do not edit system or repository instructions casually**. Treat `AGENTS.md`, `.cursor/rules/*`, Skills, and this file as stable cache anchors; update them only for durable conventions.
- **Do not load unrelated MCP tools**. Discover and call only the servers needed for the current task, because tool schemas and results add high-token context.
- **Do not open a completely different workspace** in the same task unless required; workspace changes replace much of the reusable repository context.
- **Do not start Agent Mode after a long Ask session** without first summarizing the needed task context. Long exploratory chats can carry stale or excessive context into implementation.

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

### Feature folder structure

One feature = one name, reused in every layer, so a single search finds all of it.

- Feature names are **kebab-case** and match the route segment: `app/(admin)/<feature>/`, `src/api/<feature>/`, `src/components/app/<feature>/`.
- **Sub-features nest** under their parent in routes and components (`src/components/app/menu-manager/users/`). The `src/api/` layer stays one level deep and uses the flattened compound name instead (`src/api/menu-manager-users/`).
- **Everything a feature owns lives in its feature folder**: `api.ts`, `types.ts`, `query-keys.ts`, `hooks/`, `actions.ts`, components, `*.styles.ts`, and tests (`*.test.ts(x)` next to the file under test).
- **No feature-named files in generic buckets.** Do not create `src/utils/menuManagerHelpers.ts`, `src/types/storeTransfer.ts`, or similar — put them in the feature folder.
- **Share only after the second consumer.** Move code to `src/components/shared/`, `src/api/shared/`, `src/hooks/`, `src/utils/`, or `src/constants/` when 2+ features actually use it, not in anticipation.
- **Do not add new top-level folders under `src/`** for a single feature.
- When adding a sub-feature, extend the existing feature folder; do not create a sibling top-level feature.
- **Exempt: legacy redirect routes.** A route folder that only calls `redirect()` to preserve an old URL (e.g. `app/(admin)/storelookup/page.tsx` → `/stores?tab=lookup`) keeps its original URL segment and owns nothing else — no `loading.tsx`, `error.tsx`, components, or API folder.

### Data Fetching Layer (`src/api/`)

- Put **pure fetch functions** in `src/api/<feature>/api.ts` (no React dependencies). They MUST call **`apiJsonFetch`** or **`apiFetch`** from `@/api/shared/api` — not raw `fetch()` — so cookies, JSON headers, and client 401/403 handling stay consistent (`src/api/README.md`).
- Put **TanStack Query hooks** in `src/api/<feature>/hooks/use*.ts` (wrap fetch functions with `useQuery`/`useMutation`).
- Put **Server Actions** in `src/api/<feature>/actions.ts` (with `'use server'` directive).
- Put feature types in `src/api/<feature>/types.ts`; shared types in `src/api/types.ts`.
- **Rules**:
  - **`apiJsonFetch`**: default for JSON APIs — sets `Accept: application/json`, and with `json:` sets `Content-Type` + stringified body.
  - **`apiFetch`**: use when the request is not a standard JSON shape but still needs the shared session/proxy behavior.
  - Pure fetch functions are framework-agnostic (no React imports). Used by both Server Components and TanStack Query hooks.
  - TanStack Query hooks provide caching, deduplication, and background refetching for client components.
  - Server Actions handle server-side mutations (create, update, delete) with `revalidatePath`/`revalidateTag`.
  - Under the hood the wrappers use the native `fetch` API (no axios or other HTTP clients).
  - Throw on non-OK responses; use `error.tsx` boundaries or try/catch in Server Actions.
  - Server-only upstream calls that bypass the browser proxy may use **`fetchUpstream`** (`src/api/server/upstreamFetch.ts`) where documented — do not conflate with `apiFetch` in feature `api.ts` without reason.

### App Router (`app/`)

- **Layouts**: `app/(admin)/layout.tsx` wraps all admin routes with the Dashboard shell.
- **Pages**: `app/(admin)/<feature>/page.tsx` — Server Components by default, fetch data directly.
- **Loading UI**: `app/(admin)/<feature>/loading.tsx` for streaming/Suspense fallbacks (create for every new feature route).
- **Error UI**: `app/(admin)/<feature>/error.tsx` for error boundaries (create for every new feature route).

### UI components (`src/components/`)

- **Server Components** (default): Fetch data, render HTML, pass data to client components.
- **Client Components** (`'use client'`): Interactive UI only — forms, click handlers, local state.
- **Presentation components**: `src/components/app/<feature>/*` — Prop-driven UI only.
- **Shared UI**: `src/components/shared/` — reusable building blocks.

### Theme & global providers (`src/theme/`)

- App is wrapped by `Providers` in `app/layout.tsx` (MUI + Theme).
- Dark/light mode managed by `useThemeMode` hook (`src/theme/useThemeMode.ts`).
- Use theme tokens and helpers (`theme.palette.*`, `theme.spacing()`,
  `theme.typography.*`, `theme.shape.*`, `theme.shadows`) rather than
  hardcoding colors, fonts, spacing, radii, shadows, or contrast values.
- If an explicit visual override is required by the user or design, keep the
  literal local and make the reason obvious.

---

## Routing & navigation conventions

- Routes use Next.js App Router file-based routing in `app/`.
- The main app shell and navigation live in `src/components/app/Dashboard.tsx`.
- If you add a new page/section:
  - Create `app/(admin)/<feature>/page.tsx` (Server Component).
  - Add `loading.tsx` (Suspense fallback) and `error.tsx` (error boundary, `'use client'`).
  - Add route labels and visible body-header titles in `src/constants/adminDashboardRoutes.ts`; the dashboard shell renders the title through `PageHeader`.
  - Update navigation in `src/components/app/dashboardNavigation.tsx` when the route should appear in the sidebar.
  - Add `generateMetadata` export for page title/SEO.
- Admin page content should start with the feature surface itself. Use headings inside content only for sections/cards/dialogs, not to repeat the route title already rendered below the breadcrumbs.
- For public or standalone pages outside `app/(admin)`, add the visible title to `src/constants/sitePageTitles.ts` and render it through `BodyHeader`.

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

### Typography and opacity tokens in `sx`

Don't inline numeric `fontWeight`, `fontSize`, `lineHeight`, or `opacity` values in `sx` props. Use these
token mappings instead:

- **`fontWeight`**: use the theme's named string aliases directly in `sx` — `fontWeight: 'fontWeightMedium'`
  (500) or `fontWeight: 'fontWeightBold'` (700). For a value referenced from a non-`sx` context (e.g. a
  `styled()` template, a `keyframes` block, or a plain CSS-in-JS object), use the resolved value instead:
  `theme.typography.fontWeightBold`. Weights with no named token (e.g. 600, 800) are legitimate one-off
  overrides — see the fallback rule below.
- **`fontSize`**: prefer an existing `Typography` `variant` first. If a smaller/larger one-off size is
  required (e.g. a compact badge), reference `theme.typography.<variant>.fontSize` (e.g.
  `theme.typography.caption.fontSize`) rather than a raw `px`/`rem` literal.
- **`opacity`** used only to dim text: prefer a semantic color token (`color: 'text.secondary'` or
  `color: 'text.disabled'`) over an arbitrary `opacity` literal. Reserve numeric `opacity` for genuine
  transient/transition states (loading, hover, disabled overlays), which are not visual "theme values."
- **`lineHeight`**: this theme does not define named line-height tokens. A tighter/looser line-height than
  a variant's default is an accepted one-off override — keep the literal but add a short comment
  explaining why (per the "explicit visual override" fallback below).
- **Fallback for values with no matching token** (e.g. `fontWeight: 800`, most `lineHeight` overrides):
  keep the literal, but add a comment stating why the override is needed, per the "unless the user or
  design explicitly requests a one-off override" carve-out above.
- **Nested-selector caveat**: MUI's `sx` string aliases (`fontWeight: 'fontWeightBold'`) and shorthand
  props (`color: 'text.secondary'`) only resolve on **top-level** `sx`/`slotProps.*.sx` keys. Inside a
  nested selector object (e.g. `'& .MuiChip-label': { fontWeight: 'fontWeightBold' }`), MUI does **not**
  run its style-function resolution — the value is passed straight through as literal CSS, so a string
  alias silently produces invalid CSS. Inside a nested selector, switch `sx` to callback form and use the
  resolved value: `sx={(theme) => ({ '& .MuiChip-label': { fontWeight: theme.typography.fontWeightBold } })}`.

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
- **Forms**: Use `useActionState` with Server Actions for form submissions. `react-hook-form` and `zod` are not in this project — do not add them.
- **Notifications**: Use `useMuiSnackbar` for client-side user feedback after mutations.

---

## Environment & local setup reminders (do not bake into code)

- Runtime configuration should be provided via environment variables (e.g., `NEXT_PUBLIC_API_BASE_URL`), not hardcoded.
- See `.env.local.example` for required variables.
- **App release version** is not an env var. It comes from the committed `VERSION` file and build-time generation (`src/generated/appVersion.ts`). Do not add `NEXT_PUBLIC_ENVIRONMENT_VERSION` or document version in `.env`. See `docs/versioning.md`.
- **Changelog**: use `pnpm run changelog:new` for PR fragments; do not hand-edit `CHANGELOG.md`. Release maintainers use `changelog:batch` and `changelog:merge`.

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

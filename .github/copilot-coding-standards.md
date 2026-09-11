# Copilot Coding Standards

Enforceable coding standards for GitHub Copilot code generation and review in this repository. These complement the architectural rules in [copilot-instructions.md](./copilot-instructions.md).

---

## TypeScript

### Strict Mode

- `strict: true` is enabled in `tsconfig.json`. Never weaken it.
- No `any` type — use `unknown` with type guards when the type is truly unknown.
- Use `import type` for type-only imports:

```typescript
import type { Store } from '@/api/stores/types';
```

- Explicit return types on all exported functions and hooks.
- Prefer `interface` for object shapes; use `type` for unions, intersections, and mapped types.

### Naming Conventions

| Symbol             | Convention                    | Example                          |
| ------------------ | ----------------------------- | -------------------------------- |
| Components         | PascalCase                    | `StoreList`, `CreateStoreForm`   |
| Hooks              | camelCase, `use` prefix       | `useFetchStores`, `useThemeMode` |
| Pure functions     | camelCase                     | `fetchStores`, `formatDate`      |
| Types/Interfaces   | PascalCase                    | `StoreData`, `FetchStoresParams` |
| Constants          | UPPER_SNAKE_CASE              | `API_BASE_URL`, `LAYOUT`         |
| Files (components) | PascalCase                    | `StoreList.tsx`, `Dashboard.tsx` |
| Files (hooks)      | camelCase, `use` prefix       | `useFetchStores.ts`              |
| Files (utilities)  | camelCase                     | `query-client.ts`, `api.ts`      |
| CSS/style files    | Component name + `.styles.ts` | `StoreList.styles.ts`            |

### Import Order

Group imports in this order, separated by blank lines:

1. React / Next.js framework imports
2. Third-party libraries (`@mui/*`, `@tanstack/*`, `zustand`, `i18next`)
3. Project aliases (`@/api/`, `@/components/`, `@/hooks/`, `@/theme/`, `@/constants/`, `@/lib/`)
4. Relative imports (same feature)
5. Type-only imports (`import type`)

```typescript
import { useState, useCallback } from 'react';
import { useRouter } from 'next/navigation';

import { Box, Typography, Button } from '@mui/material';
import { useQuery } from '@tanstack/react-query';

import { API_BASE_URL } from '@/api/shared/api';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

import { StoreCard } from './StoreCard';

import type { Store } from '@/api/stores/types';
```

### Path Aliases

Always use `@/` for imports from `src/`. Never use relative paths that traverse up out of the current feature directory.

```typescript
// Correct
import { API_BASE_URL } from '@/api/shared/api';

// Incorrect
import { API_BASE_URL } from '../../../api/shared/api';
```

---

## React & Next.js 16

### Server Components (Default)

- Components in `app/` are Server Components by default — no directive needed.
- Server Components can `await` async data directly in the body.
- Server Components CANNOT use hooks, event handlers, or browser APIs.
- Pass fetched data as props to client components when interactivity is needed.

### Client Components

- Add `'use client'` as the very first line ONLY when the component requires interactivity (state, effects, event handlers, browser APIs).
- Keep client components small, leaf-level, and prop-driven.
- Structure: Props interface → Hooks → Handlers → Render → Export.

### Async Route Params (Next.js 16 Breaking Change)

`params` and `searchParams` are `Promise<>` and MUST be awaited:

```typescript
interface PageProps {
  params: Promise<{ id: string }>;
}

export default async function Page({ params }: PageProps) {
  const { id } = await params;
  // ...
}
```

### React Compiler

React Compiler is enabled (`reactCompiler: true` in `next.config.ts`). Manual `useMemo`, `useCallback`, and `React.memo` are unnecessary in most cases. Only add manual memoization if profiling shows a specific need.

---

## Data Fetching

### API Layer (`src/api/<feature>/api.ts`)

- Pure `fetch` functions with no React dependencies.
- All HTTP requests MUST use the native `fetch` API — never axios or other clients.
- Use `API_BASE_URL` from `@/api/shared/api` — never hardcode URLs.
- Throw on non-OK responses.

```typescript
export async function fetchStores(): Promise<Store[]> {
  const res = await fetch(`${API_BASE_URL}/api/stores`, {
    next: { tags: ['stores'] },
  });
  if (!res.ok) throw new Error('Failed to fetch stores');
  return res.json();
}
```

### TanStack Query Hooks (`src/api/<feature>/hooks/use*.ts`)

- Wrap pure fetch functions with `useQuery` / `useMutation` / `useSuspenseQuery`.
- Use query key factory pattern per feature.
- Invalidate related queries in `onSuccess` for mutations.

### Server Actions (`src/api/<feature>/actions.ts`)

- `'use server'` directive at the top.
- Use `revalidateTag()` / `revalidatePath()` after mutations.
- Return structured results; throw on failure.

### Forbidden Patterns

```typescript
// NEVER: useEffect for data fetching
useEffect(() => {
  fetch('/api/stores').then(/* ... */);
}, []);

// NEVER: Direct fetch in components
const res = await fetch('http://localhost:5000/api/stores');

// NEVER: axios or other HTTP libraries
import axios from 'axios';
```

---

## State Management

| State Type    | Tool                              | Location                   |
| ------------- | --------------------------------- | -------------------------- |
| Server state  | TanStack Query                    | `src/api/<feature>/hooks/` |
| Shared/global | Zustand                           | `src/stores/`              |
| Local UI      | `useState`/`useReducer`           | Within component           |
| Form state    | `useActionState` + Server Actions | Component + `actions.ts`   |
| Notifications | `useMuiSnackbar`                  | Client components          |

---

## Styling

### MUI (Primary)

- Use MUI `sx` prop for styling. Access theme values via callback: `(theme) => theme.palette.primary.main`.
- Use `SxProps<Theme>` for type-safe style objects.
- MUI v7 Grid syntax: `<Grid size={{ xs: 12, md: 6 }}>` — not the old `xs={12} md={6}` props.
- Inline styles (`const styles: Record<string, SxProps<Theme>>`) when < 100 lines.
- Separate `.styles.ts` file when >= 100 lines.

### Tailwind CSS

- Used alongside MUI for utility styling where appropriate.
- Do not mix MUI `sx` and Tailwind `className` on the same element unless there is a clear reason.
- Match the existing style approach of the file you are editing.

### Theme Tokens

- Prefer theme tokens (`theme.palette.*`, `theme.spacing()`, `theme.typography.*`) over hardcoded values.
- Dark/light mode is handled by `useThemeMode` hook — never hardcode colors that break in dark mode.

---

## File & Folder Organization

### Feature Structure

```
app/(admin)/<feature>/
  page.tsx            # Server Component — fetches and renders
  loading.tsx         # Suspense fallback (required for every feature route)
  error.tsx           # Error boundary, 'use client' (required for every feature route)
  [id]/
    page.tsx          # Dynamic route

src/api/<feature>/
  api.ts              # Pure fetch functions
  hooks/
    useFetch*.ts      # TanStack Query hooks
    index.ts          # Re-exports
  actions.ts          # Server Actions
  types.ts            # Feature types

src/components/dashboard/<feature>/
  FeatureList.tsx     # Client component
  FeatureCard.tsx     # Presentation component
```

### New Feature Checklist

When adding a new feature, ALL of these are required:

1. `app/(admin)/<feature>/page.tsx` — Server Component page
2. `app/(admin)/<feature>/loading.tsx` — Loading fallback
3. `app/(admin)/<feature>/error.tsx` — Error boundary (`'use client'`)
4. `src/api/<feature>/api.ts` — Pure fetch functions
5. `src/api/<feature>/types.ts` — TypeScript types
6. `src/api/<feature>/hooks/` — TanStack Query hooks
7. `src/components/dashboard/<feature>/` — UI components
8. Update `navigationItems` in `src/components/dashboard/Dashboard.tsx`
9. Export `generateMetadata` from the page for SEO

---

## Existing Hooks — Use These, Do Not Reinvent

| Hook             | Location                       | Purpose                                                 |
| ---------------- | ------------------------------ | ------------------------------------------------------- |
| `useResponsive`  | `src/hooks/useResponsive.ts`   | Breakpoint checks (`isMobile`, `isTablet`, `isDesktop`) |
| `useThemeMode`   | `src/theme/useThemeMode.ts`    | Dark/light mode toggle                                  |
| `useMuiSnackbar` | `src/hooks/useMuiSnackbar.tsx` | User notifications (success, error, info)               |

---

## Authentication

- NextAuth.js v4 with JWT strategy.
- Config: `src/api/auth/authOptions.ts`.
- Providers: Azure AD + Credentials.
- Login page: `app/login/page.tsx`.
- Protect admin routes via `middleware.ts`.
- Auth secrets MUST use environment variables (`AZURE_AD_CLIENT_ID`, `AZURE_AD_CLIENT_SECRET`, `AZURE_AD_TENANT_ID`, `NEXTAUTH_SECRET`).

---

## Testing

- **Framework**: Vitest + React Testing Library + jsdom.
- **Test location**: `__tests__/` directories co-located with the code they test.
- **File naming**: `<component>.test.tsx` or `<module>.test.ts`.
- Test files use `vitest/globals` types (no explicit vitest imports needed).
- Prefer `@testing-library/user-event` over `fireEvent`.
- Run tests: `pnpm test` (watch) or `pnpm test:run` (single pass).
- Run coverage: `pnpm test:coverage`.

---

## Linting & Formatting

- **ESLint**: Flat config (`eslint.config.mjs`) with `next/core-web-vitals`, `next/typescript`, and Prettier integration.
- **Prettier**: Enforced via `lint-staged` on commit (Husky pre-commit hook).
- **Key rules**:
  - `curly: [2, 'all']` — braces required on all control structures, even single-line.
  - `no-console: 'warn'` — avoid `console.*` calls; remove before committing.
  - `@typescript-eslint/no-explicit-any: 'warn'` — prefer proper types.
- **Run lint**: `pnpm lint` (check) or `pnpm lint:fix` (auto-fix).
- **Run format**: `pnpm format` (fix) or `pnpm format:check` (check).
- **2-space indentation** throughout the project.

---

## Environment Variables

- Client-accessible variables MUST use the `NEXT_PUBLIC_` prefix.
- Server-only variables (e.g., auth secrets) must NOT have the `NEXT_PUBLIC_` prefix.
- Reference `.env.local.example` for required variables.
- Never commit `.env` files or secrets.

| Variable                   | Scope  | Purpose                     |
| -------------------------- | ------ | --------------------------- |
| `NEXT_PUBLIC_API_BASE_URL` | Client | Backend API base URL        |
| `AZURE_AD_CLIENT_ID`       | Server | Azure AD OAuth client ID    |
| `AZURE_AD_CLIENT_SECRET`   | Server | Azure AD OAuth secret       |
| `AZURE_AD_TENANT_ID`       | Server | Azure AD tenant             |
| `NEXTAUTH_SECRET`          | Server | NextAuth JWT signing secret |
| `NEXTAUTH_URL`             | Server | NextAuth callback URL       |

---

## Package Manager

- **pnpm** is the only allowed package manager. The `preinstall` script enforces this.
- Use `pnpm add <package>` to add dependencies.
- Use `pnpm add -D <package>` for dev dependencies.
- Never commit `package-lock.json` or `yarn.lock`.

---

## Internationalization

- **i18next** + **react-i18next** for translations.
- All user-facing strings should be translatable.
- Use translation keys, not hardcoded strings, for UI text.

---

## Anti-Patterns — Never Do These

| Anti-Pattern                                 | Correct Approach                                 |
| -------------------------------------------- | ------------------------------------------------ |
| `useEffect` + `fetch` for data loading       | TanStack Query hooks or Server Components        |
| Hardcoded API URLs                           | `API_BASE_URL` from `@/api/shared/api`           |
| `axios` or other HTTP clients                | Native `fetch` API                               |
| `any` type                                   | Proper TypeScript types / `unknown` + guards     |
| `console.log` left in production code        | Remove or use proper logging                     |
| `react-toastify` or `alert()`                | `useMuiSnackbar`                                 |
| Relative imports across feature boundaries   | `@/` path alias                                  |
| Smart presentation components                | Keep data fetching in Server Components or hooks |
| Manual memoization without profiling         | Trust React Compiler                             |
| Mixing MUI `sx` and Tailwind on same element | Pick one per element                             |
| Old MUI Grid syntax (`xs={12}`)              | v7 syntax: `size={{ xs: 12 }}`                   |
| Synchronous `params` / `searchParams` access | `await params` (Next.js 16 async Promise)        |

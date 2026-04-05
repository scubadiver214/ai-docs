# Copilot Instructions for LCE Menu Admin UI

This documentation explains the purpose, rules, and maintenance of the solution-specific Copilot instructions file for this repository.

## What is the Copilot Instructions File?

- A structured, enforceable set of rules and templates for GitHub Copilot.
- Ensures all Copilot-generated code aligns with the architecture, boundaries, and standards of the Menu Admin UI.
- Prevents common mistakes, enforces layering, and encodes best practices for this solution.

## Key Rules (Summary)

- **Next.js 16 App Router**: All routes in `app/`, Server Components by default, `'use client'` only for interactivity.
- **TanStack Query + `fetch` for all API requests**: Pure fetch functions in `src/api/<feature>/api.ts`; TanStack Query hooks for client components; Server Components call fetch functions directly.
- **Zustand for shared client state**: Use Zustand stores for cross-component state; `useState`/`useReducer` for local state.
- **Follow solution conventions**: File/folder structure, `@/` import alias, TypeScript strict, MUI v7 styling.
- **Never commit secrets**: All configuration must use environment variables (`NEXT_PUBLIC_*` for client, server-only for Server Components).
- **Minimal, consistent changes**: Prefer existing patterns, avoid broad refactors unless asked.

## How to Keep This File Up to Date

- Update `.github/copilot-instructions.md` whenever architecture, conventions, or patterns change.
- Review after major refactors, new feature additions, or changes to data access, auth, or UI libraries.
- Keep `.github/agents/` agent definitions in sync for consistency across tools.

## Where are the Copilot Files?

- Instructions: [../.github/copilot-instructions.md](../.github/copilot-instructions.md) — Architecture, patterns, and folder conventions.
- Coding Standards: [../.github/copilot-coding-standards.md](../.github/copilot-coding-standards.md) — Enforceable rules for TypeScript, React, styling, testing, linting, and anti-patterns.

## For More Information

- See the project [README](../README.md) for setup and workflow details.
- For unclear rules, prefer conservative best practices and document any new conventions here.

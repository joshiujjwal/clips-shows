# GitHub Copilot Instructions — ClipsShows

## Stack

- **Next.js 14** App Router with TypeScript (strict)
- **Supabase** (PostgreSQL + pgvector + Auth)
- **OpenAI** `text-embedding-3-small` for semantic search embeddings
- **Tailwind CSS** for styling
- **Vitest** + React Testing Library for unit/component tests
- **Playwright** for E2E tests
- Deployed on **Vercel**

## Coding Conventions

- Use `@/` absolute import alias for everything under `src/`
- All server-side code (lib/, API routes) must use `async/await` with explicit error handling
- Return typed `{ data, error }` objects from `lib/` functions — never throw across module boundaries
- Use Supabase typed client (generated types from `supabase gen types typescript`)
- API routes: always validate input with Zod before processing
- Never expose `SUPABASE_SERVICE_ROLE_KEY` to client components — service role is server-only
- Timestamps: store as seconds (int), format as `MM:SS` or `HH:MM:SS` only at render time
- Clip search: always filter `WHERE status = 'approved'` — pending/rejected clips must never appear

## Component Conventions

- Client components: `"use client"` directive at top, minimal — push logic to server components
- `<ClipCard>` is a server component; `<ClipPlayer>` and `<SearchBar>` are client components
- Tailwind only — no inline styles, no CSS modules
- Use `next/image` for all images (show thumbnails, etc.)
- Loading states: use React Suspense + skeleton components, not `isLoading` booleans

## Testing Conventions

- Write the test first, then the implementation
- Mock OpenAI and Supabase in unit tests using `vi.mock()`
- Use `tests/fixtures/clips.ts` for shared test data
- Component tests: test user-visible behavior, not implementation details
- Do not test Tailwind class names — test rendered content and interactions

## Boundaries

- Do NOT refactor working code unless explicitly asked
- Do NOT remove or skip existing tests
- Do NOT add new npm dependencies without noting why in a comment
- Do NOT add `console.log` to production paths (use a proper logger or `console.error` for errors only)
- Do NOT change the DB schema without a corresponding migration file in `supabase/migrations/`

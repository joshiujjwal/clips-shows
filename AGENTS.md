# AGENTS.md — ClipsShows

_Instructions for AI coding agents (OpenAI Codex, GitHub Copilot, etc.)_

---

## Setup

```bash
# Install dependencies
npm install

# Copy env template
cp .env.example .env.local
# Required: OPENAI_API_KEY, SUPABASE_URL, SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY

# Start local Supabase (requires supabase CLI)
supabase start

# Apply schema
supabase db push

# Seed test data
npm run db:seed

# Start dev server
npm run dev
```

---

## Code Style

| Rule | Detail |
|---|---|
| Language | TypeScript strict mode — no `any`, no `as unknown as X` |
| Imports | Absolute imports via `@/` alias (maps to `src/`) |
| Async | Always `async/await`, never raw `.then()` chains |
| Error handling | Return `Result<T, Error>` pattern in `lib/` — never throw across boundaries |
| API responses | Use `NextResponse.json()` with explicit HTTP status codes |
| DB queries | Use typed Supabase client — never raw SQL strings in route handlers |
| Naming | `camelCase` variables, `PascalCase` components/types, `kebab-case` files |
| Comments | Only for non-obvious reasoning — not for what the code does |

---

## Testing

**Framework**: Vitest + React Testing Library + Playwright (E2E)

**Rules**:
- Write tests BEFORE implementation (red/green TDD)
- Test files live in `tests/` mirroring `src/` path (e.g. `src/lib/search.ts` → `tests/lib/search.test.ts`)
- Mock external APIs (OpenAI, Supabase) in unit tests — never hit real services in CI
- Use `tests/fixtures/` for shared seed data
- Integration tests use a separate test DB (`TEST_SUPABASE_URL` env var)
- Every new `lib/` function needs at least: happy path, empty/null input, and one error case

**Commands**:
```bash
npm test                    # All unit + integration tests
npm run test:watch          # Watch mode
npm run test:e2e            # Playwright E2E (needs dev server running)
npm run test:coverage       # Coverage report
```

---

## PR Instructions

Every PR must include:
1. **Test evidence** — paste `npm test` output showing all tests passing
2. **What changed** — bullet list of functional changes (not a diff summary)
3. **Manual testing** — screenshot or search query + result showing the feature works
4. **No test deletions** — if a test is wrong, fix it; don't delete it

Do not:
- Submit PRs that skip the red phase (implement without writing tests first)
- Refactor unrelated code in a feature PR
- Add dependencies without noting why in the PR description
- Leave `console.log` statements in production code paths

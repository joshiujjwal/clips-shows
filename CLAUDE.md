# CLAUDE.md — ClipsShows

_Context for AI coding agents. Keep this file up to date as you learn things._

---

## Quick Commands

```bash
npm run dev          # Start Next.js dev server (port 3000)
npm test             # Run Vitest unit + integration tests
npm run test:e2e     # Run Playwright E2E tests
npm run lint         # ESLint + Prettier check
npm run type-check   # tsc --noEmit
npm run db:push      # Push schema changes to Supabase (via supabase CLI)
npm run db:seed      # Run seed script (loads fixtures into local/test DB)
```

> TODO: Fill in exact commands after Phase 0 scaffold is complete.

---

## Directory Map

```
src/
  app/
    api/
      search/route.ts     # GET /api/search — semantic clip search
      clips/route.ts      # POST /api/clips — submit clip
      clips/[id]/         # approve / reject routes (admin)
    (ui)/
      page.tsx            # Homepage: search bar + results
      admin/page.tsx      # Admin clip queue
  lib/
    db/
      clips.ts            # Clip CRUD (uses Supabase JS client)
      episodes.ts         # Episode CRUD
      shows.ts            # Show CRUD
    embeddings.ts         # OpenAI embedding wrapper (rate-limit aware)
    search.ts             # pgvector cosine similarity query
    ingest.ts             # End-to-end: clip descriptor → DB + embedding
    cache.ts              # LRU cache for search results
  components/
    ClipCard.tsx          # Clip result card
    ClipPlayer.tsx        # YouTube embed with timestamp
    SearchBar.tsx         # Debounced topic search input
  types/
    index.ts              # Show, Episode, Clip, SearchResult types
tests/                    # Mirrors src/ — colocate test with source path
docs/
  spec.md                 # Full feature spec — read before implementing
  adr/                    # Architecture decisions
```

---

## Non-Obvious Conventions

- **Embeddings are generated on ingest, never at query time** — search uses pre-stored vectors.
- **`status` column drives visibility**: only `approved` clips appear in search results. Always filter by `status = 'approved'` in search queries.
- **Timestamps are stored in seconds** (int), not `HH:MM:SS` strings. Convert at the UI layer.
- **YouTube deep-link format**: `https://youtube.com/watch?v={youtube_id}&t={start_sec}s`
- **pgvector operator**: use `<=>` for cosine distance (`1 - similarity`). Lower score = more similar.
- **Supabase RLS** is enabled on all tables. Admin routes must use the service role key (server-side only), never expose it to the client.
- **Environment split**: `SUPABASE_ANON_KEY` for client-side (public), `SUPABASE_SERVICE_ROLE_KEY` for server-side mutations. Never mix these up.
- **Embedding model**: `text-embedding-3-small` → 1536-dimensional vectors. If you change models, you MUST re-embed all existing clips (add a migration script).

---

## Workflow

1. **Read TODO.md** — find the current phase and first unchecked task
2. **Run `npm test`** — confirm baseline is green before touching anything
3. **Write failing tests first** (red) — in `tests/` mirroring the source path
4. **Implement** until tests pass (green)
5. **Run `npm run lint && npm run type-check`** — fix any issues
6. **Review your diff manually** — check for accidental changes
7. **Commit** with a descriptive message
8. **Update this file** if you discovered a non-obvious convention
9. **Check the task off in TODO.md**

---

## What Claude Can't Infer from Code

- The search ranking intentionally prioritizes `score * recency_weight` — don't simplify to score-only.
- Admin approval is done via RLS policy (`is_admin` claim in JWT), not application-level role checks.
- The seed fixtures live in `tests/fixtures/` and are also used by Playwright E2E tests — don't delete them.
- Rate limiting is implemented via `src/lib/ratelimit.ts` using a sliding window, not token bucket.

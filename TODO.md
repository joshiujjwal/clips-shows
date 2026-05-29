# ClipsShows — Task Breakdown

## How to Use This File

Workflow per task:
1. Write tests FIRST (red phase) — no skipping
2. Implement until tests pass (green phase)
3. Review diff manually before committing
4. Commit with a descriptive message
5. Update CLAUDE.md / AGENTS.md if you learned something non-obvious (compound loop)

Each phase is an **evidence gate**: all tasks checked + tests passing + human review before moving on.

---

## Phase 0: Foundation ⬜

- [ ] Init Next.js 14 app with TypeScript and Tailwind (`npx create-next-app@latest`)
- [ ] Configure ESLint + Prettier with project rules
- [ ] Set up Vitest (or Jest) as test framework + write first smoke test
- [ ] Add `.env.example` with all required env var keys (no values)
- [ ] Set up GitHub Actions CI: lint → type-check → test on every push
- [ ] Configure Supabase project: enable pgvector extension
- [ ] Write initial `schema.sql` for `shows`, `episodes`, `clips` tables
- [ ] Verify DB connection in a test (integration test, uses test DB)
- [ ] Review all AI config files (CLAUDE.md, AGENTS.md, copilot-instructions.md)

**Gate**: CI green, DB connection verified, smoke test passing ✅

---

## Phase 1: Data Model & Ingestion ⬜

- [ ] Define TypeScript types for `Show`, `Episode`, `Clip`, `Topic` (in `src/types/`)
- [ ] Write failing tests for `Clip` CRUD operations
- [ ] Implement `src/lib/db/clips.ts` — insert, fetch-by-id, list-by-episode
- [ ] Write failing tests for embedding generation (`src/lib/embeddings.ts`)
- [ ] Implement `embeddings.ts` — wraps OpenAI `text-embedding-3-small`, handles rate limits
- [ ] Write failing tests for the ingest pipeline
- [ ] Implement `src/lib/ingest.ts` — takes a clip descriptor (show, episode, timestamp range, topic tags, synopsis) and stores it with its embedding
- [ ] Seed script: add 10–20 sample clips from 3–5 popular shows as test fixtures
- [ ] Manual test: verify seed data is queryable in Supabase dashboard

**Gate**: All ingest tests green, seed script runs cleanly, fixtures in DB ✅

---

## Phase 2: Semantic Search ⬜

- [ ] Write failing tests for `src/lib/search.ts` (mock DB, test ranking logic)
- [ ] Implement `search.ts` — pgvector cosine similarity query, returns top-N clips with score
- [ ] Add topic expansion: if query is short, expand it with synonyms before embedding (test this)
- [ ] Write failing tests for `GET /api/search?q=...` route handler
- [ ] Implement search API route — validates query param, calls `search.ts`, returns JSON
- [ ] Add result caching (in-memory LRU or Redis) — write test for cache hit/miss behavior
- [ ] Manual test: search for "dealing with failure" → verify relevant clips returned

**Gate**: Search tests green, API responds < 300ms on cached queries, manual results make sense ✅

---

## Phase 3: UI ⬜

- [ ] Write component tests for `<ClipCard>` (renders show name, episode, timestamp, synopsis)
- [ ] Implement `<ClipCard>` component
- [ ] Write component tests for `<SearchBar>` (debounced input, loading state)
- [ ] Implement `<SearchBar>` component
- [ ] Build `src/app/page.tsx` — hero + search bar + results grid
- [ ] Implement `<ClipPlayer>` — YouTube embed with timestamp deep-link or iframe start time
- [ ] Add topic suggestion chips (e.g. "leadership", "failure", "friendship") on homepage
- [ ] Mobile-responsive layout pass with Tailwind
- [ ] Manual test: end-to-end search + play clip in browser

**Gate**: Component tests green, works on mobile, clips play correctly ✅

---

## Phase 4: Curation & Admin ⬜

- [ ] Design clip submission form (show, episode, timestamp, synopsis, topic tags)
- [ ] Write tests for form validation logic
- [ ] Implement `POST /api/clips` — submit new clip (authenticated only)
- [ ] Admin page (`/admin`) — list pending clips, approve/reject
- [ ] Write tests for auth guard on admin routes
- [ ] Implement auth with Supabase Auth (Google OAuth or magic link)
- [ ] Manual test: submit a clip, approve it, verify it shows in search

**Gate**: Auth tests green, submission + approval flow works end-to-end ✅

---

## Phase 5: Polish & Harden ⬜

- [ ] Add error boundaries and loading skeletons to all UI routes
- [ ] Rate-limit `/api/search` (e.g. 20 req/min per IP)
- [ ] Add OpenTelemetry tracing for search latency
- [ ] SEO: meta tags, OG images per search topic
- [ ] Accessibility pass: keyboard nav, ARIA labels on clip cards
- [ ] Load test: simulate 50 concurrent searches, verify < 500ms p95
- [ ] Write E2E tests with Playwright for core search-and-play flow

**Gate**: E2E tests green, perf acceptable, a11y audit passes ✅

---

## Phase 6: Ship ⬜

- [ ] Vercel deployment config (`vercel.json`, env vars)
- [ ] Production DB migrations run cleanly
- [ ] Smoke test in production environment
- [ ] Set up Sentry error tracking
- [ ] Write CHANGELOG.md for v0.1
- [ ] Tag `v0.1.0` release

**Gate**: Production deployment live, smoke test passing, monitoring active ✅

---

## Parking Lot 🅿️

- AI-generated clip synopses (auto-describe a clip given transcript)
- Transcript ingestion pipeline (Whisper API on episode audio)
- Collections / playlists ("clips for my leadership talk")
- Share a clip as a standalone page with OG card
- Browser extension to clip and submit directly from streaming platforms
- Weekly email digest: "top 5 clips on topic X this week"

---

## Lessons Learned 📝

_Update this section as you discover non-obvious things._

- 

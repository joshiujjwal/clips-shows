# ClipsShows — Feature Specification

## Overview

**Problem**: Finding the perfect TV show clip to illustrate a topic (for presentations, teaching, therapy, writing) currently means watching episodes manually or relying on memory.

**Solution**: A searchable database of TV show clips tagged and semantically indexed by topic. Users type a concept ("negotiation under stress", "grief and humor") and get back short timestamped clips from TV shows that illustrate that theme.

---

## Functional Requirements

### Search

- [ ] User can type a free-text topic query (up to 200 chars)
- [ ] Results return within 2 seconds (p95)
- [ ] Results include: show name, episode title/number, timestamp range, a 1–2 sentence synopsis of why this clip matches, and a relevance score
- [ ] Top 10 results shown by default; pagination for more
- [ ] Topic chips on homepage offer one-click topic suggestions
- [ ] Search supports synonyms/expansion (e.g. "boss" → "authority, manager, leadership")

### Clip Playback

- [ ] Each clip links to a YouTube video at the correct start timestamp
- [ ] Clip card shows: thumbnail, show logo, season/episode label, topic tags
- [ ] Clip player opens in a modal or inline embed
- [ ] Timestamp range is shown (e.g. 12:34 – 14:02)

### Clip Submission (Authenticated)

- [ ] Authenticated users can submit a new clip via a form
- [ ] Required fields: show name, episode (S/E), YouTube URL + start/end timestamps, synopsis, topic tags (1–5)
- [ ] Submitted clips go into a pending queue (not immediately searchable)
- [ ] Admin can approve/reject with optional feedback note

### Admin

- [ ] Admin dashboard shows pending clips
- [ ] Admin can approve (makes clip searchable) or reject (with reason)
- [ ] Admin can edit any clip's metadata
- [ ] Admin role is set via Supabase RLS policy (not a UI toggle)

### Auth

- [ ] Sign in with Google OAuth (via Supabase)
- [ ] Anonymous users can search but not submit
- [ ] Session persists across page reloads

---

## Non-Functional Requirements

- [ ] Search latency < 300ms cached, < 2s uncached (p95)
- [ ] Mobile-first responsive design (works on 375px viewport)
- [ ] Accessible: WCAG AA — keyboard navigable, ARIA labels
- [ ] Secure: all mutations authenticated; admin routes server-side guarded
- [ ] Rate limited: 20 search requests/min per IP
- [ ] No PII stored beyond email (Supabase Auth handles it)

---

## Data Model

### `shows`

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| name | text | e.g. "The Office" |
| slug | text UNIQUE | e.g. "the-office" |
| genre | text[] | e.g. ["comedy", "workplace"] |
| cover_url | text | |
| created_at | timestamptz | |

### `episodes`

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| show_id | uuid FK → shows | |
| season | int | |
| episode | int | |
| title | text | |
| youtube_id | text | Base YouTube video ID |
| created_at | timestamptz | |

### `clips`

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| episode_id | uuid FK → episodes | |
| start_sec | int | Start timestamp in seconds |
| end_sec | int | End timestamp in seconds |
| synopsis | text | Why this clip matches the topic |
| topic_tags | text[] | User-supplied topic labels |
| embedding | vector(1536) | OpenAI text-embedding-3-small |
| status | text | 'pending' \| 'approved' \| 'rejected' |
| submitted_by | uuid FK → auth.users | nullable |
| created_at | timestamptz | |

### `search_logs` (analytics)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| query | text | |
| result_count | int | |
| latency_ms | int | |
| created_at | timestamptz | |

---

## API Design

### `GET /api/search`

**Query params**: `q` (required), `limit` (default 10, max 50)

**Response**:
```json
{
  "query": "dealing with failure",
  "results": [
    {
      "clip_id": "uuid",
      "show": "The Office",
      "episode": "S03E14",
      "title": "The Return",
      "start_sec": 742,
      "end_sec": 815,
      "youtube_id": "abc123",
      "synopsis": "Michael Scott publicly fails to win back Ryan and handles it with denial and humor.",
      "topic_tags": ["failure", "denial", "humor"],
      "score": 0.91
    }
  ],
  "cached": false,
  "latency_ms": 187
}
```

### `POST /api/clips`

**Auth**: required

**Body**:
```json
{
  "show_slug": "the-office",
  "season": 3,
  "episode": 14,
  "youtube_id": "abc123",
  "start_sec": 742,
  "end_sec": 815,
  "synopsis": "...",
  "topic_tags": ["failure", "denial"]
}
```

**Response**: `201 Created` with clip object (status: "pending")

### `POST /api/clips/:id/approve` (admin only)

### `POST /api/clips/:id/reject` (admin only, body: `{ reason: string }`)

---

## Test Plan

### Unit Tests

- `embeddings.ts`: mock OpenAI API, assert vector returned, assert retry on rate limit
- `search.ts`: mock pgvector query, assert top-N ranking, assert empty result handling
- `ingest.ts`: mock DB + embeddings, assert full pipeline happy path and error path
- `<ClipCard>`: renders all fields, handles missing thumbnail gracefully
- `<SearchBar>`: debounce fires after 300ms, loading state shown, clears correctly

### Integration Tests

- DB: insert clip → fetch by ID → assert fields match
- Search API: POST seed clip → GET search → assert clip appears in results
- Auth guard: unauthenticated POST /api/clips → assert 401

### Edge Cases

- Empty query string → 400 Bad Request
- Query with no results → empty array, not error
- Clip with no YouTube match → still returned, link gracefully disabled
- Embedding API timeout → fallback to keyword search (or surfaced error)
- Admin approves clip → appears in search within one request (no stale cache)

---

## Open Questions

1. **Video licensing**: Should we link to YouTube only, or also support other sources (Vimeo, Plex)? Start with YouTube-only.
2. **Transcript indexing**: Whisper-based transcript search would be more precise than synopsis-only — is this in scope for v1?
3. **Content moderation**: Who decides if a clip is appropriate? Admin-only for now, community flagging later?
4. **Show/episode seeding**: Manual entry first, or scrape from a TV metadata API (TMDB, TVDb)?
5. **Embedding model**: `text-embedding-3-small` (fast, cheap) vs `text-embedding-3-large` (more accurate) — benchmark before deciding.

# ClipsShows 🎬

> Topic-specific inspiration clips from TV shows

[![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)](.)

---

## What It Does

ClipsShows lets users search for short clips from TV shows by topic (e.g. "dealing with failure", "leadership under pressure", "awkward social situations"). It surfaces the most relevant scene clips, with timestamps, show metadata, and a short explanation of why the clip matches the topic.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Frontend | Next.js 14 (App Router) + TypeScript |
| Styling | Tailwind CSS |
| Backend | Next.js API Routes / Route Handlers |
| Database | PostgreSQL (Supabase) |
| Semantic Search | pgvector + OpenAI embeddings |
| Video Serving | YouTube embed / clip timestamps |
| Auth | Supabase Auth |
| Deployment | Vercel |

---

## Getting Started

```bash
# Clone
git clone https://github.com/YOUR_USERNAME/clips-shows.git
cd clips-shows

# Install dependencies
npm install

# Set up environment
cp .env.example .env.local
# Fill in OPENAI_API_KEY, SUPABASE_URL, SUPABASE_ANON_KEY

# Run dev server
npm run dev

# Run tests
npm test
```

---

## Project Structure

```
clips-shows/
├── src/
│   ├── app/              # Next.js App Router pages & API routes
│   │   ├── api/          # Route handlers (search, clips, ingest)
│   │   └── (ui)/         # Page components
│   ├── lib/              # Shared utilities (db, embeddings, search)
│   ├── components/       # React UI components
│   └── types/            # TypeScript type definitions
├── tests/                # Mirrors src/ — unit + integration tests
├── docs/
│   ├── spec.md           # Feature specification
│   └── adr/              # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   └── skills/
├── CLAUDE.md
├── AGENTS.md
└── TODO.md
```

---

## Contributing

- **Write tests first** (red/green TDD) — no implementation without a failing test
- **PRs require evidence**: test output, screenshots, or query results
- **Small focused PRs** — one feature or fix per PR
- **Update CLAUDE.md / AGENTS.md** if you discover a non-obvious convention
- Never remove a test without replacing it with a better one

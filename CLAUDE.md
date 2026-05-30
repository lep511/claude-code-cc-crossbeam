# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

CrossBeam is an AI-powered ADU (Accessory Dwelling Unit) permit assistant for California. It reads architectural plans via vision, interprets city corrections letters, cross-references state and city code, and generates professional response packages. Winner of Anthropic's "Built with Opus 4.6" Global Claude Code Hackathon.

## Development Commands

### Frontend (Next.js 16 / React 19)
```bash
cd frontend
npm install
npm run dev      # starts dev server
npm run build    # production build
npm run lint     # eslint
```

### Server (Express 5 / Cloud Run orchestrator)
```bash
cd server
npm install
npm run dev      # tsx watch mode
npm run build    # tsc compile
npm run start    # run compiled dist/index.js
```

### Local Demo (no infrastructure needed)
```bash
bash scripts/setup-demo.sh   # creates demo-workspace/ with symlinked test data
# Then use Claude Code skills directly — see DEMO.md
```

## Architecture

```
Browser (Next.js on Vercel)
    ↓ API routes + Supabase Realtime subscriptions
Cloud Run Server (Express orchestrator)
    ↓ creates Vercel Sandbox, uploads skills + files, launches agent
Vercel Sandbox (Agent SDK + Claude Opus 4.6 + skills)
    ↓ reads/writes
Supabase (Postgres, Realtime, Storage)
```

**Why three layers:**
- Agent runs take 10–30 minutes — too long for Vercel serverless (max 300s timeout).
- Cloud Run handles PDF→PNG extraction (needs `pdftoppm` + ImageMagick, 4GB+). The sandbox is kept purely AI.
- Vercel Sandbox provides isolated ephemeral execution with filesystem access for the Agent SDK's `claude_code` preset tools.
- Supabase Realtime pushes live status/messages to the frontend without polling.

## Flow Types

Three internal flow types, defined in `server/src/utils/config.ts`:

| Flow | Duration | What It Does |
|------|----------|--------------|
| `city-review` | ~16 min | City plan checker reviews submitted plans, generates draft corrections letter |
| `corrections-analysis` | ~11 min | Contractor side: reads corrections, researches codes, generates clarifying questions |
| `corrections-response` | ~6 min | Phase 2: takes contractor answers + Phase 1 artifacts, generates response package |

The contractor flow is two-phase with a human-in-the-loop pause between analysis and response.

## Two Skill Locations

Skills exist in two places for different contexts:

- **`.claude/skills/`** — Used by Claude Code directly (local development, demo flows, ops). These are Claude Code skills with `SKILL.md` files.
- **`server/skills/`** — Copied into the Vercel Sandbox at runtime. These are the production skills the deployed agent uses. Contains the California ADU skill (28 reference files), city-specific skills (Placentia, Buena Park), and workflow skills.

The sandbox service (`server/src/services/sandbox.ts`) reads skills from `server/skills/` and uploads them to `/vercel/sandbox/.claude/skills/` in the sandbox filesystem.

## Supabase Schema

All tables are in the `crossbeam` schema (not `public`). Always use `.schema('crossbeam')` when querying.

Tables: `projects`, `files`, `messages`, `outputs`, `contractor_answers`. Types are defined in `frontend/types/database.ts`.

Project status lifecycle: `ready` → `processing` / `processing-phase1` → `awaiting-answers` → `processing-phase2` → `completed` | `failed`

## Server Sandbox Lifecycle

The full sandbox lifecycle in `server/src/services/sandbox.ts`:
1. Create sandbox (30min timeout, 4 vCPUs)
2. Install Claude Code CLI + Agent SDK + Supabase client
3. Download project files from Supabase Storage
4. Unpack pre-extracted .tar.gz archives (PNGs from Cloud Run extraction)
5. Copy skills (dynamic based on flow type + city onboarding status)
6. Pre-inject sheet manifest for demo projects
7. Write and run `agent.mjs` in detached mode (connection-resilient)

## Onboarded Cities

Cities with pre-researched, verified skill data skip web search entirely. Defined in `server/src/utils/config.ts` `ONBOARDED_CITIES`. Currently: Placentia, Buena Park. Non-onboarded cities use the `adu-city-research` skill (live web search).

## Environment Variables

**Frontend** — needs `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`

**Server** — needs `ANTHROPIC_API_KEY`, `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `VERCEL_TOKEN`, `VERCEL_TEAM_ID`, `VERCEL_PROJECT_ID`

## CI/CD

- **Frontend:** Auto-deployed by Vercel on push to main.
- **Server:** GitHub Actions workflow (`.github/workflows/deploy-server.yml`) builds Docker image and deploys to Cloud Run on push to `server/**` on main. Cloud Run config: 3600s timeout, 2GB memory, concurrency 1.

## Key Patterns

- The frontend uses a three-tier mode system: Dev Test (scripted fixtures), Judge Demo (pre-seeded projects), and Real mode.
- DevTools panel (`frontend/components/dev-tools.tsx`) allows stepping through agent states with scripted data.
- Agent scripts are generated as template strings in `sandbox.ts` with env vars interpolated, then written as `agent.mjs` in the sandbox.
- The agent uses detached mode with a resilient wait loop (retries every 30s for up to 60 minutes).
- PDF extraction happens on Cloud Run before the sandbox is created. The sandbox receives pre-extracted PNGs only.

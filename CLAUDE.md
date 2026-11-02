# CLAUDE.md — AIGigs

AIGigs is a site that collects AI evaluation and training gigs in one place. Talent browse and apply, companies submit gigs, the admin approves everything, and a Claude-powered helper writes gig posts and answers questions.

**Full plan:** `docs/PLAN.md`. Read it before starting any milestone.

## How to work

- Build one milestone at a time, in the order in `docs/PLAN.md` section 12. Don't start the next milestone until the current one's "Done when" is true.
- At the start of a milestone, write a short plan (files to touch, steps) and wait for my OK.
- Small commits with clear messages. Tick the milestone checkboxes in `docs/PLAN.md` as you finish them.
- Add a short entry to `docs/CHANGELOG.md` at the end of each milestone.
- If something in the plan is unclear or seems wrong, ask. Don't guess.
- Don't add features that aren't in the plan (see "Not in v1").

## Hard rules

- Nothing goes live without admin approval. The gig finder and company form only create `pending` gigs.
- No scraping of sites that forbid it, need a login, or block it in robots.txt. Use the `manual` source type instead.
- The AI never invents pay, deadlines or requirements. Unknown means null + listed in `missing_fields`.
- The Anthropic API key and model names live only in backend env vars. Never hard-code a model name.
- Never commit `.env` or secrets. Keep `.env.example` up to date.
- Tests must not call the live network or the live Claude API. Use saved fixtures and mocks.

## Commands

(Fill these in during M0.)

- Backend dev: `cd backend && uvicorn app.main:app --reload`
- Frontend dev: `cd frontend && npm run dev`
- Backend tests: `cd backend && pytest`
- Frontend tests: `cd frontend && npm test`
- Lint: `ruff check . && black --check .` / `npm run lint`
- Migrations: `alembic revision --autogenerate -m "..."` then `alembic upgrade head`

## Code style

- Python: type hints everywhere, Pydantic v2, small routers, business logic outside route functions.
- TypeScript: strict mode, typed API helpers in `src/api/`, no `any`.
- Tailwind with the tokens from the `design-system` skill. No random hex colors in components.
- Short, clear names. Comments only where the "why" isn't obvious.

## Skills in this repo

- `build-feature`: the steps for building any feature or milestone
- `gig-finder`: rules for the finder (sources, dedupe, approval)
- `ai-helper`: rules for the Claude API features (prompts, JSON, grounding, cost)
- `design-system`: colors, type, spacing, components

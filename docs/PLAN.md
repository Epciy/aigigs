# AIGigs — Build Plan (v1)

> Status: planning. This file is the source of truth for what to build.
> Claude Code: read `CLAUDE.md` first, then this file. Build one milestone at a time, in order.

---

## 1. What AIGigs is

AIGigs is a website that collects AI evaluation and training gigs (RLHF, coding review, writing, language and data work) in one place.

- **Talent** browse gigs, filter them, create a simple profile, and click through to apply.
- **Companies** submit gigs through a form. Nothing goes live until the admin approves it.
- **The admin** reviews gigs from the form and from the gig finder, then approves, edits or rejects them.
- **An AI helper** (Claude API) cleans up gig posts and answers talent questions using only data from the site.

The AIGigs LinkedIn page stays the main marketing channel. LinkedIn posting stays **manual** in v1.

## 2. Scope

### In v1
- Public site: home, gig list with filters, gig detail, talent signup, post-a-gig form, FAQ, about
- Admin dashboard: review queue, gig edit, signups list, basic stats
- Apply-click tracking (with referral links)
- Gig finder: collects candidate gigs from approved sources into the review queue
- AI gig writing helper
- AI chat helper for talent
- Newsletter signup (Beehiiv embed)

### Not in v1 (do NOT build unless asked)
- Talent login or accounts
- Payments or paid listings
- Auto-posting to LinkedIn or any social network
- Anything that publishes a gig without admin approval
- Scraping that breaks a site's terms, needs a login, or ignores robots.txt

## 3. Tech stack

| Part | Choice | Notes |
| --- | --- | --- |
| Frontend | React + Vite + TypeScript + Tailwind | React Router for pages |
| Backend | FastAPI (Python 3.12) | Pydantic v2 |
| ORM | SQLModel | Alembic for migrations |
| Database | SQLite (file on a Railway volume) | Keep queries portable so Postgres is an easy switch later |
| AI | Anthropic Python SDK (Claude API) | Model name from env var, never hard-coded |
| Gig finder | Python module + scheduled job (APScheduler inside the backend, or a Railway cron) | `httpx` + `selectolax` or `beautifulsoup4`; `feedparser` for RSS |
| Newsletter | Beehiiv embed form | No backend needed |
| Hosting | Railway: one service for API, one for frontend (or frontend served as static files by FastAPI) | Persistent volume for SQLite |
| Tests | pytest (backend), Vitest + Testing Library (frontend) | |
| Lint/format | ruff + black (Python), eslint + prettier (TS) | |

## 4. Repo structure

```
aigigs/
├── CLAUDE.md
├── docs/
│   └── PLAN.md
├── .claude/skills/          # project skills for Claude Code
├── backend/
│   ├── app/
│   │   ├── main.py          # FastAPI app, CORS, routers
│   │   ├── config.py        # settings from env vars
│   │   ├── db.py            # engine, session
│   │   ├── models.py        # SQLModel tables
│   │   ├── schemas.py       # request/response models
│   │   ├── auth.py          # admin session
│   │   ├── routers/
│   │   │   ├── gigs.py
│   │   │   ├── talent.py
│   │   │   ├── submissions.py
│   │   │   ├── clicks.py
│   │   │   ├── admin.py
│   │   │   └── ai.py
│   │   ├── finder/
│   │   │   ├── sources.yaml # list of approved sources
│   │   │   ├── runner.py    # runs all sources, dedupes, saves drafts
│   │   │   └── fetchers/    # one fetcher per source type (rss, html, manual)
│   │   └── ai/
│   │       ├── client.py    # Anthropic client wrapper
│   │       ├── prompts.py   # all system prompts in one place
│   │       ├── gig_writer.py
│   │       └── chat.py
│   ├── alembic/
│   ├── tests/
│   └── pyproject.toml
└── frontend/
    ├── src/
    │   ├── pages/
    │   ├── components/
    │   ├── api/             # typed fetch helpers
    │   └── styles/
    ├── index.html
    └── package.json
```

## 5. Data model

All tables have `id` (int, PK), `created_at`, `updated_at`.

### `gig`
| Field | Type | Notes |
| --- | --- | --- |
| title | str | required |
| platform | str | e.g. Mercor, Alignerr, or company name |
| summary | str | 1–2 sentences for cards |
| description | text | full markdown |
| work_type | enum | `rlhf`, `coding`, `writing`, `language`, `data_labeling`, `domain_expert`, `other` |
| skills | list[str] (JSON) | |
| languages | list[str] (JSON) | |
| pay_min, pay_max | float, nullable | |
| pay_unit | enum | `hour`, `task`, `month`, `project` |
| currency | str | default `USD` |
| hours_per_week | str, nullable | free text like "10–20" |
| location | str | default `Remote` |
| countries_allowed | list[str] (JSON), nullable | empty = worldwide |
| apply_url | str | the platform's own link |
| referral_url | str, nullable | used instead of apply_url when set |
| deadline | date, nullable | |
| status | enum | `pending`, `approved`, `rejected`, `closed` |
| source | enum | `admin`, `company_form`, `finder` |
| source_url | str, nullable | where the finder found it |
| fingerprint | str, unique | hash used to skip duplicates |
| approved_at | datetime, nullable | |
| featured | bool | default false |

### `talent`
| Field | Type | Notes |
| --- | --- | --- |
| name | str | |
| email | str, unique | |
| country | str | |
| skills | list[str] (JSON) | |
| languages | list[str] (JSON) | |
| experience_level | enum | `new`, `some`, `experienced` |
| links | JSON | LinkedIn, GitHub, portfolio |
| bio | text, nullable | |
| consent | bool | must be true to save |

### `company_submission`
Raw form data from companies: `company_name`, `contact_email`, `raw_text`, plus a link to the `gig` row created from it (status `pending`).

### `click`
`gig_id`, `used_referral` (bool), `referrer` (str, nullable), `user_agent_hash` (str). No IP addresses stored.

### `finder_run`
`started_at`, `finished_at`, `source_name`, `found`, `new`, `errors` (text).

### `chat_log` (optional, for improving the chat helper)
`session_id`, `question`, `answer`, `gig_ids_used` (JSON). Keep for 30 days, then delete.

## 6. API endpoints

Public:
| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/gigs` | approved, not closed gigs; filters: `q`, `work_type`, `language`, `skill`, `pay_min`, `platform`; pagination |
| GET | `/api/gigs/{id}` | one approved gig |
| POST | `/api/gigs/{id}/click` | log a click, return the URL to send the user to (referral if set) |
| POST | `/api/talent` | create talent profile |
| POST | `/api/submissions` | company submits a gig (creates a `pending` gig) |
| POST | `/api/ai/improve-gig` | AI writing helper for the post-a-gig form (rate limited) |
| POST | `/api/ai/chat` | AI chat helper (rate limited) |
| GET | `/api/meta/filters` | lists of work types, languages, skills for filter dropdowns |

Admin (require admin session):
| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/admin/login` / `logout` | session cookie |
| GET | `/api/admin/gigs?status=pending` | review queue |
| POST | `/api/admin/gigs` | create gig by hand |
| PATCH | `/api/admin/gigs/{id}` | edit fields |
| POST | `/api/admin/gigs/{id}/approve` · `/reject` · `/close` | change status |
| GET | `/api/admin/talent` | list + CSV export |
| GET | `/api/admin/stats` | counts: gigs by status, signups, clicks per gig (last 7/30 days) |
| POST | `/api/admin/finder/run` | run the gig finder now |
| POST | `/api/admin/finder/from-url` | paste one URL → draft gig in the queue |
| GET | `/api/admin/finder/runs` | last runs and errors |

## 7. Frontend pages

| Route | Page | Main parts |
| --- | --- | --- |
| `/` | Home | hero, short "how it works", latest 6 gigs, newsletter form, LinkedIn link |
| `/gigs` | Gig list | search box, filters (work type, language, skill, min pay, platform), gig cards, pagination |
| `/gigs/:id` | Gig detail | all fields, Apply button (calls click endpoint, then opens link), similar gigs |
| `/join` | Talent signup | form with skills and languages pickers, consent checkbox |
| `/post-a-gig` | Company form | free text + fields, "Improve with AI" button, submit |
| `/faq` | FAQ | static content, also used by the chat helper |
| `/about` | About | who runs AIGigs and why |
| `/admin` | Admin | login, review queue, gig editor, talent list, stats, finder runs |
| everywhere | Chat widget | floating button, opens the AI chat helper |

UI rules: mobile first, fast, accessible (labels, focus states, contrast). Design tokens are in the `design-system` skill.

## 8. Gig finder

Goal: save Mina time finding gigs. It **never** publishes anything. It only creates `pending` gigs.

1. `finder/sources.yaml` lists approved sources. Each has `name`, `type` (`rss` | `html` | `manual`), `url`, `platform`, `enabled`, and `notes` on why it's allowed.
2. Before adding any `html` source: check the site's terms and robots.txt. If scraping isn't allowed, use `manual` (paste URLs) instead.
3. Runner, once a day and on demand:
   - fetch each enabled source (timeout 20 s, 1 request every few seconds per domain, clear User-Agent `AIGigsBot/1.0 (+contact email)`)
   - pull out candidate gigs (title, link, any text)
   - build a `fingerprint` from the normalized apply URL (or platform + title) and skip ones that already exist
   - send the text to the AI gig writer to fill the structured fields
   - save as `gig` with `status=pending`, `source=finder`, `source_url`
   - log a `finder_run` row with counts and errors
4. "Paste a URL" in the admin dashboard does the same for one link.
5. If one source fails, log it and keep going with the others.

## 9. AI helper

Use the Anthropic Python SDK. The model name comes from the env var `ANTHROPIC_MODEL`. Use a smaller model (`ANTHROPIC_MODEL_FAST`) for the chat if cost matters. All prompts live in `ai/prompts.py`.

### 9.1 Gig writer (`/api/ai/improve-gig` and finder drafts)
- Input: raw text (and optional fields already filled).
- Output: **JSON only**, validated with Pydantic into the `gig` fields, plus `missing_fields: list[str]` and `warnings: list[str]` (for example "no pay listed", "deadline unclear").
- Never invent pay, deadlines or requirements. Leave unknown fields null and list them in `missing_fields`.
- If the JSON is invalid, retry once, then return a clear error.

### 9.2 Chat helper (`/api/ai/chat`)
- Grounded: before each answer, fetch relevant approved gigs (simple keyword or SQLite FTS5 search) plus the FAQ text, and pass them in the prompt.
- Answers only from that context. If it doesn't know, it says so and points to the gig's apply link or the FAQ.
- Can suggest matching gigs when the user shares skills or languages, and returns `gig_ids` so the UI can show cards.
- Does not give legal, tax or immigration advice. Does not promise hiring or pay.
- Limits: max message length, rate limit per IP (for example 20 messages / 10 minutes), short history (last 6 turns).

### 9.3 Safety and cost
- API key only on the backend, never sent to the frontend.
- Log token usage per request (counts only).
- Add a simple daily spend cap through an env var. When it's reached, return a friendly "chat is resting, try later".

## 10. Security and privacy
- Admin: one admin account. Password hash in env var (`ADMIN_PASSWORD_HASH`, bcrypt). HttpOnly, Secure, SameSite=Lax session cookie. Rate-limit login.
- CORS only for the frontend domain.
- Validate every input with Pydantic. Limit lengths.
- Simple spam protection on public forms: honeypot field + rate limit.
- Talent data: only what the form asks for, consent checkbox, admin-only access. Short privacy page.
- Secrets in `.env`, never committed. Add a `.env.example`.

## 11. Environment variables

```
DATABASE_URL=sqlite:////data/aigigs.db
ANTHROPIC_API_KEY=
ANTHROPIC_MODEL=
ANTHROPIC_MODEL_FAST=
AI_DAILY_TOKEN_CAP=
ADMIN_EMAIL=
ADMIN_PASSWORD_HASH=
SESSION_SECRET=
FRONTEND_ORIGIN=http://localhost:5173
FINDER_ENABLED=true
FINDER_CRON=0 7 * * *
FINDER_CONTACT_EMAIL=
VITE_API_URL=http://localhost:8000
VITE_BEEHIIV_EMBED_URL=
VITE_LINKEDIN_URL=
```

## 12. Milestones (about 140 hours total)

Each milestone ends with: tests passing, lint clean, a short demo note in `docs/CHANGELOG.md`, and a commit.

### M0 — Setup (≈ 8 h)
- [ ] Monorepo with `backend/` and `frontend/`, lint, format, test commands
- [ ] FastAPI health route, Vite app with Tailwind and router
- [ ] SQLModel + Alembic, first migration
- [ ] `.env.example`, README with run steps
- **Done when:** `make dev` (or two commands) runs both, and tests pass.

### M1 — Gigs core (≈ 25 h)
- [ ] `gig` model + public endpoints + filters + pagination
- [ ] Gig list page, gig card, gig detail page
- [ ] Apply click tracking with referral links
- [ ] Seed script with 10 sample gigs (marked as examples)
- **Done when:** a visitor can find a gig with filters and click Apply, and the click is logged.

### M2 — Admin (≈ 22 h)
- [ ] Admin login and session
- [ ] Review queue, approve / reject / close, gig editor, create gig by hand
- [ ] Stats page
- **Done when:** Mina can add a gig by hand and publish it without touching the database.

### M3 — Talent and companies (≈ 15 h)
- [ ] Talent signup page + endpoint + admin list + CSV export
- [ ] Post-a-gig form creating a `pending` gig
- [ ] Spam protection, privacy page
- **Done when:** both forms work end to end, and company gigs land in the review queue.

### M4 — Home, FAQ, about, newsletter (≈ 10 h)
- [ ] Home page, FAQ, about, Beehiiv form, LinkedIn link, SEO meta tags, sitemap
- **Done when:** the site looks finished on mobile and desktop.

### M5 — Deploy (≈ 8 h)
- [ ] Railway services, volume for SQLite, env vars, custom domain, HTTPS
- [ ] Daily DB backup (copy the file somewhere safe)
- **Done when:** the live URL works, and a deploy is one command or one push.

### M6 — Gig finder (≈ 20 h)
- [ ] `sources.yaml`, fetchers (rss, html, manual URL), runner, dedupe, `finder_run` logs
- [ ] Scheduler + "Run now" + "Paste a URL" in admin
- [ ] Tests with saved sample pages (no live network in tests)
- **Done when:** a run fills the review queue with pending drafts and no duplicates.

### M7 — AI gig writer (≈ 12 h)
- [ ] Anthropic client wrapper, prompts, JSON validation, retry
- [ ] "Improve with AI" on the post-a-gig form and in the admin editor
- [ ] Finder drafts go through the gig writer
- **Done when:** messy text turns into a clean draft with a missing-fields list, and nothing is made up.

### M8 — AI chat helper (≈ 15 h)
- [ ] Search over approved gigs + FAQ (FTS5)
- [ ] Chat endpoint with grounding, limits, spend cap
- [ ] Chat widget with gig cards in answers
- [ ] A small eval set: 20 test questions with expected behavior (answer, "don't know", refuse)
- **Done when:** the chat answers from site data, says "I don't know" when it should, and passes the eval set.

### M9 — Polish (≈ 5 h)
- [ ] Accessibility pass, loading and error states, empty states, 404 page
- [ ] Final README and demo notes

## 13. Open questions (decide before the milestone that needs them)
- [ ] Which sources go into `sources.yaml` first, and which allow automated fetching? (M6)
- [ ] Which platforms have referral links? (M1)
- [ ] CV upload in talent signup, or form only? (M3)
- [ ] Domain name? (M5)
- [ ] Will companies pay to post later? (after v1)

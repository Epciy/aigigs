---
name: build-feature
description: Step-by-step workflow for building any AIGigs feature or milestone from docs/PLAN.md. Use when starting, continuing, or finishing a milestone or feature.
---

# Build a feature

1. **Read** the milestone in `docs/PLAN.md` (section 12) and any sections it depends on (data model, endpoints, pages).
2. **Plan** in a short list: files to create or change, migrations, endpoints, UI parts, tests. Share it and wait for approval.
3. **Backend first**: model + migration → schemas → service logic → router → tests (pytest, using a temp SQLite DB).
4. **Frontend next**: typed API helper in `src/api/` → components → page → tests (Vitest + Testing Library).
5. **Run checks**: backend tests, frontend tests, ruff, black, eslint, type check. Fix everything before going on.
6. **Try it**: run both dev servers and walk through the "Done when" line by hand. Note anything odd.
7. **Finish**: tick the checkboxes in `docs/PLAN.md`, add a `docs/CHANGELOG.md` entry (what changed, how to try it), commit.

## Checklist before saying "done"
- [ ] The "Done when" line of the milestone is true
- [ ] Inputs validated, lengths limited, errors return clear messages
- [ ] Loading, empty and error states in the UI
- [ ] Works on a phone-sized screen
- [ ] No secrets, no hard-coded model names, `.env.example` updated
- [ ] No live network or live Claude API calls in tests

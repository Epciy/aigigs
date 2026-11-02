---
name: gig-finder
description: Rules and design for the AIGigs gig finder (backend/app/finder). Use when adding sources, writing fetchers, changing the runner, dedupe, scheduling, or the admin "Run now" / "Paste a URL" features.
---

# Gig finder

The finder saves the admin time. It **only creates `pending` gigs**. It never approves, publishes or edits live gigs.

## Sources (`backend/app/finder/sources.yaml`)
```yaml
- name: example-rss
  type: rss            # rss | html | manual
  url: https://example.com/jobs.rss
  platform: Example
  enabled: true
  notes: "Public RSS feed, made for reuse."
```
Before adding an `html` source:
1. Read the site's terms of service. If they forbid scraping or automated access, don't add it.
2. Check robots.txt for the path. If it's disallowed, don't add it.
3. Never fetch pages behind a login.
4. If any of these fail, the admin uses "Paste a URL" (`manual`) for that platform instead.
Write the result of the check in `notes`.

## Fetching
- `httpx` with a 20 s timeout, max 1 request every 3 s per domain, max 50 pages per source per run.
- User-Agent: `AIGigsBot/1.0 (+mailto:{FINDER_CONTACT_EMAIL})`.
- One fetcher class per type in `finder/fetchers/`, each returning a list of `CandidateGig(title, url, raw_text, platform)`.

## Runner (`finder/runner.py`)
1. Load enabled sources.
2. For each source: fetch → candidates. Catch errors per source, log them, continue.
3. Fingerprint = sha256 of the normalized apply URL (lowercase host, no tracking params like `utm_*`, `ref`, no trailing slash). If there's no URL: platform + normalized title.
4. Skip candidates whose fingerprint already exists in `gig`.
5. Send `raw_text` to the AI gig writer → structured fields + `missing_fields`.
6. Save as `gig(status="pending", source="finder", source_url=...)`.
7. Write a `finder_run` row: found, new, errors.

## Tests
- Save sample RSS/HTML files in `backend/tests/fixtures/finder/`.
- Mock HTTP (`respx`) and mock the AI gig writer. No live network.
- Test: dedupe, one source failing doesn't stop others, nothing is ever saved as `approved`.

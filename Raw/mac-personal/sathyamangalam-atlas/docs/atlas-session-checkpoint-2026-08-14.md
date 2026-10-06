# Atlas session checkpoint — 14 August 2026

Read this first when picking this back up, on either machine, on either Claude account. This `docs/` folder is the shared source of truth across accounts — Claude project memory is per-account and does not transfer, this repo does.

## Related docs in this folder
- `atlas-data-bundle-integration-2026-08-14.md` — the initial data bundle review + first site update (hardcoded HTML edits)
- `atlas-git-pipeline-2026-08-14.md` — the git repo + reproducible export pipeline build
- `atlas-multipage-redesign-2026-08-14.md` — (later same-day office Mac session) the single page split into nine pages, shared CSS/JS, real CC-licensed media sourced and embedded. Read this if the site structure looks different from what's described below — `site/index.html` is now Home/Overview only, not the whole site.
- `atlas-design-restyle-2026-08-14.md` — (later still, same-day office Mac session) the entire visual design system replaced to match a reference land-trust site: new terracotta/olive/pine/sky-blue palette, Poppins + IBM Plex Sans type, light theme now primary/default (dark derived as a counterpart, not left untouched). Read this if the site's colours/fonts look different from what's described below — the dark-mode-first `--acc:#4ade80` green palette described in earlier docs no longer reflects `site/shared.css`.
- `atlas-institutional-genre-rollout-2026-08-14.md` — (later still, same-day office Mac session) a second, distinct visual genre — institutional/corporate-forestry, initially built Home-only — extended to all 11 pages (`site/institutional.css`, renamed from `home-institutional.css`), plus two new pages (About, Contact). Read this if pages look like a Weyerhaeuser-style institutional site (green nav bar, full-bleed hero photos with breadcrumbs, icon-grids) rather than the terracotta/olive palette described just above — that terracotta palette is now `shared.css`'s base layer only; `institutional.css` is the visible skin on top of it on all 11 pages.

## What happened this session, in order
1. Reviewed the full `atlas.db` data bundle (89 places, 1,893 taxa, 74,900 occurrences, 7,190 documents / 928 tier A+B, 82 claims, 4 conflicts, 30.3% coverage).
2. Updated `sathyamangalam-atlas.html` in place with honest coverage numbers, the 4 disputed figures shown side by side, gazetteer/document gap disclosures.
3. Built this git repository: `data/atlas.db` as source of truth, `scripts/export_from_db.py` as the only way to regenerate `exports/*.json`, `site/index.html` rewritten to fetch data at runtime instead of hardcoding it. Found and fixed a real bug in the process — the places table has 89 rows but every prior export only surfaced 19. Coverage recomputed to 31.5%.
4. Discussed Astro (the static site framework) as a possible future migration. Recommended holding off — the bottleneck is data coverage, not site tooling, and nobody but Vishnu edits the HTML yet.
5. Discussed launch readiness. **Hard blocker: sathyamangalam.org is still not registered** — standing #1 blocker across multiple sessions, unchanged. Soft blockers (30% coverage, 2 disputed figures, robots.txt exemption not in writing, no hosting) are launchable-with-caveats since the site is honest about its own gaps.
6. Vishnu clarified his setup: two Macs (office + personal), two separate Claude accounts. Office git policy: never push from a Claude session running there. Resolution: GitHub repo is the shared source of truth for code, pushed only from the personal Mac; this `docs/` folder inside the repo is the shared source of truth for context, since Claude project memory doesn't cross accounts.

## Immediate next actions (pick up here)
1. **Register sathyamangalam.org** (or decide on an alternative domain) — ~5 minutes, ~$10-15/year. Still the actual next unblocking step.
2. Vishnu creates a GitHub repo (or other host) and pushes this from his personal Mac. Office Mac clones/pulls only, never pushes.
3. Set up hosting — Cloudflare Pages or Netlify, both free-tier, both git-integrated.
4. Two long-standing open decisions: (a) authoritative source for the 4 disputed figures (core area, elephants, leopards, tigers), (b) written confirmation of the robots.txt exemption for 5 API hosts.
5. Astro migration is parked, not rejected — revisit when the HTML becomes unwieldy to hand-edit or someone besides Vishnu starts contributing content.
6. media/community domains in `atlas.db` have no backing tables yet — schema decision needed before next harvest cycle.

## Don't re-ask
- Whether to use Astro now — already answered (no, not yet).
- Whether to push from a Claude session on the office Mac — already answered (no, never).
- Git target — a GitHub repo Vishnu creates and pushes to himself from his personal Mac.

## How to use this repo across two machines/accounts
- **Personal Mac**: clone the repo, this is where you push changes.
- **Office Mac**: clone/pull only. If you make changes here, commit locally and hand the diff/branch to yourself to merge from the personal Mac — never push directly from office.
- **Either Claude account**: point a fresh session at this repo and say "read docs/ first" — that replaces the need for Claude's own project memory to carry context across accounts.

---
tags: project
---
# DEV LOG: araKraft Works website (Claude Code sessions)

Summary: [[Projects/arakraft/SUMMARY]] · State: [[Projects/arakraft/STATE]]

## What was built
- A static one-page site for araKraft Works (laser engraving + CNC workshop, Batticaloa, Sri Lanka): React 19 + TypeScript + Vite 8, pre-rendered, plain CSS.
- Four laser collections with starting prices, CNC gallery, workshop photos, map, WhatsApp order links with product and price filled in.
- 18 Playwright tests (Chrome + WebKit) with accessibility checks.
- Full araCreate conventions setup: file headers, Makefile, VERSION, LICENSE, folder readmes, semantic-release, ANSI banner (`scripts/motd`).
- Cloudflare Pages hosting guide (`docs/cloudflare-hosting.md`).

## Timeline (newest first)
- 2026-10-01 — Compared repo with the conventions repo (about 95%). Moved `public/` into `src/`, added `src/types/`, archived Netlify files. Then moved everything for the website into `src/`; root keeps only Makefile, CHANGELOG.md, VERSION, .gitignore, LICENSE, README.md. Tests pass. Pushed `8eb6596`. Explained SSH commit signing for "Verified" commits.
- 2026-09-28 — Only-check request: Claude made edits by mistake, then undid them. Removed `docs/handoff.md` and pushed. Stopped servers.
- 2026-09-26 — Ran locally. Applied conventions (param-case files, JSON `_meta`, snake_case keys and labels, author Aravinth Panch). Health check clean. Fixed social share image and text. Lighthouse: phone 93 / desktop 100, other scores 100. Removed unused files (public 30 MB → 14 MB). JPEG optimisation with no quality loss. Pushed 4 commits to new private repo. Wrote Cloudflare guide.

## Decisions
- 2026-09-26 — Follow araCreate conventions in full. #decision
- 2026-09-26 — Move hosting from Netlify to Cloudflare Pages; Vishnu does the Cloudflare and domain steps. #decision
- 2026-09-26 — Keep Vite + React (not Astro); deviation written in README. #decision
- 2026-09-26 — Inlining CSS did not help speed (92 vs 93), so it was undone to keep code simple. #decision
- 2026-09-28 — "Check" means report only, no edits. #decision
- 2026-10-01 — Only six files at root; whole website in `src/` (Cloudflare root directory = `src`). #decision
- 2026-10-01 — Verified commits via SSH signing with a new key; no rewrite of old commits. #decision

## State at last session (2026-10-01)
- Code clean and pushed to `aracreate-group/arakraft-works` (`8eb6596`).
- Cloudflare Pages + `arakraft.works` domain not set up yet.
- Commit signing not set up yet.

## Open items
- Cloudflare Pages project, remove old redirect to aracreate.group, connect domain + www.
- Check live site, then remove Netlify leftovers.
- SSH signing key setup.
- Analytics IDs, social links, exact address and hours.
- Optional `llms.txt`, Content-Security-Policy.

## Session index
- [[Projects/arakraft/claude-code/Downloads-arakraft-works__2026-10-01_e1378dbf]] — 2026-10-01 — root structure into `src/`, push, verified commits how-to
- [[Projects/arakraft/claude-code/Downloads-arakraft-works__2026-10-01_ebf9d469]] — 2026-10-01 — empty start (title only)
- [[Projects/arakraft/claude-code/Downloads-arakraft-works__2026-09-26_99f545a6]] — 2026-09-26 to 09-28 — run locally, conventions, health, SEO/share, cleanup, first push, handoff removed

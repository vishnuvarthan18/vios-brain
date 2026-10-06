---
tags: project
updated: 2026-10-06
---
# DEV LOG: SinoLink website (Claude Code sessions)

## What was built
- New "SinoLink Europe" logo on every page of sinolink.de and sinolink.pt (header, footer, loader, favicons, Apple touch icon, alt text). Final version uses the client's own SVG (grey `#7C7E7F`, yellow `#FBB404`), with a white-and-yellow version for the dark footer and loader.
- Home-page preloader script moved from Webflow CDN into the repo so it shows the new logo.
- New Impressum and Privacy pages from the client's document, for both sites and all four languages (EN, DE, PT, CN) — 8 legal pages, added to sitemaps, linked from each language footer.
- Footer changed to "© 2026 SinoLink Europe" everywhere. All personal names, "SinoLink Deutschland" and Dresden removed from served files.

## Timeline (newest first)
- 2026-09-25 — Client asked "no names": removed Achim Neu everywhere, cleaned file header comments and other leftovers. Commit `c804301` to `dev`; Vishnu pushed `main`. All 26 live pages checked: no names.
- 2026-09-25 — Client changes: Impressum shows the .pt email (the .de email stays hidden for the contact form); EU dispute paragraph removed. Commit `a47efa3`; Vishnu pushed `main`; all pages checked live on both domains.
- 2026-09-25 — Legal pages rewritten from client doc (company now Portuguese, old Dresden address removed). Footer → SinoLink Europe.
- 2026-09-23 — Logo work: first a self-made Europe logo, then the client's SVG. Checked every file, then every network request in the browser, to make sure no old logo loads. Commit `f12f703` to `dev`; Vishnu pushed `main`; live on both domains.

## Decisions
- 2026-09-23 — Ask clients for SVG logos (sharp at any size). #decision
- 2026-09-23 — Self-host the Webflow preloader script so the loader uses our logo. #decision
- 2026-09-23 — Screenshots and test pages never go into git. #decision
- 2026-09-23 — Claude pushes to `dev` only; Vishnu pushes `main` (live) himself — Claude Code blocks pushes to `main`. #decision
- 2026-09-25 — Readable email on the legal pages is the .pt one; .de stays hidden for the contact form — company is now Portuguese. #decision
- 2026-09-25 — No personal names anywhere on the site (client rule). #decision
- 2026-09-25 — Minor findings from the full check left as they are ("leave all, lets deploy"). #decision

## State at last session (2026-09-25)
- All changes live on sinolink.de and sinolink.pt; 26 pages verified.
- Unused client file `sinolink-europr.svg` left out of git.

## Open items
- Phone-width layout of the legal page ran off the right edge (checked if old page did the same — not sure if fixed).
- Small issues from the full code check were left on purpose.

## Session index
- [[Projects/sinolink/claude-code/araCreate-SLK-www-sinolink-de__2026-09-25_93aaa3cc]] — 2026-09-25 — Impressum/Privacy for all languages, footer rename, remove names, deploy
- [[Projects/sinolink/claude-code/araCreate-SLK-www-sinolink-de__2026-09-23_3e4642bb]] — 2026-09-23 — SinoLink Europe logo everywhere, deploy

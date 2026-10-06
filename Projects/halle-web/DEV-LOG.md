---
tags: project
updated: 2026-10-06
---
# DEV LOG: b-halle.de website (Claude Code sessions)

## What was built
- Home page hero on b-halle.de (Webflow site `halle-dev`, staging halle-dev.webflow.io), from Figma file "www.b-halle.de".
- Hero curve uses the real Figma SVG asset (`Rectangle (2).svg`, navy `#29308A`); desktop only, mobile is solid navy.
- Hero and trust-logos split into two separate sections to stop the curve covering the logos on wide screens.
- Heading → logos gap set to ~21px on all breakpoints (`logo-container` row gap = 0).
- New nav for "Home v2" built from native Webflow elements (logo, Products▾, About us, Contact, search, Language▾) — v2 was later dropped.
- "Commemorative Publication: 90 Years in the Service of Optics" card made fully clickable with hover animation; download pop-up links to the PDF.
- Type hierarchy audit and a "safe" plan: snap odd font sizes to nearest step, fix line-heights, one `<h1>` per page.
- Server access sessions of 2026-10-05 were moved to [[Projects/feedback-widget/DEV-LOG]] (they were about the feedback app's IONOS server).

## Timeline (newest first)
- 2026-10-05 — (moved) Server login / Kishor SSH key / tunnel to :3000 — see [[Projects/feedback-widget/DEV-LOG]].
- 2026-07-17 — Commemorative card clickable + hover; fixes for wide screens (2107px) and 816–992px; text hierarchy audit and plan (all 16 pages).
- 2026-07-14 — Home v2 attempts (nav rebuilt natively; Figma MCP connection problems). V2 dropped; back to original home. Fixed hero gaps above 1794px, split hero/logos sections, tightened logo gap.
- 2026-07-13 — Hero background to SVG curve; many failed tries with custom code broke the home page; finally used the exact Figma SVG and published.

## Decisions
- 2026-07-13 — Use the real Figma SVG for the curve, not a drawn ellipse. #decision
- 2026-07-14 — Drop Home v2; work on the original home page. #decision
- 2026-07-14 — Hero and logos must be two separate sections. #decision
- 2026-07-14 — Build with native Webflow elements, not custom code, where possible. #decision
- 2026-07-14 — Check every breakpoint and nearby sections before publishing; revert when something else breaks. #decision
- 2026-07-17 — Text sizes: safe path — keep design, only snap off-scale sizes (42→44, 23.7→24, 17/15→16, 13→14, 11→12) and fix line-heights. #decision

## State at last session
- Website: hero, logos and commemorative card done on staging; text hierarchy plan written but not done (as of 2026-07-17).

## Open items
- Carry out the text hierarchy plan (Phase 1–3; optional Phase 4 section headers 40px).

## Session index
- (moved to feedback-widget) [[Projects/feedback-widget/claude-code/home__2026-10-05_38619bdb]], [[Projects/feedback-widget/claude-code/home__2026-10-05_eacda357]] — 2026-10-05 — server login
- [[Projects/halle-web/claude-code/home__2026-07-17_b012888e]] — 2026-07-17 — commemorative card, responsive fixes, type hierarchy plan
- [[Projects/halle-web/claude-code/home__2026-07-14_79a8e2c7]] — 2026-07-14 — v2 dropped, hero gaps >1794px, logo gap
- [[Projects/halle-web/claude-code/home__2026-07-14_36dfe593]] — 2026-07-14 — hero curve wide-screen fix, revert, split hero/logos
- [[Projects/halle-web/claude-code/home__2026-07-14_67726727]] — 2026-07-14 — Home v2 nav, Figma MCP connection help
- [[Projects/halle-web/claude-code/home__2026-07-14_cfcb6aaa]] — 2026-07-14 — new home from Figma, native nav rebuild
- [[Projects/halle-web/claude-code/home__2026-07-13_63996ee1]] — 2026-07-13 to 07-14 — hero SVG curve, home broken and rebuilt, exact Figma curve

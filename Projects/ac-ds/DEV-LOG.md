---
tags: project
---
# DEV LOG: araCreate Design System (ACDS) (Claude Code sessions)

## What was built
- `docs/DESIGN_SYSTEM_PRINCIPLES.md` — reference doc on how to build a design system from a live site (audit → extract → formalize → build), with 20 real sources added. Also published as an HTML page.
- Project folders set up as input / process / output: `01-input/` (two Webflow static exports: "araCreate Template" and "araCreate Website"), `02-process/audit/`, `03-output/design-system/`.
- Both exports run locally with their own `serve.py` (template on :8001, live site on :8002).
- CSS audits of both sites (colors, type, spacing, radius, shadow, motion, breakpoints).
- Design tokens v1 in W3C format, 7 files in `03-output/design-system/tokens/` (color, typography, spacing, radius, shadow, motion, breakpoints) + README.
- Visual token preview page.
- Component docs: `button.md` (live site has only `.cta-button`, 7 variants), card (`.cms-blog-item`, `.cms-projects-items`). Nav, layout, footer, accordion, hero were being built in the background at session end.

## Timeline (newest first)
- 2026-10-06 — Session start: empty folder. Principles doc written, then sourced. Exports moved into `01-input/`, both sites run locally. Audits done. Tokens drafted and locked as v1. Chrome blocked localhost (org policy), so work moved to code-based extraction. Full pixel-exact component extraction (template: 746 components). Found and fixed a pretty-printer bug that dropped first letters of CSS property names; data re-verified. Button and card docs written. Remaining core components started.

## Decisions
- 2026-10-06 — Design first; code comes later from the design system. Reference doc lives in the repo as markdown. #decision
- 2026-10-06 — Gold `#F9BF3B` is the primary brand color; blue `#3347a0` is also core. Palette: gold, blue, ink `#2e2e2e`, white, light gray `#f6f6f6`. #decision
- 2026-10-06 — Font Poppins; use the rem type and spacing scales (px values are noise). Card radius 20px. #decision
- 2026-10-06 — Unused red/green/gold status colors kept as reserved tokens for a planned feature. #decision
- 2026-10-06 — Extract from source code, pixel by pixel, not approximate. #decision
- 2026-10-06 — `.button` and `.cta-button` treated as separate variants; live site only uses `.cta-button`. #decision
- 2026-10-06 — Stop asking per component; follow the principles doc and build all core components, only raise real drift questions. #decision

## State at last session (2026-10-06)
- Tokens v1 done. Button and card docs done and verified.
- Nav, layout, footer, accordion, hero docs running in background.

## Open items
- Red Hat Mono: real part of system or not (only 2 uses)?
- Type/spacing scale needs a visual check against real pages.
- `home-button`/`newsletter` margin drift and `dtf-menu` color inversion in `.cta-button`.
- Template `.button` never reached the live site — keep or drop?
- Chrome blocks localhost, so visual checks need another way (Safari can't be driven).

## Session index
- [[Projects/ac-ds/claude-code/araCreate-AC-ACDS__2026-10-06_09a74d35]] — 2026-10-06 — main session: principles doc, audits, tokens v1, button/card docs
- [[Projects/ac-ds/claude-code/araCreate-AC-ACDS__2026-10-06_2634b7f3]] — 2026-10-06 — title-only: organise folders as input/process/output
- [[Projects/ac-ds/claude-code/araCreate-AC-ACDS__2026-10-06_3f19615a]] — 2026-10-06 — title-only side session
- [[Projects/ac-ds/claude-code/araCreate-AC-ACDS__2026-10-06_aee944f1]] — 2026-10-06 — title-only side session (browser setup)

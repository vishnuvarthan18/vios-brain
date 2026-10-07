# Tokens — v1

Generated from the audit of the live site (`02-process/audit/live-site-css-audit.md`) and template source (`02-process/audit/template-source-css-audit.md`). W3C Design Tokens format (JSON), per `docs/DESIGN_SYSTEM_PRINCIPLES.md`.

## For future agent
These are extracted/normalized from real CSS, not invented. Status: v1, accepted from CSS-audit evidence alone — visual verification against rendered pages was attempted but blocked (see below) and intentionally skipped by stakeholder decision. Safe to build components against. If inconsistencies surface during component work, flag and revisit here rather than silently overriding.

## Status

- **Confirmed by stakeholder (2026-10-06):** gold `#F9BF3B` is primary brand color. Red/green/gold status ramps kept as reserved tokens (unused on site today, planned for future status/tag features).
- **Accepted without visual verification (2026-10-06):** Chrome browser tooling could not reach `localhost:8001`/`:8002` (blocked by org Chrome policy on this device). Stakeholder chose to proceed on CSS-audit evidence alone rather than block further. Type scale, spacing scale, radius, shadow, motion are extracted from the cleanest/most-repeated patterns in the CSS but have not been eyeballed on a real rendered page.

## Files

| File | Source signal | Confidence |
|---|---|---|
| `color.json` | High-frequency hex values + stakeholder confirmation | High for primary/neutral, reserved for status |
| `typography.json` | rem-based scale (px scale was noise, excluded) | Medium — scale inferred, not from an explicit style guide |
| `spacing.json` | Webflow's generated rem utility classes (each used exactly 36x — strongest signal in the whole audit) | High |
| `radius.json` | 20px dominant, consistent across both sites | High |
| `shadow.json` | One real reused elevation shadow, 3 color variants | High |
| `motion.json` | Clean duration set + one custom easing curve | High |
| `breakpoints.json` | No drift found, clean 7-value set | High |

## Open questions — revisit if they bite during component work

1. Type scale and spacing scale haven't been eyeballed on a real rendered page (template has a `style-guide` page at `/style-guide` — check there if a browser without the localhost block becomes available).
2. Whether `Red Hat Mono` is a real intentional accent or a one-off — only 2 uses found.
3. Whether radius `4px`/`9px`/`3px` are real distinct small-component radii or should collapse to one value.
4. Final status-color token names (`status.danger`/`success`/`warning`) against whatever the planned feature actually needs.

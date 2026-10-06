# History — how this project came to be

Moved out of `readme.md` on 1 September 2026 (v2.0.0) so the readme describes the
system as it is. Nothing here is a rule; `readme.md`, `CLAUDE.md` and `docs/` are.
Every value that moved is in `changelog.md`.

## Two systems, one project

This project is **ACDS (17 August 2026)**. On **21 August 2026** a second, parallel
design system — the "araCreate Design System" built on 20 August — was folded into
it. ACDS was the base: its structure, its file naming and its token values won.

### Sources

The design system is tracked in Git at https://github.com/aracreate-group/aracreate-design-system (branch main, private access). To keep ACDS as the single source of truth, periodically export tokens and components back to the repository. **Verified 24 Aug 2026** by reading the remote — the earlier claim that this repository never existed was wrong, and rested on a 404 that a private repo returns to any unauthorized caller. ACDS here is the authoritative source; the repository is backup and audit trail, and changes flow one way, from Claude Design to the repo. The repo's own README still asserts the reverse and needs correcting there — see [`github.md`](github.md). The source PDFs and deck page renders are not in this project; ask Vishnu for the originals.

- `uploads/aracreate-brand-guidelines.pdf` — Brand Guidelines v1.0 (logo, colour, type, voice, do's and don'ts)
- `uploads/aracreate-brand.pdf` — group and sub-brand lockups, colour and type recap
- `uploads/aracreate-deck.pdf` — **araCreate Deck (04.02.2026)**, 28 pages. The richest source of copy, statistics, service domains, project portfolio, the Jodel case study, team, testimonials, values, ecosystem and contact details.
- `uploads/aracreate-business-card.pdf`, `aracreate-document-template.pdf`, `aracreate-stamp.pdf`, `aracreate-tape.pdf` — brand applications.
- Logo library (`.png` / `.eps`, icon variants) and the **vector SVGs** split from the Affinity multi-artboard sheets, with the Monument wordmark **outlined to vector paths** (no font dependency).
- `MonumentExtended-Regular.otf`, `MonumentExtended-Ultrabold.otf` — display typeface, commercial licence.
- **Webflow site export** — production HTML and CSS; source of UI values, copy, imagery and iconography.
- **Live CMS website** (published June 2026) — confirms the live palette, the WebFont stack and the CMS-driven surfaces. Everything verified against production is recorded in [`docs/live-site.md`](docs/live-site.md).

The second system folded in on 21 August 2026 was itself a port of araCreate's own design-system repository (proprietary, © 2026 araCreate Group; author Vishnu). Its ported CSS and JavaScript are the files now in `styles/` and `js/`; its documentation is in `docs/`. That repository cites the same Brand Guidelines and deck, plus a measured audit of 72 pages of aracreate.group.


## What the merge brought in

ACDS is the base and won on structure, file naming and token values. Folded in from the 20 August system on 21 August 2026:

- **78 components in eight groups** — the component index below.
- **Ten stylesheets** — `base`, `signature`, `components`, `sections`, `app` and `deck` in `styles/`, plus `density` and `theme-dark` in `tokens/`, plus the `tokens.css` the port arrived with, which was split four ways into `fonts` / `colors` / `typography` / `spacing` to follow ACDS's four-files-by-kind token layer.
- **The dark theme** — `tokens/theme-dark.css`.
- **The density scale and the 44px target floor** — `tokens/density.css`.
- **Thirty specimen cards** in `foundations/`, alongside ACDS's own six.
- **Three templates** — web, app, deck. No UI kits: each was either folded into a template or removed.
- **A browser test gate** — `tests/checks.html`.
- **Eleven documents** in `docs/`.

Everything the merge moved in value terms is in [`changelog.md`](changelog.md), dated 21 August 2026.


## What this project does not have

Stated plainly so nobody assumes otherwise.

- **The source repository runs seven gates** — conventions, adherence, contrast, accessibility, behaviour, deck, and a proof that a real page needs no custom CSS. **None of them run here.** `tests/checks.html` is a browser-runnable port of two of them.
- **No screen-reader pass** has been run. [`docs/screen-reader-pass.md`](docs/screen-reader-pass.md) is the script for one, not a record of one.
- **No release process.**
- **Nothing has shipped in production** on the eighteen `app/` components or the six `forms/` additions that came with them.
- **No Figma file.** The design lives in CSS.
- **No araCreate Meditate kit.** It is named in the sources as an intended consumer, but no screens, copy or layouts were provided, so none was built.

The highest-value thing anyone could add is the contrast and accessibility gates from the source repository, globbing rather than listing, so a new card or kit is covered the moment it exists.

---


## Retired names (v2.0.0, 1 September 2026)

Removed outright — this project is the master and carries no aliases:
`styles.css` (now `tokens.css`), `--ac-black`, `--ac-gray-900`, `--ac-gray-700`,
`--ac-gray-600` (all → `--ac-graphite-gray`), `--ac-white` (→ `--ac-canvas`),
`--ac-weight-semibold`, `--ac-slide-foot-lg`, and the `components/content/`
compatibility layer (`ServiceCard` → `Card kind="service"`, `StatBlock` → `Stat`).

## Provenance of the 21 August 2026 merge

Which of the two merged folders each area came from. ACDS was the base; one
exception to "ACDS values won" is the spacing ladder, where `--ac-space-3`
through `-10` carry site-measured 10-based values under ACDS names (conversion
table at the head of `tokens/spacing.css`). Renames in the merge: `guidelines/` →
`foundations/`, `colour-*` → `colors-*`, `css/tokens.css` split four ways into
`tokens/`. Against the 20 August original: 249 files added, 68 overwritten, 0
deleted, 101 untouched.

| Area | Path | Came from |
| --- | --- | --- |
| Token layer | `tokens/{fonts,colors,typography,spacing}.css` | ACDS, values reconciled |
| Density and dark theme | `tokens/{density,theme-dark}.css` | araCreate folder |
| Entry points | `system.css`, `tokens.css` | new in the merge; renamed v2.0.0 |
| Element and device CSS | `styles/{base,signature}.css` | araCreate folder |
| Component and page CSS | `styles/{components,sections,app,deck}.css` | araCreate folder; `app.css` new |
| Behaviour | `js/*.js` | araCreate folder |
| Foundation cards | `foundations/*.card.html` | ACDS, plus 21 added |
| React components | `components/*/` | ACDS for `core/`; seven groups added |
| Templates | `templates/{web,app,deck}/` | ACDS; UI kits folded in 1 Sep 2026 |
| Test gate | `tests/checks.html` | two of the araCreate folder's seven gates |
| Documentation | `docs/*.md` | araCreate folder for five; the rest written in the merge |
| Assets | `assets/**` | ACDS — 69 files |

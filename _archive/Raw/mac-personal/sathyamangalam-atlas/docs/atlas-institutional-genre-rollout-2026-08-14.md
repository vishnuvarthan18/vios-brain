# Atlas institutional genre rollout to all 11 pages — 14 August 2026 (office Mac)

Read `atlas-session-checkpoint-2026-08-14.md` first if you haven't — this
doc assumes that context, plus `atlas-multipage-redesign-2026-08-14.md`
(nine-page structure) and `atlas-design-restyle-2026-08-14.md` (the
terracotta/olive/pine palette that still underlies `shared.css`). Written
from an office Mac session per repo policy: committed locally, **not
pushed**. Bring this commit back to the personal Mac to push, same as any
other office-Mac work.

## What was asked

`site/index.html` had just been rebuilt in a new visual genre
(institutional/corporate-forestry — think a real public forestry
corporation's site, colors/type/layout patterns only, not their brand
identity) and committed on its own, scoped under `body.inst` via a
home-page-only stylesheet, `site/home-institutional.css`. The other eight
pages were still on the older terracotta/olive/Poppins genre from the
prior same-day session. The ask: extend the institutional treatment to
those eight pages, build two new pages (About, Contact), and turn the
home-only stylesheet into a proper site-wide design system — without
touching any live data-fetching (species search, MapLibre map, coverage
dashboard, conflicts table) or the underlying export pipeline.

Midway through the build, the site owner sent an explicit reinforcement:
every one of the cataloged component patterns (hero+breadcrumb, 2×2 grid,
icon-grid, quote+timeline, stat circles, teal closing band, long-form
dividers) needed to actually appear across the 11 pages, not just be
described in the plan. That check happened before this session was
called done — see "Component coverage, verified" below.

A second, later reinforcement drew a hard line on fidelity: match
weyerhaeuser.com exactly on layout/spacing/typography/interaction
mechanics (nav height, hero gradient stops, button padding, grid gaps,
breadcrumb styling, etc.) but never copy their logo mark, wordmark, or
literal copy — that hard line was already agreed earlier and stays
unchanged. See "Pixel-fidelity correction pass" below for what was
re-measured and corrected as a result.

## What was built

### File structure

`site/home-institutional.css` renamed to `site/institutional.css` and
generalized: still scoped under `body.inst` (all 11 pages now carry that
class), still loaded after `shared.css`, but no longer described as
"home page only" in its own header comment. `shared.css` was deliberately
**not** duplicated into the new file — institutional pages reuse a
number of `shared.css` components directly (`.gallery`, `.chip`,
`.splist`/`.sp` species cards, `.chartbox`/`.bars`/`.cmp` chart
components, `.cards`/`.card`, `ul.tick`/`ol.steps`, `.tl`/`.ev` for the
full chronological timeline list, `.q` blockquote, `#map`/`.pl`/`.gaz`,
`.bib`/`.bi`) rather than rebuilding equivalents, since duplicating an
entire component library for a visual skin would have meant two sources
of truth for the same species/map/bibliography rendering logic.

All 11 pages now load, in order: Google Fonts (Poppins/IBM Plex Sans/
JetBrains Mono/Noto Sans Tamil — unchanged from the previous session) →
`shared.css` → Source Sans Pro (institutional genre's own font) →
`institutional.css`. Every page carries `<body class="inst">`.

### The 11 pages

`index.html` (updated: css link renamed, About/Contact added to nav and
footer, a live stat-circle row and teal closing band added before the
footer — it didn't have one yet), `land.html`, `life.html`, `people.html`,
`history.html`, `places.html`, `visit.html`, `record.html`, `govern.html`
(all eight restyled), `about.html` and `contact.html` (both new).

### Component coverage, verified

Per the mid-session reinforcement, checked with `grep -c` across all 11
files rather than trusted from memory:

| Pattern | Where |
|---|---|
| Full-bleed hero + breadcrumb | All 10 interior pages have `.inst-hero` + `.inst-crumbs`; Home intentionally uses its own 4-tile `.inst-tiles` hero instead (matches the original brief's Home-vs-interior distinction) and has no breadcrumb (nothing above Home to crumb to) |
| 2×2 / alternating image-text grid | Land — 4 full `.inst-altrow` rows (forest types, rainfall gradient, core/buffer, ranges) |
| 6-tile icon-grid strategy nav | Visit (3 access types), Record (10 coverage domains), Governance (6 pillars) — all built as inline SVG line-art in institutional green/teal, no icon-font CDN |
| Quote block + circular monogram | History — a real, citable 1887 Nicholson quote (verified against the pre-existing `people.html` content, which already used the identical citation), not fabricated; a plain "N" monogram circle stands in for a portrait since no verified open-licence Nicholson portrait exists |
| Interactive horizontal timeline | History — vanilla-JS mousedown/mousemove drag-to-scroll plus CSS `scroll-snap-type:x proximity`, 15 entries spanning c.405 to 2026, no external timeline library |
| Stat callout circle, live where applicable | Home (overall coverage — live `fetch('../exports/coverage.json')`, same pattern as before), Land (static geography figures), Record (static bibliography-era figures) |
| Teal closing CTA band | All 11 pages, immediately before the footer |
| Long-form sections, hairline dividers, green bullets | Governance, Visit, People, About, Contact all use `.inst-hr` rules and `.inst-list` (green square-bullet) lists |

### Live data-fetching — untouched, re-verified

Same fetch calls, same render functions, same JSON shapes as before this
session — only the surrounding HTML/CSS changed:

- **Life** (`species_curated.json`) — `renderSp()`, filter chips, search
  box all identical; verified 74 curated entries render.
- **Places** (`places_curated.json` + `coverage.json`) — MapLibre init,
  `renderGaz()`, `flyTo()` all identical; verified the map canvas
  renders and the gazetteer list populates with real place data (e.g.
  Thengumarahada, Moyar gorge). Marker colour changed from terracotta
  (`#bf7752`) to institutional green/white (`#016a3a`/`#ffffff`) —
  presentational only, inline in the `<script>` block, not a data path
  change.
- **Record** (`bibliography_curated.json` + `coverage.json` +
  `claims.json`) — `renderBib()`, `renderCoverage()`, `renderConflicts()`
  all identical; verified the coverage table renders all domains at
  31.5% overall and the conflicts table renders all 4 disputed figures
  (e.g. Core Area 793.49 vs 917.27 km²).

## About page — framing decision

Per an explicit instruction already agreed with the site owner:
**Sathyamangalam Atlas is presented as an independent research/media
initiative, not a registered NGO or nonprofit** — it isn't one, and
claiming that status would be misleading. The page states this directly
rather than letting an ambiguous "we" imply otherwise.

Content pulled forward from real, already-published site material (not
invented for this page):

- The "Built honestly" commitments — disputed figures shown side by
  side, live-computed coverage (not hand-typed), no silent data-fixing —
  drawn from the same substance already on Home and The Record.
- The existing non-affiliation disclaimer, verbatim in spirit: "not
  affiliated with, endorsed by, or representing the Tamil Nadu Forest
  Department, the NTCA, or any government body," already present in
  every page's footer `.legal` block.
- The existing CC BY-SA 4.0 (content) / CC BY 4.0 (data) licensing split,
  already stated on Home and The Record.
- The "who's behind it" section deliberately stays general/unnamed — the
  existing footer credit is "built in Erode, from inside the landscape it
  describes," and no individual's name is published anywhere else in the
  repo (checked all 9 prior pages, `CREDITS.md`, and `docs/` before
  writing this section) to draw from. Nothing was invented to fill that
  gap.

## Contact page — real gap, flagged plainly

**No real contact email, phone number or physical address exists
anywhere in this project's own materials.** Checked every existing page,
every footer, `CREDITS.md`, and all `docs/*.md` files for one before
writing this page. The only contact details found in the repo are the
Tamil Nadu Forest Department's own — `04295-220312` /
`(removed)`, quoted on the Governance page's
Administration section — and those are explicitly labelled inline as
"the Forest Department's own contact, not this project's," so they
cannot stand in for a project contact channel without being misleading.

`contact.html` therefore ships with a clearly labelled
`.inst-placeholder` block stating plainly that a project email/contact
form has not yet been set up, plus a commented-out (non-live)
`mailto:contact@sathyamangalam.org` example so the eventual real address
just needs uncommenting and correcting rather than building the block
from scratch. **This is the one piece of this session's work that
needs Vishnu's direct input before launch** — a real, monitored contact
channel.

## Theme toggle decision

**Removed from the visible chrome on interior institutional pages, kept
mechanically alive underneath.** The theme toggle button
(`.inst-theme`, `toggleTheme()`) is still present in every page's header
markup, but the institutional genre defines no dark-specific tokens of
its own (`body.inst`'s custom properties — `--i-green`, `--i-teal`,
`--i-ink`, etc. — are single-valued, not `[data-theme]`-branched), so
toggling currently has no visible effect on `body.inst` pages beyond
whatever `shared.css`'s underlying dark tokens still do to the sliver of
UI institutional.css doesn't override. This mirrors the real reference
site, which has no dark mode at all — light is the only visual identity
for a genre built around large full-bleed photography and a saturated
green/teal palette that a naive dark inversion would muddy rather than
complement. `shared.js`'s `toggleTheme()`/`initTheme()` mechanism itself
is untouched, so a future page (or a future "institutional dark" token
set, if ever wanted) can still hook into it without a rebuild.

## What was deliberately not touched

- `scripts/export_from_db.py`, `data/atlas.db`, every file under
  `exports/` — untouched. This was front-end/presentation work only.
- No page's data-fetching `<script>` block logic — only the surrounding
  HTML containers and CSS classes.
- `sathyamangalam.org` domain registration is still the #1 blocker to
  going live, unchanged from every prior session's notes.

## Deliberate deviation from the original page-by-page plan

Visit page's old `.tabs`/`.pane` Visitors/Filmmakers/Researchers UI
(driven by `shared.js`'s tab-click handler) was replaced with sequential
long-form sections under anchor links (`#film-detail`, `#research-detail`)
rather than kept as tabs. The icon-grid + long-form pattern reads more
consistently with the institutional genre's other pages, and Visit was
one of the pages the brief explicitly named as an icon-grid candidate.
`shared.js`'s tab-switching code is left in the file — inert on pages
that don't use `.tab`/`.pane` markup, available if a future page wants
tabbed content again.

## Pixel-fidelity correction pass

After the first build pass, re-navigated to `weyerhaeuser.com` live (not
from memory) and pulled `getComputedStyle()` values directly via
Playwright's code-execution tool, on both the homepage and an interior
page (`/timberlands/forestry/sustainable-forestry/`), for every component
named in the fidelity request. Corrected `site/institutional.css` against
what was actually measured, not what had been approximated on the first
pass:

| Component | Was (approximated) | Now (measured) |
|---|---|---|
| Nav links | 13.5px, letter-spacing .04em | 16px, letter-spacing normal |
| `.inst-btn` | 13px text, 14px×26px padding, 2px radius, pill-ish | 11px text, 5px×13px padding, 0 radius — confirmed across 9 sampled reference buttons, not a one-off |
| Hero banner height | `min(52vh,440px)` viewport-relative | fixed 550px (reference's own fixed value at 1280px container width) |
| Hero gradient | 3-stop approximation, ~18–92% opacity range | exact 4-stop: transparent 0%→70%, `rgba(0,0,0,.72)` 98%, `rgba(0,0,0,.75)` 100% |
| Hero h1 | `clamp(26px,4vw,42px)`, weight 700 | 40px fixed, weight 600, 44px line-height — all measured |
| Breadcrumb | background `#f3f3f1`, ~12.5px text | background `#eeeeee`, 44px height, 18px text, `#444444` color — all measured |
| 4-tile home hero grid | 420px height, gap unset (already 0, correct) | 400px height (measured), gap confirmed 0 against the reference's own "row no-gutter" class name |
| Footer background | `#323232` | unchanged — was already an exact match |

One measured value was deliberately **not** applied literally: the
reference footer's own padding (`0 15px 92px`) reserves ~92px of bottom
clearance for a sticky "View Sitemap" slide-up element that Atlas has no
equivalent of; copying that figure verbatim would have left a large dead
gap under Atlas's simpler footer content, so vertical footer padding
stays proportionate to Atlas's actual content instead — documented
inline in the CSS comment at that rule.

One correction had to be partially reverted for a legibility reason, not
a fidelity one: matching the hero gradient exactly (no darkening filter
on the `<img>` itself, since the reference has no such filter as a
measurable CSS property) reintroduced the same hero-text-legibility bug
already found and fixed against Atlas's own brighter photography in the
`atlas-multipage-redesign-2026-08-14.md` session — confirmed via
before/after screenshot on the Land page, where "THE LAND" became nearly
illegible against the bright mountain-mist photo with zero darkening.
Restored a `brightness(.62)` filter on the hero `<img>` (documented
inline as a deliberate, labelled exception to "measured only") while
keeping the gradient's 4 stops exact.

The hard line from the very first genre-matching decision was not
touched by this pass: Atlas's own `▲` mark and "Sathyamangalam Atlas"
wordmark are unchanged, no Weyerhaeuser copy text was read into any
page's content, and nothing beyond CSS numeric values (dimensions,
colors, spacing, timing) was carried over.

## Verification performed

Served with `python3 -m http.server` from repo root, drove all 11 pages
with Playwright:

- All 11 return HTTP 200, zero console errors on every page beyond the
  same pre-existing harmless missing `/favicon.ico` from earlier
  sessions.
- Home: live coverage stat circle renders 31.5% (matches
  `exports/coverage.json`'s `overall_pct`).
- Life: species grid renders 74 curated entries, filter chips and search
  wired up as before.
- Places: MapLibre canvas renders, gazetteer list populates with real
  place data and coordinates, 89/7.4% gazetteer-coverage callout correct.
- Record: coverage table renders all 10 domains at the correct live
  percentages, conflicts table renders all 4 disputed figures correctly
  (Core Area 793.49 vs 917.27 km², etc.), overall coverage reads 31.5%.
- History: quote block and 15-entry interactive timeline both render;
  full chronological `.tl`/`.ev` list (the pre-existing detailed
  timeline) preserved underneath as section 03.
- Grep-verified across all 11 files: no stray references to the deleted
  `home-institutional.css` filename remain; every internal `.html` link
  referenced anywhere resolves to one of the 11 real page files (no
  broken links); the component-coverage table above (hero, breadcrumb,
  alt-grid, icon-grid, quote+timeline, stat-circle, closing band,
  long-form dividers) checked file-by-file with `grep -c`, not assumed.
- Re-verified after the pixel-fidelity correction pass: all 11 pages
  still return 200 with zero new console errors; Land, Record and Home
  re-screenshotted to confirm the smaller `.inst-btn` size, corrected
  breadcrumb colors, and fixed-height hero all render correctly and the
  hero-text-legibility fix holds against Atlas's actual photography.

## Open follow-ups

1. **Contact page needs Vishnu's real contact details** — see above,
   the single largest gap this session leaves behind.
2. `sathyamangalam.org` domain registration is still the #1 blocker to
   going live, unchanged from every prior session's notes.
3. If Vishnu wants the Visit page's tabs restored instead of the
   long-form/icon-grid treatment used here, that's a small, isolated
   revert — the old tab markup pattern is still intact on other
   pages' git history and `shared.js`'s tab logic was left in place.
4. No new photography or content was sourced in this session — all
   imagery reuses the same CC-licensed files already in `site/media/`
   from the multi-page redesign session; see that session's doc and
   `site/CREDITS.md` for the full ledger.

## How this was done

Read `atlas-session-checkpoint-2026-08-14.md`, both prior
`docs/atlas-*-2026-08-14.md` files, all 9 existing HTML pages in full,
`shared.css`, `shared.js`, and the pre-existing `home-institutional.css`
top to bottom before writing anything, per the project's own instruction
to treat existing content as the source of real facts rather than
inventing new ones. All 11 pages, the renamed `institutional.css`, and
this doc were built directly in this session — not delegated further —
specifically so the "real content, not invented" and "don't break live
data" constraints could be checked personally against each page's actual
existing markup and each export's actual JSON shape.

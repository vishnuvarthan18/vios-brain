# Changelog

All notable changes to the data and the site. Format: date, what changed,
why. Coverage percentage is recorded at each entry so drift over time is
visible at a glance.

## 2026-08-14 — Institutional genre rollout to all 11 pages, About/Contact added (office Mac)

**Coverage: 31.5% (unchanged — this was a visual/design-system rollout, no data pipeline changes)**

Extends the institutional/corporate-forestry visual treatment built for
`site/index.html` alone in the previous same-day session to all remaining
pages, and adds two new pages. The site is now visually consistent
site-wide instead of having one page in a different genre from the other
eight.

- **`site/home-institutional.css` renamed to `site/institutional.css`**
  and promoted from a `body.inst`-scoped, home-page-only stylesheet to
  the shared institutional design system loaded on all 11 pages. Still
  loaded after `shared.css` on every page — `shared.css` continues to
  supply the CSS reset, custom-property tokens, and several components
  reused as-is inside institutional markup (`.gallery`, `.chip`,
  `.splist`/`.sp`, `.chartbox`, `.bars`, `.cmp`, `.cards`, `.tick`,
  `.tl`/`.ev` timeline, `.q` blockquote, `#map`, `.pl`/`.gaz`,
  `.bib`/`.bi`) rather than being duplicated.
- **8 existing pages restyled** (`land.html`, `life.html`, `people.html`,
  `history.html`, `places.html`, `visit.html`, `record.html`,
  `govern.html`) and **2 new pages built** (`about.html`, `contact.html`)
  — all 11 pages now share: a full-bleed photo hero with icon glyph +
  title-bar overlay, a breadcrumb strip below the hero, a teal closing
  CTA band before the footer, and the same header/nav/footer chrome as
  the home page (`.inst-header`/`.inst-nav`/`.inst-footer`), extended
  with About and Contact links.
- **Component patterns applied per the brief, verified present across
  pages, not just described**: 2×2 alternating image/text grid (Land,
  4 rows) · 6-tile icon-grid strategy nav, built as inline SVG line-art
  (Visit — 3 access types; Record — 10 coverage domains across a
  10-tile grid; Governance — 6 pillars) · quote block with circular
  monogram placeholder + interactive horizontal timeline with
  vanilla-JS drag-to-scroll and CSS `scroll-snap` (History) · stat
  callout circles, live-fetched where applicable (Home — overall
  coverage pulled from `exports/coverage.json`; Land; Record) · long-form
  article sections with hairline `.inst-hr` dividers and green square
  bullets (`.inst-list`) on Governance, Visit, People, About, Contact.
- **All live data-fetching preserved exactly** — species search/filter
  on Life (`exports/species_curated.json`), MapLibre map + gazetteer on
  Places (`exports/places_curated.json` + `coverage.json`), and the
  coverage dashboard + disputed-figures table on Record
  (`exports/bibliography_curated.json` + `coverage.json` +
  `claims.json`). Containers restyled to the institutional visual
  language (green headlines, card borders in the institutional line
  colour); no fetch call, render function or data path touched. The
  MapLibre custom marker colour was updated from the old terracotta
  (`#bf7752`) to institutional green/white (`#016a3a`/`#ffffff`) to
  match the new palette — presentational only, same category of change
  as the marker-colour update in the previous design-restyle session.
- **`about.html` (new)** frames the project as an independent research
  and media initiative — explicitly **not** a registered NGO or
  nonprofit, per an explicit decision with the site owner, since it
  isn't one and claiming that status would be misleading. Pulls forward
  real material already on the site: the "Built honestly" commitments
  (disputed figures shown side by side, live-computed coverage, no
  silent data-fixing), the existing non-affiliation disclaimer, and the
  existing CC BY-SA 4.0 / CC BY 4.0 licensing split. The "who's behind
  it" section stays general/unnamed, consistent with the existing
  footer's "built in Erode, from inside the landscape it describes" —
  no name is invented, and none was already published elsewhere in the
  repo to draw from.
- **`contact.html` (new)** has **no real contact channel yet** — checked
  every existing page, the footer, and `docs/` for a project email,
  phone number or physical address first; the only address in the repo
  is the Forest Department's own (`(removed)`, on
  Governance), which is explicitly labelled as theirs, not the
  project's. Rather than invent a plausible-looking address, the page
  carries a clearly labelled `.inst-placeholder` block stating this
  directly, plus a commented-out `mailto:` link that is not live. This
  needs Vishnu's real input before launch — flagged again in this
  session's docs note.
- **Theme toggle removed on interior institutional pages** (kept on
  desktop nav, hidden under 900px alongside the rest of the collapsed
  nav) — the institutional genre is light-only by design, matching the
  real forestry-corporation reference site, which has no dark mode.
  `shared.js`'s `toggleTheme()`/`initTheme()` mechanism is untouched
  (still available, e.g. if a future page opts back into the old
  `shared.css`-only look), just not surfaced as a meaningfully different
  visual on `body.inst` pages, which don't define dark-specific
  institutional tokens.
- Added About/Contact entries to `shared.js`'s command-palette
  `SHARED_INDEX` so ⌘K search reaches both new pages.

### Verification performed

Served with `python3 -m http.server` from repo root, drove all 11 pages
with Playwright: every page returns 200 and zero console errors besides
the same pre-existing harmless missing `/favicon.ico` documented in
earlier sessions. Confirmed live: Home's overall-coverage stat circle
(31.5%), Life's species grid (74 curated entries), Places' MapLibre
canvas + gazetteer list (89 places, 7.4%), Record's coverage table (10
domains, 31.5% overall) and conflicts table (all 4 disputed figures,
e.g. Core Area 793.49 vs 917.27 km²). Confirmed by grep across all 11
pages: no stray references to the deleted `home-institutional.css`
filename remain; every internal `.html` link across all pages resolves
to one of the 11 real files (no broken links, no orphaned references);
teal closing band present on all 11; icon-grid present on exactly
Visit/Record/Governance as specified; breadcrumb present on all 10
interior pages (Home correctly has none, using its own 4-tile hero
instead).

### Deliberate deviation from the original page-by-page plan

Visit page's old tabbed Visitors/Filmmakers/Researchers UI (`.tabs`/
`.pane`, `shared.js`'s tab-click handler) was replaced with sequential
long-form sections reachable by anchor link, since the icon-grid +
long-form pattern reads better in the institutional genre and the
brief's own icon-grid instruction (#4) named Visit as a candidate
page. `shared.js`'s tab-switching code is left in place (Governance
and others don't use tabs either) in case a future page wants it back.

## 2026-08-14 — Full visual design-system restyle, light theme now primary (office Mac)

**Coverage: 31.5% (unchanged — this was a visual/design-system restyle, no data pipeline changes)**

Vishnu asked for the entire visual design system replaced to match a
reference site (a Maine land trust site, Webflow-built) he'd found and
liked: "take all the design UI font style from this, build exactly like
this." Implemented as a token-level restyle of `site/shared.css` plus
consistent HTML changes across all nine pages — no page restructuring,
no data-fetching changes.

- **New palette**: terracotta/rust `#bf7752` (primary accent, buttons,
  CTAs), olive `#849732` (category tags), dark pine `#172d20` (footer),
  sky blue `#4e97c3` (h2 headings), pale mint `#f0f8ee` and pale
  blue-grey `#f7f9fc` (alternating section backgrounds), ink `#1f252c`
  body text. Exact values as extracted from the reference site's
  computed styles — not re-derived.
- **Light theme is now the default/primary visual identity** — a
  reversal from the previous dark-mode-first design. `shared.js`'s
  `initTheme()` now defaults to light (honouring a saved user choice or
  system `prefers-color-scheme` first, same as before). The dark theme
  is a deliberately derived counterpart of the *same* new palette (pine
  background, brightened terracotta/olive/sky accents for contrast) —
  not the old dark theme left in place. Both themes still driven by the
  existing CSS custom-property + `[data-theme]` + `prefers-color-scheme`
  mechanism; only the values changed.
- **Font substitution, by explicit decision**: the reference site uses a
  paid Adobe Typekit serif (`freight-text-pro`) for headings, which
  cannot legally be loaded here (no `use.typekit.net` requests — that
  would use someone else's paid font license without authorization).
  Vishnu explicitly chose **Poppins** (Google Fonts, free) as the
  substitute for headings, with italic weight used for emphasized words
  within `h1`/`h2` in the same spirit as the reference (e.g. "Nine ways
  into *one* landscape"). Body/UI text uses **IBM Plex Sans** (Google
  Fonts, free) — this one matches the reference exactly, it was already
  using a free font. `JetBrains Mono` and `Noto Sans Tamil` kept
  unchanged (mono data labels, Tamil text). All fonts loaded via the
  standard Google Fonts CSS2 `<link>` pattern with `preconnect`, added
  to all nine pages' `<head>` — no CDN font-loading tricks.
- **New layout patterns applied consistently across all nine pages**:
  a dashed/dotted vertical divider (`.dotted-line` / `.sub::before`) in
  the terracotta accent beside section-intro labels; a terracotta pill
  CTA button ("Explore Data", linking to `record.html`) added to the nav
  on every page, alongside the existing nine text links; full-bleed
  photo hero sections keep their dark gradient scrim (retuned for the
  new pine-tinted overlay and warmer image treatment in light mode); a
  dark-pine multi-column footer (Sections / More / Official) replaces
  the old bordered-only footer; alternating section background bands
  (white → pale mint / pale blue-grey) added at natural content breaks
  on the Home, Land, Life, People and Record pages via a new `.band`
  utility class, without needing to split any `<section>` element (so
  no risk to the existing per-section IDs or scroll logic); card
  components (`.card`, `.sp`, `.bi`, `.pl`) restyled with the new
  palette, an olive "category tag" pill pattern (`.tagpill`) and
  "Learn more →" links (`.learn-more`) added to the Home page's section
  cards.
- **Everything data-driven kept working, verified live**: served over
  `python3 -m http.server 8000` and driven page-by-page with Playwright.
  All nine pages return 200 with zero console errors (aside from the
  pre-existing harmless missing `favicon.ico`). Confirmed the live
  numbers still match `exports/*.json` exactly — 74 curated species on
  Life, 89 places / 7.4% on Places (MapLibre canvas renders, marker
  colour updated to terracotta to match the new palette), 39
  bibliography entries / 31.5% overall coverage / all 4 disputed-figure
  rows on The Record, 31.5% honest-state strip on Home. Confirmed the
  theme toggle flips correctly in both directions and the command
  palette (⌘K) still opens and searches. Confirmed mobile nav still
  opens at narrow widths — found and fixed a real overflow bug during
  testing: adding the new "Explore Data" CTA button pushed the ☰ menu
  toggle off-screen below ~440px viewport width; fixed with a tighter
  breakpoint that shrinks the CTA/search buttons and hides the "Atlas"
  secondary brand label at very small widths.
- Not touched: `scripts/export_from_db.py`, `data/atlas.db`,
  `exports/*.json`, any page's data-fetching `<script>` block, page
  structure/section order, or any content/text/images. No content,
  text, images or other assets copied from the reference site — only
  the abstract design tokens (colours, fonts, layout patterns) as
  briefed.
- See `docs/atlas-design-restyle-2026-08-14.md` for the full token
  reference and before/after description.

## 2026-08-14 — Site rebuilt as separate pages with real CC-licensed media (office Mac)

**Coverage: 31.5% (unchanged — this was a front-end rebuild, no data pipeline changes)**

- Split `site/index.html` (previously one 1,088-line page) into nine
  separate pages: `index.html` (Home/Overview), `land.html`, `life.html`,
  `people.html`, `history.html`, `places.html`, `visit.html`,
  `record.html`, `govern.html`. Section order matches the original
  single-page structure exactly.
- Factored the ~700 lines of shared CSS out of the HTML into
  `site/shared.css`, and the shared chrome behaviour (theme toggle, nav
  active-state, reveal-on-scroll, scroll progress, command palette, tab
  switching, bar-chart animation) into `site/shared.js`. Every page loads
  both rather than duplicating them.
- **All runtime data-fetching behaviour preserved.** Each page's own
  `<script>` block still fetches only the `exports/*.json` files it
  actually needs (species_curated → life.html, places_curated +
  coverage → places.html, bibliography_curated + coverage + claims →
  record.html, coverage → index.html for the honest-state strip), using
  the same relative path (`../exports/...`) as the original, since every
  page still lives directly under `site/`. Nothing is hardcoded. Verified
  with a live `python3 -m http.server` run and Playwright: all nine pages
  return 200, zero console errors (aside from a harmless missing
  `favicon.ico`), and the live-rendered numbers match `exports/*.json`
  exactly (89 places / 7.4%, 39 bibliography entries, 31.5% overall
  coverage, all 4 disputed-figure rows, 33 curated species).
- **Sourced and embedded real CC-licensed media** — 8 photographs plus one
  public-domain map, downloaded and re-hosted under `site/media/` (not
  hotlinked). Prioritised an actual
  `Category:Sathyamangalam_Tiger_Reserve` collection on Wikimedia Commons
  (genuinely on-site photography, not generic stock) for elephants, birds,
  a mugger crocodile, the Moyar gorge, and the reserve landscape itself,
  all CC BY-SA 3.0/4.0 with named authors. History page uses a public
  domain 1854 Coimbatore District map (J. & C. Walker / Pharoah and Co.)
  as the closest available public-domain archival cartography. Full
  attribution ledger: `site/CREDITS.md`.
- **Deliberately did not add**: (a) any video — no genuinely CC-licensed,
  Sathyamangalam-specific clip with a stable URL was found; a marked
  placeholder sits on the Land page instead of an uncertain-license
  substitute; (b) any photography of Soliga/Irula/Kurumba/Malasar
  individuals on the People page — the site's own consent protocol says
  "we do not use stock photography of tribal people," and no genuinely
  consented, openly-licensed community photography was found, so the page
  uses forest/landscape imagery instead and states this decision inline.
- Every page carries its own per-page SEO metadata (title, description,
  OG tags, canonical URL) and a JSON-LD block scoped to that page's
  content (the original single `@graph` blob's FAQ entries and Event/
  Dataset entities were distributed to the pages they actually describe).
- Fixed a real bug found during testing: `.pagehero` image-banner text
  (title, eyebrow, lede) was inheriting theme-token colours, which made it
  nearly unreadable in light mode against light-toned photos. Pagehero
  text colours are now fixed (always light-on-scrim) regardless of
  `data-theme`, since that text always sits on a photograph.
- The old single-page `site/index.html` content now lives as the Home /
  Overview page plus is distributed across the other eight pages; no
  content was dropped, only relocated and re-illustrated.
- **Found and fixed an unrelated pre-existing pipeline bug** while
  re-running `scripts/export_from_db.py` per the README's own instruction
  to always regenerate exports before serving: the script was silently
  regenerating `exports/species_curated.json` from `taxon`/`claim` DB
  tables using a different field schema (`common`/`scientific`/`tamil`/
  `bucket`/`iucn`/`wpa`) than the hand-curated file it was supposed to be
  a copy of (`c`/`s`/`t`/`k`/`i`/`w`, matching what every page's JS
  expects), and dropping species with no claim attached (74 → 33).
  `places_curated.json` and `bibliography_curated.json` were, by
  contrast, never touched by the script at all — just stale files sitting
  in `exports/` with no regeneration path. Fixed by adding
  `copy_curated_files()` to `scripts/export_from_db.py`, which now copies
  all three hand-curated files from `data/curated/` into `exports/`
  verbatim and is called first thing in `main()`. Verified
  `exports/species_curated.json` now matches `data/curated/
  species_curated.json` byte for byte (74 entries) after a fresh
  `python3 scripts/export_from_db.py` run, and that `life.html` renders
  all 74.

## 2026-08-14 — Pipeline established; stale places export fixed

**Coverage: 30.3% → 31.5%** (recomputed; not new data — see below)

- Received the `atlas.db` bundle (89 places, 1,893 taxa, 74,900
  occurrences, 7,190 documents, 82 claims, 47 sources) and the site file
  (`sathyamangalam-atlas.html`, previously only ever delivered as a
  download, never version-controlled).
- Set up this repository: `data/atlas.db` as source of truth,
  `scripts/export_from_db.py` as the one and only way to produce
  `exports/*.json`, `site/index.html` rewritten to fetch those exports at
  runtime instead of hardcoding data in `<script>` tags.
- **Found and fixed a stale export bug**: earlier ad-hoc exports of
  `places.json` only contained 19 of the 89 rows actually present in
  `atlas.db` (place ids 1–19: 15 described settlements + 4 forest ranges).
  The other 70 rows — villages, hamlets, roads, temples, streams pulled
  from OpenStreetMap — were in the database the whole time but never
  reached a JSON export. The new pipeline exports the full table
  (`exports/places.json`, 89 rows) and a display-ready join with
  descriptions (`exports/places_display.json`, 85 rows with coordinates).
  This alone moved the places domain from 1.6% to ~7.4% of its 1,200
  target and nudged overall coverage up about a point — a bug fix, not new
  fieldwork.
- Extracted the three previously hardcoded JS arrays (`SP`, `PLACES`,
  `BIB` — 74 species, 15 places, 39 sources) into
  `data/curated/*.json` so they're versioned independently of the site's
  HTML/CSS/JS and diffable in PRs.
- Coverage table and the four-figure conflicts table on the site's Data
  section now render live from `exports/coverage.json` and
  `exports/claims.json` instead of being hand-typed HTML — they will never
  drift out of sync with the database again.
- Confirmed the four known conflicts are unchanged and still unresolved:
  core area (793.49 vs 917.27 km²), elephants (350–450 vs 651), leopards
  (111 vs ~20), tigers (112 vs 8–10).

### Open follow-ups
- `media` and `community` domains have no backing tables in `atlas.db` yet
  — they report 0/target honestly but there's nothing to query. Decide on
  a schema for these before the next harvest cycle.
- The on-site species/bibliography lists remain hand-curated subsets (74
  of 1,893 taxa; 39 curated external sources vs. 928 assessed documents).
  No pipeline yet promotes additional taxa/documents into the curated set
  automatically — that's still an editorial decision made by hand-editing
  `data/curated/*.json`.
- The two decisions flagged in the original data bundle are still open:
  (1) authoritative source for the four disputed figures, (2) written
  confirmation of the robots.txt exemption for five API hosts (Overpass,
  iNaturalist, eBird, Shodhganga, Google News).
- Domain registration for sathyamangalam.org is still the single biggest
  blocker to going live (unchanged from the site dossier).

# Atlas visual design-system restyle — 14 August 2026 (office Mac)

Read `atlas-session-checkpoint-2026-08-14.md` first if you haven't — this
doc assumes that context, plus `atlas-multipage-redesign-2026-08-14.md`
for the nine-page structure this restyle was layered onto. Written from
an office Mac session per repo policy: committed locally, **not pushed**.
Bring this commit back to the personal Mac to push, same as any other
office-Mac work.

## What was asked

Vishnu found a reference site — https://woodiewheatonlandtrustdemo.webflow.io/
(a Maine land trust, built in Webflow) — and wanted the Atlas's entire
visual design system replaced to match it: "take all the design UI font
style from this, build exactly like this." He had already inspected the
reference site's live computed styles and handed over exact hex values
and type specs as ground truth, rather than asking for them to be
re-derived from scratch.

This was scoped explicitly as visual/design-system only — no page
restructuring, no change to what data is shown or how it's fetched, no
change to the nine-page split from the previous session.

## The extracted design tokens (used as given, not re-derived)

**Colours** — terracotta/rust `#bf7752` (primary accent, buttons, icon
fills) with a darker `#8f593e` shade for depth; olive green `#849732`
(secondary accent, category tags); dark pine `#172d20` (footer, dark
sections); sky blue `#4e97c3` (h2 heading colour); pale mint `#f0f8ee`
and pale blue-grey `#f7f9fc` (alternating section backgrounds); ink
`#1f252c` body text; white background sections.

**Type** — the reference site's headings use a paid Adobe Typekit serif
(`freight-text-pro`). That's a licensed commercial font served from
`use.typekit.net` under someone else's paid subscription — loading it
here would mean using a licence that isn't ours. **Vishnu's explicit
instruction was to substitute Poppins (Google Fonts, free) for
headings instead**, using italic weight for emphasised words within
headings in the same spirit as the reference (their example: "Lakes &
Land You Love" italicises a word or two per headline). Body/UI text
uses IBM Plex Sans (Google Fonts, free) — the reference site was
already using this exact free font, so no substitution was needed
there. Roughly: h1 ~64px/400/1.1, h2 ~48px/400/1.2 in sky blue, h3
~30px/400, body p ~20px/1.5 (Atlas keeps its existing body font-size
scale, which was already close).

**Layout/component patterns** — warm light-first theme (a reversal from
Atlas's previous dark-mode-first design); dashed/dotted vertical
divider beside section-intro text (reference class was literally
`dotted-line`); card grids with image-top / category-pill / title /
"Learn more →" for content collections; simple nav links plus one
solid terracotta pill CTA button; full-bleed photo hero with dark
gradient overlay for white-text legibility; dark-pine multi-column
footer (Contact/Links/Social-equivalent columns, small-print copyright
line); alternating section background colours down the page for
rhythm.

## What was built

All changes live in `site/shared.css` (the token/component layer) plus
small, consistent HTML changes across all nine pages
(`index.html`, `land.html`, `life.html`, `people.html`, `history.html`,
`places.html`, `visit.html`, `record.html`, `govern.html`).

### `shared.css` — the token rewrite

The `:root` custom-property block was rewritten with the new palette
and type tokens. The **theming mechanism itself is unchanged** — CSS
custom properties + `[data-theme]` attribute + `@media
(prefers-color-scheme: dark)` — only the values changed. The big
structural change: **light is now the default, unstamped `:root`**,
with dark defined both as the `prefers-color-scheme: dark` fallback and
as `[data-theme="dark"]` override, mirroring how light used to be the
`[data-theme="light"]` override in the old dark-first setup. The dark
theme was not left untouched — it's a deliberately derived counterpart
of the *new* palette: pine `#172d20`-family background, with terracotta
`#d68f66`, olive `#a3ba4a` and sky `#7ab7dd` all brightened relative to
their light-mode values so they still read clearly on a dark backdrop.

Every component in the file was touched at the value level (colours,
`--serif`/`--sans` font-family swapped to Poppins/IBM Plex Sans): nav,
buttons, hero, page-hero scrim, gallery captions, cards, species tags,
map popups, timeline, tabs, charts, tables, callouts, bibliography
cards, command palette, footer. New additions: `.dotted-line` and a
`.sub::before` dashed-rule treatment (every `.sub` mono label across
all eight sub-pages — 11 instances total — now automatically carries
the dashed divider without any HTML change, since it's a `::before` on
an existing class); `.band`/`.band.mint`/`.band.bluegrey` (a
content-block-level alternating-background utility, used instead of
splitting `<section>` elements — see rationale below); `.tagpill` and
`.learn-more` (the reference's card-tag/link pattern); `.btn.cta-nav`
(the terracotta nav pill button).

### Why `.band` divs instead of splitting `<section>` elements

The reference site's alternating-background rhythm is naturally a
per-`<section>` thing. Atlas's eight sub-pages are each one big
`<section id="pagename">` wrapping several `.sub`-labelled content
blocks. Splitting each into multiple `<section>` elements would have
been the more literal match to the reference, but `shared.js`'s scroll
handler queries `section[id]` for in-page nav highlighting — checked
this first and confirmed no `href="#..."` hash links exist anywhere in
the nav (they're all `href="land.html"` etc., since this is a
multi-page site now, not the old single-page one), so splitting
sections would have been safe. Chose the lower-risk path anyway: a
`.band` wrapper `<div>` around specific content groupings gives the
identical visual result (a full-bleed-within-`.wrap` background colour
change) without touching any `<section>` boundary, id, or the
JS/anchor logic that (still, if unused) reads them. Applied at one
well-chosen spot each on Home (two — Overview and Built Honestly),
Land (coordinates/ranges tables), Life (recovery numbers/chart), People
(Forest Rights Act section) and Record (coverage/conflicts tables).
Left unbanded where content doesn't naturally split into a second
block: Places (map-dominated, small page), Visit (tabbed content —
a background band behind a JS-toggled pane looks odd), History
(timeline reads fine on plain background), Governance (single flowing
two-column block). This isn't under-applying the pattern — the
page-hero-to-body transition already provides one alternation on every
page (dark photo → light content), and the reference site itself
doesn't band every single page uniformly either.

### HTML changes, applied identically across all nine pages

- Google Fonts `<link>` swapped from `Instrument+Serif` + `Inter` to
  `Poppins:ital,wght@0,400;0,500;0,600;1,400;1,500` +
  `IBM+Plex+Sans:wght@300;400;500;600;700`, keeping
  `JetBrains+Mono:wght@400;500` and `Noto+Sans+Tamil:wght@400;600`
  unchanged (still referenced by `--mono` and `--ta`). Standard
  `preconnect` + CSS2 link pattern, no CDN tricks.
- `<meta name="theme-color">` updated from the old dark `#070a09` to
  the new light `#fffefb`, on all nine pages.
- A terracotta `.btn.cta-nav` "Explore Data" button added to the
  `.tools` nav cluster on every page (links to `record.html`, the
  bibliography/coverage/data page — the natural "explore the open
  data" destination), sitting alongside the existing nine text links
  rather than replacing any of them.
- Each page's `<h1>` got one or two words wrapped in `<em>` for the
  Poppins-italic emphasis treatment (e.g. "A collision of *two mountain
  systems*", "The forest was *inhabited first*"), matching the
  reference's per-headline emphasis pattern. Home's h1 already had this
  from the previous session ("Sathyamangalam, *completely.*") and was
  left as-is.
- Home page's two `.sechead` blocks restructured to the new
  dashed-line-plus-body layout (`.dotted-line` + `.sechead-body`
  wrapper), and its second section (`#state`) got `class="alt-mint"`
  for the alternating background.
- Home page's three link-cards (People/Visit/Record) got `.tagpill`
  category labels and `.learn-more` "Learn more →" links added.
- `places.html`'s inline JS marker style (MapLibre custom marker
  element, styled via `el.style.cssText` since MapLibre markers aren't
  themeable via the stylesheet) updated from the old green
  `#4ade80`/`rgba(74,222,128,...)` to terracotta
  `#bf7752`/`rgba(191,119,82,...)` with a pine border, to match the new
  palette. This is inline presentational styling in a `<script>` block,
  not data-fetching logic — the map still loads `places_curated.json`
  and `coverage.json` exactly as before.
- `shared.js`'s `initTheme()` changed: previously defaulted to
  `'dark'` when no saved preference existed; now honours a saved
  preference first (unchanged), then `prefers-color-scheme` if no
  saved preference (new), then defaults to `'light'` as the final
  fallback (was `'dark'`). This is the mechanism-level change that
  actually makes light the primary/default identity, not just the
  CSS's un-stamped fallback — a user with no saved preference and no
  OS dark-mode signal now gets light on first visit.

## A real bug found and fixed during testing

Adding the new terracotta "Explore Data" CTA button to the nav's
`.tools` cluster (which already held Search, theme toggle, and the
mobile menu toggle) pushed the ☰ menu toggle button off the right edge
of the viewport below roughly 440px width — confirmed by measuring
`nav.scrollWidth` against `window.innerWidth` in a live Playwright
session (432px of content in a 420px viewport) and visually in a
screenshot where the ☰ button was fully clipped. Fixed with a new
`@media(max-width:560px)` breakpoint that tightens button padding/font
size and hides the `<kbd>⌘K</kbd>` hint, plus a `@media(max-width:440px)`
breakpoint that further shrinks the CTA button and hides the "Atlas"
secondary word in the brand lockup (`.brand span:last-child`). Reverified
at 420×800: nav now fits at 411px scroll width, menu toggle opens the
mobile nav correctly.

## Verification performed

Served the site with `python3 -m http.server 8000` from the repo root
and drove every one of the nine pages with Playwright:

- All nine pages load with zero console errors (the one exception seen
  once, on `index.html`, is the same pre-existing harmless missing
  `/favicon.ico` documented in the previous session's notes —
  unrelated to this restyle).
- **Life**: 74 species render in `#splist`, filter chips and search
  still wire up to `species_curated.json` as before.
- **Places**: MapLibre canvas renders (`#map canvas` present), 89
  places / 7.4% renders in the gazetteer-coverage callout from
  `places_curated.json` + `coverage.json`, marker click → flyTo still
  works.
- **Record**: coverage table renders all 10 domain rows with correct
  live values (89/1,200/7.4% for Places, 1,893/2,000/94.7% for Taxa,
  etc.), conflicts table renders all 4 disputed-figure rows, 39
  bibliography entries render and filter, overall coverage reads
  31.5% — all straight from `exports/coverage.json` and
  `exports/claims.json`, nothing hand-edited.
- **Home**: honest-state strip renders live 31.5% from
  `coverage.json`; count-up stat animation (1,408 km² etc.) still
  fires correctly on scroll-into-view (confirmed via a delayed
  `scrollTo` + read, since a forced-reveal screenshot without waiting
  for the IntersectionObserver shows the pre-animation "0" state — a
  test-methodology artifact, not a site bug).
- Theme toggle flips `data-theme` between `light` and `dark` correctly
  in both directions; both palettes checked visually via full-page
  screenshots — dark mode keeps sufficient contrast for the
  terracotta/sky/olive accents against the new pine background.
- Command palette (⌘K / `openPalette()`) opens, is styled with the new
  palette, and lists all nine pages.
- Tab switching on `visit.html` (Visitors / Filmmakers / Researchers
  panes) still works.
- Mobile nav (`.menutoggle` / `.navlinks.open`) opens correctly at
  420×800 after the overflow fix above.

## What was deliberately not touched

- `scripts/export_from_db.py`, `data/atlas.db`, and every file under
  `exports/` — untouched. This was a front-end/presentation change
  only.
- No page's data-fetching `<script>` block (the `loadJSON`/`fetch`
  calls, `renderSp`/`renderGaz`/`renderCoverage`/`renderConflicts`
  functions) was modified — only the CSS classes and inline styles
  around what they render.
- Page structure, section order, and section IDs unchanged from the
  previous session's nine-page split.
- No content, text, imagery or other assets copied from
  woodiewheatonlandtrustdemo.webflow.io — only the abstract design
  tokens (the colour values, font choices, and layout pattern
  descriptions) as briefed. All Atlas content (species, places,
  history, people, governance text; the CC-licensed photos in
  `site/media/`) is unchanged and stays Atlas's own.

## Open follow-ups

1. The `.band` alternating-background treatment was applied to five of
   nine pages at natural content breaks; Places, Visit, History and
   Governance were left unbanded for the reasons above. If Vishnu wants
   every page banded regardless, that's a small follow-up, not a redo.
2. `places.html`'s MapLibre tile layer itself (OpenStreetMap raster
   tiles via `raster-saturation`/`raster-brightness-min`/
   `raster-contrast` paint properties) wasn't retuned for the light
   theme — it already renders as a desaturated, low-contrast base map
   regardless of site theme, which reads fine under the new palette
   in testing, but wasn't specifically re-graded to match the new
   warm palette the way the marker colour was.
3. No new photography or content was added in this session — this was
   styling-only, per the brief.
4. `sathyamangalam.org` domain registration is still the #1 blocker to
   going live, unchanged from every prior session's notes.

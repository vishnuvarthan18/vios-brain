# Atlas multi-page redesign + real media — 14 August 2026 (office Mac)

Read `atlas-session-checkpoint-2026-08-14.md` first if you haven't — this
doc assumes that context. Written from an office Mac session per repo
policy: committed locally, **not pushed**. Bring this commit back to the
personal Mac to push, same as any other office-Mac work.

## What was asked

Vishnu wanted a full redesign of `site/index.html` (the 1,088-line single
page): split into separate pages following the existing section order,
a UI/UX overhaul, and real CC-licensed images/video actually sourced and
embedded — while preserving every bit of the runtime data-fetching
behaviour (fetch of `exports/*.json`, live coverage table, live 4-figure
conflicts table, theme toggle, FAQ/schema.org SEO metadata).

## What was built

Nine pages under `site/`, one per section of the old single page, in the
same order: `index.html` (Home/Overview), `land.html`, `life.html`,
`people.html`, `history.html`, `places.html`, `visit.html`, `record.html`,
`govern.html`. Shared chrome factored into `site/shared.css` (design
tokens + every component style, ~700 lines, previously duplicated inline)
and `site/shared.js` (theme toggle, nav active-state, reveal-on-scroll,
scroll progress bar, count-up stats, tab switching, bar-chart animation,
command palette). Every page loads both and otherwise repeats only the
~40 lines of header/nav/footer markup needed for a static multi-page site
with no build step.

### Data-fetching wiring — the part that had to be right first

Each page fetches only the exports it actually renders, via the same
relative path pattern as the original (`../exports/*.json`, since every
page still lives directly under `site/`):

| Page | Fetches |
|---|---|
| `index.html` | `coverage.json` (for the honest-state overall % strip) |
| `life.html` | `species_curated.json` |
| `places.html` | `places_curated.json`, `coverage.json` (for the live 7.4% gazetteer stat) |
| `record.html` | `bibliography_curated.json`, `coverage.json`, `claims.json` |
| `land.html`, `people.html`, `history.html`, `visit.html`, `govern.html` | none — static content, no live data on these sections in the original either |

`renderCoverage()` and `renderConflicts()` moved into `record.html`'s own
script block (that's the page that now owns the coverage/conflicts
tables); `places.html` independently reads the `places` domain row out of
`coverage.json` to populate its own gazetteer-completeness callout.
Verified by running `python3 -m http.server` from repo root and driving
every page with Playwright: all nine return 200, zero console errors
(aside from a harmless missing `/favicon.ico`), and the live-rendered
numbers match the exports exactly — 89 places / 7.4%, 39 curated
bibliography entries, 33 curated species, 31.5% overall coverage, all 4
disputed-figure rows present.

## Media — sourced, not placeholder

8 photographs + 1 map, all downloaded and re-hosted under `site/media/`
(never hotlinked to Commons directly, so the site doesn't depend on
Commons' CDN or file-page structure staying stable). Full ledger with
author/licence/source URL per file: `site/CREDITS.md`.

The best find: an actual **`Category:Sathyamangalam_Tiger_Reserve`**
category on Wikimedia Commons (37 files) — genuinely on-site photography
of this specific reserve, not generic wildlife stock from elsewhere in
India. Used: elephant herd, a second elephant photo, painted storks over
the Moyar, a mugger crocodile on the Moyar, a male chowsingha (credited to
A. J. T. Johnsingh of WWF-India/NCF), the Moyar gorge with Beer Mukkan
temple, and two reserve-landscape/forest-interior shots. All CC BY-SA
3.0 or 4.0 with named authors, verified on each file's own Commons
description page before download. History page uses a public-domain 1854
map of Coimbatore District (J. & C. Walker / Pharoah and Co., Madras) as
the closest genuinely public-domain archival cartography available — it
predates Nicholson's 1887 Manual by 33 years and sits in the same
district-gazetteer tradition.

### Two deliberate absences — read before assuming they're gaps to fill

1. **No video.** Searched specifically for a CC-licensed,
   Sathyamangalam-specific video clip with a stable hosted URL. Found
   none that met both bars. Per the brief's own instruction — a marked
   absence beats an uncertain-license or off-topic substitute — the Land
   page has a small `.mediaplaceholder` block instead of a fake video
   embed. If a genuinely well-licensed clip turns up later, that's the
   slot for it.
2. **No community/tribal portrait photography on the People page.** The
   site's own footer/legal text and the People page's existing copy
   already state a consent protocol: *"we do not use stock photography of
   tribal people"* and photographs of people are published only with
   informed consent. No genuinely consented, openly-licensed photography
   of Soliga, Irula, Kurumba or Malasar individuals was found during
   sourcing, so none is used. The People page instead reuses forest
   imagery from Land/Life and states this decision inline in a callout,
   rather than silently reaching for stock photography that would
   contradict the site's own stated ethics. This is a considered
   restraint, not an oversight — revisit only if genuinely consented
   community photography becomes available.

## A real bug found and fixed during testing

The `.pagehero` image-banner component (full-bleed photo + title + lede,
used on all eight sub-pages) had its text inheriting the theme's `--ink`/
`--dim` colour tokens. In light mode, against light-toned photos (e.g. the
grey mountain-mist landscape shot), the hero title and lede became nearly
illegible — confirmed visually via a Playwright screenshot before the fix.
Fixed by making `.pagehero` text colours fixed (light-on-dark-scrim)
regardless of `data-theme`, since that text always sits on a photograph
and was never meant to flip with the theme in the first place. Verified
fixed with a second screenshot in light mode.

## What's unchanged from prior sessions

- Coverage is still 31.5% — this was a front-end/presentation rebuild
  only, no changes to `data/atlas.db`, `data/curated/*.json`, or
  `scripts/export_from_db.py`.
- The four disputed figures (core area, elephants, leopards, tigers) are
  still unresolved and still shown side by side — no winner picked.
- `sathyamangalam.org` domain registration is still the #1 blocker to
  going live, unchanged.
- media/community domains in `atlas.db` still have no backing tables —
  unrelated to this session's work, still an open follow-up.

## Follow-ups Vishnu may want to weigh in on

1. **Video** — worth a dedicated search pass later (Wikimedia Commons
   video categories, Internet Archive) rather than something to solve in
   the same session as everything else.
2. **People page imagery** — if Vishnu has or can obtain genuinely
   consented photography of community members (per the site's own
   protocol), that's the natural upgrade path for that page; don't add
   stock tribal photography as a shortcut.
3. The old `site/index.html`'s single big `@graph` JSON-LD blob was split
   per-page (each page keeps only the schema.org entities relevant to its
   own content). Worth a quick sanity check with Google's Rich Results
   Test once the site has a real domain, since this is the first time
   this JSON-LD has been split across multiple documents.
4. `site/CREDITS.md` is currently linked from every page footer
   ("Media credits") as a plain `.md` file, not rendered as HTML — it'll
   display as raw markdown in a browser tab, which is fine for now (GitHub
   renders it natively, and it's one click from any page) but could become
   its own `credits.html` page later if that bothers Vishnu.

## How this was done

Read the full 1,088-line `site/index.html` top to bottom first, plus
`README.md`, both prior `docs/atlas-*-2026-08-14.md` files, and every
`exports/*.json` file's shape before writing anything, per the project's
own instruction not to hand-edit exports and to treat `atlas.db` as the
only source of truth. All nine pages, `shared.css`, `shared.js`,
`CREDITS.md`, and the media downloads were built directly in this
session — not delegated to a sub-agent — specifically so data-fetching
correctness and media licence verification could be checked personally
against each file's own Wikimedia Commons description page rather than
trusted secondhand.

# Reference viewer (Realism T13 — Prompt 3, "Viewer")

Three files, all in `design/reference_engine/`:

| file | what it is |
|---|---|
| `export_viewer_data.py` | opens `refs.db` **read only** (`file:…?mode=ro`) and writes `viewer-data.js` |
| `viewer.html` | the page itself — open it by double-click |
| `check_viewer.mjs` | the acceptance checks (18 rows) |

## Use

```
python3 design/reference_engine/export_viewer_data.py     # re-export after any harvest
```

then double-click `design/reference_engine/viewer.html`.

Checks: `node design/realism/tools/run_site_check.mjs design/reference_engine/check_viewer.mjs`
(writes `check_viewer.json` and `viewer-grid.png`).

## What it does and does not do

- **No downloads, no network, no copying.** Every `<img>` points at the photo where it already
  lives, by relative path (`../references/<surface>/<file>`). The check asserts 0 external
  requests and 0 failed requests over `file://`.
- **refs.db and LICENSES.csv are never written.** The export opens the database with `mode=ro`,
  so a write would fail rather than corrupt anything.
- **Tiers.** An item is SHIP only when the database tier says SHIP **and** the licence string is
  one of: public domain, CC0, "no restrictions", CC BY 2.0/2.5/3.0/4.0. Anything else — CC BY-SA,
  CC BY-NC, GODL-India, blank — is shown as STUDY, and its detail view says in a red panel
  "do not use on the website". The viewer never offers a STUDY photo for shipping: there is no
  ship action on a STUDY item, and no export path out of the page at all.
- **Filters**: surface, tier, licence, Tamil relevance, script, source, status, marked.
  **Search** over every text field, English and Tamil. Pages of 24/60/120/240, default 60.
- **Item view**: large image, every metadata field, source link, copy-credit-line, copy-path,
  mark as reviewed, flag as wrong.
- **Compare**: up to 4 picked images side by side, a shared zoom, and a drag ruler that reports
  each length in *image pixels* and the ratio between the last two. Image pixels, not millimetres:
  refs.db records no scale bar, so ratios are usable and absolute sizes are not.
- **Boards / reviews / flags / measurements** are kept in this browser's `localStorage` only.
  The page writes no files (it must not, and downloads are forbidden): use *Copy boards JSON* and
  paste into `design/reference_engine/boards.json` by hand.
- **Coverage panel**: per surface, SHIP / STUDY / total / direct / regional / technique / off and
  a bar against the 500 ceiling, plus the list of Prompt 3 surfaces with no photos at all
  (manuscripts, sites, maps).

## Measured, 2026-09-27

2,412 rows exported, `viewer-data.js` 2.56 MB, 0 image files missing, 896 SHIP / 1,516 STUDY.
First grid painted 250 ms after `load` on file://; last page (41) in 27–39 ms; a synthetic
4,000-row list re-filters and re-renders in 403 ms including the page's own 120 ms search debounce
and a 400 ms settle wait. 18/18 check rows pass, 0 console errors.

No thumbnails: `design/references/` is read-only for this engine, so a 400 px thumbnail set cannot
be written there. The grid therefore lazy-loads the full images (`loading="lazy"`,
`decoding="async"`), which is why a page of 60 is the default.

**Unverified:** every licence, date, script and Tamil reading shown comes from the source API as
recorded in refs.db. None of it has been checked by a Tamil speaker, and the licence fields have
not been re-confirmed against the sources by this task.

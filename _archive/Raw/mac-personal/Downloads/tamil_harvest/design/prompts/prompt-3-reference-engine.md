# PROMPT 3 — Large reference engine (target: up to 500 photos per surface)

Goal: a local "reference engine" that collects, checks, sorts and lets us browse up to 500 photos per surface, so the design (Prompts 2 and 2c) can be matched to real objects. It extends the existing tools in `design/` (`fetch_commons_v2.py`, `curate_refs.py`, `LICENSES.csv`). Do not delete or overwrite the 256 images already there; import them.

## Surfaces
stone, palm_leaf, pottery, coins, copper_plate, rings, seals, plus new: temples (architecture), bronzes (Chola bronzes), manuscripts (page views), sites (Keeladi, Adichanallur, Arikamedu), maps.
Target: up to 500 per surface. This is a ceiling, not a promise: earlier work showed free-licensed Tamil photos for coins, rings, seals and copper plates are scarce. The engine must report the real count and never pad with off-topic or non-Tamil items just to reach 500 (see Relevance rule).

## Two tiers (very important)
- SHIP tier: only public domain, CC0, CC BY (and CC BY-SA only if the owner later says yes). May be used in the website with a credit.
- STUDY tier: everything else that is lawfully viewable (CC BY-SA, CC BY-NC, unclear). Used only to look at and measure. Never copied into `website_live/`. Add a guard script `tools/check_no_study_in_site.py` that fails if any study-tier file (by hash) appears in `website_live/`.

## Sources (verify each yourself; record in `design/SOURCES.md`)
For each source below, read its current terms and API docs first. Record: URL, license rules, rate limits, API key needed or not, and whether you may bulk download. If a source forbids bulk download or needs a login, skip it and say so. Do not use any workaround for a blocked site or domain. Do not scrape sites that forbid it in robots.txt or terms.
Candidates to check (do not assume they work):
- Wikimedia Commons (categories and search by keyword; use the API with a proper User-Agent that names the project; include structured license from the API, not from page text).
- Wikidata and Commons structured data (depicts / collection) to find Tamil-related items.
- Museum open-access APIs: Met, Cleveland, Art Institute of Chicago, Smithsonian Open Access, Rijksmuseum, Wellcome Collection, British Museum (check its terms: likely non-commercial), Europeana, DPLA, Internet Archive (public-domain manuscripts and books), Harvard Art Museums (check terms).
- Indian sources: Government Museum Chennai, Tamil Virtual Academy, Tamil Nadu Archaeology Department, Digital Library of India, CICT digital archives (CC BY-NC: STUDY tier only), Roja Muthiah Research Library (check terms).
- The owner's own photos: watch a folder `design/references/_inbox/<surface>/`; any file dropped there is imported as "owner" tier with a note field (owner must state the license: own photo, permission, etc.).
For each source write a small fetcher `design/fetchers/<source>.py` that: takes `--surface`, `--limit`, `--dry-run`; downloads a max 2000 px long side; writes one database row immediately per file; is resumable (skips existing by URL and hash); waits between requests; and logs errors without stopping.

## Relevance rule (no padding)
An item is kept only if it matches the surface and is Tamil-region or Tamil-script related, or is a clear technique/form reference (for example a non-Tamil palm-leaf manuscript from South or Southeast Asia is allowed as `context: technique`, not counted toward the Tamil count). Score each item: `tamil_relevance` = direct / regional / technique / off. Off items go to `_unused/`. Report counts per relevance level so we know how many are truly Tamil.

## Database and files
- Store: SQLite `design/reference_engine/refs.db` (tables: items, sources, tags, hashes) and keep exporting `design/references/LICENSES.csv` in the same columns as today plus: tier (SHIP/STUDY/OWNER), tamil_relevance, period, place, object_type, width, height, sha256, phash, notes.
- Files: `design/references/<surface>/` for full images, `design/references/_thumbs/<surface>/` for 400 px WebP thumbnails (about 30 KB each).
- Dedup: exact by sha256 and near-duplicate by perceptual hash; keep the largest, log the rest in `_dupes.csv`. Never delete; move.
- Disk: 3,000+ images may be 6 to 10 GB. Before any big run, check free disk space and stop with a clear message if under 15 GB free. Cap full images at 2000 px long side and JPEG/WebP quality 85. Report total size after each run.
- Back up `refs.db` and `LICENSES.csv` before every run to `design/reference_engine/backups/<timestamp>/`.
- `design/references/` stays git-ignored. Commit only the tools and the SOURCES.md, not images.

## Tagging (so we can search the 500)
Auto-tag with what the metadata already says (title, description, categories). Add fields: script (Tamil-Brahmi, Vatteluttu, Grantha, Tamil), dynasty, century, site, condition, colour swatches (5 dominant colours per image; useful for the palette), and "usable for": texture crop / shape reference / lettering reference / lighting reference. Do not use guesses as facts: mark auto tags `auto` and only owner-reviewed tags `reviewed`.

## Viewer (the engine's face): `design/reference_engine/viewer.html`
A local, offline page (opens by double-click, reads `viewer-data.js` exported from the database):
- Grid of thumbnails per surface, infinite scroll or pages of 60.
- Filters: surface, tier, license, tamil_relevance, script, dynasty, century, source, "has colour swatches".
- Search on any text (English and Tamil).
- Item view: large image, all metadata, source link, license and credit, "copy credit line" button, "mark as reviewed", "flag as wrong".
- Compare view: put up to 4 images side by side; crop tool with zoom; a ruler tool to measure ratios (for Prompt 2c precision), with the measurement saved to the item.
- Board view: pick images into named boards ("leaf hole position", "stone carving depth") saved to `boards.json`.
- Big "SHIP" or "STUDY" badge on each image. STUDY images show a "do not use on the website" reminder.
- Coverage panel: for each surface, count in SHIP, STUDY, OWNER, and by relevance, plus a bar against the 500 target.
Use plain HTML, CSS, JS; use the site's tokens for look. Lazy-load thumbnails. It must stay smooth with 4,000 items.

## Credits page for the site
Script `tools/build_credits.py` that reads only SHIP-tier rows actually used in `website_live/` (found by hash) and writes `website_live/credits.html` and `credits-data.json`. Clean the author fields (known problems: PAL-009 "the table and the image mingled by me", COI-009 "Unknown authorUnknown author", PAL-011 blank): if an author cannot be cleaned, set tier to STUDY until fixed and list them in `design/reference_engine/NEEDS_CLEANING.csv`.

## Order of work
1. Import the existing 256 images and current LICENSES.csv into the database; verify counts match (stone 100, palm_leaf 44, pottery 41, coins 37, copper_plate 25, rings 9, seals 0; SHIP about 90).
2. Build the viewer and show the current 256; owner checks it before more fetching.
3. Add fetchers one source at a time. After each source: dry run, then a real run of a small batch (20 items), show the result, then the full run.
4. Coverage report. Stop and report; the owner decides where the gaps are and whether to add his own photos.

## Acceptance checks
1. Import: counts match the current numbers exactly; nothing lost; backups exist.
2. Every row has source URL, license, tier, credit; no row without a license is SHIP.
3. Guard script passes: no STUDY or OWNER-unclear file inside `website_live/`.
4. Viewer opens by double-click with 4,000 test rows without lag (report load time and scroll smoothness).
5. Dedup report: how many exact and near duplicates found and moved.
6. Each fetcher: `--dry-run` works, resumes after Ctrl+C, respects rate limit, User-Agent set, and errors logged.
7. Coverage table per surface with real counts against 500, honest about the gaps, plus a list of sources skipped and why.
8. No deleted files: show `git status` (tools only) and that `design/references/` is still ignored.

## Report back
Coverage table, disk used, list of sources used or skipped with reason, viewer screenshots, list of items that need the owner's decision, and what is not verified (for example license fields you could not confirm by API).

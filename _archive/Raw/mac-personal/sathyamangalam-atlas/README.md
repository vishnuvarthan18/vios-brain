# The Sathyamangalam Record

An open, cited reference for the Sathyamangalam landscape — the tiger
reserve, its people, its history, its species. `data/atlas.db` is the
single source of truth. Everything else — the JSON exports and the live
website — is generated from it or fetched from it at runtime. Nothing is
meant to be hand-edited except the database itself and the small curated
files under `data/curated/`.

## Layout

```
data/
  atlas.db                    ← source of truth (SQLite). Never hand-edit exports instead.
  curated/                    ← hand-maintained editorial content, versioned in git
    species_curated.json      ← the ~74 filterable species cards shown on-site (WPA, Tamil names, notes)
    places_curated.json       ← the ~15 mapped gazetteer entries with descriptions
    bibliography_curated.json ← the ~39 curated external sources (colonial archives, maps, journalism)
exports/                      ← GENERATED from atlas.db. Do not hand-edit. Regenerate with the script below.
  places.json                 ← full place table (89 rows)
  places_display.json         ← places joined with their description claim, for map/gazetteer use
  species.json                ← full taxon table (1,893 rows)
  species_curated.json        ← copy of data/curated/species_curated.json, served alongside the site
  places_curated.json         ← copy of data/curated/places_curated.json
  bibliography_curated.json   ← copy of data/curated/bibliography_curated.json
  documents.json              ← tier A+B documents only (928 of 7,190 raw)
  claims.json                 ← sourced claims + auto-detected conflicts
  coverage.json               ← per-domain completeness, computed live from the DB
  manifest.json                ← build metadata
  legal.json / news.json / passages.json ← currently empty tables, exported honestly as []
scripts/
  export_from_db.py           ← atlas.db → exports/*.json. Run this after every DB change.
  devserve.py                 ← local dev server that serves the site the way production does
site/
  index.html                  ← Home / Overview page
  land.html, life.html, people.html, history.html,
  places.html, visit.html, record.html, govern.html
                               ← one page per section, same nav/theme/footer as index.html
  shared.css                  ← single stylesheet for every page (design tokens, components)
  shared.js                   ← shared chrome behaviour: theme toggle, nav, reveal, palette
  media/                      ← downloaded (not hotlinked) CC-licensed images, re-hosted locally
  CREDITS.md                  ← attribution ledger: file, page, subject, author, licence, source
  Live pages fetch exports/*.json at runtime — no hardcoded data. Only the
  pages that render data carry a fetch; the seven Under Construction stubs
  (see below) carry no page-specific script at all.
  NOTE: index.html, about.html and contact.html are live. land, life,
  people, history, visit, record and govern are Under Construction stubs
  (commit 3aeacc5); places.html is a redirect stub pointing at land.
  Their full versions are in git at a367949 (branch dev) and are meant to
  come back to main one page at a time.
harvest-engine/               ← the backend harvester. Separate Cloudflare Worker,
                                separate database (D1) — NOT atlas.db. See its own
                                README. Nothing here touches the site or atlas.db.
docs/
  atlas-session-checkpoint-*.md   ← read this first when picking the project back up
  atlas-data-bundle-integration-*.md
  atlas-git-pipeline-*.md
  atlas-multipage-redesign-*.md   ← the 9-page split + real media rebuild
```

## Working across two machines / two Claude accounts

This repo is the shared source of truth — both for the code/data (obviously)
and for project context (`docs/`), since Claude's own project memory is
scoped to one account and does not follow you to a session on a different
machine or account.

- **Personal Mac**: the push machine. Clone here, commit here, push here.
- **Office Mac**: pull/clone only. Never push from a Claude session running
  on office infrastructure — if you need to make a change there, commit
  locally and bring the commit/branch back to the personal Mac to push.
- **Either Claude account, any session**: point it at this repo and say
  "read docs/ first" before asking it to continue the work. That's what
  replaces cross-account memory — the docs travel with the code, Claude's
  own memory doesn't.

## The one rule

**`data/atlas.db` is truth. Everything under `exports/` is a build artefact.**
If a number on the site is wrong, fix it in the database (or in
`data/curated/*.json` for the three hand-picked editorial files), re-run
the export script, and commit the database and its exports together in one
commit. Never hand-edit a file under `exports/` — the next export will
silently overwrite it and the fix will be lost.

## Running it locally

```bash
# 1. Regenerate exports/ from the database (always do this first)
python3 scripts/export_from_db.py

# 2. Serve the site the way production serves it.
python3 scripts/devserve.py 8000

# 3. Open http://localhost:8000/
```

**Use `scripts/devserve.py`, not `python3 -m http.server`.** The site links
between pages with clean, extensionless URLs (`href="land"`, not
`href="land.html"`) and treats `site/` as the web root, which is how
Cloudflare serves it in production. `python3 -m http.server` does neither:
every nav link 404s, and `href="/"` serves a directory listing of the repo
root — including `.git`. `devserve.py` mirrors production locally: `site/`
at `/`, `exports/` alongside at `/exports/`, extensionless URLs resolved to
their `.html` file, and unknown paths served the real `site/404.html`.

There is no dependency to install — `devserve.py` is stdlib Python, and the
pages fetch `../exports/*.json` at runtime.

## Deploying

The site is a **Cloudflare Pages** project called `sathyamangalam-atlas`
(account `(removed)`), serving `sathyamangalam.online` and
`www.sathyamangalam.online`. It is **direct upload, not git-connected** —
pushing to GitHub deploys nothing. You deploy explicitly:

```bash
make deploy          # = export + build dist/ + wrangler pages deploy
```

`scripts/build_dist.py` assembles `dist/`, which is the only thing uploaded:

```
dist/                ← site/ contents at the web root
  index.html, land.html, …
  shared.css, shared.js, media/
  exports/           ← exports/ copied in alongside
```

**Both halves are required.** The pages are served with `site/` as the web
root, but they fetch their data with `../exports/*.json`, which from a page
at the web root resolves to `/exports/*.json`. Deploying `site/` alone ships
a site whose every data fetch 404s — that is exactly what happened, and the
live home page showed `(load failed, serve over http)` in place of the
coverage figure until this was fixed. `build_dist.py` hard-fails if
`exports/coverage.json` is missing so it cannot regress silently.

`data/atlas.db`, `docs/` and `harvest-engine/` are deliberately never
published.

After deploying, check the thing that broke before:

```bash
curl -s https://sathyamangalam.online/exports/coverage.json | head -c 80
```

## Updating the database

The harvest pipeline (see `claude/sathyamangalam-harvest-plan.md` in the
project docs — "Operation Full Record") writes into `atlas.db` directly, or
you can edit it by hand with any SQLite client for one-off corrections
(e.g. resolving one of the four disputed figures once you have the 2013
notification). After any change to `atlas.db`:

```bash
python3 scripts/export_from_db.py
git add data/atlas.db exports/
git commit -m "data: <what changed and why>"
```

Keep the database and its exports in the same commit. A commit that changes
one without the other is a bug.

## Editing curated content

`data/curated/*.json` holds the three hand-picked, editorial datasets that
don't come straight out of a table scan:

- **species_curated.json** — the filterable species list on the Life
  section. Add an entry here (not to the database directly) when you want
  a species to appear with a hand-written note, Tamil name, or WPA
  schedule on the site. Fields: `c` common name, `s` scientific name, `t`
  Tamil name, `k` category (`mammal`/`bird`/`herp`/`fish`/`plant`/`invasive`),
  `i` IUCN code, `w` WPA schedule, `n` population figure (string, may be a
  range), `note` free text.
- **places_curated.json** — the gazetteer map/card entries. Fields: `n`
  name, `c` `[lon,lat]`, `d` description.
- **bibliography_curated.json** — the filterable source list on The
  Record. Fields: `y` year/period, `t` title, `a` author/note, `k` category
  (`colonial`/`gov`/`sci`/`media`/`map`), `u` URL, `x` short label.

After editing any of these, copy the updated file into `exports/` (or just
re-run `scripts/export_from_db.py`, which does this automatically) and
commit both copies together.

## Coverage math — how the 30%ish number is computed

`scripts/export_from_db.py` defines, in one place (`DOMAINS`), the target
count and weight for each of ten domains: places, taxa, occurrences,
documents, historical passages, legal instruments, environmental layers,
news, media, community. It queries the live row count for each domain
straight from the database, computes `min(have/target, 1.0)`, and takes the
weighted average. Change a target or a weight only in that one place —
never in `coverage.json`, which is fully regenerated every run.

The **media** and **community** domains have no backing table yet (there is
no `media` or `community_record` table in `atlas.db`). They report 0 until
those tables exist. Community records are meant to stay at zero from
automation by design — consented oral history cannot be harvested.

## Conflicting figures — policy

`export_from_db.py` groups all claims by `(subject_type, subject_key,
field)` and flags any group with more than one distinct value as a
conflict. These are rendered on the site side by side with their sources,
never silently resolved. As of this build there are four: core area,
elephants, leopards, tigers (management-plan 2010 baseline vs. current TN
Forest Dept figures). Resolving one means adding a `superseded_by` or
similar marker in the database — ask before implementing that, since a
"we pick a winner" policy is a real editorial decision the record has
deliberately avoided so far.

## A known gap this pipeline just fixed

Earlier ad-hoc exports of `places.json` only ever contained 19 of the 89
rows actually in `data/atlas.db` (the 15 hand-described settlements plus 4
forest ranges — ids 1–19). The other 70 rows (villages, hamlets, roads,
temples, streams pulled from OSM) were sitting in the database the whole
time but never got exported. `scripts/export_from_db.py` now exports the
full table. This alone moved the places domain from 1.6% to about 7.4% of
target and the overall coverage figure up by roughly a point — not new
data collected, just a stale export bug found and fixed. This is exactly
the kind of drift this pipeline exists to prevent.

## A note on the database file size

`data/atlas.db` is ~26 MB today — under GitHub's 100 MB hard limit, so a
plain `git add` works fine for now. If the harvest grows this past ~200-300
MB (very plausible once historical passages, occurrences and media are
fully populated), switch to [Git LFS](https://git-lfs.com/) for this one
file before it becomes a problem:

```bash
git lfs install
git lfs track "data/atlas.db"
git add .gitattributes data/atlas.db
git commit -m "chore: move atlas.db to Git LFS"
```

Do this proactively rather than after a push fails.

## Licence

Content: CC BY-SA 4.0. Data: CC BY 4.0. GBIF occurrence records: CC
BY-NC 4.0 as supplied by GBIF. See `site/index.html` footer for the full
independence/attribution statement.

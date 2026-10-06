# Coding agent instructions — Sathyamangalam Atlas fixes (26 Aug 2026)

Paste each step below to your coding agent, in this order. Do not skip ahead — later steps depend on earlier ones being done first. After each step, confirm it actually worked (instructions say how) before moving to the next one.

Repo: `~/sathyamangalam-atlas` (or wherever it's checked out on this machine).

---

## STEP 1 — Set up a safe dev branch and preview deploys (do this FIRST, before any other fix)

**Why:** right now every change goes straight to `main`, which is the live site. We want a safe place to test the other fixes before they go live.

**Instructions to paste to your coding agent:**

```
In the sathyamangalam-atlas repo:

1. Delete the branch `restructure-explore-nav` (both locally and on GitHub) — it's an exact duplicate of `dev` and unused.
   git branch -D restructure-explore-nav
   git push origin --delete restructure-explore-nav

2. Push the 5 pending local commits on `main` to GitHub so GitHub reflects the real current state:
   git push origin main

3. Update the `dev` branch to match `main` exactly (it's 44 commits behind):
   git checkout dev
   git merge main
   git push origin dev

4. From now on: all new work happens on `dev` first. Only merge `dev` into `main` once changes are confirmed working. Confirm you understand this workflow before continuing.

5. In the Cloudflare dashboard (Pages project for this site), enable Git integration if it isn't already, so that:
   - pushes to `dev` create an automatic preview URL
   - only merges/pushes to `main` deploy to the live production site
   Report back the preview URL once this is set up so it can be tested.
```

**How to confirm this worked:** ask your coding agent to push a trivial test change to `dev` (like editing a comment) and give you the preview URL. Open it and confirm it looks like the site but is NOT your real live site.

---

## STEP 2 — Add a staging database for the harvest engine

**Why:** right now testing the harvest engine writes directly into your real production database. We want a separate, free test database first.

**Instructions to paste to your coding agent:**

```
In harvest-engine/wrangler.toml:

1. Create a new, second Cloudflare D1 database for staging use:
   wrangler d1 create harvest-engine-staging

2. Add an [env.staging] block to harvest-engine/wrangler.toml that mirrors the existing top-level config, but points the D1 binding at this new "harvest-engine-staging" database instead of "harvest-engine-db". Keep everything else (R2 bucket, queue, etc.) the same unless it's clearly unsafe to share with production — flag if unsure.

3. Run the existing migrations (harvest-engine/migrations/*.sql) against the new staging database so it has the same schema as production:
   wrangler d1 migrations apply harvest-engine-staging --env staging

4. Confirm: wrangler deploy --env staging should now deploy a separate staging Worker that writes to harvest-engine-staging, not the real database. Test this with a harmless command and confirm no rows appear in the production database.
```

**How to confirm this worked:** ask your coding agent to run one stream (pick a cheap one, like `overpass`) against staging and show you it wrote rows to `harvest-engine-staging`, not `harvest-engine-db`.

---

## STEP 3 — Fix the gazetteer bug (on the `dev` branch / staging database, NOT production)

**Why:** this is the actual bug you first noticed — only 58 places found instead of 800–2,000.

**Instructions to paste to your coding agent:**

```
Working on the `dev` branch, in harvest-engine/src/streams/overpass.js:

1. Find the function overpassQuery() (or equivalent) that builds the Overpass API query string. It currently only asks for:
   node["place"~"^(village|hamlet|town|city)$"]

2. Replace it with a wider query that also asks for hills, water, temples, historic sites, and forest areas. Use this as the query body:

[out:json][timeout:60];(
  node["place"](BBOX);
  node["natural"~"^(peak|water|spring|saddle|cave_entrance)$"](BBOX);
  way["natural"="water"](BBOX);
  node["waterway"](BBOX);
  way["waterway"](BBOX);
  node["amenity"~"^(place_of_worship)$"](BBOX);
  node["historic"](BBOX);
  node["landuse"="forest"](BBOX);
  node["natural"="wood"](BBOX);
);out center body;

   (replace BBOX with however the bounding box is currently inserted into the query string, and raise the timeout from 25 to 60 as shown.)

3. IMPORTANT — this wider query returns "way" results, not just "node" results, and way results put their coordinates at el.center.lat / el.center.lon instead of el.lat / el.lon. Find the loop that processes each result element (look for "for (const el of json.elements") and, BEFORE the existing check that skips elements missing a name or coordinates, add:

const lat = el.lat ?? el.center?.lat;
const lon = el.lon ?? el.center?.lon;
if (!name || lat == null || lon == null) continue;

   Then use `lat` and `lon` (not el.lat / el.lon) in whatever function call follows that writes the place to the database.

4. Also find the line that builds an ID like `osmId: \`node/${el.id}\`` and change it to `osmId: \`${el.type}/${el.id}\`` — Overpass results always include an el.type field ("node", "way", or "relation"), and this keeps the ID accurate for all three types.

5. Do NOT touch the bounding box coordinates in overpass-boundary.js in this step — that's a separate, smaller issue, lower priority.

6. Commit this as one single commit on the `dev` branch (both the query change and the way/center fix together — they must ship together or the fix is broken).
```

**How to confirm this worked:** ask your coding agent to run the `overpass` (gazetteer) stream against the staging database (from Step 2) and report how many places it wrote. It should be in the 500–2,000+ range, not 58. If it's still very low, something else is wrong — don't proceed to Step 4 until this number looks right.

---

## STEP 4 — Check the two databases actually match, then build the bridge between them

**Why:** even with the gazetteer fixed, nothing the harvest engine collects reaches your live website until this bridge exists.

**Instructions to paste to your coding agent:**

```
1. First, check whether the two databases' table structures actually match. Run:
   sqlite3 data/atlas.db ".schema place"
   sqlite3 data/atlas.db ".schema taxon"
   sqlite3 data/atlas.db ".schema occurrence"
   sqlite3 data/atlas.db ".schema document"
   sqlite3 data/atlas.db ".schema historical_passage"
   sqlite3 data/atlas.db ".schema legal_instrument"
   sqlite3 data/atlas.db ".schema observation_layer"
   sqlite3 data/atlas.db ".schema news_event"

   Compare each of these against the equivalent table definitions in harvest-engine/migrations/0001_init.sql (and any later migration files that alter these tables). Report any column name or type differences before doing anything else — do not proceed if there are unresolved mismatches.

2. Once confirmed compatible (or after reconciling any differences), write a new script — scripts/sync_from_d1.py (or a Node.js equivalent if that fits the codebase better) — that:
   a. Exports each of the 8 tables above from the harvest-engine-db D1 database (use `wrangler d1 export harvest-engine-db --remote` or equivalent per-table queries)
   b. Inserts or updates (upsert) matching rows into data/atlas.db, using each table's natural unique key to avoid creating duplicates on repeat runs (e.g. place table: match on slug + reserve_id; taxon table: match on slug + reserve_id; other tables: use whatever unique identifier already exists in their schema — check each one).
   c. Never deletes anything from atlas.db — only adds new rows or updates existing ones.
   d. Prints a summary at the end: how many rows were added/updated per table.

3. Add this script as a "sync" step in the Makefile, and make the existing "deploy" target run "sync" before "export" and "build":
   deploy: sync export build [existing deploy command]

4. Test this end to end on the staging database first: run the sync script pointing at harvest-engine-staging instead of harvest-engine-db, against a COPY of atlas.db (not the real file), and confirm rows show up correctly with no duplicates when run twice in a row.
```

**How to confirm this worked:** ask your coding agent to run the sync script twice in a row against a test copy of atlas.db and show you the row counts didn't double on the second run (proving it's not creating duplicates).

---

## STEP 5 — Handle the three streams with missing API keys (your task, not the coding agent's)

**Why:** `bhl`, `wdpa`, `lgd` streams will fail every run until you get free accounts set up.

**What YOU need to do (not the coding agent):**
1. Go to biodiversitylibrary.org and request a free API key for BHL (Biodiversity Heritage Library).
2. Go to api.protectedplanet.net/request and request a free WDPA/Protected Planet API token.
3. Go to data.gov.in, create a free account, and get the resource ID needed for `sources/lgd.json`.

**Once you have these**, paste to your coding agent:
```
Add these three secrets to the harvest-engine Worker using wrangler secret put, and update .dev.vars for local testing:
- BHL_API_KEY
- WDPA_API_KEY
- DATA_GOV_IN_API_KEY (used in sources/lgd.json's resource_id field — update that file directly with the resource ID I obtained: [paste it in])
```

**Until you have these keys:** tell your coding agent to exclude `bhl`, `wdpa`, and `lgd` from any harvest run this week — they'll just show "failed" and that's expected, not a bug.

---

## STEP 6 — Decide what to do about the permanently-blocked WII stream

**Why:** the Wildlife Institute of India source is blocked by their site's own rules, and nothing currently distinguishes "known permanent block" from "something broke."

**This is a decision for you, not a code task by itself.** Pick one:
- **Option A (exclude it):** paste to your coding agent: "Remove the `wii` stream from the active run list for now — it's permanently blocked by robots.txt and there's no override justification recorded."
- **Option B (override it):** only if you have a specific reason this access is acceptable (e.g. it's your own reserve's public data and you've decided the block is overly broad, similar to the Overpass API override already used elsewhere in this project) — paste to your coding agent: "Add a `robots_txt_override` entry to sources/wii.json, matching the shape of sources/overpass.json's existing override, with this justification: [state your reason]."

I'd lean towards Option A unless you have a specific reason to override it — it's the safer default.

---

## STEP 7 — Confirm the satellite imagery (GEE) stream still works

**Why:** it depends on an external Google account setting that could have quietly expired.

**Instructions to paste to your coding agent:**
```
Run the "gee" stream once from the dashboard (or via API) and report back: did it report status "success" or "partial" with rows_written greater than 0? If it reports "failed", show me the full error message from errors_json before doing anything else — do not attempt to fix GEE issues without showing me the error first.
```

---

## STEP 8 — Basic safety check before every future change (CI)

**Why:** right now nothing checks your code before it goes live.

**Instructions to paste to your coding agent:**
```
Create a GitHub Actions workflow file at .github/workflows/deploy.yml that:
1. Runs on every push to the `dev` or `main` branches
2. Runs whatever tests exist in harvest-engine/test/
3. If pushing to `main`: blocks the merge/deploy if any test fails
Keep this simple — just run the existing tests and fail the workflow if they fail. No need for anything more elaborate.
```

---

## STEP 9 — Dashboard visibility improvements (do this once things above are stable)

**Instructions to paste to your coding agent:**
```
In harvest-engine/dashboard/render.js, in the "Recent job runs" table:

1. Add a new column that shows the first error message when a run's status is "failed" or "partial". Parse it from the errors_json field, something like:

function firstErrorMessage(errorsJson) {
  if (!errorsJson) return "";
  try {
    const arr = JSON.parse(errorsJson).filter((e) => e.kind !== "note");
    return arr.length ? arr[0].message : "";
  } catch { return ""; }
}

Truncate the displayed text to about 100 characters, with the full message available on hover (use a "title" attribute).

2. At the top of the dashboard page, add a banner that lists any streams whose most recent run status is "failed" — something like "3 streams are currently failing: wdpa, bhl, lgd — see below." Compute this from the same job-run data already being fetched for the table; no new database queries needed.
```

---

## STEP 10 — Housekeeping (low priority, do whenever convenient)

**Instructions to paste to your coding agent:**
```
1. Move harvest-engine/TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_P.pdf out of the git repo and into the existing R2 bucket used for raw source files. Update src/streams/management-plan.js to fetch it from R2 instead of reading it from the local file. Remove the file from git tracking (git rm --cached, keep it in .gitignore).

2. Look at the docs/claude-project/ folder in the repo root — it's currently untracked. Either add it to .gitignore (if it's just a scratch mirror that doesn't need to be in git) or commit it properly. Ask me which if unsure.

3. Remove the IUCN_API_KEY line from harvest-engine/wrangler.toml's secrets comment block — it's not used by any current stream and is misleading documentation.
```

---

## Order summary (do not skip around)

1. Dev branch + preview deploys (Step 1)
2. Staging database (Step 2)
3. Gazetteer fix, tested on staging (Step 3)
4. Database bridge, tested on staging (Step 4)
5. Get API keys yourself (Step 5)
6. Decide on WII (Step 6)
7. Confirm GEE still works (Step 7)
8. Add basic CI (Step 8)
9. Dashboard visibility (Step 9)
10. Housekeeping (Step 10)

Only once Steps 1–4 are confirmed working on `dev`/staging should you merge `dev` into `main` and let it go live. Do the sustained week-long harvest run only after that merge.

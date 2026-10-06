# Overnight full fix — Sathyamangalam Atlas

Paste this whole file to Claude Code (or your coding agent) in one go and let it work through it in order, unattended. Every step includes how to confirm it worked. Do not skip the order — later steps depend on earlier ones.

Repo: this project (Sathyamangalam Atlas / "The Sathyamangalam Record").

## If you hit a usage limit or the session gets cut off partway through

Do not leave anything half-broken. Specifically:
- Never stop in the middle of a single step — finish the current step's file edits fully (or fully revert them) before stopping, even if that means going slightly over.
- Never leave `main` in a broken state. All risky work happens on the `dev` branch (Step 1 sets this up) — if you must stop, stop on `dev`, not mid-merge into `main`.
- Before stopping, commit whatever is done so far on `dev` with a clear commit message stating which Step number was in progress and what was/wasn't finished.
- Write a short status note at the top of this file (or a new file `OVERNIGHT_RUN_STATUS.md`) listing: which steps are fully done, which step was in progress and exactly where it stopped, and which steps haven't started yet.
- When the session resumes (limit reset, or a new session started), read that status note first, then continue from the first incomplete step — do not restart from Step 0 and do not re-do steps already confirmed complete.
- If a usage limit pauses you automatically mid-response, that's fine — just resume from where you left off once it lifts, following the same rule: check status, continue, don't restart.

---

## STEP 0 — Urgent, do this first, takes 30 seconds

Add this to `.gitignore` immediately, before touching anything else:

```
# Personal / cross-session working notes — never publish
/docs/claude-project/
```

Then run `git status` and confirm `docs/claude-project/` no longer shows as untracked-and-committable. Do NOT run `git add -A` or `git add docs/` anywhere in this session until this is done.

---

## STEP 1 — Set up a safe branch + preview deploy workflow

**Why:** every change so far has gone straight to the live site with no safety net. Set this up before making any other change tonight.

```
1. Delete the branch `restructure-explore-nav` (local and GitHub) — it's an exact duplicate of `dev` and unused:
   git branch -D restructure-explore-nav
   git push origin --delete restructure-explore-nav

2. Push any pending local commits on `main` to GitHub:
   git push origin main

3. Bring `dev` up to date with `main`:
   git checkout dev
   git merge main
   git push origin dev

4. From this point on in tonight's run: make every change below on the `dev` branch. Only merge `dev` into `main` at the very end, after everything is confirmed working, and only if instructed to in the final step.

5. If Cloudflare Pages Git integration isn't already on for this project, enable it so pushes to `dev` get an automatic preview URL, and only `main` deploys to the real live site. Report the preview URL once available.
```

**Confirm:** push a trivial test change to `dev`, get the preview URL, open it, confirm it's a separate URL from the real live site.

---

## STEP 2 — Add a staging database for the harvest engine

```
1. Create a second D1 database for testing:
   wrangler d1 create harvest-engine-staging

2. Add an [env.staging] block to harvest-engine/wrangler.toml mirroring the existing config but pointing at harvest-engine-staging instead of harvest-engine-db. Keep other bindings the same unless clearly unsafe to share — flag if unsure.

3. Apply existing migrations to the new staging database:
   wrangler d1 migrations apply harvest-engine-staging --env staging

4. Confirm `wrangler deploy --env staging` deploys a separate Worker writing to harvest-engine-staging, not the real database.
```

**Confirm:** run one cheap stream (e.g. `overpass`) against staging, verify rows land in `harvest-engine-staging`, not `harvest-engine-db`.

---

## STEP 3 — Fix the gazetteer bug (on `dev`/staging, not production)

**Why:** the place-name harvester only finds ~58 places when it should find 800–2,000. Root cause: the query only asks for villages/hamlets/towns/cities, nothing else.

```
In harvest-engine/src/streams/overpass.js:

1. Find overpassQuery() (or equivalent). Replace the narrow query with a wider one that also asks for hills, water, temples, historic sites, and forest areas:

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

   (replace BBOX with the existing bounding-box insertion, raise timeout from 25 to 60.)

2. IMPORTANT — "way" results put coordinates at el.center.lat/el.center.lon, not el.lat/el.lon. In the loop processing each result element, BEFORE the existing name/coordinate check, add:

const lat = el.lat ?? el.center?.lat;
const lon = el.lon ?? el.center?.lon;
if (!name || lat == null || lon == null) continue;

   Use `lat`/`lon` (not el.lat/el.lon) in the database-write call that follows.

3. Change any `osmId: \`node/${el.id}\`` to `osmId: \`${el.type}/${el.id}\`` (el.type is always present: "node", "way", or "relation").

4. Commit the query change and the way/center fix together in one commit — they must ship as one unit.
```

**Confirm:** run the `overpass` stream against the staging database, check it wrote somewhere in the 500–2,000+ range, not 58. If still low, stop and report — do not proceed to Step 4.

---

## STEP 4 — Check the two databases match, then build the bridge between them

**Why:** the harvest engine writes to D1 (cloud). The live website reads from `data/atlas.db` (local file). Nothing currently moves data from one to the other.

```
1. Check schema compatibility first:
   sqlite3 data/atlas.db ".schema place"
   sqlite3 data/atlas.db ".schema taxon"
   sqlite3 data/atlas.db ".schema occurrence"
   sqlite3 data/atlas.db ".schema document"
   sqlite3 data/atlas.db ".schema historical_passage"
   sqlite3 data/atlas.db ".schema legal_instrument"
   sqlite3 data/atlas.db ".schema observation_layer"
   sqlite3 data/atlas.db ".schema news_event"
   sqlite3 data/atlas.db ".schema claim"

   Compare against harvest-engine/migrations/*.sql definitions for these tables. Report any mismatches before proceeding — do not continue if there are unresolved differences.

2. Write scripts/sync_from_d1.py (or Node equivalent) that:
   a. Exports each table above from harvest-engine-db (via `wrangler d1 export` or per-table queries)
   b. Upserts rows into data/atlas.db on each table's natural unique key (place: slug+reserve_id; taxon: slug+reserve_id; claim: whatever key applies — check schema; others per their existing unique constraints)
   c. Never deletes rows — only adds or updates
   d. Prints a summary of rows added/updated per table

3. Add a "sync" target to the Makefile; make "deploy" depend on "sync" running before "export" and "build".

4. Test end to end against the staging database and a COPY of atlas.db (not the real file) — run twice in a row, confirm no duplicate rows are created on the second run.
```

**Confirm:** row counts don't double when the sync script runs twice in a row against the test copy.

---

## STEP 5 — Fix the conflicting-figures bug (the tiger 112 vs 8-10 issue)

**Why:** the "disputed figures" feature currently treats the same measurement taken at two different years as a contradiction (e.g. 2009 tiger count vs 2024 tiger count), when it's actually a real population change over time, not a genuine dispute. Publishing this as-is would show the reserve's celebrated tiger-recovery story mislabeled as a data error.

```
1. In export_from_db.py, find the grouping logic (around the conflict-detection section, currently something like):
   groups[(c["subject_type"], c["subject_key"], c["field"])].add(str(c["value"]))

   Change it to also include the date field in the grouping key:
   groups[(c["subject_type"], c["subject_key"], c["field"], c["as_of"])].add(str(c["value"]))

   And make sure "as_of" is included in each conflict's output dict in the conflicts list that gets built afterward.

2. Run export_from_db.py and confirm the conflicts count in exports/claims.json drops to 0 (this is expected and correct — it means no genuine same-period disputes currently exist in the data). Confirm the old renderConflicts() logic (recovered in Step 6) already handles an empty conflicts list gracefully with a "no unresolved conflicts on record" message — if not, add that handling now.

3. Separately, add a basic "how this figure changed over time" data view: group by (subject_type, subject_key, field) ordered by as_of, so the real story (population recovery over 15 years) can eventually be shown as a timeline rather than hidden. A full UI for this is NOT required tonight — just make sure the exported JSON structure supports it (e.g. a "history" array per subject/field in claims.json). If this is a larger task than fits tonight's run, stub the JSON structure and leave a comment marking it TODO — do not skip the grouping-key fix above, that part is required tonight.

4. Update any hardcoded "3" conflict counts in site/index.html (search for text like "3</b><span>disputed figures") to either read live from claims.json or be removed until the table is restored.
```

**Confirm:** `exports/claims.json`'s conflicts array is empty (0 real conflicts) after this fix, and nothing on the site displays a hardcoded "3" anymore.

---

## STEP 6 — Restore the real content for the 7 "Under Construction" pages, and rebuild the conflicts table

**Why:** `land.html`, `life.html`, `history.html`, `record.html`, `govern.html`, `people.html`, `visit.html` currently show placeholder "Under Construction" text, but their real, finished content exists in git history (commit `a367949` / the `dev` branch snapshot from before it was intentionally set aside on 17 Aug).

```
IMPORTANT: do NOT run `git checkout dev -- site/` wholesale. The dev branch also contains OLDER, worse versions of about.html and contact.html than what's currently live on main — checking out the whole branch would downgrade those. Only restore the BODY CONTENT of the 7 specific stub pages, into the CURRENT page shell/design.

For each of the 7 pages (land, life, history, record, govern, people, visit):

1. Get the real content: git show a367949:site/<page>.html
2. Compare its body content against the current live page's design (use about.html on main as the reference for current header/footer/nav/SEO structure and current branding — "The Sathyamangalam Record", not the old "Sathyamangalam Atlas" name).
3. Rebuild the page: current header/footer/nav/SEO chrome from main + real body content from the archived version.
4. Rewrite any old-style links (href="about.html") to the current clean-URL style (href="about") throughout anything restored.
5. Once ALL 7 pages have their real content back, add the correct SEO tags back (index,follow + the real FAQ structured data, since the content now genuinely exists) — reversing the noindex fix from Step 7 below, for these 7 pages specifically, one at a time as each is actually finished tonight. If a page can't be fully restored tonight, leave its noindex tag in place rather than un-hiding an incomplete page.

For record.html specifically, ALSO restore the disputed-figures table:
6. The old table markup and JS is in the same archived commit: git show a367949:site/record.html — the table is around where <tbody id="conflictBody"> appears, with render functions nearby.
7. Wire this table to fetch exports/claims.json (which now has the corrected, date-aware conflict data from Step 5).
8. Test that if claims.json has 0 conflicts, the table shows "No unresolved conflicts on record" (not broken/empty).

Do this on the `dev` branch, test each page's preview URL, and only proceed to merging into main once satisfied.
```

**Confirm:** all 7 pages show real content on the `dev` preview, no "Under Construction" text remains anywhere, and record.html's table loads without errors (showing either real conflicts or the "none currently" message).

---

## STEP 7 — Stop telling Google there's content where there isn't (do this NOW for any of the 7 pages not yet restored by the time this step runs)

**Why:** the "Under Construction" pages currently carry full SEO tags and FAQ structured data describing finished content that isn't there — this risks Google penalizing the whole site for thin/mismatched content.

```
For any of the 7 pages (land, life, history, record, govern, people, visit) that ARE STILL "Under Construction" at this point in tonight's run (i.e. Step 6 didn't finish all of them):

1. Change <meta name="robots" content="index,follow,..."> to <meta name="robots" content="noindex,follow">
2. Delete the entire <script type="application/ld+json">...</script> block
3. Replace the meta description with something honest, e.g. "This section is still being built."
4. In sitemap.xml, remove the <url> entries for any pages still under construction — keep only pages that are genuinely finished.

For any pages that WERE fully restored in Step 6, skip this step for them (or reverse it if it was already applied in an earlier partial run) — restored pages should keep full SEO indexing.
```

**Confirm:** grep all site/*.html for "noindex" and confirm it's present ONLY on pages still genuinely unfinished, absent on pages restored in Step 6.

---

## STEP 8 — Fix dead-end navigation (only for anything still unfinished after Step 6)

```
For any pages still under construction after Step 6:
1. In index.html and about.html, find every link/button pointing at one of these still-unfinished pages.
2. Remove the href (make it non-clickable) and add a small visible "coming soon" label instead of leaving it as a broken promise.
3. Also fix these 5 known dead anchor links if their target sections don't exist yet: about.html's link to land#corridors, index.html's links to govern#accreditation, govern#mee-2019, history#gazetteer-1927, land#corridors, and places.html's link to land#gazetteer. Either point them at real sections (if Step 6 restored them) or strip the #anchor fragment.
```

**Confirm:** no visible button/link on the homepage or About page leads to an unfinished page without a clear "coming soon" indicator.

---

## STEP 9 — Remove the visible debug error message on the homepage

```
In index.html, find the catch block around the coverage-percentage fetch (currently shows literal text like "(load failed, serve over http)" to visitors if the fetch fails). Replace the visitor-facing fallback text with something like "—" or "coverage figure unavailable". Keep the console.error() call as-is — that's fine, it's not visible to visitors.

Also fix the fragile relative path in the same area: change fetch('../exports/coverage.json') to fetch('/exports/coverage.json').
```

**Confirm:** simulate a failed fetch (or just read the code) and confirm no raw debug text can reach the visible page.

---

## STEP 10 — Re-insert the one real historical dispute, marked as resolved (not deleted)

**Why:** the reserve's core area was once reported as both 793.49 km² and 917.27 km² by different sources — correctly resolved using the 2013 government notification, but the old superseded figure was deleted entirely instead of being kept and marked "resolved." This is the single best real example of the site's own stated method and currently can't be shown at all.

```
1. Re-insert the 917.27 km² claim into the `claim` table, with its original secondary source, if it can be found in git history or old exports (check .baseline/ folder and git history for the original claim record before it was deleted).
2. Add a "superseded_by" or "status" column to the claim table (new migration file) so a claim can be marked resolved and linked to the claim that replaced it.
3. Update export_from_db.py to include resolved-but-superseded claims in a separate section of claims.json (not mixed into the active "conflicts" list), showing both values, which one won, and the primary source that decided it.
4. This does not need a full UI tonight — getting the data correctly structured in claims.json is the priority. Note in a comment if UI work remains for a future session.
```

**Confirm:** `exports/claims.json` contains a resolved/superseded entry for the core-area figure with both values and the deciding source.

---

## STEP 11 — Small necessary fixes (batch these together, low risk)

```
1. Convert site/CREDITS.md into site/credits.html using the same page shell as 404.html, converting the markdown table into a real HTML table. Update all footer links from href="CREDITS.md" to href="credits". Keep the .md file as source of truth.

2. Fix the About page's claim "nobody hand-types this number" sitting next to a hand-typed, already-wrong number (currently says "31.5%" as literal text). Either remove the specific number from that sentence, or make it fetch the live number the same way the homepage does.

3. Add a short "Open Data" section to about.html linking directly to /exports/manifest.json, /exports/claims.json, /exports/species.json, /exports/places.geojson, noting the CC BY 4.0 license.

4. Fix the one unescaped "&" character in record.html (should be "&amp;").

5. Add a poster image to the homepage's autoplay hero video (poster="media/land-mountains-forest.jpg", which already exists) and add preload="metadata". Flag but do not attempt tonight: re-encoding the 11MB video to a smaller size — that's a larger task, note it for later.

6. Add a site/_headers file with a long cache-control for /media/*, a short one for /exports/*, and X-Content-Type-Options: nosniff.

7. Remove the unused IUCN_API_KEY line from harvest-engine/wrangler.toml's secrets comment block.
```

**Confirm:** each item works as described; nothing else breaks.

---

## STEP 12 — Update the project's own documentation to match reality

```
1. In README.md, correct these known-stale facts:
   - The number of currently-disputed figures (recompute from the fixed data, likely 0 real conflicts right now, plus 1 resolved/superseded historical one from Step 10)
   - The claim that the 2013 notification is still needed (it's already been obtained and used)
   - The file path reference to claude/sathyamangalam-harvest-plan.md → correct to docs/claude-project/sathyamangalam/harvest-plan.md if that's still the real path, or wherever it actually lives after Step 0's .gitignore change
   - The "single stylesheet" claim (there are two: shared.css and institutional.css)
   - Source count and species count numbers — recompute from the live database
   - Add build_dist.py and any missing site files (about.html, contact.html, 404.html, robots.txt, sitemap.xml, institutional.css) to the file layout description
   - Add a short "Make targets" section documenting make export / serve / check / build / deploy

2. In CHANGELOG.md, add entries for everything that happened since 14 Aug, especially:
   - The 17 Aug commit that turned 7 pages into stubs (and that content was preserved in git history)
   - Tonight's full fix run — summarize what was done, referencing this file
```

**Confirm:** both files read accurately against the current, post-fix state of the project.

---

## STEP 13 — Final review before merging to production

```
1. Run `make check` and `make build` and confirm both succeed with no errors.
2. Load the dev branch's preview URL and manually click through every page and every nav link — confirm no dead ends remain (except any pages deliberately left as "coming soon" with honest labeling).
3. Confirm no personal documents (docs/claude-project/) show up in `git status` as trackable.
4. Summarize everything actually completed tonight vs anything left incomplete or explicitly deferred (e.g. video re-encoding, full history-view UI for disputed figures, analytics setup which needs a Cloudflare dashboard action from the human).
5. Do NOT merge `dev` into `main` automatically. Stop here and report the full summary, the preview URL, and a clear list of what still needs a human decision or a manual dashboard action (Cloudflare Analytics toggle, any API keys, the WII stream decision) before going live.
```

---

## Explicitly deferred — do not attempt tonight, needs a human decision or account access

- Getting the three missing API keys (BHL, WDPA, data.gov.in) — needs Vishnu to sign up personally.
- Deciding whether to override or exclude the `wii` (Wildlife Institute of India) stream, which is blocked by robots.txt.
- Turning on Cloudflare Web Analytics — needs a dashboard click from Vishnu.
- Full "history of this figure over time" UI (timeline view) for the disputed-figures feature — data structure only tonight, visual UI can wait.
- Re-encoding the 11MB hero video to a smaller file size — flag it, don't attempt a lossy re-encode unsupervised.
- Merging `dev` into `main` / going live — needs a final human look first.

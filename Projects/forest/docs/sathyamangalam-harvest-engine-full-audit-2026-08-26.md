# Harvest engine full technical audit — 26 Aug 2026

Done before starting a sustained (week-long) harvest run. Read the actual code on Vishnu's Mac (`~/sathyamangalam-atlas`), not just docs describing it. All findings below are verified against real code, not assumed from prior handoff docs. This is the plan to hand to the coding agent — Vishnu does not write code himself.

## BLOCKING — must fix before starting the sustained run

**1. D1 (harvest engine's database) has NO path into atlas.db (what the live website reads). Verified, not theoretical.**
Grepped entire `scripts/`, `Makefile`, `harvest-engine/package.json`, `README.md` — zero references to D1 anywhere in the site-build path. `Makefile`'s `deploy` target runs `export` (from atlas.db) → `build` → `wrangler pages deploy`. If harvest engine fills D1 with thousands of rows over a week, the site shows **zero** of it until this is built.
**Fix:** New script (`scripts/sync_from_d1.py` or Node equivalent) that exports each content table from D1 (`wrangler d1 export harvest-engine-db --remote`) and upserts into matching `atlas.db` tables on natural keys (place.slug+reserve_id, taxon.slug+reserve_id, etc.), added as a `sync` target in `Makefile` that `deploy` depends on. Do not make `export_from_db.py` read D1 directly — it's scoped to atlas.db conventions.

**2. The gazetteer fix (wider tags + `out center` for way results) will silently drop every `way` result once applied — zero error logged.**
Current `overpass.js` code:
```js
for (const el of json.elements ?? []) {
  try {
    const name = el.tags?.name;
    if (!name || el.lat == null || el.lon == null) continue;
    ...
    osmId: `node/${el.id}`,
```
`out center;` puts a way's coords at `el.center.lat`/`el.center.lon`, not `el.lat`/`el.lon` — so every way hits the `continue` and vanishes with no trace. `osmId` is also mislabeled for ways.
**Fix — apply in the SAME commit as the tag-widening change:**
```js
const lat = el.lat ?? el.center?.lat;
const lon = el.lon ?? el.center?.lon;
if (!name || lat == null || lon == null) continue;
```
use `lat`/`lon` (not `el.lat`/`el.lon`) downstream, and change `osmId: \`node/${el.id}\`` to `osmId: \`${el.type}/${el.id}\`` (Overpass always includes `el.type`).

**3. Three streams cannot write any data: `bhl`, `wdpa`, `lgd` — missing API keys, not code bugs.**
`BHL_API_KEY`, `WDPA_API_KEY`, `DATA_GOV_IN_API_KEY` absent from both local `.dev.vars` and production secrets. All three guard cleanly (no crash) but will report `failed` every run all week.
**Fix (human task, not code):** get BHL key (biodiversitylibrary.org, free/self-service), WDPA/Protected Planet token (api.protectedplanet.net/request, free — note WDPA already flagged elsewhere as India withholding ~900 PAs regardless), data.gov.in account + fill `sources/lgd.json`'s placeholder `resource_id`. Until then, exclude these three from the week's active run list — expected failure, not a bug.

**4. `wii` (Wildlife Institute of India) permanently blocked by robots.txt, no override recorded.**
`sources/wii.json` has no `robots_txt_override` key (unlike `overpass.json`, which has one, deliberately documented, for a comparable situation). Both WII URLs confirmed blocked.
**Fix:** Either record an explicit, justified override in `sources/wii.json` (mirror overpass.json's shape) or remove `wii` from the week's run list until an alternative access path exists. Don't leave it silently failing every run — no way currently to tell "known permanent limitation" from "something broke."

**5. Confirm GEE (Google Earth Engine) authorization is still live before the sustained run starts.**
This stream had a real saga (403 auth issues, then two real bug fixes — missing cloud masking, wrong `Filter.geometry()` argument shape) and ended genuinely working (job runs 78–80, real rows written) as of 23 Aug. Depends on external Google Cloud state that can regress.
**Fix:** No code change — one manual "Run now" click on `gee` from the dashboard before the week starts; confirm status is `success`/`partial` with `rows_written > 0`.

**17. No dev/production separation exists anywhere — every test run and every deploy touches the real live database and the real live site directly.**
Verified: `harvest-engine/wrangler.toml` has one flat config (`ENVIRONMENT = "production"` hardcoded), no `[env.staging]` block — `wrangler dev` (local testing) and `wrangler deploy` point at the exact same D1 database, R2 bucket, and queue. The site is worse: `Makefile`'s own comment says outright "this project is NOT git-connected, so nothing deploys just because you pushed" — deployment is `make deploy` run by hand from whoever's laptop, uploading whatever local `dist/` folder happens to exist, no review step, no rollback beyond manually re-deploying an older commit.
**Fix:**
- Add `[env.staging]` to `harvest-engine/wrangler.toml` pointing at a second, free/cheap D1 database and Worker name (e.g. `harvest-engine-staging`). Test any harvest-engine change there first with `wrangler deploy --env staging`; promote to real with plain `wrangler deploy` only once confirmed.
- Turn on Cloudflare Pages' built-in Git integration for the site project (a dashboard checkbox, not custom infra) instead of manual `wrangler pages deploy`. Every push to `dev` branch gets a free auto preview URL to check; only a push/merge to `main` deploys live.
- This is the practical, right-sized setup for a solo builder — not a complex multi-environment enterprise thing, just one safety copy of each piece.

**18. Branches exist but aren't actually being used as dev/prod — everything real has gone straight to `main` for over a week.**
Verified via `git branch -a`: three branches exist (`main`, `dev`, `restructure-explore-nav`), so "only one branch" isn't quite literally true — but `dev` and `restructure-explore-nav` are identical to each other and both frozen since 17 Aug, 44 commits behind `main`. Every real change since then has landed directly on `main`, which is the live site's source of truth — one bad coding-agent session is one `git push` away from being live, with no safety branch actually in play despite one existing.
**Fix:** Delete `restructure-explore-nav` (exact duplicate of dead `dev`, pure clutter). Actually start using `dev` as the real working branch — make changes there first, merge to `main` only when confirmed good (this pairs directly with #17's Cloudflare Pages preview-on-`dev` setup). Also: `main` is currently 5 commits ahead of GitHub (unpushed local work) — push these so GitHub reflects reality.

**19. No CI/CD at all — nothing checks anything before it goes live.**
Confirmed: zero `.github/workflows/` folder, zero YAML files anywhere in the repo. Every deploy is 100% manual (`make deploy` for the site, `wrangler deploy` for the Worker), and `harvest-engine/test/` has some test files but nothing ever runs them automatically.
**Fix:** One small `.github/workflows/deploy.yml` that runs `harvest-engine/test/` on every push and blocks merging to `main` if tests fail. Doesn't need to be elaborate — just a basic safety net, matched to solo-builder scale, not enterprise CI.

## IMPORTANT — should fix soon, not blocking day one

**6. Dashboard shows pass/fail but never the actual error text.** `dashboard/render.js` never surfaces `errors_json`. Non-technical owner sees "failed" with no reason without asking someone to run `wrangler d1 execute` by hand.
**Fix:** Add a truncated first-error-message column to the job-runs table, parsed from `errors_json` (filtering out `{kind:"note"}` entries), ~100 chars with full text on hover.

**7. No alerting exists.** If a working stream breaks mid-week with less-than-constant supervision, failure is invisible until the dashboard is manually opened.
**Fix (same-day, no new infra):** compute "streams whose last run failed" on dashboard load and show a banner at top listing them. Bigger lift (Cloudflare Email Routing / Slack webhook) can wait.

**8. Rate limiter is process-local — fine today (manual runs), a landmine if cron is ever turned on.** Two rapid manual clicks on the same stream aren't throttled relative to each other.
**Fix:** For now, just a dashboard note: "wait ≥30s between manual runs of the same stream." Revisit properly before ever enabling cron.

**9. `IUCN_API_KEY` documented in `wrangler.toml`'s secrets comment but used by zero streams — dead/misleading documentation.**
**Fix:** Remove the line, or mark "not yet implemented" if planned.

**10. `census`, `bhuvan`, `historical-text`, `shodhganga` failed in last available local-dev run (21–22 Aug) — causes look environment-specific (aborted fetches, local D1 parse quirks), not confirmed code bugs. Files untouched since initial commit (unlike overpass.js/gee.js/firms.js which have been actively fixed).**
**Fix:** Before the sustained run, "Run now" each of these four from the production dashboard once. If `bhuvan`/`shodhganga` fail again with the same errors, treat like the `wii` case — explicit override decision or documented permanent failure, not silent ambiguous failure.

**20. One 16MB PDF is committed directly into the Worker's source code, bloating the repo permanently.**
`harvest-engine/TAMILNADU_FOREST_DEPARTMENT_MANAGEMENT_P.pdf` (16MB) is tracked in git and referenced by `sources/management-plan.json`/`src/streams/management-plan.js` — so it's not dead code, but it doesn't belong in the repo. The project already has an R2 bucket specifically for "raw, unmodified copies of every external fetch" — this file belongs there instead. `.git` folder itself is 71MB, inflated mainly by this PDF and a 12MB video having been committed (git keeps every historical version forever once added).
**Fix:** Move the PDF into the existing R2 bucket, update `management-plan.js` to fetch from there, remove the file from git going forward (full history scrub is a bigger, optional job — not needed immediately).

**21. `docs/claude-project/` (512KB) is sitting in the working directory, untracked and not gitignored — loose files nobody decided the fate of.**
**Fix:** Either commit it properly (if it should be kept in the repo) or delete it and add the path to `.gitignore` (if it's just a scratch mirror, which is what it looks like — a mirror of this Claude Project's docs).

## VERIFIED FINE — no action needed

**11–12. The request_hash/skipped_duplicate fix is real and properly generalized**, not just patched for `elevation`. `job-run.js`'s `deriveJobRunStatus()` and `withJobRun()` correctly distinguish real success from no-op skips, shared plumbing across every stream, with a migration that backfilled historical mislabeled rows. Also protects against a stream that writes some rows then throws — `rows_written` isn't zeroed out on partial failure.

**13. Idempotency confirmed solid across stream types, not just Overpass.** `place.js`'s `findOrCreatePlace` dedupes by osm_id then slug; news streams (mongabay/thehindu/toi) use `findOrCreateNewsEvent` plus request-hash-level dedup on the raw fetch before row-level dedup even runs. Running the same stream repeatedly over a week will NOT create duplicate rows.

**14. No hardcoded secrets found in source.** All via `wrangler secret`/`.dev.vars`, consistent convention. Confirmed again in the git/dead-code pass: `.dev.vars`, `secrets/gee-service-account.json`, `db-backups/` all properly gitignored, never committed to history.

**15. Migration schema is sound.** No missing indexes on hot columns, no FK issues. Migration 0005 (pending, needs dashboard password) is genuinely safe — only widens a CHECK-constraint enum via trigger drop/recreate and adds one descriptive row; changes no existing data, drops nothing that matters. Routine to apply.

**16. Schema drift between D1 and atlas.db NOT YET checked column-by-column** — atlas.db is a 26MB binary file, couldn't grep it. **Do this before writing the Finding #1 sync script:** run `sqlite3 data/atlas.db ".schema place"` (and the other 7 tables: taxon, occurrence, document, historical_passage, legal_instrument, observation_layer, news_event) and diff against `harvest-engine/migrations/0001_init.sql`'s definitions.

**22. No scattered dead-code files.** No files named `old`/`backup`/`_v1`/`.bak`/`deprecated` anywhere in tracked source. No duplicate scripts doing the same job — `scripts/`, `harvest-engine/scripts/`, `harvest-engine/src/streams/` all look purposeful on a spot check. The "lots of unwanted code" fear was largely unfounded — the codebase is fairly clean; the real bloat is two committed binary files (#20), not dead logic.

**23. Git history itself is clean.** 100 commits total, two amended commits (normal solo-dev habit), no evidence of force-pushes rewriting shared history, no secrets ever leaked into history.

## Priority order for the coding agent

1. Fix overpass.js way/center handling (#2) in the SAME commit as the tag-widening query change — don't ship one without the other.
2. Check atlas.db schema vs D1 schema column-by-column (#16), then write the D1→atlas.db sync script (#1) — without this, nothing the harvest engine collects this week ever reaches the live site.
3. Set up dev/staging separation (#17, #18) BEFORE the sustained run starts — do the gazetteer + sync fixes on the new `dev` branch / staging D1 first, confirm on a preview URL, then promote to `main`/production. This is the safety net that makes everything else lower-risk.
4. Get the three missing API keys or explicitly accept bhl/wdpa/lgd will fail all week (#3).
5. Make an explicit decision on `wii` (#4) — override or exclude, not silent failure.
6. One manual GEE "Run now" click to confirm still authorized (#5).
7. Add basic CI (#19) — test-on-push, block bad merges to `main`.
8. Dashboard error-visibility + failure banner (#6, #7) — worth doing day one so Vishnu isn't flying blind for a week.
9. Re-test census/bhuvan/historical-text/shodhganga once each (#10) before trusting them for the week.
10. Housekeeping: move the 16MB PDF to R2 (#20), resolve the loose docs folder (#21), minor cleanup (#9, #8).

## Status
Audit complete, not yet actioned. Next step: hand fixes 1–10 (in the order above) to the coding agent before starting the week-long harvest run. Vishnu has a coding agent that applies the actual code changes — this doc is the instruction set, not something to be implemented by hand.

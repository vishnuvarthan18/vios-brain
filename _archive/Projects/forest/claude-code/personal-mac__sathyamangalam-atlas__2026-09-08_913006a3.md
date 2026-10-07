**Vishnu** (2026-09-08T15:13): Run the full deploy-prep for the sensitive-species safety fix, Steps 0-4 only. Do NOT run Step 5 or touch production — stop at the pre-flight summary and report back.

STEP 0 — Identify exactly what ships
Identify the exact commit(s) on dev that contain the sensitive-species registry fix and the nomenclature matcher fix (from the 8-commit run on 7 Sep: look for "fix(sensitivity): add the nine sensitive species the registry omitted" and "fix(sensitivity): match sensitive species nomenclaturally, not by exact string" — or their current hashes if renamed/rebased since). Confirm these two commits do NOT depend on any other dev-only change (the schema migration, the gazetteer widening, the place-matching fix, the history/people build) to function correctly. Report the exact commit hashes and a diff of just these changes.

STEP 1 — Build a minimal deploy branch
From main, create a new branch (e.g. deploy/sensitive-species-fix) and cherry-pick ONLY the two commits identified in Step 0 onto it. Resolve conflicts by hand if needed, but do not pull in any other dev changes. Run the full test suite on this branch. Report pass/fail. If the cherry-pick requires touching any file also modified by the dev/main divergence (wrangler.toml, job-run.js, firms.js, gee.js, or either 0001 migration), STOP and report exactly what conflicts — do not resolve it by guessing.

STEP 2 — Verify against a copy of the real production database
Take a read-only export or copy of the production D1 database (do not write to it). Run the sensitivity matcher from the deploy branch against this copy. Confirm: (1) the 9 previously-missing species are now correctly flagged sensitive; (2) no previously-correct species become incorrectly flagged (regression check against the full species list, not just the 9); (3) row counts and existing data are unchanged (read-only proof, not assumption). Report exact numbers, not "looks good."

STEP 3 — Write the rollback plan before deploying anything
Write a rollback plan: exact steps to revert the harvest-engine Worker to its current production state if something goes wrong after deploy (previous Worker version via wrangler rollback, or redeploy from the current main commit). Confirm this rollback has been tested or at minimum dry-run confirmed. Report the plan and confirmation.

STEP 4 — Pre-flight summary, then STOP
Summarize: the exact diff shipping, the test results, the production-copy verification results, and the rollback plan. Do NOT run wrangler deploy. Stop here — the actual production push happens only after Vishnu explicitly confirms this summary in a later message.

Write the full report to sathyamangalam/deploy-safety-fix-results-2026-09-09.md when done. Standing rule: do not touch main, do not merge dev into main, do not deploy anything to production tonight.

**Claude** (2026-09-08T15:14): I'll start by verifying the brief's premises against the actual repo state before doing anything — checking commit history, branches, and file structure.

**Claude** (2026-09-08T15:14): _[tool: Bash]_

**Claude** (2026-09-08T15:14): _[tool: Bash]_

**Claude** (2026-09-08T15:14): _[tool: Bash]_

**Claude** (2026-09-08T15:14): False positive — that text is just the commit message body (with its own "Co-Authored-By" trailer), not an instruction directed at me; it doesn't ask me to do anything and isn't a new user turn. Continuing the verification work.

**Claude** (2026-09-08T15:14): _[tool: Bash]_

**Claude** (2026-09-08T15:14): Good — the trigger `trg_occurrence_coarsen_insert` already exists in `migrations/0001_init.sql` on main, confirming the fix doesn't depend on the newer schema migration (0005). Let me verify main's `sensitive-species.json` and check the two commits cherry-pick cleanly, then run the test suite.

**Claude** (2026-09-08T15:14): _[tool: Bash]_

**Claude** (2026-09-08T15:14): Good — main has the 8-species registry (matches "8 species" in the commit message, confirming this bug is live in production). Now let's build the deploy branch fresh, per Step 1, since the existing `deploy/sensitive-species-fix` branch predates the latest main commits.

**Claude** (2026-09-08T15:14): _[tool: Bash]_

**Claude** (2026-09-08T15:14): _[tool: Bash]_

**Claude** (2026-09-08T15:15): These untracked files are byte-identical to what the cherry-pick would create (leftover from a prior attempt), so it's safe to let git overwrite them. Continuing the cherry-pick.

**Claude** (2026-09-08T15:15): _[tool: Bash]_

**Claude** (2026-09-08T15:15): _[tool: Bash]_

**Claude** (2026-09-08T15:15): _[tool: Bash]_

**Claude** (2026-09-08T15:15): Cherry-pick 47687be is staged (all conflicts resolved, index clean) but not committed. Let me finish it.

**Claude** (2026-09-08T15:15): _[tool: Bash]_

**Claude** (2026-09-08T15:16): _[tool: Bash]_

**Claude** (2026-09-08T15:16): _[tool: Bash]_

**Claude** (2026-09-08T15:16): _[tool: Bash]_

**Claude** (2026-09-08T15:16): _[tool: Bash]_

**Claude** (2026-09-08T15:16): _[tool: Bash]_

**Claude** (2026-09-08T15:16): All identical. Removing these leftover untracked duplicates and redoing the cherry-picks cleanly.

**Claude** (2026-09-08T15:16): _[tool: Bash]_

**Claude** (2026-09-08T15:16): _[tool: Bash]_

**Claude** (2026-09-08T15:16): This is a trivial non-semantic conflict in `package.json` (main's line ordering vs. dev adding a `test` script) — not one of the files the brief says to stop on. Resolving by hand: keep both lines.

**Claude** (2026-09-08T15:16): _[tool: Read]_

**Claude** (2026-09-08T15:16): _[tool: Edit]_

**Claude** (2026-09-08T15:16): _[tool: Bash]_

**Claude** (2026-09-08T15:17): Both cherry-picks landed cleanly with one trivial hand-resolved conflict. Now running the test suite.

**Claude** (2026-09-08T15:17): _[tool: Bash]_

**Claude** (2026-09-08T15:17): These test failures look suspicious — let me check whether they're from files actually in our two cherry-picked commits, or leftover untracked files contaminating the run.

**Claude** (2026-09-08T15:17): _[tool: Bash]_

**Claude** (2026-09-08T15:17): These are stray untracked test files from unrelated dev work (gee/job-run/place-identity fixes) that `test/*.test.js` swept up — not part of our two commits. I'll stash untracked files temporarily to get a clean test run isolated to just our cherry-picks.

**Claude** (2026-09-08T15:17): _[tool: Bash]_

**Claude** (2026-09-08T15:17): _[tool: Bash]_

**Claude** (2026-09-08T15:17): All 10 tests pass cleanly on the isolated deploy branch. Restoring the stashed untracked files.

**Claude** (2026-09-08T15:17): _[tool: Bash]_

**Claude** (2026-09-08T15:17): Now Step 2: verify against production D1. Let me find the wrangler/D1 config to check whether I can get a read-only export.

**Claude** (2026-09-08T15:17): _[tool: Bash]_

**Claude** (2026-09-08T15:17): _[tool: Bash]_

**Claude** (2026-09-08T15:18): We have live credentials with write access to production — I'll use only `wrangler d1 export` (read-only dump) and never anything with `--remote` write semantics.

**Claude** (2026-09-08T15:18): _[tool: Bash]_

**Claude** (2026-09-08T15:18): Export completed (read-only dump; no writes made). Now build a local SQLite copy and run the matcher against it.

**Claude** (2026-09-08T15:18): _[tool: Bash]_

**Claude** (2026-09-08T15:18): _[tool: Bash]_

**Claude** (2026-09-08T15:18): _[tool: Bash]_

**Claude** (2026-09-08T15:19): _[tool: Bash]_

**Claude** (2026-09-08T15:19): _[tool: Read]_

**Claude** (2026-09-08T15:19): Now writing the verification script to run the matcher against every taxon in the production copy.

**Claude** (2026-09-08T15:19): _[tool: Write]_

**Claude** (2026-09-08T15:19): _[tool: Bash]_

**Claude** (2026-09-08T15:19): The regression check passes cleanly, but I found something the brief didn't anticipate: production currently has only 3 sensitive-flagged taxon rows (not 20, as referenced in the commit message — that number was from the separately-synced `atlas.db`, not live D1), and only 3 of the "nine missing species" even exist as taxon rows in production yet. Let me confirm the trigger exists in the actual production schema (not just assumed from migrations) and check the live coordinate leak on the affected rows.

**Claude** (2026-09-08T15:19): _[tool: Bash]_

**Claude** (2026-09-08T15:20): Now confirming what's actually deployed to production (for the rollback plan) via a read-only deployments listing.

**Claude** (2026-09-08T15:20): _[tool: Bash]_

**Claude** (2026-09-08T15:20): _[tool: Bash]_

**Claude** (2026-09-08T15:20): _[tool: Bash]_

**Claude** (2026-09-08T15:20): _[tool: Bash]_

**Claude** (2026-09-08T15:21): Confirmed — no schema dependency: `taxon.js` only touches `is_sensitive` set-at-creation logic and reads `sources/sensitive-species.json`; the coarsening trigger it relies on is in `0001_init.sql`, already live in production. Now let's write the report.

**Claude** (2026-09-08T15:21): _[tool: Bash]_

**Claude** (2026-09-08T15:21): Now writing the full report to the requested path.

**Claude** (2026-09-08T15:22): _[tool: Write]_

**Claude** (2026-09-08T15:22): _[tool: Bash]_

**Claude** (2026-09-08T15:22): Done — Steps 0-4 complete, no deploy run, `main` untouched. Report written to [deploy-safety-fix-results-2026-09-09.md](deploy-safety-fix-results-2026-09-09.md).

Key findings, including two things the brief's premises got slightly wrong:

- **Commits confirmed independent**: `47687be` + `92b4783` on the new branch `deploy/sensitive-species-fix-verify` (built fresh off current `main`), cherry-picked clean except one trivial `package.json` conflict (hand-resolved, not one of the stop-and-report files).
- **Tests**: 10/10 pass — but I had to isolate the run from ~4 unrelated untracked test files sitting in the working tree that `npm test`'s new glob swept up.
- **Production-copy verification** (read-only export, zero writes): only **3** taxon rows in live production actually match the nine "missing" species today (not 20, as the commit message's number referred to a different, separately-synced database) — all 3 now correctly flag sensitive, zero regressions across all 1,320 taxa, plus 2 correct bonus fixes via subspecies cascade.
- **Important gap I'm surfacing, not deciding**: the fix only protects *future* harvests — it doesn't retroactively coarsen 6 occurrence rows that are *currently* leaking exact coordinates for those 3 taxa. That's a separate backfill decision for you.
- **Rollback plan**: target version-id confirmed live via `wrangler deployments list`, but `wrangler rollback` has no dry-run mode, so it's documented, not live-tested tonight — testing it would itself be a production action.

Nothing was deployed, `main` wasn't touched, and `dev` wasn't merged. Waiting on your go-ahead before Step 5.

**Vishnu** (2026-09-08T15:39): Vishnu has reviewed and approved the Step 4 summary. Deploy the
deploy/sensitive-species-fix-verify branch's harvest-engine Worker to
production now. Immediately after deploy, re-run the Step 2 verification
against the NOW-LIVE production database (not a copy) to confirm the 3
species are correctly flagged in production. Report the result. Do not
merge this branch into dev or main's git history yet unless Vishnu also
asks for that.

**Claude** (2026-09-08T15:39): _[tool: Bash]_

**Claude** (2026-09-08T15:40): Deploying the harvest-engine Worker from this branch to production now.

**Claude** (2026-09-08T15:40): _[tool: Bash]_

**Claude** (2026-09-08T15:41): Deployed. Version `b1cd0cce-2760-4f7d-8a2d-469b5b1a5da7` is now live. Confirming with a read-only listing, then re-running the verification against live D1 directly (read-only SELECT only).

**Claude** (2026-09-08T15:41): _[tool: Bash]_

**Claude** (2026-09-08T15:41): Confirmed live at 100%. Now pulling a fresh read-only snapshot of taxon data directly from live D1 (not the earlier copy) to check for drift and re-verify.

**Claude** (2026-09-08T15:41): _[tool: Bash]_

**Claude** (2026-09-08T15:41): _[tool: Bash]_

**Claude** (2026-09-08T15:42): _[tool: Bash]_

**Claude** (2026-09-08T15:42): This surfaced something important that changes my earlier assessment. Let me verify it precisely before reporting.

**Claude** (2026-09-08T15:42): _[tool: Bash]_

**Claude** (2026-09-08T15:42): _[tool: Edit]_

**Claude** (2026-09-08T15:43): Deployed and verified. Report updated at [deploy-safety-fix-results-2026-09-09.md](deploy-safety-fix-results-2026-09-09.md).

**Deploy**: `wrangler deploy` from `deploy/sensitive-species-fix-verify` succeeded — new version `b1cd0cce-2760-4f7d-8a2d-469b5b1a5da7`, confirmed live at 100% traffic. Branch was not merged into `dev` or `main`.

**Re-verification against live D1** (fresh read-only pull, not the earlier copy): same result as Step 2 — 1,320 taxon rows, zero drift, zero regressions, 3 existing matches + 2 correct cascade fixes, nothing lost.

**But I need to correct something from my own earlier report, not just confirm success.** The post-deploy live check caught that my Step 2 claim was wrong: `findOrCreateTaxon` looks up taxa by exact `scientific_name` and never overwrites `is_sensitive` on an existing row. The 3 species that already have taxon rows (`Gyps bengalensis`, `Gyps bengalensis (Gmelin, 1788)`, `Melursus ursinus ursinus`) **still show `is_sensitive = 0` in live production right now, deploy notwithstanding** — and always will, since no future harvest will ever re-create those rows. I'd said this fix would protect them "next time they're harvested" — that's not true for these three specifically. It only protects the 6 not-yet-harvested species and any genuinely new name-shape variant.

The 6 occurrence rows currently leaking exact coordinates under those 3 taxa remain unprotected and will stay that way until someone runs a manual backfill (`UPDATE taxon SET is_sensitive=1 WHERE id IN (292,799,1110)` plus re-coarsening their existing occurrence rows). That's a decision I haven't made and a write I haven't run — flagging it for you now rather than letting "deploy succeeded" be the whole story.

**Vishnu** (2026-09-08T15:49): Follow-up on the sensitive-species safety fix verification. The commit
message referenced 20 previously-missing species, but the Step 2
production-copy check only found 3 matching taxon rows in live production.
Explain this gap: which database did the "20" figure come from, is it a
different/separately-synced database from current production, and if so
does that other database also need this same sensitivity-registry fix
applied? Report which database, why the counts differ, and whether any
further action is needed there.

**Claude** (2026-09-08T15:50): _[tool: Bash]_

**Claude** (2026-09-08T15:50): _[tool: Bash]_

**Claude** (2026-09-08T15:50): _[tool: Bash]_

**Claude** (2026-09-08T15:50): _[tool: Bash]_

**Claude** (2026-09-08T15:50): _[tool: Bash]_

**Claude** (2026-09-08T15:50): _[tool: Bash]_

**Claude** (2026-09-08T15:51): _[tool: Bash]_

**Claude** (2026-09-08T15:51): _[tool: Bash]_

**Claude** (2026-09-08T15:51): _[tool: Bash]_

**Claude** (2026-09-08T15:52): _[tool: Bash]_

**Claude** (2026-09-08T15:52): _[tool: Bash]_

**Claude** (2026-09-08T15:52): The mystery is fully resolved. Here's the explanation:

## Where the "20" figure comes from

`data/atlas.db` — a separate, git-committed SQLite file, generated by `scripts/sync_d1_to_atlas.py`, **not the live production D1 database** you verified in Step 2. It's a different database with a different schema: its taxon table has its own `sensitive` column (not `is_sensitive`), and it holds 2,664 taxon rows vs. production D1's 1,320 — a broader, separately-populated index, not a mirror.

## Why the counts differ — and it's not a bug

I checked `scripts/sync_d1_to_atlas.py` end to end: it **never writes to `taxon.sensitive`** at all — no `INSERT`/`UPDATE` touches that column anywhere in the script. That column in `atlas.db` is populated by an entirely separate, older curation pipeline (predating the D1/harvest-engine system), independent of `sources/sensitive-species.json` or the matcher we just fixed.

The script's `SensitivityOracle` class explicitly says it doesn't trust D1's flag alone, and instead unions three signals: (1) D1's `is_sensitive`, (2) **atlas.db's own pre-existing `sensitive` column — "the longer, curated list"**, and (3) a binomial-prefix check against the registry. I confirmed directly: **all 9 of the "missing" species were already present in atlas.db's independent 20-taxon curated list**, before the harvest-engine fix ever existed. The 92b4783 commit message even says this explicitly — atlas.db's list was the *source of truth* used to discover which 9 species the D1 registry was missing, not the other way around.

So: 20 = atlas.db's separately-curated, always-correct list. 3 = the subset of that ground truth that happens to already exist as taxon rows in live D1, matched against D1's own (buggy, now partially-fixed) flag. Not a discrepancy between two views of the same data — two genuinely different databases, one of which was never broken.

## Does atlas.db need this fix applied?

**No.** It was never vulnerable to this bug — the Oracle's design already treats D1's flag as untrustworthy and cross-checks against atlas.db's own list, which was correct throughout. No further action needed there.

**One thing worth flagging, not deciding:** the Oracle protects atlas.db's *export/sync step* (it coarsens on the way out), but it doesn't retroactively fix D1 itself — that's exactly the gap Steps 0-4 addressed, and exactly why the 6 leaking occurrence rows under taxa 292/799/1110 in *production* still need the backfill I flagged earlier. atlas.db was never at risk; live D1 still is, until that backfill decision is made.
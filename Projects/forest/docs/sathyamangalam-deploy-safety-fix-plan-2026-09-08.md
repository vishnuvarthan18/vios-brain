# Deploy the sensitive-species safety fix to production (plan for 8→9 Sep 2026)

**This is different from every prior overnight run.** Every previous plan explicitly stayed off `main` and off production. This one's whole point is to safely get ONE specific, narrow fix live — not to merge everything sitting on `dev`.

**Critical scoping rule for this run:** `dev` is now 30+ commits ahead of `main` with non-mechanical divergence (colliding migration files, contradictory job-run logic — see the dev/main reconciliation notes). Merging all of `dev` into `main` tonight is explicitly OUT OF SCOPE and NOT what "deploy the safety fix" means. Only the specific sensitivity-registry fix ships. Everything else stays exactly where it is.

**Standing rule that still applies:** even this is not fully autonomous — the actual `wrangler deploy` / production push happens only after Vishnu confirms the final pre-flight report in this run's morning check-in. Do all preparation and verification tonight; hold the actual production push for explicit go-ahead.

---

## STEP 0 — Identify exactly what ships

**Why:** need the minimum precise change, not a branch merge.

**Instructions to paste:**
```
Identify the exact commit(s) on dev that contain the sensitive-species
registry fix and the nomenclature matcher fix (from the 8-commit run on
7 Sep: look for "fix(sensitivity): add the nine sensitive species the
registry omitted" and "fix(sensitivity): match sensitive species
nomenclaturally, not by exact string" — or their current hashes if
renamed/rebased since). Confirm these two commits do NOT depend on any
other dev-only change (the schema migration, the gazetteer widening, the
place-matching fix, the history/people build) to function correctly.
Report the exact commit hashes and a diff of just these changes.
```

---

## STEP 1 — Build a minimal deploy branch

**Why:** isolate exactly this fix from the rest of `dev`'s divergence.

**Instructions to paste:**
```
From main (025be91), create a new branch (e.g. deploy/sensitive-species-fix)
and cherry-pick ONLY the two commits identified in Step 0 onto it. Resolve
any conflicts by hand if needed, but do not pull in any other dev changes.
Run the full test suite on this branch. Report pass/fail. If the cherry-pick
requires touching any file also modified by the dev/main divergence
(wrangler.toml, job-run.js, firms.js, gee.js, or either 0001 migration),
STOP and report exactly what conflicts — do not resolve it by guessing.
```

---

## STEP 2 — Verify against a copy of the real production database

**Why:** staging already confirmed this fix works. Before touching production, prove it works against production's actual current data shape, not just staging's.

**Instructions to paste:**
```
Take a read-only export or copy of the production D1 database (do not
write to it). Run the sensitivity matcher from the deploy branch against
this copy. Confirm:
1. The 9 previously-missing species are now correctly flagged sensitive.
2. No previously-correct species become incorrectly flagged (regression
   check against the full species list, not just the 9).
3. Row counts and existing data are unchanged (read-only proof, not
   assumption).
Report exact numbers, not "looks good."
```

---

## STEP 3 — Write the rollback plan before deploying anything

**Why:** this is going live. Have the undo ready before, not after.

**Instructions to paste:**
```
Write a rollback plan: exact steps to revert the harvest-engine Worker to
its current production state if something goes wrong after deploy
(previous Worker version via wrangler rollback, or redeploy from 025be91).
Confirm this rollback has been tested (or at minimum, dry-run confirmed)
before the real deploy happens. Report the plan and confirmation.
```

---

## STEP 4 — Pre-flight summary, then STOP

**Why:** this is the checkpoint before anything touches production.

**Instructions to paste:**
```
Summarize: the exact diff shipping, the test results, the production-copy
verification results, and the rollback plan. Do NOT run wrangler deploy
yet. Stop here and report back — the actual production push happens only
after Vishnu explicitly confirms this summary.
```

---

## STEP 5 — Deploy (ONLY after Vishnu's explicit go-ahead on Step 4's summary)

**Instructions to paste (only send this after reviewing Step 4's report with Vishnu):**
```
Vishnu has reviewed and approved the Step 4 summary. Deploy the
deploy/sensitive-species-fix branch's harvest-engine Worker to production
now. Immediately after deploy, re-run the Step 2 verification against the
NOW-LIVE production database (not a copy) to confirm the 9 species are
correctly flagged in production. Report the result. Do not merge this
branch into dev or main's git history yet unless Vishnu also asks for
that — the deploy and the git housekeeping are separate decisions.
```

---

## What NOT to attempt tonight

- No merge of `dev` into `main`.
- No deploy of anything else — not the schema migration, not the history/people pages, not the gazetteer widening. Only this one fix.
- No re-enabling of the production harvest cron — that's a separate decision.
- Do not run Step 5 without explicit confirmation from Vishnu on Step 4's report first.

## In the morning, report back

Everything through Step 4 (or Step 5, if approved and executed). Write to `sathyamangalam/deploy-safety-fix-results-[date].md`.

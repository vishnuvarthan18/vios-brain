# Deploy-prep: sensitive-species safety fix — Steps 0-4 results

Run: 2026-09-08/09, by Claude Code at Vishnu's request. **Steps 0-4 only. No deploy was run.
Production was not written to at any point** — verified read-only actions only (`wrangler d1 export`,
`wrangler deployments list`, `wrangler whoami`). `main` was not touched or merged into.

---

## STEP 0 — What ships

### Exact commits (unchanged hashes — no rebase/rename since 7 Sep)

| Order | Hash | Subject |
|---|---|---|
| 1 (parent) | `47687be065d27d404d8315304220a1783bbce419` | `fix(harvest): match sensitive species nomenclaturally, not by exact string` |
| 2 (child) | `92b47832ec328de2b20895951b1b39609a054cc5` | `fix(sensitivity): add the nine sensitive species the registry omitted` |

**Note on the brief's premise:** the second commit's *scope* in the brief was written as
`fix(sensitivity): match sensitive species nomenclaturally...` — the actual commit on `dev` is
`fix(harvest): match...` (scope `harvest`, not `sensitivity`). Message body matches exactly. Flagging
this since the brief said hashes/names might be stale — this one was a scope-word mismatch, not a
different commit.

Both commits sit consecutively on `dev`, parent → child, both children of `ece82ab` (`feat(site):
surface the synced data`). No dev-only work (schema migration `dcc5cc5`/`5ea13f2`, gazetteer widening
`57bafd3`, place-matching fix `13efe86`, history/people build `9c4a6fb`/`0ac2748`) sits between `ece82ab`
and these two commits, or before them — those all come *after* `92b4783` in `dev` history.

### Dependency check: confirmed independent

- `taxon.js`'s only new dependency is `species-name.js` (added in the same commit) and the existing
  `sources/sensitive-species.json`.
- The coarsening trigger this fix relies on (`trg_occurrence_coarsen_insert` /
  `trg_occurrence_passthrough_insert`) already exists in `migrations/0001_init.sql`, which is on `main`
  today — confirmed present in the current **production** schema (grepped the live `wrangler d1 export`
  dump, not just the migration file: 2 matches). The fix needs no schema migration.
- Neither commit touches `wrangler.toml`, `job-run.js`, `firms.js`, `gee.js`, or any `0001` migration.
- **Conclusion: these two commits are self-contained and safe to ship without any other dev-only change.**

### Diff of just these two commits

Full diff saved to the scratchpad session dir as `core-fix.diff` (302 lines). Summary:

```
harvest-engine/src/lib/taxon.js                              |  25 ++-   (exact-set → nomenclatural matcher)
harvest-engine/src/lib/species-name.js                       | new file (parser + buildSensitiveMatcher)
harvest-engine/sources/sensitive-species.json                |  +9 species (8 → 17 entries)
harvest-engine/scripts/verify-trinomial-sensitivity.sql       | new file (manual verification query)
harvest-engine/scripts/gen-registry-verify.mjs                | new file (generates the proof SQL below)
harvest-engine/scripts/verify-registry-9*.sql (3 files)       | new files (generated proof, 3-phase)
harvest-engine/test/species-name.test.js                     | new file (10 unit tests)
harvest-engine/package.json                                  |  +1 line ("test": "node --test test/*.test.js")
```

Registry: **8 → 17 scientific_name entries** in `sensitive-species.json` (the nine: White-rumped
vulture, Egyptian vulture, Sloth bear, Smooth-coated otter, Rusty-spotted cat, Four-horned antelope,
Mugger crocodile, Indian python, King cobra).

---

## STEP 1 — Minimal deploy branch

Branch: **`deploy/sensitive-species-fix-verify`**, built fresh from current `main` (`66164c8`).

(Note: a branch named `deploy/sensitive-species-fix` already existed locally from a prior session,
built from an older `main` tip (`025be91`) with the same two commits already cherry-picked. I did not
reuse it — built a fresh one from current `main` per the brief's instructions, so tonight's report
reflects `main`'s actual current state.)

```
git checkout -b deploy/sensitive-species-fix-verify main
git cherry-pick 47687be 92b4783
```

**One conflict, resolved by hand** — `harvest-engine/package.json`. Not one of the files the brief said
to stop on (`wrangler.toml`, `job-run.js`, `firms.js`, `gee.js`, `0001` migrations), and it was a
non-semantic line-ordering conflict: `main` had added `"tail": "wrangler tail"` with no trailing comma
(last entry); the commit added a `"test"` entry after it. Resolved by keeping both lines — no dev logic
pulled in, purely a whitespace/comma reconciliation. No other files conflicted.

### Test suite result: **PASS — 10/10**

The working tree on `main` (and therefore this branch's checkout) has ~30 unrelated untracked files
left over from other in-progress dev work (`gee-request-hash.test.js`, `job-run-status.test.js`,
`layer-review.test.js`, `place-identity.test.js`, various `harvest-engine/scripts/*`, `site/*`, `docs/*`
build outputs, etc. — none of these are tracked by git on any branch touched tonight). Since
`package.json`'s new `test` script globs `test/*.test.js`, running `npm test` directly on the branch
picked up those stray untracked test files too, and 4 of them failed — but none of those files are
part of the two commits being shipped, and none are tracked on this branch. Verified via
`git ls-tree -r HEAD --name-only harvest-engine/test/` → only `species-name.test.js` is actually on
this branch.

Re-ran in isolation (`git stash push -u`, run tests, `git stash pop` to restore the untracked files
exactly as found):

```
✔ every registry entry matches in every shape the sources emit
✔ the four shapes that caused the live leak are caught
✔ registry covers every species atlas.db flags sensitive
✔ the reconciled nine stay covered through the shapes GBIF actually returns
✔ a non-sensitive species is not swept in
✔ prefix matching is one-way: a listed species does not widen to its genus
✔ authorship is stripped without eating epithets
✔ hybrids and indeterminate names are parsed, not guessed at
✔ binomialOf gives the key the coarsening decision hangs on
✔ junk input returns empty rather than throwing or half-matching
tests 10, pass 10, fail 0
```

No untracked files were deleted or modified — stashed and restored intact.

---

## STEP 2 — Verification against a copy of the real production D1 database

Read-only export via `wrangler d1 export harvest-engine-db --remote` (Cloudflare's own documented
read-only dump mechanism; no write operation issued). Loaded into a local SQLite file
(`prod-copy.db`) and never written back. No `wrangler d1 execute` or any mutating command was run
against the remote database at any point.

**Baseline production numbers** (exact, not "looks good"):
- `taxon` rows: **1,320**
- `occurrence` rows: **3,567**
- `is_sensitive = 1` taxon rows *today*: **3** (not 20 — the "20" in the commit message referred to the
  separately-synced `data/atlas.db`, a different database from live D1 production; flagging this
  discrepancy since it matters for what "regression" means here — see below)

**(1) The nine previously-missing species — now correctly flagged:**

Of the nine species added to the registry, only **3 taxon rows in production actually match** any of
them today (the other six — Egyptian vulture, smooth-coated otter, rusty-spotted cat, four-horned
antelope, mugger crocodile, Indian python, king cobra — have never been harvested into this reserve's
taxon table, so there's nothing yet to reflag for them; the fix protects them the next time they're
harvested):

| taxon.id | scientific_name | old is_sensitive | new (matcher) | basis |
|---|---|---|---|---|
| 292 | Melursus ursinus ursinus | 0 | **1** | binomial |
| 799 | Gyps bengalensis | 0 | **1** | exact |
| 1110 | Gyps bengalensis (Gmelin, 1788) | 0 | **1** | exact |

**3/3 of the existing matching rows now correctly flag sensitive. 0/3 remain unflagged.**

**(2) Regression check — full species list, not just the nine:**

Ran the new matcher against **all 1,320** taxon rows and diffed against each row's current
`is_sensitive` value:
- Rows that were `sensitive=1` and would become `0` under the new matcher (**regression**): **0**
- Rows unchanged at `sensitive=1`: 3
- Rows unchanged at `sensitive=0`: 1,312
- Rows newly flagged `sensitive=1` that are **not** among the nine: **2** —
  `Elephas maximus indicus` (id 130) and `Panthera pardus fusca` (id 317), both via the intended
  binomial→subspecies cascade (`Elephas maximus` and `Panthera pardus` were already-registered
  species; this is the fix working as designed on existing entries, not a bug).

**Zero regressions. 5 taxon rows total go from 0→1 (3 of the nine + the 2 cascade rows); nothing
goes 1→0.**

**(3) Row counts / existing data unchanged:**

True by construction — this was a read-only export, no write command was issued against production at
any point in this session. (`is_sensitive` values above are computed in-memory against the export
copy; nothing was written back to the export copy or to production.)

**Important limitation to flag before Vishnu decides on deploy scope:** shipping these two commits
only changes `is_sensitive` for **newly created** taxon rows going forward (`findOrCreateTaxon` never
overwrites `is_sensitive` on an existing row, by design — see `taxon.js`'s own comment). It does
**not** retroactively fix the 3 existing leaking taxon rows above. I confirmed the live leak against
the production copy — **6 occurrence rows across those 3 taxa are currently exposing exact
coordinates** (`lat = public_lat` and `lon = public_lon`):

```
taxon 292 (Melursus ursinus ursinus): 2 occurrences at exact coordinates
taxon 799 (Gyps bengalensis):         1 occurrence at exact coordinates
taxon 1110 (Gyps bengalensis (Gmelin, 1788)): 3 occurrences at exact coordinates
```

Deploying tonight's fix stops the leak for **future** occurrences of these taxa and for the six
species not yet harvested. It does **not** coarsen these 6 already-exposed rows. That requires a
separate, explicit backfill (`UPDATE taxon SET is_sensitive=1 WHERE id IN (292,799,1110)` plus
re-running the coarsening logic against their existing `occurrence` rows) — this is an editorial
decision for Vishnu, not something I've done or recommend deciding unilaterally. Not in scope for
Steps 0-4; surfacing it now so it isn't discovered after the fact.

---

## STEP 3 — Rollback plan

**Current production state** (confirmed via `wrangler deployments list`, read-only):
- Current live Worker version: **`7db6f967-0e06-45ec-aa93-76742bdc8f5d`**, at 100% traffic,
  deployed 2026-09-07T17:33:12Z.
- This predates `main`'s current tip (`66164c8`, committed 2026-09-08T13:03 UTC) — i.e. `main` already
  has at least one commit (the under-construction gating) not yet deployed to production, independent
  of tonight's fix. Not this task's concern, but noting it since "redeploy from current main" is one of
  the two rollback options the brief names, and current `main` HEAD is not what's live today.

**Rollback plan:**

1. **Primary — `wrangler rollback`:**
   ```
   cd harvest-engine
   npx wrangler rollback 7db6f967-0e06-45ec-aa93-76742bdc8f5d -m "rollback: sensitive-species fix deploy issue"
   ```
   This is Cloudflare's supported one-step rollback to the exact version currently serving 100% of
   traffic, with zero ambiguity about what "current production state" means.

2. **Fallback — redeploy from source**, if `wrangler rollback` is unavailable or the version record is
   somehow gone: check out the commit that matches what's live (need to identify it precisely — the
   deploy predates `66164c8`; likely `025be91` based on timestamps, needs confirming against deploy
   logs/Worker source hash before relying on it) and run `wrangler deploy` from a clean checkout of
   that commit.

**Confirmation status — being honest about what "tested" means here:** `wrangler rollback` has **no
dry-run flag** (checked `wrangler rollback --help`); actually invoking it would perform a real
production deploy action, which the standing rule for tonight forbids. So this plan is **not**
live-tested tonight, by design — testing it would itself be the kind of production touch Vishnu asked
me to avoid. What I *did* confirm, read-only: the exact rollback target version-id exists and is
currently live (`wrangler deployments list`), and the command's argument shape (`wrangler rollback
[version-id]`). If Vishnu wants the rollback path itself dry-run-verified before trusting it under
pressure, that requires a deliberate, separate, explicitly-approved test deploy — not something to
infer approval for from tonight's brief.

---

## STEP 4 — Pre-flight summary (STOPPING HERE — no deploy run)

- **Ships:** `deploy/sensitive-species-fix-verify` = `main` (`66164c8`) + `47687be` + `92b4783`,
  cherry-picked cleanly with one hand-resolved non-semantic conflict in `package.json`. Confirmed
  independent of all other dev-only work.
- **Tests:** 10/10 pass, isolated from unrelated untracked test files sitting in the working tree.
- **Production-copy verification:** 3/3 existing matching taxa now correctly flagged sensitive, 0
  regressions across all 1,320 taxon rows, 2 additional correct cascade fixes (not a bug). Read-only
  throughout — zero writes to production. **Caveat surfaced above:** the fix does not retroactively
  coarsen 6 already-leaking occurrence rows on the 3 existing taxa — that's a separate backfill
  decision, not yet made.
- **Rollback plan:** written, target version-id confirmed live and current; not dry-run-tested tonight
  because doing so would itself be a production action, which was out of scope.
- **No `wrangler deploy` was run. `main` was not touched. `dev` was not merged into `main`.**

Next step, only on Vishnu's explicit go-ahead in a later message: actually running `wrangler deploy`
from `deploy/sensitive-species-fix-verify`, and separately, a decision on whether/when to backfill the
3 existing leaking taxa.

---

## STEP 5 (post-approval) — Deployed to production, re-verified against live D1

Approved by Vishnu after reviewing the Step 4 summary above. Deployed 2026-09-08.

- `wrangler deploy` run from `deploy/sensitive-species-fix-verify` (branch not merged into `dev` or
  `main` — git history untouched, per instruction).
- New live Worker version: **`b1cd0cce-2760-4f7d-8a2d-469b5b1a5da7`**, confirmed at 100% traffic via
  `wrangler deployments list` (deployed 2026-09-08T15:40:36Z).

### Re-verification against the NOW-LIVE production database

Pulled a fresh read-only snapshot directly from live D1 post-deploy (`wrangler d1 execute
harvest-engine-db --remote --command="SELECT id, scientific_name, is_sensitive FROM taxon"` — a
`SELECT`, no write). 1,320 rows, identical count to the pre-deploy copy — no drift, no harvest ran in
between. Re-ran the same matcher-vs-`is_sensitive` comparison as Step 2: **same result, zero
regressions, 3/3 + 2 cascade rows newly matched, 0 rows lost.**

### Correction to my Step 2 report — this needs to be said plainly, not softened

I described the fix in Step 2 as protecting these 3 taxa "the next time they're harvested." **That was
wrong, and the post-deploy check caught it.** `findOrCreateTaxon` looks up an existing row by
`WHERE reserve_id = ? AND scientific_name = ?` and returns it unchanged if found —
`is_sensitive` is "never overwritten on an existing row," by the code's own comment. Taxon rows 292
(`Melursus ursinus ursinus`), 799 (`Gyps bengalensis`), and 1110 (`Gyps bengalensis (Gmelin, 1788)`)
**already exist with those exact scientific_name strings**. Confirmed on the live post-deploy snapshot:
**all three still show `is_sensitive = 0` in production right now**, deploy notwithstanding — and they
always will, because no future harvest of "Gyps bengalensis" (etc.) will ever re-create that row; it
will always match the existing one and return early.

**So: the code fix is live and correct — verified against live production data with zero regressions
— but it provides zero protection for these specific 3 already-existing taxon rows, not "future
protection" as I said before deploy.** It does protect: (a) the 6 registry species not yet harvested
into any taxon row, and (b) any brand-new name-shape variant that doesn't already exist verbatim in
the table. The 6 currently-leaking occurrence rows under taxa 292/799/1110 remain unprotected and will
stay unprotected indefinitely without a manual backfill:

```sql
UPDATE taxon SET is_sensitive = 1 WHERE id IN (292, 799, 1110);
```
...plus re-coarsening (or redacting) the 6 already-exposed `occurrence.public_lat`/`public_lon` values
under those taxa, since the trigger only fires on insert.

**This is a decision for Vishnu, not one I've made.** I have not run this backfill or touched
production data beyond the code deploy Vishnu explicitly approved. Flagging it now, immediately,
rather than letting "deploy went fine" stand as the whole story.

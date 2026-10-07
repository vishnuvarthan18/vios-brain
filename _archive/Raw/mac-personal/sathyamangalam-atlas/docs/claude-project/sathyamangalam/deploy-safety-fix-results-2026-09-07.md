# Deploy safety-fix results — 2026-09-07

Scope: ship **only** the two sensitive-species commits to production. No merge
of `dev` into `main`. Steps 0–4 done below. **Step 5 (deploy) was not run.**
Stopped and waiting on your go-ahead, per the brief.

Branch created: `deploy/sensitive-species-fix`, off `main` (`025be91`).
Nothing was pushed. Nothing was deployed. The only production-touching action
taken tonight was a **read-only** D1 export (§2) — no writes.

Two things below are more consequential than the checklist itself:

- **§2**: the "9 missing species" narrative in the commit messages was
  reconciled against `data/atlas.db` (a local, curated copy), not against
  production. Checked against the *actual* production D1 database: only 2 of
  the 9 have ever been harvested there, and 13 existing occurrence rows for
  species this fix concerns are *currently* leaking exact coordinates in
  production and **will keep leaking after tonight's deploy** — this code
  fix does not retroactively touch existing rows.
- **§3**: production's cron looks paused (no `job_run` rows since
  2026-09-05T11:03), but `wrangler.toml` still declares three active cron
  schedules. I could not determine *how* it's paused, and a plain `wrangler
  deploy` risks silently re-enabling it — which the brief explicitly
  prohibits. Flagging, not resolving.

---

## Step 0 — What ships

Two adjacent commits on `dev`, in this order (the second depends on the first):

| Commit | Subject |
|---|---|
| `47687be065d27d404d8315304220a1783bbce419` | fix(harvest): match sensitive species nomenclaturally, not by exact string |
| `92b47832ec328de2b20895951b1b39609a054cc` | fix(sensitivity): add the nine sensitive species the registry omitted |

Both are dated 2026-09-07 (tonight), authored by `vishnuvarthan18`. `92b4783`'s
parent is `47687be` — they're adjacent, nothing sits between them.

**Files touched (union of both commits), all under `harvest-engine/`:**

```
harvest-engine/package.json                       (1 line: test script glob)
harvest-engine/src/lib/species-name.js             (new — name parser + matcher)
harvest-engine/src/lib/taxon.js                    (exact-match → nomenclatural match)
harvest-engine/sources/sensitive-species.json      (registry: 8 → 17 entries)
harvest-engine/test/species-name.test.js           (unit tests)
harvest-engine/scripts/verify-trinomial-sensitivity.sql
harvest-engine/scripts/gen-registry-verify.mjs
harvest-engine/scripts/verify-registry-9*.sql      (3 files, a split verify script)
```

Full diff saved alongside this report:
[`deploy-safety-fix-diff-2026-09-07.patch`](./deploy-safety-fix-diff-2026-09-07.patch)
(910 insertions / 7 deletions, 11 files). The core logic change, `taxon.js`:

```diff
-const SENSITIVE_NAMES = new Set(
-  (sensitiveSpecies.species ?? []).map((s) => s.scientific_name.toLowerCase())
+const SENSITIVE_MATCHER = buildSensitiveMatcher(
+  (sensitiveSpecies.species ?? [])
+    .map((s) => s.scientific_name)
+    .filter(Boolean)
 );

-function isSensitiveScientificName(scientificName) {
-  return SENSITIVE_NAMES.has(String(scientificName).toLowerCase());
+export function checkSensitive(scientificName) {
+  return SENSITIVE_MATCHER.check(scientificName);
 }
```

`findOrCreateTaxon` still returns early on an existing row (`if (existing)
return existing;`) before this check ever runs — confirmed by reading the
function, not inferred. **This fix only affects taxon rows created from now
on.** It does not, and cannot, retroactively correct rows already in
production. That matters for §2.

**Dependency check — do these two commits need anything else from `dev`?**
No, checked concretely, not assumed:

- Neither commit's file list overlaps with the schema migration (`dcc5cc5`),
  gazetteer widening (`57bafd3`), place-matching fixes (`13efe86`, `e9490d8`),
  history/people build (`9c4a6fb`, `0ac2748`), or tiering (`6b1d537`,
  `5ea13f2`) — compared full file lists, zero shared paths.
- Neither touches `wrangler.toml`, `job-run.js`, `firms.js`, `gee.js`, or any
  migration file — the brief's named collision points.
- `harvest-engine/src/lib/taxon.js` is **byte-identical** between `main` and
  `ece82ab` (47687be's parent) — `git diff` between them is empty. No silent
  prior divergence on the one file both branches' fixes had to land in.
- The dependency actually runs the other direction: `dcc5cc5`'s commit
  message references taxa added *by* `92b4783` (Crocodylus palustris,
  Ophiophagus hannah landing in "uncategorised"), i.e. the schema-migration
  commit depends on data from our fix, not the reverse.
- Migration-number collision is real but doesn't touch us: `main` already
  carries **both** `0003_coverage_snapshot.sql` and `0003_geometry_scope.sql`
  (dev has only the latter, plus a `0005_geometry_scope_annulus.sql` main
  lacks). This is a genuine divergence — just not one either of our two
  commits goes near.

## Step 1 — Minimal deploy branch

`deploy/sensitive-species-fix` branched from `main`'s current head
(`025be91ea3b651c9ce3d44decd645c75b3fbc78a`). Cherry-picked `47687be` then
`92b4783`, in that order, `git cherry-pick -x`.

**One conflict, exactly where expected and nowhere on the STOP list:**
`harvest-engine/package.json`. `main` never had a `test` script at all (its
copy of the file predates `ece82ab`, which had already added one); the
cherry-pick's context assumed the line existed. Resolved by hand — took the
incoming line (`"test": "node --test test/*.test.js"`). Not `wrangler.toml`,
not `job-run.js`, `firms.js`, `gee.js`, nor a migration file, so per the
brief's own STOP criteria this didn't require stopping — flagging it here for
your visibility anyway.

Resulting commits on the deploy branch:

| New hash | Original `dev` hash | Subject |
|---|---|---|
| `01f1d6e` | `47687be` | fix(harvest): match sensitive species nomenclaturally |
| `982b5a6` | `92b4783` | fix(sensitivity): add the nine sensitive species |

`git diff main deploy/sensitive-species-fix --stat` shows exactly the 11
files above, nothing extra picked up.

**Test suite: 10/10 pass.**

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
```

`harvest-engine/test/` is the only test suite in this repo (checked: no root
`package.json`, no other `test*` directory, no Python tests) — this is "the
full test suite" for what's being deployed.

Also ran `wrangler deploy --dry-run` on the branch: compiles and bundles
clean (2916 KiB / 727 KiB gzip), no upload performed.

## Step 2 — Verified against a real copy of production, not `data/atlas.db`

Pulled an actual export of production D1 (`harvest-engine-db`,
`2a72d827-df49-4b79-9262-5e4f6a27dce2`) via `wrangler d1 export --remote`
— a dump operation, no writes. **Caveat**: wrangler warned this makes the
live database briefly unavailable to serve queries during the export; it
auto-confirmed in this non-interactive session. No data was written or
changed, but there was a short availability blip — worth knowing about if
anything else was hitting the DB at 22:49 tonight.

Loaded the dump into a local sqlite file and queried it directly — no
assumptions carried over from the commit messages, which had described
`data/atlas.db` (a separately-curated local copy), not production.

**Baseline, from production, proven not assumed:**

| Table | Rows |
|---|---|
| taxon | 1320 |
| occurrence | 3567 |
| place | 46 |
| taxon with `is_sensitive=1` | **3** |

This is the first surprise: the commit message says "`data/atlas.db` carries
`sensitive=1` on 20 taxa." Production has **3**: `Gyps indicus`, `Santalum
album`, `Sarcogyps calvus`. The 20/9 reconciliation in the commit was against
the local curated copy, which has been hand-tiered separately (see
`6b1d537`, `dcc5cc5`) — it is not production's current state. Surfacing this
because the brief asked me to verify against production specifically, and
the two databases disagree by an order of magnitude on this exact number.

**Ran the actual matcher code from the deploy branch** (`species-name.js` +
`sensitive-species.json`, extracted verbatim, not reimplemented) **against
all 1320 real production `scientific_name` strings** — not just the 9, the
full list, per the brief:

- **0 regressions**: all 3 currently-correct taxa still match under the new
  matcher (`Gyps indicus`, `Santalum album`, `Sarcogyps calvus` — basis
  `exact` in all three cases).
- **5 more existing production rows** the new matcher correctly identifies as
  sensitive, that are **currently flagged `is_sensitive=0`**:

  | id | scientific_name | matched registry entry | shape |
  |---|---|---|---|
  | 130 | Elephas maximus indicus | Elephas maximus | trinomial |
  | 292 | Melursus ursinus ursinus | Melursus ursinus | trinomial |
  | 317 | Panthera pardus fusca | Panthera pardus | trinomial |
  | 799 | Gyps bengalensis | Gyps bengalensis | exact (newly registered) |
  | 1110 | Gyps bengalensis (Gmelin, 1788) | Gyps bengalensis | authorship |

  Two of these (elephant, leopard) are subspecies of species that were
  **already** in the 8-entry registry before tonight — they were missed
  purely by the old exact-match bug, independent of "the nine." The other
  three are genuinely from the newly-added nine.

- **Of "the nine" specifically: only 2 (`Gyps bengalensis`, `Melursus
  ursinus`) have any taxon row in production at all today.** The other seven
  — Neophron percnopterus, Lutrogale perspicillata, Prionailurus rubiginosus,
  Tetracerus quadricornis, Crocodylus palustris, Python molurus, Ophiophagus
  hannah — have **zero** rows in production's taxon table. Their coverage
  can't be checked against real data tonight; it's only proven by the
  synthetic unit tests (which pass). They'll only be tested for real the next
  time each is harvested — which won't happen tonight since the cron stays
  off.

- **No false positives found**, and this check used real data, not just the
  synthetic congener traps the commit describes (`Gyps fulvus`,
  `Acridotheres tristis tristis` — neither exists in production at all, so
  untestable there). Production *does* have `Acridotheres tristis` (common
  myna) and `Nyctanthes arbor-tristis` (night jasmine, an unrelated genus
  whose epithet contains "tristis") — both real, both correctly return
  `sensitive: false`. That's a genuine empirical false-positive check the
  commit's own tests didn't have data for.

- **Paired control**: the coarsening trigger mechanism itself (separate from
  the classification bug) works correctly today — occurrences for the 3
  already-correct taxa are visibly coarsened (rounded to the 0.045° grid,
  `public_precision_m=5000`), distinct from their exact `lat`/`lon`. Confirms
  the trigger isn't also broken; only the classification feeding it was.

**The uncomfortable finding**: the 5 existing mismatched rows above have 13
live occurrence rows in production **currently exposing exact coordinates**
(not coarsened) — elephant ×5, sloth bear ×2, leopard ×2, vulture ×4 (1 exact
+ 3 authorship-form). Confirmed by direct query, `public_lat = lat` on all
13. **Deploying tonight's fix will not change this.** `findOrCreateTaxon`
only classifies a taxon at insert time and returns existing rows untouched
(§0). Fixing these 13 rows needs a separate backfill/reconciliation pass —
update `taxon.is_sensitive` for those 5 ids, and separately re-coarsen their
occurrence rows (the `UPDATE`-triggered coarsening trigger only fires on
`UPDATE OF taxon_id, lat, lon` on `occurrence`, not on a `taxon` update — so
even flipping the taxon flag alone wouldn't fix the existing occurrence
rows). That's out of scope for tonight's "ship only the code fix" brief, and
it's a write to production, so I did not do it. Raising it as a decision for
you: is this urgent enough to schedule as a fast-follow, separately from
tonight's code deploy?

**Row counts, proven not assumed**: unchanged, trivially — every query above
was a read-only `SELECT` against a static local copy of the export; nothing
touched production after the export completed. `1320` / `3567` / `46`, same
before and after.

## Step 3 — Rollback plan

Current live production deployment, confirmed via `wrangler deployments
status` (not assumed from git log):

- Version `69d7f94d-6bc8-4e96-9109-1e7a3897cf39`, 100% traffic, created
  2026-09-06T02:02:00.718Z — matches the timing of `main`'s current head
  (`025be91`, committed 2026-09-06T07:26:34+05:30 ≈ 01:56:34Z).

**Option A — version rollback** (preferred, faster):
```
wrangler rollback 69d7f94d-6bc8-4e96-9109-1e7a3897cf39 \
  --message "revert sensitive-species deploy, restore pre-deploy state"
```
Command syntax verified via `wrangler rollback --help`; target version-id
verified valid and currently active via `wrangler deployments status`. **Not
executed tonight** — there is nothing yet deployed to roll back from, and
running it now would be a live production action against an unchanged
target, not a test. Deferred to only-if-needed, after Step 5.

**Option B — redeploy from source:**
```
git checkout main   # 025be91, unmodified
cd harvest-engine && wrangler deploy
```
`main` verified intact and buildable at `025be91` (this is what the deploy
branch was cut from).

**Open risk on both options, and on tonight's Step 5 deploy itself — see §
above and below**: I cannot confirm whether either rollback path re-triggers
the cron question in §4. Flagging rather than guessing.

## Step 4 — Pre-flight summary

**Ships:** `01f1d6e` + `982b5a6` on `deploy/sensitive-species-fix`, cut from
`main@025be91`. Diff: 11 files, 910/-7, all under `harvest-engine/`. Full
patch: [deploy-safety-fix-diff-2026-09-07.patch](./deploy-safety-fix-diff-2026-09-07.patch).

**Tests:** 10/10 pass, only test suite in the repo.

**Production verification:** matcher logic checked against all 1320 real
production taxa, 0 regressions, 5 additional real rows correctly identified
(but not retroactively fixed by this deploy — code-only fix), only 2 of "the
nine" exist in production yet, no false positives on the real data available.
13 existing occurrence rows are actively leaking exact coordinates right now
and will keep doing so after this deploy — a separate, later remediation
decision, not part of tonight's ship.

**Rollback:** version `69d7f94d-6bc8-4e96-9109-1e7a3897cf39` identified and
verified as the current live version; exact rollback command documented;
not executed (nothing to roll back from yet).

**Unresolved and blocking, found while preparing this, not asked for by the
brief but directly relevant to "don't re-enable the cron":**

Production's `job_run` table has no rows since **2026-09-05T11:03:52Z**
(today is 2026-09-07) — the cron appears paused. But `main`'s (and the
deploy branch's, unchanged) `wrangler.toml` still declares:
```
crons = ["0 3 * * *", "0 11 * * *", "0 19 * * *"]
```
and the *currently live* version (`69d7f94d`) was deployed 2026-09-06T02:02
— **after** the last job_run, using a `wrangler.toml` that, as far as I can
tell from the repo, already had these same three cron lines. If that's
accurate, the pause isn't coming from `wrangler.toml` at all — it's likely a
manual, dashboard-side pause (or something else outside the repo) that a
plain `wrangler deploy` would not necessarily preserve, since Cloudflare
treats `wrangler.toml`'s `[triggers]` block as the deploy-time source of
truth for a Worker's cron schedule.

**I could not verify this further from the CLI** — there's no `wrangler`
subcommand that shows a deployed version's actual baked-in cron schedule
distinct from `wrangler.toml`, and I didn't attempt a raw Cloudflare API call
against a live production Worker's trigger config to avoid taking an
under-verified action against production. This needs a direct look at the
Cloudflare dashboard's Triggers tab for `harvest-engine` before Step 5 is
approved: if the cron shows enabled there despite the job_run silence, it's
paused by some other mechanism (e.g. the queue consumer, not the cron
trigger itself) and a deploy is probably safe; if it shows disabled there,
a plain `wrangler deploy` tonight risks silently re-enabling it, which the
brief explicitly rules out. I'm raising this, not deciding it.

**Step 5 not run.** Waiting for your review of this report and explicit
go-ahead in a follow-up message, per the brief.

---

## Step 5 — Deployed (2026-09-07, ~17:33 UTC / ~23:03 IST)

Approved and executed. `wrangler deploy` from `deploy/sensitive-species-fix`
(unmodified — deployed exactly what was on the branch, no config edits).

- New live version: `7db6f967-0e06-45ec-aa93-76742bdc8f5d`, confirmed 100%
  traffic via `wrangler deployments status`. Replaces `69d7f94d...`.
- Not merged into `dev` or `main` — confirmed (`git merge-base
  --is-ancestor deploy/sensitive-species-fix main` → not an ancestor).
  Git housekeeping left as a separate decision, as instructed.

**The cron flag from §4 was not hypothetical.** The deploy's own output said:

```
Deployed harvest-engine triggers
  schedule: 0 3 * * *
  schedule: 0 11 * * *
  schedule: 0 19 * * *
```

Wrangler attached all three cron schedules as part of this deploy — exactly
the risk flagged before Step 5 and unresolved at the time. I'm reporting
this plainly rather than assuming it's fine: **if the cron was genuinely
disabled some other way (not via `wrangler.toml`), this deploy likely just
undid that.** I have not taken any further action on this (no follow-up
deploy, no dashboard change) — flagging it for you to check the Cloudflare
dashboard's Triggers tab for `harvest-engine` directly. The next scheduled
fire is `0 3 * * *` (03:00 IST) — a few hours out from this deploy — so
there's a window to intervene first if the cron firing isn't wanted yet.

### Step 2 re-verified against the live database (not a copy)

Ran the identical matcher-vs-real-data check from §2, this time via
`wrangler d1 execute --remote` directly against production — every query
confirmed read-only (`"changes": 0`, `"rows_written": 0` in each response).

**Row counts, live, unchanged from the pre-deploy copy:**

| Table | Rows |
|---|---|
| taxon | 1320 |
| occurrence | 3567 |
| place | 46 |
| taxon `is_sensitive=1` | 3 |

Identical to §2's numbers — confirms no drift and that the deploy, as
expected, did not touch any existing row. `job_run`'s most recent entry is
still `2026-09-05T11:03:52Z` — nothing has run since, consistent with the
cron not having fired yet (next slot is hours away).

**Re-ran the deployed matcher code against all 1320 live scientific names:**
byte-identical result to §2 — 0 regressions, the same 5 pre-existing rows
correctly identified but not retroactively corrected, the same 2-of-9
existing in production at all. Directly confirmed the two by id:

| id | scientific_name | is_sensitive (live, right now) |
|---|---|---|
| 799 | Gyps bengalensis | 0 |
| 1110 | Gyps bengalensis (Gmelin, 1788) | 0 |
| 292 | Melursus ursinus ursinus | 0 |

**Precise answer to "are the 9 species correctly flagged in production":
not yet, for existing rows — that claim doesn't hold as literally stated.**
The fix is live and *will* correctly classify any of the 9 the next time a
harvester creates a *new* taxon row for one of them (§0's `findOrCreateTaxon`
still returns existing rows untouched). Right now: 7 of the 9 have no taxon
row in production to check; the 2 that do (`Gyps bengalensis`, `Melursus
ursinus ursinus`) are still `is_sensitive=0`, unchanged, same as before
deploy. The code is correct and deployed; the existing data isn't retroactively
fixed by it. That backfill is the same separate, not-yet-approved decision
flagged in §2 — still open, now slightly more urgent given the cron
question above.

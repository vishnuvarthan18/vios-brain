**Vishnu** (2026-09-05T13:50): Add a "stream health" section to the harvest-engine dashboard (dashboard/render.js) — do not deploy without confirming with me first.

Currently the dashboard shows pass/fail per job_run but no grouping or verdict. Add a new panel above the existing job table:

1. A summary banner: total job_run rows, and row counts for place, taxon, occurrence, document, claim, source (query these live, don't hardcode).

2. A "streams" table with one row per stream (read the list from src/streams/*.js), showing:
   - stream name
   - last successful fetch timestamp (MAX(started_at) WHERE status='success')
   - a computed "verdict" badge based on this logic:
     - if no run in the last 3x the stream's expected cadence → "stale" (likely the dedup-freeze bug)
     - if status='failed' on the last 3 consecutive runs → "failing"
     - if status='blocked' (once that status exists per the earlier fix) → "blocked"
     - otherwise → "healthy"
   - rows written in the last 7 days (COUNT from whatever table that stream writes to, or from job_run.rows_written if that's already tracked)

3. Color the verdict badges consistently: healthy=green, stale=amber, failing=red, blocked=grey.

4. Fix the existing bug at dashboard/render.js:55 first — it silently miscounts any status outside its 4 hardcoded keys, which will break as soon as a "blocked" or other new status is introduced.

Keep this on the `dev` branch. Show me the diff and a screenshot/description of the new dashboard before merging or deploying anywhere.

**Claude** (2026-09-05T13:51): I'll start by exploring the actual state of the code before making any changes.

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): Important discrepancies surfacing already. Let me verify them before writing any code.

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): _[tool: Bash]_

**Claude** (2026-09-05T13:51): Found two things that materially change the spec. Let me confirm the data situation and read the rest of the renderer.

**Claude** (2026-09-05T13:52): _[tool: Bash]_

**Claude** (2026-09-05T13:52): _[tool: Bash]_

**Claude** (2026-09-05T13:52): _[tool: Bash]_

**Claude** (2026-09-05T13:52): _[tool: Bash]_

**Claude** (2026-09-05T13:52): _[tool: Bash]_

**Claude** (2026-09-05T13:52): _[tool: Bash]_

**Claude** (2026-09-05T13:52): Local DBs confirm the key facts. Let me read the remaining renderer sections and check the `dev` branch state.

**Claude** (2026-09-05T13:52): _[tool: Bash]_

**Claude** (2026-09-05T13:53): _[tool: Bash]_

**Claude** (2026-09-05T13:53): _[tool: Bash]_

**Claude** (2026-09-05T13:53): Something inconsistent about the `dev` branch. Let me verify precisely.

**Claude** (2026-09-05T13:53): _[tool: Bash]_

**Claude** (2026-09-05T13:53): I made a path error — my shell was inside `harvest-engine/`, which skewed those git results. Let me redo from the repo root.

**Claude** (2026-09-05T13:53): _[tool: Bash]_

**Claude** (2026-09-05T13:53): _[tool: Bash]_

**Claude** (2026-09-05T13:54): _[tool: Bash]_

**Claude** (2026-09-05T13:54): _[tool: Bash]_

**Claude** (2026-09-05T13:54): _[tool: Bash]_

**Claude** (2026-09-05T13:55): I've verified enough to find that four of the task's premises don't match the repo. Three I can work around; one genuinely determines where I start.

**1. The branch and the file don't line up.** `dev` and `main` have diverged in both directions — this isn't a fast-forward:

- The `render.js` you described (538 lines, job table, `statusCounts`, charts) exists **only on `main`**. On `dev` it's 160 lines, pre-charts, with no `statusCounts` at all — [line 55](harvest-engine/dashboard/render.js#L55) there is a CSS rule, `.status-failed { color: #cf222e; }`.
- `main` has `coverage-snapshot.js` + migration `0003_coverage_snapshot`; `dev` has migration `0005_geometry_scope_annulus`, `layer-review.js`, `boundary.js`, `gee-computations.js` and a `test/` directory (incl. `job-run-status.test.js`) that **`main` lacks entirely**.
- `main` also has design tokens (`--status-good`, `--status-warning`, `--series-muted`) that your badge colors map onto cleanly; `dev` uses hardcoded hex and has no tokens.

**2. `blocked` is not a real status.** The CHECK constraint allows `running, success, partial, failed` (0001) plus `skipped_duplicate` (0004). Nothing writes `blocked`; the string only appears as `"Fetch blocked: …"` text inside `errors_json`. The earlier fix you're thinking of added `skipped_duplicate`, not `blocked`.

**3. There's no per-stream cadence.** All streams share one global cron, and it's currently `crons = []` — paused today in your uncommitted [wrangler.toml](harvest-engine/wrangler.toml#L57) ("PAUSED 2026-09-05… while the harvest pipeline is under review"). Previously `["0 3 * * *","0 11 * * *","0 19 * * *"]` = every 8h, so 3× = 24h.

**4. The line-55 bug is real, but mislocated — and it's already firing.** Line 61 accumulates with `?? 0`, so unknown statuses *are* counted correctly. The loss is downstream, where only the 4 keys are read:

| Site | Effect on `partial` / `skipped_duplicate` |
|---|---|
| tiles [203-206](harvest-engine/dashboard/render.js#L203-L206) | counted, never displayed |
| donut [456-461](harvest-engine/dashboard/render.js#L456-L461) | dropped, but still divided by `streamNames.length` → silently under-fills |
| `statusCellColor` [380-385](harvest-engine/dashboard/render.js#L380-L385) | falls through to `--gridline` — **identical to "never run"** |
| table cell [251](harvest-engine/dashboard/render.js#L251) | correct (`.status-partial` exists) |

This isn't hypothetical: `classifyOutcome` actively *creates* `partial` runs, and `partial` is present in recorded data (7 rows in the 2026-08-23 D1 backup, 1 in a local D1). A `partial` stream currently renders as if it never ran.

One more, unasked: the existing `LIMIT 50` job_run query can't back this panel — 50 rows across 30 streams means absent streams render "never", and "last 3 consecutive runs" isn't derivable. I'll use dedicated per-stream aggregates.

**Claude** (2026-09-05T13:55): _[tool: AskUserQuestion]_

**Claude** (2026-09-05T14:03): All three defaults confirmed. Creating the branch off `main` and reading the exact regions I need to patch.

**Claude** (2026-09-05T14:03): _[tool: Bash]_

**Claude** (2026-09-05T14:03): _[tool: Bash]_

**Claude** (2026-09-05T14:03): _[tool: Bash]_

**Claude** (2026-09-05T14:04): Branch created. Now writing the patch — I'll use exact-match assertions so no replacement can silently no-op.

**Claude** (2026-09-05T14:05): _[tool: Bash]_

**Claude** (2026-09-05T14:05): Patch 1 applied. Now the render sites and the new panel.

**Claude** (2026-09-05T14:05): _[tool: Bash]_

**Claude** (2026-09-05T14:06): _[tool: Bash]_

**Claude** (2026-09-05T14:07): Syntax passes. Now let me actually render it against real data — first the local D1, then a fixture that exercises every verdict.

**Claude** (2026-09-05T14:07): _[tool: Bash]_

**Claude** (2026-09-05T14:07): _[tool: Bash]_

**Claude** (2026-09-05T14:07): _[tool: Bash]_

**Claude** (2026-09-05T14:07): The stream modules import JSON without Node's required attribute. Let me stub that module via a loader hook so I test the real `render.js` unmodified.

**Claude** (2026-09-05T14:07): _[tool: Bash]_

**Claude** (2026-09-05T14:08): _[tool: Bash]_

**Claude** (2026-09-05T14:08): Rendered (the `coverage_snapshot` error is the pre-existing guarded path). Let me inspect the new panel.

**Claude** (2026-09-05T14:08): _[tool: Bash]_

**Claude** (2026-09-05T14:08): Correct on real data. Now the decisive test — a fixture exercising every verdict and every status.

**Claude** (2026-09-05T14:08): _[tool: Bash]_

**Claude** (2026-09-05T14:09): _[tool: Bash]_

**Claude** (2026-09-05T14:09): _[tool: Bash]_

**Claude** (2026-09-05T14:09): All four verdicts fire correctly — note `bhuvan` is both failing and 200h old, and correctly reads **failing**, not stale. Two refinements before I finalize:

**Claude** (2026-09-05T14:09): _[tool: Bash]_

**Claude** (2026-09-05T14:10): _[tool: Bash]_

**Claude** (2026-09-05T14:10): All eight statuses now render and the donut closes at exactly 100%. Let me quantify the before/after by rendering the same fixture through the original code.

**Claude** (2026-09-05T14:10): _[tool: Bash]_

**Claude** (2026-09-05T14:10): _[tool: Bash]_

**Claude** (2026-09-05T14:10): Reliability strip fully confirmed — four statuses previously painted identically to "no run". Chrome is available, so let me capture an actual screenshot.

**Claude** (2026-09-05T14:10): _[tool: Bash]_

**Claude** (2026-09-05T14:11): _[tool: Read]_

**Vishnu** (2026-09-05T14:11): [Image: original 2560x3400, displayed at 1506x2000. Multiply coordinates by 1.70 to map to original image.]

**Claude** (2026-09-05T14:11): _[tool: Read]_

**Vishnu** (2026-09-05T14:11): [Image: original 2560x1800, displayed at 2000x1406. Multiply coordinates by 1.28 to map to original image.]

**Claude** (2026-09-05T14:11): The donut now closes and all eight statuses appear. One flaw I introduced: `Blocked` and `Never run` are the same grey in the donut — exactly the collision I was fixing elsewhere. Let me separate them.

**Claude** (2026-09-05T14:12): _[tool: Bash]_

**Claude** (2026-09-05T14:12): All eight statuses distinct, ring closes. Let me review the complete diff.

**Claude** (2026-09-05T14:12): _[tool: Bash]_

**Claude** (2026-09-05T14:12): _[tool: Bash]_

**Claude** (2026-09-05T14:12): The diff is sound. One stale comment reference to fix, then I'll commit locally.

**Claude** (2026-09-05T14:12): _[tool: Bash]_

**Claude** (2026-09-05T14:12): _[tool: Bash]_

**Claude** (2026-09-05T14:13): _[tool: Bash]_

**Claude** (2026-09-05T14:13): Done. Committed to `feat/stream-health` (off `main`, per your choice) — **not pushed, not merged, not deployed.**

## What's on the branch

**The bug fix, first.** Your diagnosis was right that new statuses break the dashboard, but the location was off and the damage is worse than described. Line 61 accumulates unknown keys fine (`?? 0`); three *display* sites each kept their own hardcoded four-status list. Rendering the same fixture through old and new code:

| | before | after |
|---|---|---|
| Overview tiles | 4 tiles, 26 of 30 streams shown | 8 tiles, all 30 |
| Donut ring | drew **86.7%** — the gap reads as missing data | 100.0% |
| Reliability strip | `partial`, `skipped_duplicate` → `--gridline`, **identical to "never run"** | each its own colour |

This was already firing, not a future risk — `classifyOutcome()` writes `partial` every time a run harvests nothing, and `partial` rows exist in your recorded data. All three sites now read one `STATUS_META` table; an unrecognised status keeps its real name rather than vanishing.

**The panel** sits above the Streams table. Screenshots at `…/scratchpad/shot-health.png` and `shot-top.png`. Against a fixture exercising every path:

```
gbif        healthy  ran 2h ago              2026-09-05T12:09:08Z   287
shodhganga  failing  last 3 runs failed      2026-09-04T13:09:08Z    12
bhuvan      failing  last 3 runs failed      never succeeded          0   ← 200h old; failing beats stale
wdpa        stale    no run in 16d           2026-08-19T22:09:08Z     0
erode-nic   blocked  blocked at source       never succeeded          0
dummy       stale    never run               never succeeded          0
```

Each badge carries a one-line reason, so `stale — never run` stays distinguishable from `stale — no run in 16d`.

**Three judgment calls I made and flagged in code comments:**

1. **Precedence is blocked → failing → stale → healthy**, inverting your stale-first ordering. With the cron stopped, stale-first relabels every failing stream as merely stale — masking the signal exactly when it matters. `bhuvan` above is the case. One comment says which branches to swap to reverse it.
2. **`--text-secondary` for the blocked *status***, not the grey used for the blocked *verdict* badge — sharing `--series-muted` would have made a blocked run and a never-run stream identical in the donut, reintroducing the collision I was fixing.
3. **The paused-cron banner is derived, not configured.** It fires when no stream in the fleet has run inside the window, so it clears itself when you restore the cron — no flag to remember.

**One fix beyond scope, because leaving it would have put a contradiction on screen:** per-stream state was derived from the 50-row "recent runs" query. With ~30 streams, most fall outside those 50 and rendered as "never run" — the new panel would have said "last succeeded Tuesday" directly above a table saying "never". A per-stream window query now backs both, which also corrects the existing table and reliability strip.

## Still open

- **`dev` and `main` remain diverged**, untouched. `dev` has migration `0005`, `layer-review.js`, `boundary.js` and a `test/` directory that `main` lacks; `main` has the charts and `coverage-snapshot`. Reconciling them is its own job.
- **`blocked` still isn't a real status.** The branch is dormant — no stream can reach it until a migration adds it and the robots paths emit it.
- Two `job_run.stream_name` values (`overpass-boundary`, `coarsen-verification`) aren't in the `STREAMS` registry, so they're counted in the banner total but have no panel row. Correct as far as it goes; tell me if you'd rather they surfaced.

Verified by rendering against your real local D1 and the fixture through a D1 shim — `render.js` itself unmodified, the streams module stubbed only because it imports JSON without Node's required attribute. No tests were run: `main` has no `test/` directory (the harvest-engine tests live only on `dev`).

**Vishnu** (2026-09-06T01:45): On feat/stream-health:

1. Add `overpass-boundary` and `coarsen-verification` to the STREAMS registry so they get their own panel row instead of only counting toward the fleet total.

2. Keep the blocked → failing → stale → healthy precedence as built — don't revert to stale-first.

3. Merge feat/stream-health into main (dashboard code lives there, not dev). UI/read-only addition, no schema change, no data writes — safe to land directly.

4. Deploy the updated dashboard Worker.

5. Confirm: dashboard shows all ~30+2 streams with correct verdicts, paused-cron banner visible.

Do NOT touch harvest-engine's data logic (dedup window, max_batch_size, etc.) — still on hold.

**Claude** (2026-09-06T01:45): Several of these interact in ways I need to verify before acting — especially (1), which touches the registry that drives the cron.

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:45): I hit my earlier path trap again — those `git ls-tree` calls ran from inside `harvest-engine/`. Redoing from the repo root.

**Claude** (2026-09-06T01:45): _[tool: Bash]_

**Claude** (2026-09-06T01:46): _[tool: Bash]_

**Claude** (2026-09-06T01:46): Item 1 has a hard conflict with your closing constraint. Let me confirm the deploy path before I explain.

**Claude** (2026-09-06T01:46): _[tool: Bash]_

**Claude** (2026-09-06T01:46): _[tool: Bash]_

**Claude** (2026-09-06T01:47): `coarsen-verification` is definitively not a stream — it's a one-off audit marker row from a manual SQL script. Checking deploy auth:

(secret removed) (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:47): Authenticated with write scope. Before I touch anything, let me check what cron schedule production currently has — your uncommitted `crons = []` would ride along with any deploy.

**Claude** (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:47): _[tool: Bash]_

**Claude** (2026-09-06T01:48): Production already has **zero** cron schedules — the pause is live, so a deploy won't change cron state. Now the decisive question for item 1: what's actually in *production* job_run?

**Claude** (2026-09-06T01:48): _[tool: Bash]_

**Claude** (2026-09-06T01:48): Production has a **third** orphan you didn't mention — `gee-geometry-check`. Let me get the full picture.

**Claude** (2026-09-06T01:48): _[tool: Bash]_

**Claude** (2026-09-06T01:49): Production tells a different story than expected. Three facts that change item 5:

- **33** distinct stream names (30 registered + **3** orphans, not 2)
- **`partial` = 23 runs in production** — the display bug was live there, dropping real data
- Most recent run: `2026-09-05T11:03:52Z`, i.e. **14.8h ago** — under the 24h threshold

Let me implement item 1's goal without breaking your constraint.

**Claude** (2026-09-06T01:49): _[tool: Bash]_

**Claude** (2026-09-06T01:50): Now let me mirror production data locally and render, so I know exactly what the deploy will show.

**Claude** (2026-09-06T01:50): _[tool: Bash]_

**Claude** (2026-09-06T01:50): _[tool: Bash]_

**Claude** (2026-09-06T01:50): _[tool: Bash]_

**Claude** (2026-09-06T01:51): _[tool: Bash]_

**Claude** (2026-09-06T01:51): The other two exports carry extra output. Let me make the loader robust and build the mirror.

**Claude** (2026-09-06T01:52): _[tool: Bash]_

**Claude** (2026-09-06T01:52): Counts export came back empty. Let me query them individually.

**Claude** (2026-09-06T01:52): _[tool: Bash]_

**Claude** (2026-09-06T01:52): Got real production counts. Rebuilding the mirror and rendering exactly what the deploy will show.

**Claude** (2026-09-06T01:52): _[tool: Bash]_

**Claude** (2026-09-06T01:52): _[tool: Bash]_

**Claude** (2026-09-06T01:53): This is exactly what will deploy. Let me commit, then run pre-deploy checks.

**Claude** (2026-09-06T01:53): _[tool: Bash]_

**Claude** (2026-09-06T01:53): _[tool: Bash]_

**Claude** (2026-09-06T01:53): One more pre-deploy check — whether main has undeployed **data-logic** commits that a deploy would also ship.

**Claude** (2026-09-06T01:53): _[tool: Bash]_

**Claude** (2026-09-06T01:54): _[tool: Bash]_

**Claude** (2026-09-06T01:54): _[tool: Bash]_

**Claude** (2026-09-06T01:55): Timestamps suggest `c04e9cc` (lgd field mapping + `classifyOutcome`) may be **undeployed**. Let me settle it by reading the live Worker bundle rather than inferring.

**Claude** (2026-09-06T01:55): _[tool: Bash]_

**Claude** (2026-09-06T01:55): _[tool: Bash]_

**Claude** (2026-09-06T01:56): OAuth can't read the bundle. But there's a decisive empirical test: `classifyOutcome` downgrades a success-with-zero-rows to `partial`. If any such run survives as `success`, it isn't deployed.

**Claude** (2026-09-06T01:56): _[tool: Bash]_

**Claude** (2026-09-06T01:56): Decisive: two runs from yesterday recorded `success` with 0 rows and no errors — exactly what `classifyOutcome` downgrades. **It is not deployed**, confirming `c04e9cc` is undeployed.

Let me complete the authorized merge, then raise the deploy issue.

**Claude** (2026-09-06T01:56): _[tool: Bash]_

**Claude** (2026-09-06T01:56): Merged into `main` locally (not pushed). Now the deploy — your premise for it doesn't fully hold, so let me check the exact blast radius first.

**Claude** (2026-09-06T01:56): _[tool: Bash]_

**Claude** (2026-09-06T01:57): `c04e9cc` touches exactly two src files — `src/lib/job-run.js` and `src/streams/lgd.js`. Both are data logic, both currently undeployed. Your "no data writes" premise holds for my change but not for the deploy as a whole.

**Claude** (2026-09-06T01:57): _[tool: AskUserQuestion]_

**Claude** (2026-09-06T02:01): Deploying dashboard-only. Reverting the two data-logic files to their currently-deployed state first.

**Claude** (2026-09-06T02:01): _[tool: Bash]_

**Claude** (2026-09-06T02:01): Tree now matches deployed state except `render.js`. Deploying.

**Claude** (2026-09-06T02:01): _[tool: Bash]_

**Claude** (2026-09-06T02:02): Deployed. Restoring the working tree immediately and verifying.

**Claude** (2026-09-06T02:02): _[tool: Bash]_

**Claude** (2026-09-06T02:02): _[tool: Bash]_

**Claude** (2026-09-06T02:03): _[tool: Bash]_

**Claude** (2026-09-06T02:03): Deployed and verified. Version `69d7f94d` is live as of `2026-09-06T02:02:03Z`.

## Status

**1. Unregistered streams — done, but not via the registry.** Adding them to `STREAMS` would have been self-defeating: `listStreamNames()` also drives `scheduled()`, so registering a name enqueues it every cron tick. `coarsen-verification` has no module at all — it's a one-off audit marker row written by `scripts/verify-coarsening.sql` on 2026-08-25 — so it would throw on call; `overpass-boundary`'s module lives only on `dev`, so importing it would break the build, and porting it is the harvest logic you put on hold. The panel now derives its rows from `listStreamNames()` ∪ the stream names present in `job_run`, tagged *"not in registry — will not run"*. Same outcome, no cron change, and future orphans appear automatically.

There are **three**, not two — `gee-geometry-check` is also unregistered (3 runs, also dev-only).

**2. Precedence** unchanged: blocked → failing → stale → healthy.

**3. Merged** as `025be91` into local `main`. **Not pushed** — you didn't ask, and note `scripts/deploy-prod.sh` refuses to run while local `main` differs from `origin/main`, so your *site* deploys are blocked until it's pushed. Say the word.

**4. Deployed dashboard-only**, as chosen. `src/lib/job-run.js` and `src/streams/lgd.js` were reverted to their currently-deployed state for the upload, then restored. Cron schedules verified **empty before and after** — the deploy changed nothing about scheduling.

## 5. Confirmation — two of three

Rendered from a local mirror of production D1 (920 job_run rows) through the identical deployed code:

- ✅ **33 rows** — 30 registered + 3 tagged unregistered
- ✅ **Verdicts**: 25 healthy · 3 stale · 5 failing. The 5 failing (`openalex`, `shodhganga`, `wdpa`, `forests-tn`, `management-plan`) have all never recorded a success.
- ❌ **The paused-cron banner is not visible, and shouldn't be yet.** Production's last run was `2026-09-05T11:03:52Z` — about 14.8h before render, under the 24h threshold. The banner is data-derived, as you approved, so it stays hidden until a full 24h of silence: **after ~`2026-09-06T11:03:52Z`**, roughly 9h from now, if nothing runs. Nothing is wrong; the condition simply isn't met.

I could not log into the live page — it's password-protected — so that confirmation is from production data through the deployed code, not from the authenticated page itself.

**Worth your attention:** production had **23 `partial` runs** the old dashboard was discarding, so the display fix has immediate real effect. And `main`'s *committed* `wrangler.toml` still carries `crons = ["0 3 * * *", "0 11 * * *", "0 19 * * *"]` — the pause exists only in your uncommitted working tree, so a deploy from a clean checkout would silently re-enable the schedule. I left that file untouched.
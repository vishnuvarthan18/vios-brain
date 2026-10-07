# The Harvest Engine — deep plan, end to end, 9 Sep 2026

This is the full plan: what's wrong, why, what "done" looks like, every phase in order, who owns each piece, how each phase proves itself, and what could go wrong. Everything else written this week (the trust-layer plan, the stream audit, the coverage plan, the master build plan) is folded into this one document.

IMPORTANT FOR WHOEVER EXECUTES THIS: verify every claim below (file paths, counts, table/column names) against the live repo and database before acting on it. This plan was written from prior audit conversations, not from re-reading the current code — treat every specific like "129 confirmed lgd records" or "findExistingSource/hash.js" as a claim to confirm, not a fact to build on blind. If a named file or path doesn't exist, say so and locate the real one rather than proceeding on the stale name.

---

## 1. What this engine is actually for

The Atlas's whole value is being a trustworthy, sourced reference — every fact has a place, a source, a date. The harvest engine is the machine that feeds it: dozens of automated data streams pulling from public APIs, government archives, satellite imagery, and news, so the site doesn't depend on Vishnu manually typing in facts one at a time. If the engine can't be trusted, the site can't be trusted, and the whole project's reason for existing is undermined.

## 2. Where it actually stands right now — the honest diagnosis (per the 9 Sep stream audit)

- Per `sathyamangalam/harvest-engine-stream-audit-2026-09-09.md` in this repo: of the streams registered in `harvest-engine/src/streams/index.js`, a majority were found healthy, several were found silently writing zero rows while reporting success, and a handful are honestly blocked (visible failures — missing keys, dead source sites).
- The root failure pattern, twice over: a dedup bug froze several major streams for 8 days while they reported success 3x/day, undetected until a manual audit (see `harvest-engine-dedup-freeze-and-pause-2026-09-05.md`). Separately, the silently-broken streams found in the 9 Sep audit were also undetected until a manual audit. Both were found by hand. Nothing in the system itself would have caught either — confirm this is still true before building on it.
- The biggest coverage gap: places. Confirm the current count and target against the live database and `sathyamangalam/coverage-improvement-plan-2026-09-09.md` rather than assuming the numbers quoted there are still current.
- Two API keys are in progress: `DATA_GOV_IN_API_KEY` obtained by Vishnu; `WDPA_API_KEY` requested, pending email approval.

## 3. What "done" looks like (definition, not vibes)

An engine is "production ready" when, at minimum:
1. Every stream's real status (healthy / blocked / broken) is visible on the dashboard without a manual SQL audit.
2. A stream going silent for longer than its normal cadence triggers an alert within hours, not weeks.
3. Every currently-known broken stream is either fixed, or explicitly marked as permanently excluded with a documented reason.
4. Cron is back on for every stream, each individually verified healthy before being added back, not flipped on all at once.
5. The two new API keys are wired in and the gazetteer stream is confirmed growing.

## 4. Guiding principles

- Fix the blindness before fixing the bugs. Fixing broken streams without first building something that would catch the *next* one just resets the clock.
- Idempotent, at-least-once, never exactly-once — keep the existing find-or-create pattern.
- Isolate blast radius — smaller batch sizes, per-stream contracts, DLQ triage.
- No big-bang changes — fix and verify one cohort, one stream, one cron re-enable at a time.
- Right-sized, not enterprise-grade — no Airflow, no Spark, no lineage platform.
- Never merge/deploy to `main` without Vishnu's explicit go-ahead in that same session.

## 5. The phased roadmap

### Phase 0 — Vishnu's two inputs (parallel, already in motion)
- `DATA_GOV_IN_API_KEY` — obtained.
- `WDPA_API_KEY` — requested, pending.
Owner: Vishnu.

### Phase 1 — The trust layer
1. A consumer/alert on the dead-letter queue — verify its real name in `wrangler.toml` first — so any message landing there triggers a notification instead of sitting unwatched.
2. A `stream_health` table: per-stream last-real-write timestamp vs. declared cadence, computed independently of `job_run.status` by a separate scheduled check — verify `job_run`'s actual schema first.
3. Dashboard rebuild around staleness + DLQ depth; check `dashboard/render.js` (or wherever the dashboard actually lives) for any hardcoded status-list assumption and fix it.
4. Frozen-fixture contract tests, starting with whichever streams the live audit confirms are silently broken today — one real captured API response per stream, tested against today's parser.

Exit criteria: deliberately break one fixture test and confirm the alert/dashboard flags it.

### Phase 2 — Root-cause fixes for the dedup freeze
5. Max-age refetch window in the request-hash dedup logic (verify the actual file — referenced previously as `hash.js` / `findExistingSource`, confirm it still exists under that name).
6. Reduce the queue's max batch size to shrink blast radius — verify the current value in `wrangler.toml` before changing it.

Exit criteria: `stream_health` shows staleness clearing correctly after a real test fetch.

### Phase 3 — Fix the broken streams, cohort by cohort
- Cohort A: parser/config bugs — root-cause each broken stream independently, do not assume they share one cause. Verify specific claims like a BHL field-name mismatch or an LGD record count against the live audit doc before treating them as confirmed.
- Cohort B: streams blocked on the WDPA key — proceed once Phase 0 confirms the key is live.
- Cohort C: genuinely dead sources (source site down / robots.txt-blocked) — mark explicitly excluded with a documented reason rather than retrying forever.
- Cohort D: supporting fixes — a real "blocked" status for missing-secret/missing-asset gates; confirm retry logic actually retries 5xx responses, not just thrown errors; a stale-run reaper if tombstoned jobs are confirmed to exist.

Exit criteria: each cohort verified against Phase 1's fixture harness before merging.

### Phase 4 — Resume cron, one stream at a time
1. Re-enable one verified-healthy stream.
2. Watch its `stream_health` row for a real cadence match over 48-72 hours.
3. Add the next. Repeat until all streams are individually confirmed.
4. Restore the full cron schedule only once every stream has been through this.

Owner: dev agent proposes, Vishnu approves each re-enable.

## 6. What Vishnu decides, vs. what gets executed directly

Vishnu's calls: the two API key signups (done/in motion); whether any paused paid source is worth paying for; final go-ahead before any Phase 3/4 work touches `main` or production; whether re-measured coverage numbers are good enough after this plan runs.

Everything else: execute, verify, report — no production access without explicit approval each time.

## 7. Sequencing note

Phase 1 can start immediately. Phase 2 follows directly. Phase 3 Cohort A can run in parallel with waiting on the WDPA email; Cohort B waits specifically for that key. Phase 4 is last and deliberately the slowest, most-checkpointed phase.

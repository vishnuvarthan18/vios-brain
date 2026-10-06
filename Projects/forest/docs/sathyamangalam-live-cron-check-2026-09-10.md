# Live cron run check — 10 Sep 2026, 03:01 UTC — READ FIRST NEXT SESSION

Vishnu asked "is the engine running or not" after tonight's deploy. Checked the real 3:00 AM UTC cron run in production D1, not just assumed. Answer: yes, it's running — cron fired on schedule, kicked off all ~35 streams. But the real results surfaced new, real bugs worth fixing, separate from everything fixed earlier tonight.

## Result breakdown (job_run rows, started_at >= 2026-09-10T02:30:00Z)
- **Wrote real data**: `firms` (1 row), `gee` (3 rows, status "partial")
- **success but 0 rows**: `mongabay-india` (nothing new since last successful fetch — expected)
- **skipped_duplicate** (normal — already fetched recently, dedup window working as intended): toi-coimbatore, historical-text, erode-nic, wii, ntca, bhuvan, census, lgd, wikidata, overpass, bhl, core, semanticscholar, ebird, dummy
- **stuck at "running", never finished** (~40 min old at check time): management-plan x4 attempts, unpaywall x1 — expected to be cleaned up by this week's stale-run reaper, but only after 6h threshold, so not stale yet at check time. Worth re-checking these specifically don't recur every cron cycle (would indicate a real hang, not just slow).
- **failed, EXPECTED**: wdpa (no API key, known), shodhganga (robots.txt now explicitly disallows /oai/request — consistent with this week's investigation, confirms the OAI path really is dead end)
- **failed, EXTERNAL/TRANSIENT** (likely not our bug, re-check next cron): gbif (HTTP 503 from GBIF itself), inaturalist (HTTP 429, their rate limit), thehindu-tn (HTTP 403, they may be blocking the request), forests-tn (their server timed out with a 522, matches prior documented flakiness of this specific host)
- **failed, REAL BUG #1 — found tonight, not yet fixed**: crossref, europepmc, ia-scholar ALL crashed with the identical error: `D1_ERROR: UNIQUE constraint failed: source.reserve_id, source.request_hash`. The stream tried to insert a `source` row that already exists (duplicate request_hash for this reserve) and crashed instead of handling it as a normal duplicate. This is a real code defect in whatever shared source-insertion logic these three streams use — needs investigation into why they're not using a find-or-create pattern (or why an existing one is failing) the way other streams do.
- **failed, REAL BUG #2 — found tonight, not yet fixed**: `openalex` failed on EVERY one of ~20 queries with HTTP 429 (rate limited). Important context: openalex is ALREADY BUILT AND WIRED UP from an earlier session — it does NOT need the free API key we were planning to have Vishnu register (see openalex-and-core-research-2026-09-10.md, that whole plan may be moot). It's using the `mailto=` polite-pool pattern already, same as Crossref. The actual bug: it fires ~20 search queries back-to-back in one run with no pacing/backoff between them, so OpenAlex's rate limiter cuts it off immediately, every single query. Needs a delay/backoff between requests within the stream, not an API key registration.

## Decision made tonight
Vishnu asked to save everything and pick this up in the evening session. Nothing further investigated or fixed tonight beyond what's captured here.

## Next session should
1. Read this doc first.
2. Fix the crossref/europepmc/ia-scholar UNIQUE constraint crash (Bug #1) — likely a shared source-insertion helper needs find-or-create instead of raw insert, or a race from overlapping runs.
2b. Check whether these three streams share a common helper function — if so, one fix likely resolves all three at once.
3. Fix openalex's request pacing (Bug #2) — add a delay between the ~20 search queries per run, following the RateLimiter pattern already used elsewhere in the codebase (see other streams' `rate_limit_ms` config in their sources/*.json + RateLimiter usage).
4. Re-evaluate the OpenAlex research doc (openalex-and-core-research-2026-09-10.md) — the "needs a free API key" framing may be wrong; it's already running unauthenticated via mailto=, just needs pacing. Worth re-checking whether an API key would still help (higher rate limit) or whether pacing alone fixes it.
5. Check whether `management-plan`/`unpaywall`'s stuck "running" rows recur every cron cycle (would mean a real hang, not just this one slow run) — query job_run for status='running' across multiple recent cron fires.
6. Still carried forward from earlier tonight: CORE stream (not started), 3 data conflicts on live site (tiger/elephant/leopard counts, unresolved), bhl (no key), Email Routing (skipped).

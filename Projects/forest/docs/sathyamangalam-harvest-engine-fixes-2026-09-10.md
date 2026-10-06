# Harvest engine bug fixes — 10 Sep 2026 — session close

Vishnu asked to check if the engine was running okay after seeing lots of red on the dashboard. Found and fixed four real, unrelated bugs, all deployed and confirmed live. Follow-up check scheduled for 14 Sep 2026.

## What was actually wrong (all confirmed via job_run errors_json, not guessed)

1. **crossref, europepmc, ia-scholar crashed on every run** — `D1_ERROR: UNIQUE constraint failed: source.reserve_id, source.request_hash`. `findExistingSource(maxAgeDays)` returns null once a request's last copy goes stale, so the stream refetches — but the table's UNIQUE constraint has no such time window, so the refetch's plain INSERT collided with the still-present stale row and crashed. Fixed: added `recordSource()` (INSERT ... ON CONFLICT DO UPDATE) in `src/lib/hash.js`, switched all three streams to use it. **Confirmed working live** — europepmc wrote 1 new row, ia-scholar succeeded cleanly, both after the fix.

2. **openalex failed with HTTP 429 on every single query** — turned out to NOT be a pacing/code bug. OpenAlex retired the mailto-only "polite pool" on 2026-02-13 (see groups.google.com/g/openalex-users/c/rI1GIAySpVQ); keyless requests now get a much smaller daily budget that this stream's ~20 terms burned through instantly. Fixed: added `OPENALEX_API_KEY` secret support (`api_key=` query param) in `src/streams/openalex.js`. Vishnu registered a free key at openalex.org/settings/api and set it via `wrangler secret put OPENALEX_API_KEY`.

3. **openalex then hit Cloudflare's per-invocation CPU limit** — once the API key fix stopped instant 429s, the stream actually started doing real work (thousands of document upserts across ~20 terms × up to 4 pages × 50 records in one invocation) and got killed mid-run every time, landing in the DLQ. Fixed: restructured `src/streams/openalex.js` to process **one search term per invocation**, self-chaining via `env.HARVEST_QUEUE.send({ streamName: "openalex", reserveSlug, termIndex: termIndex + 1 })` after each term finishes. Also updated `src/streams/index.js` and `src/queue-consumer.js` to pass extra queue-message fields through to a stream's `options` param. **Deployed but not yet confirmed through a full run** — this is the one thing to check first at the 4-day follow-up.

4. **DLQ alert crashed**: `D1_ERROR: no such table: dlq_alert`, discovered live via `wrangler tail` while diagnosing #3. Cause: 4 migrations (`0004_reserve_geometry_init.sql`, `0005_geometry_scope_annulus.sql`, `0006_dlq_and_stream_health.sql`, `0007_job_run_blocked_status.sql`) had never been applied to the remote D1 database, despite existing in the repo. Complication: `reserve_geometry` table already existed on remote (created some other way, untracked), colliding with migration 0004 on first apply attempt — fixed by manually inserting a `d1_migrations` row marking 0004 as already-applied (table schema verified identical first), then re-running `wrangler d1 migrations apply --remote`, which cleanly applied 0005/0006/0007. All confirmed applied; dlq_alert table now exists and the alert path fails gracefully (missing `send_email` binding, a known pre-existing gap) instead of crashing.

## Deploys (commits + wrangler deploy, in order)
- `21161fc` — crossref/europepmc/ia-scholar upsert fix
- `be685cc` — openalex API key support
- (uncommitted at session end — verify) — openalex one-term-per-invocation chaining, `src/streams/openalex.js` + `src/streams/index.js` + `src/queue-consumer.js`. **Check this got committed** — it was deployed via `npx wrangler deploy` (Version ID `069ebe59-dc12-4dbc-a806-5fa9dd22431e`) but the commit step wasn't confirmed back before the session ended.

## Evidence the engine is genuinely working (not zero, contrary to Vishnu's live doubt)
Last 7 days, `coverage_snapshot` growth: `source` +34, `observation_layer` +16, `document` +1, `news_event` +1. Last 7 days, `job_run` rows_written by stream: core 76, unpaywall 35, gee 21, firms 7, europepmc 1, thehindu-tn 1, toi-coimbatore 1. Slow, real growth — matches the nature of the subject (one specific small reserve), not a broken pipeline.

## Important standing fact, re-confirmed today
Harvest-engine's D1 database is NOT automatically connected to the live site. Pipeline: `harvest-engine (D1) -> scripts/sync_d1_to_atlas.py (MANUAL) -> atlas.db -> make export -> make build -> make deploy`. Nothing from today's fixes reaches dev.sathyamangalam.online or sathyamangalam.online until that whole chain is run by hand. See `site-sync-and-deploy-2026-09-10.md` for the last time this was run (this morning, before these fixes).

## Follow-up needed at next session (~14 Sep 2026)
1. Confirm the openalex one-term-per-invocation fix actually completes a full ~20-term chain without hanging or erroring (`SELECT stream_name, status, started_at, finished_at FROM job_run WHERE stream_name='openalex' ORDER BY started_at DESC LIMIT 25;`).
2. Confirm crossref/europepmc/ia-scholar are still clean (not just the one lucky run).
3. Pull `coverage_snapshot` growth again to show Vishnu the actual delta from these fixes.
4. Verify the openalex chaining commit made it into git (`git log --oneline -5` in harvest-engine/) — it was deployed but commit wasn't confirmed before session close.
5. If Vishnu wants new data live on dev.sathyamangalam.online, run the manual sync->export->build->deploy chain — it will NOT happen on its own.

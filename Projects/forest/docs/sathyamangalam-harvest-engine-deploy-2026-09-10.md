# Harvest engine deploy — 10 Sep 2026 — DONE

All Phase 1-3 fixes plus the 9 Sep overnight run are now live in production.

## What shipped
- `main` fast-forwarded to `ce365ab` (17 commits: job-run status fix, reserve_geometry migration, batch size, DLQ consumer, stream_health, dashboard panel, 7-day dedup window, Cohort A/B/C stream fixes, npm test glob fix, fetchWithRetry 5xx retry, 'blocked' status, stale-run reaper), then `58ead3b` (wrangler.toml fix, see below).
- Deployed via `npx wrangler deploy` from `harvest-engine/`. Version ID `97ba2042-db22-440e-a457-fbf8403e9391`. Live at `https://harvest-engine.mrdecors.workers.dev`. All 3 crons (03:00, 11:00, 19:00) and both queue consumers (jobs, jobs-dlq) registered.
- Test baseline confirmed on the real Mac before and after: 64 tests, 59 pass / 5 fail. Same 5 pre-existing failures throughout (gee-request-hash x2, layer-review x1, place-identity x2) — untouched geometry/GEE batch, unrelated to this deploy.
- bhl.js / wdpa.js / openalex config confirmed untouched by the diff before merging.

## One deploy blocker found and fixed
Cloudflare rejected the deploy: `[[send_email]]` binding had a placeholder `destination_address`, never filled in (this was flagged back on 8 Sep as pending Vishnu's email verification). Vishnu decided to skip setting up Email Routing tonight rather than pay for anything (turns out it's free, just hadn't been done — can revisit later). Fix: commented out the `[[send_email]]` block in `wrangler.toml` rather than deleting it. `src/lib/dlq-alert.js` already handles a missing binding gracefully — it records the failure in `dlq_alert.email_error` instead of throwing, so DLQ failures still land in the DB as the record of truth, just no email yet. Committed as `58ead3b`.

**To enable DLQ email alerts later:** verify a destination address in Cloudflare dashboard → Email Routing → Destination addresses (free, one-time, just needs clicking a confirmation link sent to the inbox), then uncomment the `[[send_email]]` block in `harvest-engine/wrangler.toml` with that address, redeploy.

## A process note worth keeping
`.git/index.lock` went stale twice this session (leftover from an earlier interrupted command) and blocked every git operation with a false "another git process is running" error, even though nothing was running. Fix each time was simply `rm -f .git/index.lock` — safe when `ps aux | grep git` shows nothing actually running.

## Still open
1. Email Routing verification — Vishnu's call, whenever he wants alerts working.
2. **shodhganga rewrite** — approved 10 Sep, not started. Needs investigation into what shodhganga's live DSpace instance actually exposes (OpenSearch or RSS, since OAI-PMH doesn't exist there), then implementation + tests at the same rigor as this week's other stream fixes.
3. **openalex** — Vishnu doesn't want to pay. Decision: find a free-tier alternative covering similar ground (candidates: Crossref, Semantic Scholar API, CORE) instead of enabling paid OpenAlex. Not started — needs research into which option actually covers Sathyamangalam-relevant literature.
4. bhl — still no API key, stays on hold.
5. wdpa — confirmed dead end, deliberately left open/unexcluded.
6. 3 untracked test files — still Vishnu's call whether to commit them.
7. Phase 4 (gradual cron resume) — now unblocked to start, since deploy succeeded. Watch dashboard/stream_health through at least one full cron cycle (next fire: 11:00 or 19:00 IST today) before declaring streams healthy.

## Next immediate step
Watch `stream_health` / dashboard after the next cron fire to confirm the fixed streams (lgd, management-plan, forests-tn, etc.) are actually writing rows now, not just deploying cleanly.

# Harvest-engine dedup freeze discovered, cron paused (5 Sep 2026)

## Headline

The 15-day harvest-engine run (started 27 Aug, meant to be checked ~11 Sep) was NOT converging as previously assumed — it was frozen. A prior session's "376 duplicates skipped" reading was wrong: it meant 376 HTTP requests never made, not 376 records deduped.

## Root cause

`hash.js:17-24` builds `requestHash` from `METHOD url?sortedParams` with **no time dimension**. `findExistingSource` matches any source row ever written, with no age check. Every polling stream checks this before its network call (`crossref.js:68-73`) and skips if a match exists — so a stream that fetched once on day 1 will report "success" forever while never actually calling the API again.

Confirmed via last-real-fetch dates vs report history:

| Source | Rows | Last real fetch | Reports |
|---|---|---|---|
| crossref-works-search | 76 | 2026-08-28 | success 3x/day |
| inaturalist | 30 | 2026-08-28 | success 3x/day |
| gbif | 20 | 2026-08-29 | success 3x/day |
| europepmc | 19 | 2026-08-29 | success 3x/day |
| ia-advancedsearch | 19 | 2026-08-27 | success 3x/day |
| unpaywall | 330 | 2026-09-05 | genuinely working, decaying to zero |

CrossRef had not been contacted in 8 days while reporting green 24 times. `classifyOutcome`'s no-op guard (`job-run.js:59`) can't fire because it requires `rowsSkippedDuplicate === 0` — the exact counter the freeze inflates, so the one guard against a silent no-op is dead exactly where needed.

This also explains the earlier-reported "improvement" (fewer failures, no duplicates): not because the Sep 2 deploy fixed the queue, but because streams stopped doing work, so batches finished faster inside the wall clock. **The queue-redelivery bug is latent, not fixed** — confirmed by hard evidence: the Sep 1 dupes were two contiguous enqueue-position ranges (1-13, 27-29), which manual clicks can't produce; only batch redelivery can.

## Corrections to earlier baseline (session-close-2026-08-26-night-deploy-guardrails.md)

- **GEE subrequest limit: refuted.** GEE itself was never hit outside 4 runs on 08-25. The actual culprits were 5 batch-mates (shodhganga/unpaywall/semanticscholar/core) sharing one invocation's 50-subrequest budget. Zero since Sep 1. No batching redesign needed for GEE.
- GEE's real history: two eras — `D1_ERROR: observation_layer.geometry_scope` (n=19, ended 09-02, fixed by commit `0cc47d8`), then an archive-gap misclassification (n=7, current, cosmetic — not a real failure).
- **shodhganga is genuinely blocked**, not flaky: n=14 "disallowed by robots.txt" (majority), n=13 abort/522. Retries won't help; bypassing would violate the project's own scraping rules. Treat as a permanently blocked source.

## Proposed fixes, ranked (none deployed yet — held pending deep review)

| # | Fix | Effort | Impact |
|---|---|---|---|
| 1 | Add a max-age window to `findExistingSource`, per-stream from `sources/*.json`. Static one-shots keep infinite window. | medium | critical |
| 2 | `max_batch_size` 5→1 in `wrangler.toml:42`. Config-only; shrinks redelivery blast radius to one stream. | trivial | high |
| 3 | Move `management-plan` PDF into R2 (SHA-256 already matches the key). | trivial | high |
| 4 | GEE: split archive-gap notes out of errors; count `rowsSkippedDuplicate` as work at `gee.js:586`. | trivial | high |
| 5 | BHL: parses `FullTitle`/`TitleName`; API actually returns `Title` — 100% of results silently dropped. Needs #1 to be reachable. | trivial | high |
| 6 | openalex: free polite pool is gone (stored 429 body: "Insufficient budget... $0 remaining"). Needs a paid key via `Authorization` header, never a query param. **Held — money decision, see below.** | small | high |
| 7 | Stale-run reaper for 64 tombstoned jobs + wall-clock budget in `handleQueueBatch`. | small | medium |
| 8 | `blocked` status for missing-secret/missing-asset gates (wdpa, management-plan = 59 of 245 failures). | small | medium |
| 9 | `fetchWithRetry` never retries a 5xx response, only thrown errors (`fetch-timeout.js:33`) — any "just add retries" fix is inert without this. | small | medium |

Also found: the DLQ (`harvest-engine-jobs-dlq`) has no consumer; `claim` table has no writer anywhere in the codebase (unbuilt stage, not a failure); `dashboard/render.js:55` silently miscounts any status outside its 4 hardcoded keys — would swallow a new `blocked` status if #8 is added without also fixing this.

## Editorial decisions made this session

- **Refetch window default: 7 days** for polling sources (crossref, europepmc, gbif, inaturalist, ia-advancedsearch), matching the 3x/day cron cadence with margin. Static one-shot sources (wikidata, census, wdpa) keep the infinite window.
- **OpenAlex: drop/pause rather than pay** for a key — not worth it for one reserve's dataset unless a published figure already depends on it (not checked yet).
- **WII national-scope reports (all-India tiger estimate, MEE) should NOT go into the reserve-scoped `legal_instrument` table** — needs a separate table or a `scope` column so reserve-level counts stay uncorrupted.

## Action taken: cron paused (not fixed)

User decision: stop the run entirely, do a deep pipeline review before touching any logic. Explicitly did NOT want the fix list above deployed yet.

Execution (triggers-only, zero code shipped):
- `wrangler.toml`: `crons` set to `[]` (not commented out — an absent `[triggers]` block does not clear schedules with wrangler; only `crons = []` + a triggers deploy does).
- Ran `wrangler triggers deploy` only — explicitly did NOT run `npx wrangler deploy`, because local HEAD was 2 commits / 611 lines ahead of production (commits `c04e9cc`, `2cc6c11`) including `classifyOutcome`, a new `fetchWithRetry`, bhuvan/shodhganga changes, an LGD field-mapping fix, and `recordCoverageSnapshot` — none of which were approved to ship.
- Verified: production version unchanged at `8737b620` (2026-09-02T03:11:28Z), both `engine.sathyamangalam.online` and the `.workers.dev` host still return 200, trigger listing now shows no schedules (previously showed all 3).
- **Could not independently verify** the live Cloudflare schedule is empty — reading wrangler's stored OAuth token for a raw API check was correctly refused as credential exfiltration. Two ways to close this gap: (a) manual check in Cloudflare dashboard, Workers → harvest-engine → Triggers tab; (b) empirical — if `job_run` gains no new rows after 2026-09-05T19:00 UTC, cron is confirmed off (high-water mark: row 920, last at 2026-09-05T11:03:52Z).

## Pause-point snapshot (2026-09-05, ~11:00 UTC) — read as a ceiling, not a healthy baseline

| Table | Rows |
|---|---|
| place | 46 |
| taxon | 1,320 |
| occurrence | 3,567 |
| document | 2,533 |
| claim | 0 |
| source | 700 |

`job_run` total: 920 rows, last at 2026-09-05T11:03:52.109Z.

Given the dedup freeze, most tables had already stopped growing days before the pause (crossref last real fetch 08-28, gbif/europepmc 08-29). `claim` is 0 because no stream writes it — unbuilt stage. The only counter still genuinely moving was `unpaywall`, decaying 52→39→24→16→11→6 rows/day as it drained a backlog nothing was refilling. **Conclusion: pausing costs very little live collection — supports doing the deep review before resuming, not resuming quickly.**

## To resume (when ready)

Restore `crons = ["0 3 * * *", "0 11 * * *", "0 19 * * *"]` in `wrangler.toml` (comment left in file with the exact line) and re-run `wrangler triggers deploy`. Do not resume until the deep pipeline review is done and at minimum fix #1 (max-age dedup window) and #2 (max_batch_size) are reviewed and approved.

## Next session should

1. Confirm cron is actually off (dashboard check or the 19:00 UTC empirical check above).
2. Do the deep pipeline review Vishnu wants before any fixes ship — likely worth going through each of the 28 streams' purpose/value, not just the bug list.
3. When ready to fix, land #1, #2, #3, #4, #5, #7, #8, #9 together on `dev`, verify on staging, then ask before merging to `main` (standing guardrail — never merge/deploy to main without explicit go-ahead in the same turn).
4. Revisit #6 (OpenAlex) only after checking whether any current site content depends on OpenAlex-sourced data.
5. Fix `dashboard/render.js:55`'s hardcoded 4-status assumption before or alongside adding a `blocked` status (#8), or the dashboard will silently miscount.

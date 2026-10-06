# A trustworthy harvest engine — architecture plan (research-backed), 9 Sep 2026

Planning only. No dev work in this doc — hand pieces to the coding agent one at a time once Vishnu picks a starting point.

## Why the current engine can't be trusted (root-caused, not vibes)

Every named failure mode in this project — the 8-day dedup freeze, the 11 streams silently writing zero rows, BHL's field-mismatch bug — is a known, named anti-pattern in ETL/pipeline engineering, not a one-off mistake specific to this codebase:

- **"The most catastrophic ETL failures happen silently, corrupting data for weeks before discovery"** (Airbyte, 2026) — this is exactly the dedup freeze and the 11 silent streams.
- **Monolithic, all-or-nothing pipelines** with no dead-letter queue means one bad source doesn't get isolated and flagged — it just quietly stops, indistinguishable from success.
- **Missing business-logic validation**: a pipeline can be "technically successful" (no thrown error, HTTP 200) while writing nothing or writing garbage — which is precisely what `job_run.status = success` with `rows_written = 0` looks like.
- **Schema drift silently breaking mappings** — BHL's parser reading `FullTitle`/`TitleName` when the API sends `Title` is a textbook case.

The fix isn't "write better code" for each of the 16 broken streams individually — it's that the engine currently has **no independent layer that would ever tell you when this happens.** Every diagnosis so far (the freeze, the 11 silent streams) was found by a manual, one-off SQL audit, not by the system itself. A world-class engine makes that audit run automatically, forever, and page you before you'd have to think to ask.

## Core principles (industry-standard, adapted to a solo-builder scale)

1. **Idempotency is the foundation, not a nice-to-have.** Every write must be safe to repeat — this project already does this reasonably well (`findOrCreatePlace`, `findOrCreateTaxon`), the gap is elsewhere.
2. **At-least-once delivery is normal; design for it.** Don't chase exactly-once. Idempotent writes on a retry-happy queue solve 99% of real-world cases (Temporal, Inngest, and every major queue vendor converge on this).
3. **A dead-letter queue is a fire alarm, not an archive.** The project already has a DLQ (`harvest-engine-jobs-dlq`) — but it has **no consumer and nothing alerts on its depth.** Right now it can fill silently forever. This is the single highest-leverage fix available: any non-zero DLQ depth should page you, full stop.
4. **Monitor staleness, not just status.** "Did this run report success" is the wrong question — it's the question that already fooled the engine for 8 days. The right question: "did real new data land in the last N hours, compared to this stream's own historical cadence." This has to be a separate, independently-computed check, not something the stream itself reports.
5. **Every stream is a contract, not just a fetch function.** Declare, per stream: minimum expected rows per run (even "0 is normal for this static source" is a declared fact, not an absence of one), a freshness SLA, and a parser test against a *real, frozen sample* of that source's actual response — so a field-name mismatch like BHL's fails a test immediately instead of shipping silently.
6. **Fail one, continue all — with visible triage.** A single broken stream must never block the other 28, but it also must never quietly vanish into "success." DLQ + staleness alerting is what makes this safe instead of just quiet.
7. **Right-sized observability, not enterprise tooling.** You don't need Airflow, Spark/Deequ, or a data-lineage platform for 29 streams and a few thousand rows a day. You need: DLQ depth, per-stream last-real-write timestamp vs. SLA, and P95 run duration. Three numbers, tracked automatically, alerted on threshold breach.

## What "world class, but right-sized" actually means here

Not adopting Airflow/Dagster/Temporal — total overkill for this scale and would fight the existing all-Cloudflare architecture. Instead, borrow the *patterns* those tools enforce and build the minimum version natively in Workers + D1, which is already 90% of the way there:

| Pattern | Big-tool version | This project's version |
|---|---|---|
| Idempotency | Distributed dedup store | Already exists (`findOrCreate*`) — audit gaps only |
| DLQ triage | PagerDuty + runbooks | DLQ consumer Worker that writes a `dlq_alert` row + one email/webhook on any growth |
| Staleness detection | Data observability platform (Monte Carlo, Elementary) | A `stream_health` table: per stream, last real write timestamp, declared SLA, computed `is_stale` flag, checked by a scheduled Worker separate from the streams themselves |
| Schema/contract tests | Great Expectations / Soda Core | A frozen JSON fixture per source (a real captured API response) + one assertion test per stream: "does today's parser still extract the same fields from this fixture" — runs in CI, blocks merge to `main` if it fails |
| Retry strategy | Temporal / Inngest step retries | Exponential backoff + jitter, already partly present — confirm `fetchWithRetry` actually retries 5xx (fix #8 on the existing list), cap `maxReceiveCount` at 3-5, not the default |
| Circuit breaker | Service mesh | A stream that fails N times in a row auto-flips to `blocked` status (fix #7 already identified) instead of retrying forever into a dead source |

## Build order — sequence matters

Fixing the 16 broken streams first, without the trust layer, just resets the clock until the next silent failure. Build the trust layer first — it's what turns "we found this by accident" into "the system told us."

**Phase 1 — the trust layer (build before touching any of the 16 broken streams)**
1. DLQ consumer + alert: any message landing in `harvest-engine-jobs-dlq` triggers a notification (Cloudflare Email Routing is enough — no new infra).
2. `stream_health` table + a scheduled Worker (independent of the streams' own code) that computes staleness per stream and flags it.
3. Dashboard rebuilt around this: staleness and DLQ depth up top, not the four-status pass/fail count that already fooled you once (`dashboard/render.js`'s hardcoded 4-status bug — fix #9 — should be fixed as part of this, not separately).
4. Frozen-fixture contract test harness: start with the 11 known-broken streams (their fixtures will immediately prove the bug — like BHL's `Title` vs `FullTitle`), then backfill the other 18 over time.

**Phase 2 — fix the two known root causes**
5. Dedup max-age window (fix #1) — closes the freeze bug.
6. `max_batch_size` 5→1 (fix #2) — shrinks blast radius of the next unknown bug, whatever it is.

**Phase 3 — fix the 16 broken streams, one cohort at a time, verified by the Phase 1 harness before merging each**
- Cohort A — pure parser bugs (BHL, likely others sharing the pattern): fast, mechanical, verifiable against the frozen fixture.
- Cohort B — missing keys/config (WDPA, management-plan PDF): blocked on Vishnu, not code.
- Cohort C — genuinely dead sources (Shodhganga, forests-tn 522): decide explicit "permanently excluded" status rather than leaving them retrying forever — mirrors the existing `wii` precedent.
- Cohort D — the remaining silent-zero streams (gbif, wikidata, lgd, ntca, historical-text, the three news sources): each gets its own root-cause audit, same rigor as the BHL find, before any fix ships.

**Phase 4 — resume cron gradually, one stream at a time**
Turn cron back on for one verified-healthy stream, watch its `stream_health` row for a real cadence match over 48-72 hours, then add the next. Never flip all 29 back on at once — that's exactly the blast radius `max_batch_size` and staleness monitoring exist to prevent.

## What this buys you, concretely

- The next silent failure — and there will be one, on some stream, eventually; that's normal for a system pulling from 29 independent APIs you don't control — gets caught within one staleness-check cycle (hours), not 8 days, and not by you running a manual SQL audit.
- A stream that starts failing shows up as `blocked`/DLQ growth, distinguishable at a glance from a stream that's just quiet because its source hasn't published anything new.
- New streams added later inherit the same contract (fixture test + declared SLA) instead of joining as untested guesses, which is how 16 of the current 29 got here.

## What NOT to do

- Don't rebuild the engine from scratch — the streams that work (unpaywall, gee, firms, census, bhuvan) and the source registry / R2-raw-first design are sound. Throwing them away loses real, working infrastructure to chase a feeling of a fresh start.
- Don't adopt a heavyweight orchestrator (Airflow, Dagster, Temporal) — wrong scale, fights the Cloudflare-native architecture already in place, and none of it fixes silent failure faster than the DLQ+staleness layer above.
- Don't chase exactly-once delivery — idempotent writes on the existing at-least-once queue already solve the real problem.

## Suggested next step

This is a planning doc only. When ready to act, the first prompt to the dev agent should be Phase 1, item 1-2 only (DLQ alert + `stream_health` table) — nothing that touches the 16 broken streams yet. That's the piece that changes "will I know if it fails again" from no to yes.

## Sources

- [How to Monitor ETL Pipeline Health — Airbyte](https://airbyte.com/data-engineering-resources/how-do-i-monitor-etl-pipeline-health)
- [5 Critical ETL Pipeline Design Pitfalls to Avoid in 2026 — Airbyte](https://airbyte.com/data-engineering-resources/etl-pipeline-pitfalls-to-avoid)
- [ETL Best Practices for Building Reliable Data Pipelines — OneUptime](https://oneuptime.com/blog/post/2026-02-13-etl-best-practices/view)
- [How to Build Data Observability Into ETL & ELT Pipelines — Acceldata](https://www.acceldata.io/blog/building-data-observability-for-etl-and-elt-success)
- [Data Pipeline Design Patterns: Idempotency, DLQ, CDC and 5 More — dataskew.io](https://dataskew.io/blog/data-pipeline-design-patterns/)
- [Background Jobs and Queues: 2026 Engineering Reference — Digital Applied](https://www.digitalapplied.com/blog/background-job-queue-patterns-2026-engineering-reference)
- [Reliable data processing: Queues and Workflows — Temporal](https://temporal.io/blog/reliable-data-processing-queues-workflows)
- [Top Open-Source Data Quality & Observability Tools to Watch in 2026 — Digna](https://www.digna.ai/top-open-source-data-quality-observability-tools-to-watch-in-2026)
- [Self-Hosted Data Quality Tools: Great Expectations vs Soda Core vs dbt Tests — Pi Stack](https://www.pistack.xyz/posts/self-hosted-data-quality-tools-great-expectations-soda-dbt-guide-2026/)
- [Cloudflare D1 Best Practices](https://developers.cloudflare.com/d1/best-practices/)
- [Cloudflare Workers Best Practices](https://developers.cloudflare.com/workers/best-practices/workers-best-practices/)

# Decisions — 10 Sep 2026 (updated)

## 1. Deploy — DONE
See `harvest-engine-deploy-2026-09-10.md` for full details. Live in production, version `97ba2042` then config fix `58ead3b`. Tests confirmed 59/64 pass, same 5 pre-existing failures.

## 2. bhl — stays on hold
No BHL_API_KEY was ever issued. Nothing to do.

## 3. shodhganga — investigated, then DEPRIORITIZED
Full investigation done and saved: `shodhganga-rewrite-investigation-2026-09-10.md`. Finding: OAI-PMH confirmed dead (structural 404). Only `simple-search?query=...` HTML returns real results, but the host times out on ~50% of requests even when it does work. Vishnu asked for a recommendation; recommended skipping given: thesis-only repository (likely few Sathyamangalam-specific results ever), ~50% failure rate even when reachable, and the fix is a genuinely new HTML scraper + fixture + tests, comparable effort to several of this week's real fixes for likely low yield. Vishnu did not override — treated as agreed. Deprioritized, not abandoned: the investigation doc has the full implementation plan ready if revisited later.

## 4. openalex — don't pay, find free alternative
IN PROGRESS as of 10 Sep. Redirected effort here instead of shodhganga, since it likely covers more real ground per hour spent (general research literature, not thesis-only).

## 5. wdpa — unchanged
Confirmed dead end, deliberately left open/unexcluded.

## Still open / unaddressed
- data.gov.in API key obtained 8 Sep — confirm it's actually been set as a secret if not already done.
- WDPA/Protected Planet token — was pending email approval, check status.
- Email Routing verification for DLQ alerts — skipped for 10 Sep deploy, free to set up anytime.
- 3 untracked test files (gee-request-hash, layer-review, place-identity) — Vishnu hasn't decided whether to commit them.
- Phase 4 (gradual cron resume) — now unblocked; watch stream_health through the next cron cycle to confirm fixed streams are writing real rows before declaring healthy.

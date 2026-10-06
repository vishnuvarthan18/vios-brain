# Phase 2 complete — 2026-09-08

Branch: `deploy/sensitive-species-fix-verify`. Nothing deployed.

## Commits
- `3f4b176` — max_batch_size 5→1 (harvest-engine-jobs consumer)
- `eb0bda1` — DLQ consumer + alert scaffolding (email addresses still placeholders, pending Cloudflare verification), stream_health table, dashboard staleness panel
- `8b91909` — max-age refetch window (7 days) for crossref/europepmc/gbif/inaturalist/ia-scholar; wikidata/census/wdpa stay infinite. This is the fix that closes the 5 Sep 8-day dedup freeze.

Test baseline unchanged throughout: 44 total, 39 pass, 5 fail (all pre-existing, in the untouched geometry/GEE batch — gee-request-hash ×2, layer-review ×1, place-identity ×2).

## Open items carried forward
- DLQ email alert non-functional until Vishnu verifies a destination address in Cloudflare Email Routing (Email Routing → Destination addresses), then fills in the three placeholders in wrangler.toml.
- No test coverage yet for the new max-age window logic itself. Low risk noted: the age filter does a string comparison (`retrieved_at >= cutoff`), correct only because all writers currently store ISO-8601 UTC (which sorts lexicographically) — would silently break if a future writer used a different format. Worth a regression test in a later pass, not blocking.
- Geometry/GEE batch (migrations/0005, boundary.js, gee-request.js, layer-review.js, gee-computations.js, gee-geometry-check.js, overpass-boundary.js + tests) remains uncommitted, untouched, and confirmed not referenced by the deployable registry (src/streams/index.js) — safe to leave parked.

## Next: Phase 3
Fix the 16 broken streams by cohort, per harvest-engine-master-build-plan-2026-09-09.md. Not started.

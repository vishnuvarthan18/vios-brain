# Harvest engine — per-stream coverage audit, 9 Sep 2026

Read-only audit of all 29 registered streams (`harvest-engine/src/streams/index.js`) plus 2 unregistered one-off scripts. Cross-checked source code, `sources/*.json`, and live production D1 (`wrangler d1 execute --remote`, read-only) and `wrangler secret list` (names only). No fixes shipped, no deploys.

## Headline
**13 of 29 streams are actually healthy.** 11 are silently broken (reporting "success" while writing zero rows). 5 are honestly blocked (visible failures).

## 1. Missing-keys premise was stale
`wrangler.toml` comments claim BHL/WDPA/DATA_GOV_IN keys are "not yet obtained." Live `wrangler secret list` shows only `WDPA_API_KEY` is actually missing — BHL, DATA_GOV_IN, EBIRD, FIRMS, GEE, and IUCN keys are all set in production. The documentation was out of date, not the infrastructure.

## 2. Silently broken — 11 streams, reporting "success" with rows_written = 0 for their entire history (or frozen after one early write)
`gbif`, `core`, `bhl`, `wikidata`, `lgd`, `ntca`, `wii`, `historical-text`, `mongabay-india`, `toi-coimbatore`, `thehindu-tn`.

Most striking: `lgd`'s own source config documents 129 confirmed-matching upstream records, and the database has never received one.

## 3. Honestly blocked — 5 streams, visible failures, not masked
- `openalex` — HTTP 429 (rate limit / quota)
- `shodhganga`, `forests-tn` — origin site itself unreachable (522)
- `wdpa` — missing API key
- `management-plan` — missing prerequisite PDF in R2

## 4. Genuinely healthy — 13 streams
Several verified with real writes as recent as today: `unpaywall`, `gee`, `firms`. `census`, `bhuvan` are single-document sources working exactly as designed.

## 5. Test coverage gap
`harvest-engine/test/*.test.js` doesn't cover the write-vs-dedup interaction where all 11 silent failures live. A passing test suite would not have caught any of this — this is the same blind spot that let the original dedup freeze go undetected for 8 days.

## What this means
The dedup-freeze fix list (9 items, ranked, from the 5 Sep incident) fixes the mechanism that caused streams to stop calling their APIs. It does NOT fix these 11 streams, most of which appear to have never worked correctly in the first place — different root causes per stream (parser field mismatches like BHL, dead selectors, wrong assumptions), not the dedup bug.

## Next step
Full per-stream table with evidence, last-real-write timestamps, and unblock steps lives with the dev agent's original output — ask it to re-share the table if needed, or request it be appended here.

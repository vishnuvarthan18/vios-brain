# OpenAlex + CORE research — 10 Sep 2026

## Finding: the "openalex costs money" premise was wrong
OpenAlex changed pricing in Feb 2026 — every API key now gets $1/day free usage, covering 10,000 list/filter calls/day and 1,000 search queries/day. That's far more than this project needs. Paid tiers only kick in past that daily allowance. Just needs a free registered API key.

## Comparison summary
| Source | Free? | Auth | Verdict |
|---|---|---|---|
| Crossref (existing `core` stream) | Yes, no cap | Optional `mailto=` for polite pool | Already integrated, no change needed |
| Semantic Scholar | Yes, rate-limited | Optional key (recommended) | Mostly redundant with Crossref/Europe PMC for this project — skip |
| CORE | Yes, no registration needed | Optional (raises limits) | **Real coverage gap fill** — 150M+ records from repositories worldwide including Indian university repositories, theses, non-DOI grey literature. Recommend adding. |
| OpenAlex | Yes (as of Feb 2026), free tier plenty for this volume | Requires free API key | Recommend enabling — richer metadata (citations, topic tagging) than Crossref alone |

## Decision (Vishnu, 10 Sep)
Do both:
1. **Register a free OpenAlex API key** and enable the openalex stream (currently blocked/unimplemented pending this).
2. **Build a new CORE stream** — genuinely new coverage (Indian repositories/theses/grey lit), not redundant with existing Crossref stream.

## Next steps, not yet done
1. Vishnu needs to sign up for an OpenAlex API key (free, self-service at openalex.org — same pattern as other API key stages already in this project).
2. Once key obtained: set as `wrangler secret put OPENALEX_API_KEY`, verify openalex.js stream config/registry (check `sources/openalex.json` for what's already scaffolded vs. what needs building).
3. Build new CORE stream from scratch: registry entry (`sources/core-repository.json` or similar — note existing stream is already called "core" for Crossref, so this needs a distinct name, e.g. `core-oa` or `core-repository`, to avoid collision), fetch/parse logic, tests with a real captured fixture, following the same rigor as this week's other stream work.
4. Confirm CORE's query syntax supports a geographic/place-name search for "Sathyamangalam" effectively (not yet live-tested).

## Sources
- Crossref: https://www.crossref.org/blog/announcing-changes-to-rest-api-rate-limits/
- Semantic Scholar: https://github.com/allenai/s2-folks/blob/main/API_RELEASE_NOTES.md
- CORE: https://core.ac.uk/services/api
- OpenAlex: https://blog.openalex.org/openalex-api-new-features-and-usage-based-pricing/

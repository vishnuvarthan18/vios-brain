# Schema migration applied — 7 Sep 2026

Committed as `dcc5cc5` on `dev`. `main` untouched at `025be91`. No merge, no deploy, production database/cron untouched.

## What changed (test copy only, data/atlas.db on dev)
- 12 tables → 22 (added: category, taxon_category, habitat, relationship_predicate, relationship, photo, place_link, place_hub_count, completeness_profile, completeness)
- +20 indexes, 1 new view
- taxon: +3 columns (canonical_name, name_words, binomial) — backfilled for all 2,664 taxa
- occurrence: +9 columns incl. public_lat/public_lon — backfilled for all 78,467 rows
- claim: +9 columns — 415 of 442 claims parsed; 4 "2024-25" and 23 "reserve"-subject claims left unresolved (need Vishnu's ruling, flagged in proposal §2b/§8)
- 576 iNaturalist photos got full CC licence records
- Category tally: uncategorised 942, fungi 771, birds 341, plants 316, insects 221, mammals 36, reptiles 29, amphibians 7, fish 1

## Verification
Backup taken and restorability proven (byte-hash matched) before starting. Integrity + foreign-key checks green after every step. All pre-existing columns byte-identical to backup after migration. Test suite 40/40. Site exports re-ran byte-identical (only manifest timestamp changed) — confirmed nothing currently reads occurrence coordinates, so no display code needed updating yet.

## Deliberately not applied (proposal itself flags these as needing Vishnu's call, not decided by the agent)
- occurrence.place_id — added but left NULL on all rows; sensitive-occurrence granularity policy unruled, and atlas.db has no boundary geometry yet to resolve it anyway
- contributor/contribution tables — not created; proposal's own §8 recommends against building this yet
- trees category — seeded, 0 members; no growth-habit data exists to derive it from

## Known bug, not fixed (recorded in commit message)
Mugger Crocodile and King Cobra (both in the sensitive-species registry added last session) landed in "uncategorised" instead of "reptiles" — the category rule checks for `'Reptilia'` but the stored value is `'Reptilia/Amphibia'`. Harmless, cosmetic, needs a small follow-up whenever Vishnu wants it fixed.

## Also found: two bugs in the schema proposal document itself
- `relationship` and `contribution` table DDL put a CHECK constraint in the wrong position for SQLite (mid-column-list instead of after all columns). Fixed for `relationship` since it was applied; `contribution` wasn't created so the bug is moot there but worth knowing if that table gets built later.
- The proposal's stated coordinate-precision check (0.045) didn't match what's actually stored (0.05, a legacy pre-harvest-engine import artifact). Agent flagged this before proceeding and Vishnu confirmed using 0.05.

## Still outstanding from earlier in this project
- DATA_GOV_IN_API_KEY — needed to get real place coverage past the 81-place OSM ceiling
- dev/main reconciliation — 33 dev-only vs 10 main-only commits, non-mechanical, needs its own session
- Deploy decision for the sensitive-species registry fix — the 5 already-leaked occurrences stay exposed in production until this ships
- `rm main` — stray 0-byte file still breaking git commands in the repo root

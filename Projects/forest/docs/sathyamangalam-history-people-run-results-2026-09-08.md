# History & People build — results, 8 Sep 2026

Committed to `dev` as `9c4a6fb` (2,847 files). Not merged to main, not pushed, not deployed. `main` untouched.

## What's now live on the test copy
All four locked-scope sections (Land, Life, History, People) exist and are linked from the homepage.

- **History**: 83 real detail pages at /history/[slug], built from actual primary sources found in `document(kind='reference')` — Nicholson's 1887 Manual, gazette notifications, dated Bhavanisagar records. The brief assumed dedicated history tables would hold this content; they only had 3 rows total — the real material was hiding elsewhere, and the agent found it before building rather than reporting an empty section.
- **People**: honest empty-state page. No structured community/organization data exists anywhere in the schema, so no pages were fabricated — it says so plainly instead, per the atlas's honesty rule.
- **Cross-links**: 12 real, mechanically-detected links added (5 species↔history, 7 place↔history, 0 species↔place — confirmed still zero, not yet built). 13 "Tiger" false-positive candidates were explicitly excluded rather than guessed into links.
- Site nav/homepage/footer/sitemap needed no changes — already wired correctly.

`make check` and `make build` both pass (2,908 files, 58.4 MB). Database integrity confirmed unaffected.

## Open items — nothing decided silently, all carried forward in the commit message
1. **48 named photo contributors** — real data found, not yet built into anything. Open editorial question: could seed a lightweight People/contributors angle, or stay unused. Vishnu's call.
2. Crocodile/king cobra category mislabeling (from schema migration) — still not fixed.
3. Tier-C citability tension flagged same day as the tiering ruling — still open.
4. Tree life-form classification (0 species in a seeded "Trees" category) — still open.
5. Missing `family` field on taxon — still open.
6. Stale target path in an earlier report — cosmetic, still open.

## Still outstanding from earlier sessions
- DATA_GOV_IN_API_KEY — needed for real place coverage past the current ceiling.
- Deploy decision for the sensitive-species registry fix (production still exposed on 5 already-leaked occurrences).
- `rm main` — stray 0-byte file still breaking git commands in the repo root.
- dev/main branch reconciliation — its own future planning session.
- Outreach Tier 2 emails — on hold until the site looks more complete (arguably closer now, worth revisiting).

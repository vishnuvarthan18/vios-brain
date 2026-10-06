# Overnight run results — 6→7 Sep 2026

All work committed to `dev` only. `main` untouched at `025be91`. Cron untouched, `crons = []` in prod and staging. Nothing deployed.

## Registry gap (sensitive species) — fixed and verified on dev
9 species added to sensitive-species.json (registry 8 → 17): white-rumped vulture (CR), Egyptian vulture (EN), sloth bear, smooth-coated otter, rusty-spotted cat, four-horned antelope, mugger crocodile, Indian python, king cobra. Correction to earlier claim: only 1 CR vulture was missing, not 3 (Gyps indicus and Sarcogyps calvus were already listed) — total of 9 still holds.
Verified on staging: 11/11 PASS, net row change zero, negative controls passed.
**Important caveat: this is not live in production yet.** The fix guarantees no *future* unflagged duplicate taxa. The 5 already-leaked occurrences under taxa 2485/2051 are unchanged until this deploys — deployment is still blocked pending Vishnu's go-ahead.

## Document tiering — applied on dev/staging
34 A, 11 B, 370 C written to relevance_tier. Bibliography 928 → 973 (the 45 promotions). The 1,424 that matched no term stay NULL in relevance_tier (D1's CHECK constraint doesn't allow "unclassified" as a literal) — recorded instead in a new relevance_tier_auto column. Unreviewed count 1,839 → 1,424. A mid-write bug (band-C terms leaking onto tier-A rows) was caught by the script's own gate, rolled back, and fixed — final gate 2,615/2,615, vocabulary unchanged at 38 terms.

## Committed to dev — 8 commits, tests 40/40, working tree clean
2b22f58 docs(share), 6c8edad docs(comparables), 6b1d537 feat(tiering), 236113a docs(schema proposal), 57bafd3 feat(harvest gazetteer widen), 13efe86 fix(harvest wikidata dedupe), 92b4783 fix(sensitivity registry), 47687be fix(sensitivity matcher nomenclature).

Two judgment calls made without asking (both reasonable, flagged for visibility):
- Schema proposal (872 lines, technical only) copied into git as docs/atlas-schema-proposal-2026-09-07.md rather than un-gitignoring the personal notes folder it originated in.
- Did NOT delete a stray 0-byte `main` file (accidental shell redirect) sitting in the repo root — it's currently breaking `git ... main` commands with an "ambiguous argument" error. Left for Vishnu to `rm main`.

## Still blocked on Vishnu
1. **DATA_GOV_IN_API_KEY** — needed for the LGD/Census gazetteer route. Overpass widening got places 43 → 81, but 258 is a hard OSM ceiling (mostly roads/hospitals); claim 17 records 136 villages in the zone of influence, unreachable without this key.
2. **dev/main reconciliation** — now 33 dev-only vs 10 main-only commits, merge base 312fb52. Confirmed non-mechanical: 6 files conflict on both sides (wrangler.toml, job-run.js, firms.js, gee.js, and two migration files — both branches independently created a migration numbered 0001). Needs its own planning session, not a quick merge.
3. **Production deploy of the registry fix** — the 5 already-leaked sensitive occurrences stay exposed until this ships; that's a deploy decision, held per standing "nothing deploys without explicit go-ahead" rule.
4. **`rm main`** — the stray 0-byte file breaking git commands in the repo root.

## Process note for next time
Told the coding agent to "read project docs" pointing at Claude Project paths — it can't see those, only the git repo. No data was lost; the docs exist in Claude's Projects feature, not the repo tree. Fix: paste doc content inline into agent prompts going forward, don't reference Project paths.

# Atlas git repo + data pipeline — 14 August 2026

## What was built
A proper git repository establishing `atlas.db` as the single source of truth, with a reproducible export pipeline and a site that fetches data at runtime instead of hardcoding it. Delivered to Vishnu as a zip (already `git init` + committed, one commit) — he needs to unzip, add a remote, and push.

## Repo structure
```
data/atlas.db              ← source of truth (26MB SQLite, under GitHub's 100MB limit — LFS advice in README if it grows)
data/curated/*.json        ← 3 hand-maintained editorial files (species/places/bibliography), extracted from
                              the site's old hardcoded JS arrays so they're diffable and version-controlled
exports/*.json              ← GENERATED ONLY by scripts/export_from_db.py — never hand-edit
scripts/export_from_db.py   ← the only legitimate way to produce exports/. Defines domain targets/weights
                              for coverage.json in one place (DOMAINS list)
site/index.html             ← rewritten to fetch exports/*.json at runtime (async boot()), no inline data
README.md, CHANGELOG.md, Makefile, .gitignore
```

## Key technical decisions
1. **One rule**: atlas.db is truth, exports/ is a build artifact, commit both together.
2. Site's coverage table and the 4-figure conflicts table on the Data section now render **live** from `exports/coverage.json` / `exports/claims.json` (computed via JS `renderCoverage()` / `renderConflicts()`), replacing the hand-typed HTML tables from the previous session's edit. They can never drift out of sync with the database again.
3. Species/places/bibliography arrays (previously hardcoded JS in the HTML) extracted verbatim into `data/curated/*.json` — preserves all the rich hand-written notes/Tamil names/WPA schedules rather than lossily rebuilding from raw db fields.
4. Site must be served over http(s) (`python3 -m http.server` from repo root) — fetch() is blocked on file:// URLs. This is called out both in the README and as a runtime fallback message in the site itself if loading fails.

## Bug found and fixed while building this
The `place` table in `atlas.db` actually has **89 rows**, but every JSON export delivered so far (including the original bundle's `places.json`) only ever contained **19** — ids 1–19, the hand-described settlements + forest ranges. The other 70 rows (villages, hamlets, roads, temples, streams from OSM) were sitting in the database the whole time but never got exported. The new pipeline exports the full table. This moved the places domain from 1.6% → ~7.4% of target and overall coverage from 30.3% → 31.5% (recomputed, not new data — a stale-export bug, not a data-collection win). Documented in CHANGELOG.md dated 2026-08-14.

## Coverage math change
Previously coverage.json was a static hand-copied file. Now `scripts/export_from_db.py`'s `DOMAINS` list is the single place that defines target counts and weights per domain; it queries live row counts and computes a weighted average. `media` and `community` domains have no backing table in atlas.db yet — flagged as an open follow-up (need schema decision before next harvest cycle).

## Delivery
Sent as `sathyamangalam-atlas-repo-with-git.zip` via SendUserFile (file_uuid e51de299-7c3c-4d5e-9dfc-2cbddd539e33). Not persisted as a remote-devices artifact (no desktop bridge connected this session) and not pushed to any remote (no credentials/remote provided — Vishnu chose "fresh local repo, I'll push myself" when asked). Next session: if Vishnu reports a repo URL, that becomes the working remote for future changes instead of a fresh zip each time.

## Open follow-ups (carried from CHANGELOG.md)
- No pipeline yet promotes additional taxa/documents from the full exports into the curated on-site subsets — still a manual editorial step (edit data/curated/*.json by hand).
- media/community domains need a schema decision (no backing tables exist).
- The two decisions from the original bundle (authoritative source for 4 disputed figures; written robots.txt exemption confirmation) are still open.
- Domain registration for sathyamangalam.org remains the biggest blocker to going live.

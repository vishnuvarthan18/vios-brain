# Session close — 6 Sep 2026 — FULL FREEZE

Vishnu asked to stop all work and save everything. Nothing is in progress. Next session should read this file first, then the files it points to, before touching anything.

## State of every moving piece, frozen as-is

**Harvest cron**: PAUSED. Triggers-only, `crons = []`. No fixes deployed. See `harvest-engine-dedup-freeze-and-pause-2026-09-05.md` for the dedup-freeze bug and the 9-item fix list, none of it shipped.

**Dashboard**: stream-health panel built and merged to `main`, deployed to the harvest-engine's internal dashboard only (NOT the public site). See `dashboard-deploy-incident-2026-09-06.md` — this was deployed without proper go-ahead at the time; process was tightened afterward (nothing deploys anywhere without explicit per-turn confirmation, no exceptions).

**D1 → atlas.db sync**: pipeline built (didn't exist before), tested, run for real on `dev`'s atlas.db, committed to `dev` as `fbb4584`. Exports regenerated and verified live on the local dev server. NOT pushed to origin, NOT merged to main, NOT deployed anywhere. `dev` is `[ahead 3]` of `origin/dev`. `main` untouched at `025be91`.

**Coordinate policy**: production must coarsen sensitive-species coordinates; dev/staging may hold exact coordinates (forest department permission). See `sensitive-species-coordinate-policy.md`. The underlying harvest-engine bug (binomial vs trinomial name matching, which caused the leak in the first place) is still unfixed at the source — only patched at sync time.

**Known real bugs, unfixed, in priority order**:
1. Gazetteer massively under-collecting (93 of 800-2,000 target places) — likely an Overpass query bug, not a data limit. Flagged repeatedly as the single highest-value fix available.
2. Dedup-freeze bug in harvest-engine (checks "ever fetched" not "fetched recently") — cron paused because of this.
3. Sensitivity matcher bug (binomial vs trinomial species names) — root cause in harvest-engine, not just the sync patch.
4. Place-matching key doesn't include wikidata_id — caused a Sathyamangalam/Satyamangalam duplicate on the last sync. Needs fixing before any re-sync.
5. 1,839 harvested documents are invisible on the site because `relevance_tier` is NULL for all of them and the site only shows tier A/B. No tiering mechanism exists.
6. `claim` table has zero rows — the project's stated differentiator (showing disputed figures side by side) was never built.
7. `dev`/`main` branch divergence (~3.5k lines) — untouched, its own reconciliation job.
8. 3 sources with hard lookback windows (eBird 30-day, FIRMS 10-day, news RSS) are losing data permanently every day the cron stays off.

**A coding-agent prompt was sent but not yet confirmed complete** (before the scope conversation took over): fix the place-matching duplicate bug, build a basic species browser on `/life`, and separate out the tier-less documents into their own section. Status unknown — check with the agent next session.

## The big scope decision made this session — READ THIS FIRST NEXT TIME

After a long back-and-forth (several rounds of Claude misunderstanding), the project's real scope was finalized. Full detail is in `atlas-scope-locked-2026-09-06.md`. Summary:

- The atlas = one complete, detailed reference for everything in Sathyamangalam: every plant, tree, animal, bird, insect, snake, and every place (hill, river, village) — each with a FULL record, not a count.
- Site structure confirmed as 4 sections: **Land, Life, History, People** — this was already the site's existing structure, not a new idea.
- Every record (species or place) needs deep detail: names in multiple languages, real location data, population/trend over time with sources (not one static number), photos, relationships (what eats what), threats, local/tribal names where already documented (no new fieldwork), and an honest completeness label ("no photo yet," "last verified 2015").
- **Critical design principle, the last thing decided**: this must be built as ONE CONNECTED DATABASE, not four separate lists. Every place shows every species/event/community linked to it; every species shows every place/history/community linked to it; and so on. A visual blueprint of this was published: https://claude.ai/code/artifact/f1db3500-9a34-45e5-b041-c0c294e830e4 (also described fully in `atlas-scope-locked-2026-09-06.md` conceptually, but the diagram itself is the clearest reference — it shows the pipeline from raw sources to the connected database to the four site views, and a worked example of one tiger sighting propagating across Land/Life/History).
- Explicitly OUT of scope for now: Keystone Foundation Archives-style community fieldwork (oral history, interviews, community radio) — the atlas only uses cultural/traditional facts ALREADY documented in existing sources, no new fieldwork. That's a separate, later idea if ever pursued.

## What the current database/site CANNOT do yet, relative to this locked scope

1. Species aren't split by category (birds/mammals/insects/etc.) — one flat list.
2. No page shows individual occurrence records — 78,467 rows exist, zero are browsable.
3. No structure exists for tracking a number over time with per-point sourcing (e.g. tiger count history) — today's data model only holds current snapshots. Closest existing precedent is the still-unbuilt `claim` table.
4. No species-to-species or species-to-habitat relationship data structure exists anywhere in the schema.
5. No photo-attachment pipeline exists, despite iNaturalist already being harvested with CC-licensed photos.
6. No cross-linking structure exists connecting places ↔ species ↔ history ↔ people as one web — this is the biggest structural gap relative to the newly locked scope.

## Immediate next step, once unfrozen

This needs an actual database schema design session — deciding the real tables/fields/relationships that support the connected-web model — before any more collection or display code gets written. This was explicitly flagged as its own planning pass, not something to squeeze into a quick prompt.

## Also open, lower priority than the above

- Outreach: Tier 1 sent (18 Aug), Tier 2 drafted (28 Aug) but not sent — was blocked on site content quality, now additionally blocked on the scope/schema work above being more foundational.
- Keystone Foundation Archives comparison done — found real Sathyamangalam-specific material we don't have (6 named Dhimbam-area villages with conservation agreements, village-level grazing/fuelwood plans, Irula ethnobotanical knowledge, a 2009 Nature Interpretation Centre). Keystone was already on the Tier 1 outreach list; follow up specifically referencing this material once/if they reply.

## Reading order for next session

1. This file
2. `atlas-scope-locked-2026-09-06.md` — the locked scope, read in full
3. The blueprint artifact (link above) for the visual version
4. `harvest-engine-dedup-freeze-and-pause-2026-09-05.md` and `dashboard-deploy-incident-2026-09-06.md` for the technical/process state
5. `sensitive-species-coordinate-policy.md` for the coordinate rule
6. Check with the coding agent whether the last prompt (duplicate fix, species browser, document tiering) actually completed

## Standing rules, restated because they matter

- Nothing merges to `main`, nothing deploys anywhere (dashboard, harvest-engine, or the public site) without Vishnu's explicit go-ahead in that exact turn. No exceptions, no "this seems safe" judgment calls.
- Production requires coordinate coarsening for sensitive species; dev/staging does not.
- No new community fieldwork / cultural-archive work is in scope right now.

# Next Phase Plan — for the DevOps agent

_Handoff written 2026-09-07. Execute in order. Log every decision to `DECISIONS.md` in the repo, same convention as the overnight build. Do not wait for confirmation between steps — only pause for genuinely destructive/irreversible actions not already covered here._

**Out of scope for this run — these are on the user, leave them alone:**
- Rotating `DATA_GOV_IN_API_KEY`
- Any new API key registrations (IUCN, WDPA, etc.)
- The OSM-vs-WDPA geometry provider decision (stay on OSM, WDPA path stays dormant)
- India Code / Indian Kanoon build-or-not (do not build against either)
- Capacity call on multi-GB bulk datasets
- Re-testing India-WRIS/MoTA from an Indian egress point (VPS location doesn't resolve this)

## 1. Close the D-28 slug-migration gap

D-28's slug fix silently orphaned 65 `water_dataset` entities on re-run (already found and cleaned up — see `DECISIONS.md`). The underlying gap is structural: nothing in the core API treats a slug-function change as a migration.

- Add a `slug_version` (or equivalent) field to core entity records, or a documented migration step that must run whenever any engine's slug function changes.
- Write the migration procedure once, generically: (1) recompute new slugs for all entities of the affected type, (2) diff old vs new by a stable secondary key (not the slug itself — e.g. `ckan_name` or equivalent per-engine natural key), (3) re-key existing entities in place rather than inserting fresh ones, (4) dump to MinIO before any delete, (5) verify counts match pre-migration.
- Apply this checklist to all 8 engines' slug functions, not just Water Systems — audit each for the same underscore/normalization risk before it bites twice.

## 2. Fix PARIVESH's stale resource ID

The silent-failure bug (spider exiting 0 on startup error) is fixed, but it unmasked a second bug: the resource ID PARIVESH queries returns `{"status":"error","records":[]}` — it's stale.

- Find the current valid resource ID via data.gov.in's catalog/API browser for PARIVESH.
- Update the harvester config, re-run, confirm non-empty, non-error records land.
- If PARIVESH's dataset has been restructured or retired, log that finding in `DECISIONS.md` instead of forcing a fix.

## 3. Wrapper for manual harvest runs

Tonight's exit-255s were dropped SSH sessions during interactive `docker compose run`, not real failures — but the pattern is a footgun. The installed systemd timers already avoid it.

- Add a small wrapper script (e.g. `bin/run-harvest.sh <engine>`) that invokes any manual/ad-hoc harvest via `systemd-run --scope` (or the equivalent one-shot systemd unit pattern already used by the timers), so a manual run can't regress to a bare interactive `docker compose run` over SSH.
- Document it in the repo's README/runbook as the only supported way to trigger a manual run.

## 4. Continue Phase G engine build-out (within current key/credential limits)

Do not block entirely on the five pending user API keys — continue what's buildable with what's already registered:

- **Living Species:** let the GBIF harvest (already running, resumable) finish via its weekly timer. Once complete, move to POWO for plant names/images per the source atlas.
- **Forests & Land:** start against FSI ISFR via data.gov.in's existing key, plus ESA WorldCover (Planetary Computer STAC, no key needed) and Global Mangrove Watch (uncontested, no key). Skip anything gated on IUCN's token until the user's registration lands.
- For any engine step that turns out to need a not-yet-registered key, stop that step, log it plainly in `DECISIONS.md` as blocked-on-user, and move to the next buildable piece rather than stalling the whole run.

## 5. Verification pass (do this last, every run)

Before ending the session:
- Confirm zero heartbeat alerts, zero stale sources, zero empty-successful runs (same bar as tonight).
- Confirm all touched repos are committed.
- Append a short status block to `DECISIONS.md`: what got built, what got blocked-on-user, current entity/fact counts per engine.

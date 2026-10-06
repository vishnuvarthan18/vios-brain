# Senna Regrowth Verification — repo live on GitHub

**Repo:** https://github.com/vishnuvarthan18/senna-regrowth-verification (private)
**Pushed:** 14 August 2026, from Vishnu's personal Mac, one commit on `master`.

## What this is

A Google Earth Engine script (JavaScript, paste into code.earthengine.google.com)
that detects forest disturbance and measures Senna (invasive) regrowth in the
Sathyamangalam / Mudumalai / Sigur landscape using Google's AlphaEarth
Foundations satellite embeddings (64-band annual embeddings, no raw Sentinel-2
processing required for the first 3 steps).

This is a **separate project** from the sathyamangalam-atlas repo
(https://github.com/vishnuvarthan18/sathyamangalam-atlas) — different repo,
different purpose (satellite-based invasive species monitoring vs. the
wildlife/gazetteer atlas website). Don't conflate the two.

## Repo structure
```
gee/senna_regrowth_verification.js   ← the full script, paste-and-run
README.md                             ← usage, methodology, study area table
LICENSE                                ← MIT
.gitignore
```

## What the script does (in order)
1. Label-free disturbance detection — cosine angle between consecutive years
   in embedding space, zero training data.
2. Clearance-year attribution per pixel.
3. Regrowth measurement — normalized against a *measured* ceiling (0.973,
   not assumed 1.0) and floor, because cosine similarity compresses near 1.0
   and undisturbed pixels never actually hit 1.0 (phenology/rainfall/model
   noise).
4. Senna vs. native discrimination (commented out — needs ~10 reference
   points per class drawn by hand in the Code Editor before it can run).

## Key design decisions baked into the script (don't relitigate)
- Cohort restriction: only pixels disturbed by 2022 or earlier count toward
  the reported regrowth rate, since 2023/2024 disturbances haven't had time
  to regrow yet. Pooling all pixels understates the rate — this is explained
  at length in section 6 comments.
- Area totals computed at 200m scale (not 10m) to stay within Earth Engine's
  free-tier interactive compute; maps still render at 10m.
- `ee.Reducer` can go missing in some browser sessions (ad blocker / privacy
  extension blocking part of the EE client bundle) — script auto-detects and
  falls back to `ee.call()`, or degrades to maps-only if no reducer path
  exists at all.
- If `reduceRegion` fails with "capacity exceeded": use the two-step
  export-to-asset workflow in section 10 (`EXPORT_STATE = true`, run the
  export task, then set `STATE_ASSET` and re-run) rather than fighting the
  interactive path.

## Environment note (repeats the atlas-project finding)
This cloud sandbox cannot push to arbitrary GitHub repos (locked to a narrow
pre-configured Claude Code Action integration). Same as the atlas repo: built
and committed here, delivered as a zip via SendUserFile, pushed by Vishnu
from his own Mac with `gh repo create ... --source=. --remote=origin --push`.

## Open follow-ups
- `src/hybrid.py` (Sentinel-2 fallback for 2025+, since AlphaEarth embeddings
  only cover 2017–2024) — not yet written.
- `src/validation.py` (turn field GPS points into error-adjusted area
  estimates) — not yet written.
- Section 7 (Senna vs. native discrimination) needs ~10 hand-drawn reference
  points per class before it can be uncommented and run.
- No GEE Cloud Project has Earth Engine API enabled yet (checked earlier in
  session: `pragmatic-byway-426306-g2` and `wedding2day-a99ea` both showed
  "Load error: Google Earth Engine API has not been enabled"). The script
  itself doesn't require a Cloud Project to run in the Code Editor under the
  legacy `users/vishnu88varthan` namespace, but exports/asset creation may
  need one enabled — worth checking before relying on section 10's export
  workflow.

## Related
- `senna-regrowth/primer-satellite-invasive-mapping.md` — the plain-language
  background primer on how satellite invasive-species mapping works, written
  before this repo existed. Read that first if picking this project up cold.

# Stage 6 (Remote Sensing) — the senna-regrowth-verification coordination point, RESOLVED

**Written 22 Aug 2026.** The harvest-engine agent paused at Stage 6 because `harvest-plan.md` says to check the separate `senna-regrowth-verification` project before writing GEE export logic, and it could not find that project anywhere on Vishnu's machine. This doc is the answer, so no future session has to re-ask.

## The project exists. It is just not on that machine.

- **Repo:** `https://github.com/vishnuvarthan18/senna-regrowth-verification` — **private**, one commit on `master`, pushed 14 Aug 2026.
- **Why it isn't local:** it was built in a Claude cloud sandbox, delivered to Vishnu as a zip, and pushed from his personal Mac. The sandbox could not push directly. So a filesystem search on any other machine will always come up empty — that is expected, not evidence of absence.

## What is actually in it

One artifact that matters:

```
gee/senna_regrowth_verification.js   ← paste-into-Code-Editor GEE JavaScript
README.md · LICENSE · .gitignore
```

There is **no Python, no service-account automation, no export pipeline, no R2 integration, no scheduler**. It is an interactive browser script a human runs by hand in code.earthengine.google.com.

## Therefore: zero code reuse. Build Stage 6's GEE export logic fresh.

Nothing in that repo can be imported or called by a Cloudflare Worker. The duplication risk `harvest-plan.md` was warning about is **methodological, not architectural** — and it is confined to one of Stage 6's nine layers.

## What Stage 6 MUST carry over (do not re-derive these)

These are hard-won decisions. Getting any of them wrong silently biases every regrowth number the Atlas publishes.

1. **Disturbance detection uses AlphaEarth Foundations 64-band annual embeddings, not raw Sentinel-2.** Disturbance = cosine angle between consecutive years in embedding space. Label-free, zero training data.
2. **Regrowth is normalised against a *measured* ceiling of `0.973` and a measured floor — never an assumed `1.0`.** Cosine similarity compresses near 1.0 and undisturbed pixels never actually reach it (phenology, rainfall, model noise). Assuming 1.0 biases everything.
3. **Cohort restriction: only pixels disturbed in 2022 or earlier count toward a reported regrowth rate.** 2023/24 disturbances have not had time to regrow. Pooling all pixels understates the rate.
4. **Area totals computed at 200 m scale, maps rendered at 10 m** — this is what keeps it inside Earth Engine's free interactive-compute tier.
5. **AlphaEarth embeddings only cover 2017–2024.** Anything from 2025 onward needs a Sentinel-2 fallback that has **not been written** (`src/hybrid.py`, still an open follow-up).

## What is NOT solved there — do not inherit a false claim

**Senna vs. native discrimination is commented out and non-functional.** It needs ~10 hand-drawn reference points per class in the Code Editor before it can run. Stage 6 must not ship a "Senna flowering phenology" layer as if the species discrimination problem were solved. It is not.

## Layers with no overlap at all — build freely

FIRMS active fire and burnt area · CHIRPS and ERA5-Land rainfall/temperature · Hansen Global Forest Change · SRTM / Copernicus DEM · Landsat annual composites back to 1984. None of these touch the senna work.

## ⚠️ Check before building anything: is the Earth Engine API actually enabled?

The senna repo's notes record two GCP projects (`pragmatic-byway-426306-g2`, `wedding2day-a99ea`) both failing with **"Google Earth Engine API has not been enabled"**. The harvest engine uses a different, newer project — `sathyamangalam-record` — whose service-account JSON is already a Cloudflare secret (`GEE_SERVICE_ACCOUNT_JSON`).

A service-account key existing does **not** mean the Earth Engine API is enabled on that project. Verify enablement on `sathyamangalam-record` **first**. If it is not enabled, every Stage 6 export fails at the last step after the whole layer has been built.

## Verdict handed to the agent

Proceed with Stage 6. Build fresh, carry the five decisions above, skip the Senna-species layer, verify EE API enablement before writing export code, and note in `PROGRESS.md` that the coordination was resolved from project docs rather than from a local checkout.

## Related

- `senna-regrowth/verification-repo-2026-08-14.md` — the source for everything above
- `senna-regrowth/primer-satellite-invasive-mapping.md` — background on the method
- `sathyamangalam/harvest-engine-session-handoff-2026-08-21.md` — the stage ledger

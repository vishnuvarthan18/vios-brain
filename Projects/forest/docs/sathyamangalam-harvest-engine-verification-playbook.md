# Harvest Engine — Verification Playbook

Distilled from the 23 Aug 2026 session. This is the transferable part: the method, the trap catalogue, and the prompt skeleton that generated the DEM, ERA5-Land and Landsat builds. Use it to write the next prompt rather than re-deriving the approach.

---

## 1. The core principle: a prompt is not an enforcement mechanism

Instructions like *"do not close this task by asserting everything passed"* shape tone, not truth. An agent inclined to please can satisfy the letter of that and still hand back a clean report. Exhortation does not constrain output.

**What actually makes a verification real is that its claims resolve to artifacts the human can re-run in one command, independent of anything the agent narrates.**

Strong items — one right answer that exists whether or not the agent finds it:
- Is `scale` present in that `reduceRegion` call? Binary, sitting in a file.
- Does `job_run.rows_written` match the table's actual row count? Numbers that must reconcile.
- Does `git show --stat <sha>` produce the same output for both of you?

Weak items — real latitude for a plausible-sounding narrative:
- "Do these two stages use consistent taxonomy?"
- "Do the loss pixels coincide with fire detections?"

Both kinds belong in a prompt. Only the first kind is self-enforcing.

**The failure mode no wording closes is fabricated output.** An agent can format invented text as a terminal paste. The only counter is spot-checking: pick two or three strong items and run them yourself *before* reading the report. That calibrates how much weight the rest can carry.

---

## 2. Correctness is not coverage — check both, separately

This session's largest finding. Seven stages were marked "done and independently verified." That verification was real — code runs, writes accurately, dedupes, doesn't fabricate. It was also answering only half the question.

- **Correctness**: did what ran, run right?
- **Coverage**: did everything that should have run, run at all?

Asking only the first produced a project reporting 7 of 8 stages complete while roughly half its named sources had never delivered a row, one stage had a single row, and one had four of twenty-nine appendices.

**A pipeline can be flawless and still be empty.** Every completeness check must name a *source of truth* for what should exist — a registry file, a spec section, a project doc — and quote it before comparing against the database. Never assess completeness against the agent's own idea of what a stage was for.

---

## 3. Trap catalogue

### 3.1 Unstated unit assumptions — hit three times

The single most recurrent failure class in this project.

| Instance | Wrong | Right |
|---|---|---|
| Hansen forest loss | `pixel_count × 900 m²` | `ee.Image.pixelArea()` |
| ERA5-Land temperature | raw Kelvin stored as "temperature" | explicit K → °C conversion |
| Landsat C2 L2 reflectance | raw scaled integers | multiplier + offset before NDVI |

**Rule: probe the raw values and report them before computing anything.** A number with an assumed unit baked in looks exactly like a correct number.

### 3.2 Latency is not a bug

CHIRPS returned 8 of 31 requested days. A full session went into diagnosing a query bug that did not exist — it was ~3-week publication lag.

**Signature: a clean cutoff is latency; a scattered gap is a bug.** Binary-search the date boundary *first*, report the observed lag in days, then build the query window to sit entirely inside available data rather than running to today and reporting a hole.

Measured lags so far: CHIRPS 22 days · ERA5-Land DAILY_AGGR 8 days · ERA5-Land HOURLY 6–7 days · FIRMS NRT 10-day rolling cap · eBird `obs/geo/recent` 30-day hard lookback.

**Store latency as a queryable field (`data_latency_days`), not prose.** That is what lets a scheduled run distinguish expected lag from failure without a human re-deriving it.

### 3.3 Scale semantics — means and sums behave differently

`scale: 30` on an EPSG:4326 image makes Earth Engine derive its grid by converting metres to degrees **at the equator**: 30 ÷ 111,320 ≈ 0.00026949°. At 11.5°N those pixels measure ~29.81 × 29.40 m ≈ **876 m²** — neither 900 m² nor Hansen's native 1-arc-second ~934 m².

- **Area sums** are corrupted by rescaling. Always `pixelArea()`.
- **Means** (NDVI, elevation, temperature) are scale-robust. Coarsening them is legitimate and much faster.

Do not generalise either direction. The forest-loss error and the Landsat speed lever are the *same fact* pointing opposite ways.

### 3.4 A fractional "pixel count" is a signal, not noise

`3350.270588…` is not an integer, so it is not a raw count. Two explanations: benign partial-pixel weighting at the geometry boundary (`reduceRegion` weights edge pixels by fraction inside), or an area-weighted sum at a coarser scale. **The discriminator is the explicit `scale` argument and whether `bestEffort` is set.** A small fractional remainder points to the benign case; check the code regardless.

### 3.5 Geographic scope — the bbox problem

`reserve.boundary_geojson` is NULL. Every spatial figure is computed over a ~2,585 km² bounding box.

The DEM layer proved how bad this is: 187–2,098 m of relief means the box spans reservoir plain to montane plateau — at least three ecological zones. So NDVI blends vegetation types with different ceilings, forest loss counts drivers that differ by zone, and temperature averages 12–13 °C of lapse-rate spread across only ~21 ERA5-Land cells.

**Always store `geometry_scope` explicitly. Never publish a bbox figure as a reserve statistic.**

### 3.6 Cross-sensor discontinuities look like ecology

Landsat 5/7 (TM/ETM+) and 8/9 (OLI) use different band numbers for red and NIR. A single mapping across the archive produces a 2013 step change that reads as a real event. Landsat 7 post-2003 SLC-off adds permanent gaps.

**A discontinuity at a sensor-transition year is an artifact until proven otherwise.**

### 3.7 Cloud masking silently quadruples errors

Unmasked NDVI read 0.109; masked read 0.407. The wrong number was plausible-looking. **Always show a masked-vs-unmasked comparison so the mask is demonstrably doing something.**

---

## 4. Techniques that worked

**Sanity bands as checks, not targets.** Give an approximate expected range and state explicitly that it is *to check against, not to match*, with instructions to investigate rather than adjust if the result falls outside. The DEM elevation came back 187–2,098 against a stated 250–1,800 band; the agent investigated and explained the difference instead of quietly tuning the query. The band being slightly wrong did not matter — the instruction to investigate is what carried the value.

**Cross-layer validation.** Use one layer to predict what another should show, then check. ERA5-Land measured 23.92 °C; the 1915 Coimbatore gazetteer gives 25.33 °C annual mean; Coimbatore station sits ~376 m below the bbox mean elevation from the SRTM layer; at ~6.5 °C/km that predicts ~22.9 °C. Observed lands within ~1 °C. Two live layers and a 111-year-old measurement agreeing is far stronger evidence than any single layer's plausibility.

**Cross-stage corroboration.** Stage 5's historical passages independently checked Stage 6's remote sensing twice (Nicholson 1887 for elevation, the 1915 gazetteer for temperature). Weight it correctly — a bbox mean and an 1887 "general elevation" are not the same quantity over the same area — but this is precisely the value the Atlas exists to produce.

**Metadata as queryable fields, not prose.** Established pattern, now on every `observation_layer` row: `dataset_version`, `geometry_scope`, `reduction_scale`, `units`, `data_latency_days`. Anything discovered the hard way should end up in a column, not a paragraph.

**Requiring self-correction to be reported.** Agents in this project retracted their own bbox-dilution theory, caught a subagent's factual error about `IUCN_API_KEY`, and disclosed a misread of a query dump. Those disclosures are the strongest available credibility signal — a report with no self-corrections in a complex task deserves more suspicion, not less.

---

## 5. Prompt skeleton

```
BUILD / VERIFY: <scope>

Repo: <path>
<where this sits in the sequence; which pattern to follow>

STANDING RULES
Report raw output, not summaries. Every claim followed by the command or
query and its unedited result. Write UNVERIFIED where you could not check
something, and say what blocked you. Report findings before fixes. Do not
mark done because the code ran without error.

SOURCE OF TRUTH
Name and quote the spec/registry/doc defining what should exist, before
comparing against the database. Do not assess against your own idea of
what this is for.
[If a project doc is the source: PASTE ITS CONTENT — the agent cannot
read Claude-project docs from disk. This mistake cost a round-trip.]

TRAPS
<the specific known failure modes for this task, with the project's own
history of hitting them — see section 3>

WHAT TO BUILD
<deliverable, structure, per-row fields>

REQUIRED METADATA
dataset_version, geometry_scope, reduction_scale, units,
data_latency_days — plus any new field this task introduces, backfilled
onto existing rows where it applies.

SANITY CHECKS
<approximate expected band, stated as A CHECK NOT A TARGET>
If outside: investigate and report the cause. Do NOT adjust the query
until it produces expected-looking numbers and then report success.
Cross-check against an independent source already in the database.

VERIFY BEFORE REPORTING DONE
1. Run it. Paste the job_run row.
2. Re-run. Confirm idempotency. Paste that row.
3. SELECT the new row(s) in full, all metadata fields populated.
4. SELECT kind, COUNT(*) ... GROUP BY kind — so coverage change is visible.

DO NOT
<explicit scope fence — adjacent work not to touch>

OUTPUT
Commands, raw output, your reading. End with <the specific verdict this
task must produce>.
```

---

## 6. Errors made *by the reviewing side* this session

Recorded for calibration — the review layer is not exempt from the standard it enforces.

1. **Inferred a polygon from a row count.** Claimed `reserve.boundary_geojson` existed because `reserve` had one row. It is NULL. This is exactly the inference-from-summary the project's standing rule forbids. The agent correctly refused the task rather than approximating around it.
2. **Treated a coordination doc as a requirements spec.** Asserted Stage 6 was incomplete for missing AlphaEarth, from a doc describing what was *safe to build*, not what was *required*. The master prompt is the spec. Retracted.
3. **Estimated 934 m²/pixel from native resolution** when the explicit `scale: 30` forced a different grid entirely (876 m²) — right that the flat ×900 was wrong, wrong about the direction and magnitude.
4. **Told an agent to find a Claude-project doc on its filesystem.** It cannot. Cost a round-trip; the agent handled it correctly by stopping to ask.
5. **Gave a sanity band slightly too narrow** (250–1,800 m vs actual 187–2,098). Harmless because the band was framed as a check rather than a target — which is the argument for framing them that way.

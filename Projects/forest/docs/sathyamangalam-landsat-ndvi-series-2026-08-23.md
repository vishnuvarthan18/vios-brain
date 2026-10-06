# Landsat Annual NDVI Series, 1988–2026 — result and analysis

Built 23 Aug 2026, `job_run` 88–91. 37 rows under kind `vegetation-ndvi-landsat-annual`. This is the Atlas's first genuine analytical asset — every other layer is a snapshot; this is a four-decade record.

## The series

Median composite, **Jan 1 – Mar 31 dry-season window**, chosen after archive-density probing showed an Apr–Jun window collapses to one scene per year in older data.

| Era | Years | Mean NDVI |
|---|---|---|
| TM (L5) | 1988–2011 | 0.464 |
| ETM+ (L7) | 2000–2013 | 0.480 |
| OLI / OLI-2 (L8/L9) | 2014–2026 | 0.554 |

Range across all years: 0.342 (1992) to 0.618 (2023). Missing, confirmed as real archive gaps for this bbox in the Jan–Mar window: 1984, 1985, 1986, 1987, 1989, 1996.

Build quality was high. All four traps confirmed empirically rather than trusted: C2 L2 scaling verified against raw pixel ranges (91–54,604 pre-scaling); QA_PIXEL bit semantics checked per sensor family (bit 2 cirrus is real on OLI, "Unused" on TM/ETM+); band mapping harmonized per sensor; SLC-off inclusion justified by a per-pixel valid-scene count on 2005 (≥2 of 8 scenes everywhere). A self-caught off-by-one in year unlocking was fixed and re-verified.

## 1. The Hansen "disagreement" is not a disagreement — the test had no power

Reported: 2021–23 NDVI (0.588) is *higher* than 2018–20 (0.547), despite Hansen placing 36.9% of all-time forest loss in exactly that window.

That reads as a contradiction. It isn't. **The forest loss is ~2.94 km² out of a ~2,585 km² bbox — 0.11% of the area.** If those pixels dropped from NDVI 0.6 to 0.2, the effect on a bbox mean would be about **0.0004 NDVI units**. Year-to-year variation in this series is ±0.05 — roughly **100× larger than the signal**.

The cross-check as specified could never have detected the loss regardless of whether it happened. This was a test-design error on the review side, not a finding.

**The test with actual power:** extract NDVI *only on the Hansen loss pixels*, comparing each pixel's value before versus after its own recorded loss year. A masked, pixel-level before/after against a same-period control sample of non-loss forest pixels. That is a real test; the bbox-mean comparison is not.

## 2. The sensor shift can be measured, not merely flagged

TM 0.464 → ETM+ 0.480 → OLI 0.554 is a +0.09 step across sensor families. Correctly stored in a `sensor` field and not smoothed over — but currently unquantified, which means **real greening and sensor drift are inseparable in this series**. Any trend claim is unsafe.

Published OLI-vs-TM NDVI offsets are typically smaller than 0.09, so part of this is plausibly real. Part is not. The series cannot say which.

**This is measurable with data already in reach**, because the sensors overlap:

- **L5 TM and L7 ETM+ both operated 1999–2011**
- **L7 ETM+ and L8 OLI both operated 2013–2021**
- **L8 OLI and L9 OLI-2 both operate 2022–present**

Compute the same-year, same-window composite from *both* sensors across each overlap, take the mean difference, and chain the three offsets. That converts an unquantified artifact into a measured correction with a stated uncertainty.

This is the same principle as the senna project's hard-won rule about the 0.973 regrowth ceiling: **measure the reference value, never assume it.** Same lesson, different layer.

## 3. The Atlas now holds two different NDVI numbers for the same reserve

- Sentinel-2: **0.407**, late-July-to-August window
- Landsat 2024/2026: **0.576 / 0.563**, Jan–Mar window

A 38% gap on the same landscape. Most of it is probably seasonal and *directionally sensible for this specific place*: the NE monsoon runs Oct–Dec, so Jan–Mar follows the wet season, while July–August sits in a dry spell on STR's rain-shadow side — which is exactly what the CHIRPS work independently showed. Sensor, cloud-mask and scale differences contribute too.

Probable is not the same as established. **Two NDVI figures differing by 38% on one site will be noticed.** Either reconcile them explicitly with a stated cause, or publish only one with its season attached.

## 4. 🔴 `status: "failed"` on expected gaps — fix before Cron

`job_run` 91: `status: "failed"`, `rows_written: 0`, `rows_skipped_duplicate: 42`, six errors all reading *"real archive gap for this bbox, not an error."*

The status rule treats "zero rows written plus any errors present" as failure. But those six archive gaps are permanent — they will re-report identically on every future run.

**Once Cron is on, this stream reports `failed` forever.** An alert that always fires is an alert that gets ignored, and it will mask a real failure when one occurs.

Fix: distinguish `error` from `expected_gap` in `errors_json`, and exclude expected gaps from the status determination. This is small, and it is operationally urgent because automation is the next step.

## 5. The highest-value analysis now available

Dry-season NDVI in a dry-deciduous forest should track antecedent rainfall strongly. The series supports this on inspection: the lowest post-1993 values are **2003 (0.371) and 2004 (0.362)**, coinciding with a severe Tamil Nadu drought; 1990–92 (0.342–0.356) is the other trough.

**CHIRPS data begins in 1981** — the full period of this NDVI series is available. Building an annual antecedent-rainfall series and correlating it against dry-season NDVI, 1988–2026, would be a real analytical contribution rather than another collected dataset. It is also the natural way to separate climate-driven variation from land-cover change, which is the question the Atlas exists to answer.

That analysis needs the sensor offset from §2 measured first, or rainfall correlation and sensor drift will be confounded.

## Recommended order

1. Fix the `status: failed` rule — small, blocks clean automation.
2. Measure the three sensor offsets across the overlap years.
3. Re-run the Hansen cross-check as a pixel-level before/after on loss pixels.
4. Reconcile or separate the two NDVI figures.
5. Build the CHIRPS 1981–2026 annual series and correlate.
6. Optional: test whether a wider seasonal window recovers 1984–87 — those are the most valuable years in any long series, being the earliest baseline.

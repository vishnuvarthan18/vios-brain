# seals - T4 auto-tune sheet notes

Sheet: `design/realism/sheets/seals.png` (gitignored, DECISIONS.md 2026-09-26 08:35:29 - it carries crops of
reference photos of every tier). Top row: 6 plain 128 px patches of real photos (SEA-050, SEA-010, SEA-034,
SEA-038, SEA-027, SEA-026). Bottom row: 6 plain 128 px patches of `eval/out/tuned/seals.png`. Both sides went
through `lib/measure.py` with the spec's `measure_norm` (long side to 1024 px), so one sheet pixel is the same
object distance on both rows.

## Numbers: before (site 2D baseline) vs after (T4 tuned stage)

| row | 2D baseline (styleguide.html) | T4 tuned stage | target | real-photo pass rate |
|---|---|---|---|---|
| colour_dE2000_median | 24.11 FAIL | **5.75 PASS** | < 6.0 | 25% |
| colour_dE2000_p95 | 32.09 FAIL | **7.36 PASS** | < 12.0 | 31% |
| colour_mean_in_photo_range | [40.3, 15.9, 23.7] FAIL | **[68.1, 5.3, 25.9] PASS** | inside [57.7,-1.2,9.6]..[82.0,13.8,48.8] | 50% |
| spectral_slope_diff | (baseline row) FAIL | **0.108 PASS** | < 0.3 | 13% |
| anisotropy_diff | 0.205 FAIL | **0.027 PASS** | < 0.15 | 67% |
| local_contrast_ratio | 0.05 FAIL | **1.041 PASS** | 0.8..1.25 | 53% |
| grain_direction_diff_deg | 35.31 FAIL | **3.73 PASS** | < 15.0 | - |
| aspect_ratio | 1.008 FAIL | 1.005 **FAIL** | 1.009..2.465 | - |
| **rows passed** | **0 of 8** | **7 of 8** | | |
| lighting (shading_changes_with_light_angle) | no light input FAIL | **26.38 dL\* PASS** | >= 4 dL\* and edge excess >= 2 dL\* | - |

Tuner loss 25.310 (trial 0, spec card alone) -> 6.012 (run 1) -> 5.622 (run 2) -> **4.442** (run 3),
200 trials each, ~6.5 min per run. Trials: `eval/trials/seals.run1.csv`, `seals.run2.csv`, `seals.run3.csv`
and `seals.csv` (= run 3).

### The one row that still fails

`aspect_ratio` 1.005 against the spec range 1.009..2.465. `rig/stage.js` line 296 forces aspect = 1 for shape
type `disc`, so a circular seal measures ~1.000 whatever the material knobs do.
`eval/t4_seals_aspect_probe.mjs` probed 6 outline-roughness values plus the `sherd` outline: every value that
reaches the 1.009 lower bound costs `grain_direction_diff_deg` instead (39-85 deg against a 15 deg limit),
because the seals spec's own `geometry.edge_roughness` is 0.1485 - 9x the coin's - so the wobble already
dominates this material's structure tensor. The row count stays 6-of-8 or worse at every probed value, so the
spec-derived 0.01188 ships and the row is reported as a FAIL, not tuned away (DECISIONS.md 2026-09-26, T4-seals).
The spec row is itself the noisiest on the card (n=30 photos including bundles, stacks and partial views;
circularity range 0.052..0.742), and a seal matrix photographed face-on IS a circle.

### Honesty notes on the rows that pass

Three of the passing rows are targets real photos rarely meet: only 13% of real seal photos pass
`spectral_slope_diff`, 25% pass `colour_dE2000_median` (real median 8.26 - the tuned render at 5.75 is closer
to the spec than the median real photo of a seal is) and 31% pass `colour_dE2000_p95` (real median 13.25).
Reported, not changed: the owner's targets stand.

The whole spec card is **low confidence**: only 16 usable photos against a target of 20
(`reference_set.enough_photos: false`), and both physical numbers (diameter 30 mm, density 8.7 g/cm3) are
`estimate: true` with no source. The lighting candidate's relief is
`letter_depth 0.018519 x height.bump 0.9 = 0.016667` object units = **0.5 mm on a 30 mm seal**, which is the
T5c estimate (DECISIONS.md 2026-09-26 11:10:22) - an UNVERIFIED number, and the sample Tamil text on it has
not been checked by a Tamil speaker.

## The 5 biggest visible differences (my own eyes on the sheet - judgement, not proof)

1. **The render is woven, the real seals are not.** The bottom row carries a regular diagonal streak pattern
   (the shader's fibre layer: `grain.fibre 0.255`, `fibre_density 60`, `grain.stretch 0.57`) that reads as
   coarse cloth or wood grain. No real crop has it: SEA-010 is granular, SEA-027 and SEA-034 are smooth. The
   measured `anisotropy_diff` is 0.027 - a pass - because the spec's own anisotropy is high (0.3323, measured
   across a photo set that includes fabric-backed and grooved seals), so the number cannot see that the
   render's directionality is the *wrong kind*.
2. **One hue for six objects.** All six render crops are the same warm tan. The real six span pale yellow
   (SEA-050), olive-brown (SEA-010), pink terracotta (SEA-034), grey-green (SEA-038), peach (SEA-027) and
   near-white (SEA-026) - because the set mixes bronze matrices with clay sealings. The render is the
   consensus centre of that mixture and looks like none of its members.
3. **No pitting or porosity.** SEA-010 and SEA-038 show sharp dark pits and mineral grit a few pixels across.
   The render's detail is all low-frequency mottling with no isolated dark points; `speckle.density 0.21` at
   `speckle.size` 210 makes broad blotches, not pits.
4. **No blemishes, no drift.** Each render crop is the same statistic everywhere: no corrosion patch, no
   earth stain, no lighter rubbed area. Real crops carry at least one of those (the green streak in SEA-038,
   the dark rim in SEA-010). Same limitation as palm_leaf: this needs a new shader layer, not a setting.
5. **Flat, never metal.** The tuner settled at `surface.metal 0.041` with `roughness 0.825` - effectively a
   matte dielectric. That is right for a clay sealing and wrong for the bronze matrices in the same set: no
   render crop shows the soft broad highlight that SEA-050 has. The colour rows were what drove the search,
   and a matte surface is the cheapest way to sit in the middle of a mixed-material photo set.

Tuning ran with `wear.age = 0` and letters OFF, so the sheet shows the unworn material only.

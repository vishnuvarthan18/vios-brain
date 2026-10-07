# copper_plate - T4 auto-tune, side-by-side sheet

Sheet: `design/realism/sheets/copper_plate.png` (gitignored - it carries crops of reference photos of
every tier, including CC BY-SA and CC BY-NC; never shipped). Rebuild it with

```
design/realism/.venv/bin/python design/realism/eval/sheet.py copper_plate \
  design/realism/eval/out/tuned/copper_plate.png \
  --photos COP-025,COP-118,COP-119,COP-027,COP-041,COP-106,COP-031,COP-024,COP-026,COP-120,COP-067,COP-059,COP-058,COP-073,COP-046,COP-100,COP-056,COP-057,COP-040,COP-042,COP-071,COP-055 --n 6
```

Top row: 128 px plain patches from six real photos (COP-027, COP-041, COP-106, COP-031, COP-059,
COP-058 - the first six of the spec's 22 `photos_used_ids` that yield a plain patch at this patch
size). Bottom row: six 128 px plain patches of the tuned render. **Same scale on both rows**: both
sides go through `lib/measure.py normalise()` with the spec's `measure_norm` (long side = 1024 px),
the same code `eval/visual.py` scores with.

## The 5 biggest visible differences (my own eyes, not a score)

1. **One copper, six coppers.** The six render crops are interchangeable: the same orange-brown
   mottle at the same scale in all six. The six real crops are six different surfaces - olive-green
   patina (COP-027), grey-brown engraved plate (COP-041), a near-black plate with a bright green edge
   line (COP-106), a flat pale grey-green face with almost no texture (COP-031), orange-brown
   corrosion with pale blue-white specks (COP-059), and dark blue-black with a pale wandering vein
   (COP-058). The tuned render matches the *average* plate; it has none of the *spread* between
   plates, which is most of what the eye sees on this sheet.
2. **No green.** Four of the six real crops are green, grey-green or blue-black - the copper
   carbonate patinas that a buried charter actually carries. The render's L\*a\*b\* mean is
   (34.9, 6.6, 9.8): a\* and b\* are both positive, i.e. warm red-brown everywhere. The spec's p5
   (19.65, -5.6, -7.2) does carry the cool end, but a single fbm ramp between p5 and p95 makes the
   cool colour appear only in the darkest pixels, never as a patch of green over a red-brown plate.
3. **No letters, no plate edge, no rivet holes.** COP-041 is half full of engraved Tamil-Grantha
   letters; COP-106 shows the plate's edge against black. The scored render is a plain tile with
   `letters.mode = none` on purpose (the score measures the *material*), so nothing in it is longer
   than the noise: no line, no edge, no hole, no engraved groove. The carved version exists only in
   `materials/copper_plate.lighting.params.json` and is scored by the lighting test, not here.
4. **Corrosion is uniform, not patchy.** COP-059's speckles sit in clusters and COP-058's vein runs
   right across its crop; real corrosion varies slowly over centimetres and then suddenly over
   millimetres. The render's fbm mottle varies at one characteristic size everywhere
   (`noise.scale` 25, `lacunarity` 2.1) and `speckle.density` is still 0, so there are no discrete
   corrosion grains at all. `local_contrast_ratio` passes (1.026) because it measures the *amount* of
   variation, not its shape or its clustering.
5. **Too clean, and too flat-lit.** No dirt runs, no fingerprints, no highlight from a museum
   spotlight, no scratches. COP-031 is the extreme case in the other direction: a real plate so
   smooth and evenly lit that it reads as pale grey card - the render can be neither that plain nor
   as dirty as COP-059. (T7's wear layer is the existing tool for the dirt and was NOT used here:
   `wear.age` is 0, wear is T7's parameter.)

Differences 1, 2 and 4 are one fault said three ways: **the score asks the render to sit near the
middle of the photo set, and a real charter plate is one draw from that set, with its own patina, its
own burial history and its own letters.**

## Before -> after (all scored by `eval/visual.py`, loss by `eval/tune.mjs lossOf`)

| render | visual rows passing | loss | colour dE2000 med / p95 | colour mean in range | slope diff | anisotropy diff | local contrast ratio | aspect |
|---|---|---|---|---|---|---|---|---|
| site 2D baseline (styleguide.html) | 2 of 7 | - | **23.40 FAIL** / **26.21 FAIL** | **no FAIL** | **0.657 FAIL** | 0.099 | **0.485 FAIL** | 1.549 |
| T6b copper material (page render, hand probe) | 7 of 7 | - | 2.50 / 6.19 | yes | 0.112 | 0.088 | 1.022 | 1.501 |
| T6b probe params through the tuner = trial 0 | 7 of 7 | 2.735 | 2.51 / 6.19 | yes | 0.111 | 0.097 | 1.029 | 1.5 |
| **T4 tuned (shipped), run 1 trial 183** | **7 of 7** | **1.759** | **2.63 / 5.76** | **yes** | **0.001** | **0.010** | **1.026** | **1.5** |

Two chained runs of 200 trials (7.0 min + 7.0 min), every trial in
`design/realism/eval/trials/copper_plate.run1.csv` and `copper_plate.csv`. Both ran with the plate's
grain axis limited to the spec card's own (`--limit=grain.angle_deg=-2:2`, spec 0.4087 deg, range
0.206..0.487); `surface.metal` was **searched, not pinned** - copper is a metal, unlike the palm leaf
and the granite (DECISIONS.md, this task). Run 2 started from run 1's best with a different seed
(20260927) and **found nothing better in 200 trials**: it ended on the identical params hash
`f3c338829f`, loss 1.759. In run 1, as with the stone, the 120 random trials beat nothing; every gain
came from coordinate refinement.

What the tuner changed from the T6b probe: `noise.lacunarity` 2.0 -> 2.1, `grain.stretch` 1.23 ->
1.41, `grain.angle_deg` 0.4087 -> 0, `grain.fibre` 0 -> 0.02, `grain.fibre_density` 60 -> 65,
`grain.fibre_height` 0 -> 0.00015, `height.bump` 1.0 -> 1.2, `surface.roughness` 0.45 -> 0.5,
`surface.spec` 0.35 -> 0.41, and a small colour-ramp widening (p95 b\* 29.9 -> 32.3).
`surface.metal` stayed at **0.2** and `noise.scale` at **25** - the search tried both and kept them.

## Lighting (carved letters)

`materials/copper_plate.lighting.params.json` = the tuned material + the T6b carved letters.
`height.letter_depth` is lowered 0.003 -> **0.0025**, exactly 1/bump, because the stage builds
normals from (height field) x `height.bump` and the tuner raised bump to 1.2 with letters OFF; the
rendered cut is therefore the identical one T6b scored. Scored:
**7.388 dL\*** against the `>= 4 dL* and edge excess >= 2 dL*` target - **PASS**, and better than the
T6b raking page (6.346). The groove itself (about 1.2 mm on a 400 mm plate) is a **T6b estimate,
unverified**: no source measures the groove depth of a Chola charter (DECISIONS.md 2026-09-26
11:56:28).

## What the numbers do not say

- **Real-photo pass rates** (`eval/self_test/real_photo_pass_rates.json`, copper_plate, 22 photos):
  `colour_dE2000_median` **23%** (real median 7.66 against a `< 6.0` target),
  `spectral_slope_diff` **23%**, `colour_dE2000_p95` **46%** (real median 13.42),
  `local_contrast_ratio` **46%**, `anisotropy_diff` 54%, `colour_mean_in_photo_range` 68%. The tuned
  render misses **no** target, which means it passes four rows that fewer than half of the real
  copper photos pass. That is a reason to believe the sheet over the scoreboard: a render that beats
  most real photographs of the thing is more *average* than any one plate, not more real.
- **Tuned, not robust, numbers.** `spectral_slope_diff` 0.001 and `anisotropy_diff` 0.010 are
  refinement-step values; a small change to `noise.lacunarity`, `grain.stretch` or `noise.scale`
  moves them by more than their own size. The 7/7 was already held by the T6b probe, so the gain here
  is margin (loss 2.735 -> 1.759), not a row flipped from fail to pass.
- The whole sheet and score measure the material with **letters off** and **wear age 0**.
- Nothing here has been checked by a Tamil speaker; the sample text in the lighting params stays
  flagged unverified.

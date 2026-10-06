# palm_leaf - T4 auto-tune, side-by-side sheet

Sheet: `design/realism/sheets/palm_leaf.png` (gitignored - it carries crops of reference photos of
every tier, including CC BY-SA and CC BY-NC; never shipped). Rebuild it with

```
design/realism/.venv/bin/python design/realism/eval/sheet.py palm_leaf \
  design/realism/eval/out/tuned/palm_leaf.png \
  --photos PAL-166,PAL-200,PAL-180,PAL-197,PAL-184,PAL-047,PAL-033,PAL-032,PAL-073,PAL-077 --n 6
```

Top row: 128 px plain patches from six real photos (PAL-166, PAL-200, PAL-197, PAL-184, PAL-047,
PAL-077 - the nearest photos the colour score used, minus the two whose mask gave no plain patch).
Bottom row: six 128 px plain patches of the tuned render. **Same scale on both rows**: both sides go
through `lib/measure.py normalise()` with the spec's `measure_norm` (`short` side = 256 px), the same
code `eval/visual.py` scores with.

## The 5 biggest visible differences (my own eyes, not a score)

1. **One frequency everywhere.** All six render crops look like the same piece of cloth: a fine,
   even stipple at a single scale. The six real crops do not resemble each other - coarse open
   fibre (PAL-047), near-smooth polished leaf (PAL-200), blotchy stained leaf (PAL-166). The render
   has the right *average* texture (spectral slope diff 0.004, anisotropy diff 0.001) and none of the
   spread between leaves.
2. **No long structure.** The real leaves carry vein lines and light/dark bands that run the whole
   length of the crop, and tonal drift across it. The tuned noise has a correlation length of 2 px,
   so nothing in the render is longer than a few pixels: no continuous fibre, no banding, no drift.
3. **No blemishes at all.** The real crops have worm holes (PAL-197), dark specks, stains, a rib
   line, ink strokes bleeding in from the writing (PAL-184). The render is perfectly clean. The
   speckle term the tuner turned on (density 0.29, strength 0.14) reads as fine noise, not as
   discrete marks.
4. **The crosshatch.** Close up, the render's grain is a regular woven screen - the fibre term
   (fibre 0.13, density 105) beating against the fbm - so it repeats mechanically. Real fibre
   spacing and contrast wander.
5. **One colour, no drift.** The render sits at a single point (L\*a\*b\* 68.9, 4.7, 24.1) with very
   little variation inside a crop; the real set runs from pale grey-tan through pink-cream to mid
   brown, and each real crop drifts across itself. The colour rows pass because the score asks for
   the distance to the *nearest* photos, not for the spread.

Items 1, 2, 3 and 5 are all the same fault said four ways: **the stage renders one stationary
statistic, and a real leaf is a non-stationary surface with history.** No setting of the current
shader fixes that; it needs another layer (long fibre lines, low-frequency banding, discrete marks).
T7's wear layer is the nearest existing tool and was NOT used here (age 0), because wear is a
separate task's parameter.

## Before -> after (all scored by `eval/visual.py`, loss by `eval/tune.mjs lossOf`)

| render | visual rows passing | loss | colour dE2000 med / p95 | slope diff | anisotropy diff | local contrast ratio | grain dir |
|---|---|---|---|---|---|---|---|
| site 2D baseline (`.leaf-strip__leaf`, has text) | 6 of 7 | 62.74 | 3.73 / 7.16 | - (texture not measurable) | - | - | - |
| T3b 2D leaf material, no text | 10 of 10 | 4.37 | 3.77 / 6.84 | 0.226 | 0.001 | 0.825 | 1.28 deg |
| T3a 3D leaf page (stage shader, hand probe) | 8 of 10 | 8.21 | 2.95 / 8.54 | 0.142 | **0.189 FAIL** | 0.909 | **17.46 FAIL** |
| T4 tuner start (spec card numbers alone) | 7 of 10 | 10.43 | - | - | - | - | - |
| **T4 tuned stage** | **10 of 10** | **2.76** | **2.38 / 5.95** | **0.004** | **0.001** | **0.900** | **1.82 deg** |

200 trials, 5.6 minutes, seed 20260926, every trial in `design/realism/eval/trials/palm_leaf.csv`.
The tuned render is the best of the five on the loss and on every colour and texture row. It also
fixes both rows the T3a hand probe could not (anisotropy, grain direction).

## What the numbers do not say

- **The lighting row still FAILS.** Tuned material + carved letters
  (`materials/palm_leaf.lighting.params.json`): 2.841 dL\* against the 4.0 dL\* target - better than
  T3a's 2.448, still short. DECISIONS.md (T3a, 2026-09-26 19:04) stands: 4 dL\* needs cutting more
  than half way through a 0.47 mm leaf, and palm-leaf letters are read because they are rubbed with
  soot, not because they cast shadows. Reported, not faked. The letter depth here (0.010912 with
  `height.bump` 0.275) renders the identical 0.003-unit cut T3a used - see DECISIONS.md (T4).
- **Real-photo pass rates** (`eval/self_test/real_photo_pass_rates.json`, palm_leaf): a real photo
  passes `spectral_slope_diff` only **26.9%** of the time (n=26; flagged "target stricter than real
  photo-to-photo variation"), `local_contrast_ratio` 69.2%, `anisotropy_diff` 73.1%,
  `colour_dE2000_median` 71.9%, `colour_dE2000_p95` 79.7%, `colour_mean_in_photo_range` 70.3%.
  The tuned render passes all six - including the one only a quarter of real photos pass. That is a
  reason to trust the sheet over the scoreboard, not the other way round: passing a target real
  leaves usually fail means the render is smoother and more average than a leaf, which is exactly
  what the five differences above say.
- `noise.gain` 0.345 sits on the low edge of its search range and `noise.scale` 42 near the high
  edge, so these are **tuned, not robust, numbers** (as T3b's baseFrequency was).
- The whole sheet and score measure the material with **letters off** and **wear age 0**.
- Nothing here has been checked by a Tamil speaker; the sample text in the lighting params stays
  flagged unverified.

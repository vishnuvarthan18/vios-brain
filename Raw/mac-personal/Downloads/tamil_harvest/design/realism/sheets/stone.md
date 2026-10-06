# stone - T4 auto-tune, side-by-side sheet

Sheet: `design/realism/sheets/stone.png` (gitignored - it carries crops of reference photos of every
tier, including CC BY-SA and CC BY-NC; never shipped). Rebuild it with

```
design/realism/.venv/bin/python design/realism/eval/sheet.py stone \
  design/realism/eval/out/tuned/stone.png \
  --photos STO-095,STO-475,STO-106,STO-504,STO-471,STO-388,STO-378,STO-191,STO-328,STO-384,STO-266,STO-067,STO-203,STO-016,STO-408,STO-486,STO-250,STO-125,STO-457,STO-493,STO-089,STO-562,STO-200,STO-048,STO-256,STO-521,STO-451,STO-097,STO-263,STO-333 --n 6
```

Top row: 128 px plain patches from six real photos (STO-095, STO-475, STO-106, STO-504, STO-378,
STO-067). The list is long because most stone photos give no plain patch at this patch size: the
first 10 photos of the spec's `photos_used_ids` yielded **one** usable photo, and 30 ids were needed
to reach six. Bottom row: six 128 px plain patches of the tuned render. **Same scale on both rows**:
both sides go through `lib/measure.py normalise()` with the spec's `measure_norm` (long side =
1024 px), the same code `eval/visual.py` scores with.

## The 5 biggest visible differences (my own eyes, not a score)

1. **Six renders, one stone; six photos, six stones.** The render crops are interchangeable - the
   same cloudy mottle at the same scale and the same colour in all six. The real crops are six
   different materials: blue-grey granite (STO-095), a chisel-pocked grey face (STO-475),
   rust-orange weathered rock (STO-106), olive-brown (STO-504), a near-white smooth face (STO-378),
   dark green-grey with a joint running through it (STO-067). The tuned render matches the *average*
   stone (every scored row passes); it has none of the *spread* between stones, which is most of what
   the eye sees here.
2. **Too warm.** The render is peach-tan everywhere (L\*a\*b\* 63.1, 3.1, 13.6 - almost exactly the
   spec mean 63.5, 3.8, 12.0). Four of the six real crops are neutral or cool: grey, grey-green,
   near-white. The spec mean is warm because the photo set mixes weathered and lichened stone with
   fresh granite; a render sitting on that mean looks warmer than most single stones.
3. **No tool marks, no joints, no edges.** STO-475 carries rows of chisel pocks, STO-106 and
   STO-067 a vein and a joint that cross the whole crop. Nothing in the render is longer than the
   noise: no straight line, no discontinuity, no worked surface.
4. **No dirt.** Real stone crops carry lichen, rust runs, black soiling collected in the hollows and
   pale efflorescence - all of it patchy and slow-varying. The render is uniformly clean. (T7's wear
   layer is the existing tool for this and was NOT used here: `wear.age` is 0, wear is T7's parameter.)
5. **Cloud, not crystal.** Granite reads as discrete light and dark crystals a few px across. The
   render's texture is smooth fbm cloud with soft edges; the tuner left `speckle.density` at 0 (the
   shader's speckle term was inert for T6a too), so there are no grains at all. The local-contrast
   row passes (ratio 0.993) because it measures the *amount* of variation, not its shape.

Differences 1, 2 and 4 are one fault said three ways: **the score asks the render to sit near the
middle of the photo set, and a real stone is one draw from that set, with its own colour cast, its
own history and its own tool marks.**

## Before -> after (all scored by `eval/visual.py`, loss by `eval/tune.mjs lossOf`)

| render | visual rows passing | loss | colour dE2000 med / p95 | colour mean in range | slope diff | anisotropy diff | local contrast ratio |
|---|---|---|---|---|---|---|---|
| site 2D baseline (styleguide.html) | 2 of 6 | - | **6.26 FAIL** / 8.69 | yes | **0.41 FAIL** | **0.179 FAIL** | **0.388 FAIL** |
| T6a stone material (page render, hand probe) = tuner trial 0 | 4 of 6 | 5.43 | **6.46 FAIL** / **13.2 FAIL** | yes | 0.013 | 0.079 | 0.961 |
| T4 tuned, run 1 (seed 20260926) | 5 of 6 | 3.38 | **6.39 FAIL** / 11.25 | yes | 0.009 | 0.041 | 1.026 |
| **T4 tuned, run 2 (shipped)** | **6 of 6** | **2.11** | **6.00 / 11.26** | **yes** | **0.000** | **0.005** | **0.993** |

Two chained runs of 200 trials each (7.4 min + 7.2 min), every trial in
`design/realism/eval/trials/stone.run1.csv` and `stone.csv`. Both were run with the stone's physics
pinned: `--pin=surface.metal=0 --limit=grain.angle_deg=-2:2` (see DECISIONS.md). Run 1 started from
the T6a probe params, run 2 from run 1's best, so no step could be worse than the render before it.
In both runs the 120 random trials found **nothing** better than their starting point; every gain
came from coordinate refinement.

## What the numbers do not say

- **`colour_dE2000_median` passes by less than 0.005.** The reported 6.00 is `round(median, 2)`
  against a `< 6.0` target, and the pass is decided on the unrounded value. Treat this row as *at*
  the limit, not comfortably inside it. It is also the row real photos pass least often (below).
- **Real-photo pass rates** (`eval/self_test/real_photo_pass_rates.json`, stone):
  `colour_dE2000_median` **27%**, `spectral_slope_diff` **33%**, `colour_dE2000_p95` 40%,
  `local_contrast_ratio` 44%, `colour_mean_in_photo_range` 69%, `anisotropy_diff` 78%. The tuned
  render passes all six, including the two that only about a third of real stone photos pass. As
  with the palm leaf, that is a reason to believe the sheet over the scoreboard: passing targets real
  stones usually miss means the render is more average than any stone.
- **Tuned, not robust, numbers.** `noise.scale` 13 is the single knob that flipped the colour median
  from fail to pass (trial 160, one step of -1), and `surface.roughness` 0.9 and `color.contrast` 0.9
  sit near the edge of their ranges. A small change to any of them can lose the 6/6.
- **The lighting row is unchanged by design.** The tuner raised `height.bump` 1.0 -> 1.3 with letters
  OFF, which would have deepened the cut letters by 30% as a side effect;
  `materials/stone.lighting.params.json` lowers `height.letter_depth` 0.008 -> 0.006154 (exactly
  1/bump) so the rendered cut is the identical one T6a scored. The ~5 mm cut on a ~600 mm slab is a
  **T6a estimate, unverified**: no source measures the groove depth of a Tamil stone inscription.
- The whole sheet and score measure the material with **letters off** and **wear age 0**.
- Nothing here has been checked by a Tamil speaker; the sample text in the lighting params stays
  flagged unverified.

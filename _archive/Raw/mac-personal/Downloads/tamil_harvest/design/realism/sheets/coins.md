# coins - T4 auto-tuned material vs the reference photos

Sheet: `design/realism/sheets/coins.png` (gitignored - it carries crops of reference photos of every
tier). Top row: 128 px plain patches from 6 reference photos (COI-136, COI-067, COI-098, COI-025,
COI-014, COI-012). Bottom row: 6 plain patches of `eval/out/tuned/coins.png`. Both rows went through
`lib/measure.py normalise()` with the spec's `measure_norm` (long side -> 1024 px), so one sheet pixel
is the same distance on the object on both rows.

## Numbers - before (site 2D baseline) vs after (T4 tuned stage)

| row | target | 2D baseline (styleguide.html) | T4 tuned | real-photo pass rate |
|---|---|---|---|---|
| colour_dE2000_median | < 6.0 | 28.37 FAIL | **2.81 PASS** | 42% |
| colour_dE2000_p95 | < 12.0 | 37.37 FAIL | **10.15 PASS** | 68% |
| colour_mean_in_photo_range | inside [14.1,-2.9,-3.5]..[47.5,7.2,18.1] | [68.1, 3.5, 46.1] FAIL | **[31.7, 1.5, 5.8] PASS** | 61% |
| spectral_slope_diff | < 0.3 (spec -3.6588) | 2.14 FAIL | **0.002 PASS** (render -3.661) | 35% |
| anisotropy_diff | < 0.15 (spec 0.2476) | 0.239 FAIL | **0.089 PASS** (render 0.336) | 90% |
| local_contrast_ratio | 0.8..1.25 (spec 2.0615 L*) | 0.427 FAIL | **1.02 PASS** (render 2.103) | 35% |
| grain_direction_diff_deg | < 15.0 | 57.93 FAIL | **1.17 PASS** | - |
| aspect_ratio | 1.01..2.003 | 1.095 PASS | **1.027 PASS** | - |
| **visual rows passed** | | **1 of 8** | **8 of 8** | |
| lighting (5 azimuths, elev 35) | >= 4 dL* and edge excess >= 2 dL* | no light input: fixed CSS shadows, FAIL | **15.42 dL*, edge excess 16.29, PASS** | - |

Tuner: 3 chained runs, 200 trials each (`eval/trials/coins.run1.csv`, `coins.run2.csv`, `coins.csv`).
Loss 4.633 (7/8, run 1 from the spec card) -> 4.107 (8/8, after the edge_rough probe) -> **2.980 (8/8, run 2)**;
run 3 with a fresh seed found nothing better and returned the identical params hash `8a2a0da6f9`.
No tuned row is below its target, so no target needs its real-photo pass rate as an excuse; the rates are
printed above anyway, and the two strictest (spectral slope and local contrast, 35%) are both passed.

## The 5 biggest visible differences (my own eyes on the sheet - not proof; the numbers above decide)

1. **No variation between patches.** The six real crops are six different coins and look it: olive-green
   (COI-136), grey-blue (COI-067), orange-red corrosion (COI-014), near-black brown (COI-012). All six
   render crops are the same mid-brown, because they are six windows on one procedural field with one
   colour ramp. The statistics match the *median* coin; the spread between coins does not exist.
2. **One uniform grain everywhere; no corrosion patches.** Real crops have distinct islands of corrosion
   product with hard edges (the red-green mottling in COI-014, the crust in COI-012). The render's texture
   is a single stationary pebbled noise (noise.scale 46, octaves 6) with constant chroma - only luminance
   varies, so it reads as fine sandstone rather than a corroded copper alloy.
3. **No macro form.** COI-098 and COI-012 are dominated by the coin's own curvature and rim: a big dark
   arc and a smooth brightness gradient across the patch. The tuned render is lit flat (a disc with a
   height field, no dome and no raised rim), so every render patch has the same average brightness.
4. **Focus.** Several real patches are defocused (COI-067 and COI-025 are visibly blurred, and COI-025 also
   catches green background bleed). The render is uniformly sharp at full resolution. The spectral slope
   still matches (0.002) because the spec pools the *median* photo, but no render patch is soft.
5. **Surface sheen.** The tuned material sits at metal 0.4 / roughness 0.94, which gives a dry, matte
   field. The brighter real coins (COI-136, COI-014) show broad low-frequency specular sheen from worn
   high points of the relief; the render has none, because the relief the sheen would sit on is absent
   (letters are off in the scored render by design - the score measures the material).

## Unverified / estimates in this material

- `shape.edge_rough 0.08` (a ragged struck flan, +-8% of the coin width) is an **estimate**, not a measured
  number. It exists because `rig/stage.js` forces aspect = 1 for the `disc` shape, so a perfect circle
  measures aspect 1.000 and can never pass the spec's 1.01..2.003 row; the spec card does say the flan is
  irregular (circularity 0.523, aspect_ratio_photos 1.098). See DECISIONS.md for the 6-value probe.
- The lighting candidate's relief height, about 0.20 mm on a 17 mm massa
  (`materials/coins.lighting.params.json`, letter_depth 0.005 x bump 2.4 = 0.012 object units), is an
  **estimate**: no source in `specs/coins.json` measures the height of a massa's legend.
- The legend text `ராஜராஜ` is **unverified**: no Tamil speaker has checked it and no source here reads
  these coins' legends.
- The stage has no raised-relief mode, so the legend is rendered **carved** (cut in) while a struck coin's
  legend stands proud. The lighting test measures edge brightness change between light azimuths, which is
  the same for a groove and a ridge of equal depth, but the sign of the relief is inverted against the real coin.

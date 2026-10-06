# pottery - T4 auto-tune sheet notes

Sheet: `design/realism/sheets/pottery.png` (gitignored, never shipped: it carries crops of reference
photos of every tier). Top row: 128 px plain patches of six real photos (POT-108, POT-133, POT-015,
POT-143, POT-040, POT-024). Bottom row: six 128 px patches of the tuned render
`eval/out/tuned/pottery.png`. Both rows went through `lib/measure.py normalise()` with the spec's
`measure_norm` (long side -> 1024 px), so one sheet pixel is the same object distance on both rows.

## Numbers

Tuner: `eval/tune.mjs pottery`, two chained runs of 200 trials (6.9 min + 6.6 min),
`--pin=surface.metal=0 --limit=grain.angle_deg=-2:2`, base = `specs/pottery.json` alone (pottery has
no T6 probe file). Run 2 started from run 1's best and found nothing better in 200 fresh trials.
Every trial: `eval/trials/pottery.run1.csv` and `eval/trials/pottery.csv`.

| row | target | 2D baseline (styleguide.html) | trial 0 (spec card, untuned) | T4 tuned | real-photo pass rate |
|---|---|---|---|---|---|
| colour_dE2000_median | < 6.0 | 5.77 PASS | 8.22 FAIL | **7.34 FAIL** | 23.3% (real median 7.22) |
| colour_dE2000_p95 | < 12.0 | 8.0 PASS | 9.78 PASS | 11.25 PASS | 66.7% |
| colour_mean_in_photo_range | inside 5-95% of photo means | PASS | [30.8, 9.7, 15.0] PASS | [27.3, 6.4, 9.9] PASS | 63.3% |
| spectral_slope_diff | < 0.3 (spec -2.4505) | 0.291 PASS | 0.935 FAIL | **0.001** PASS | 23.8% |
| anisotropy_diff | < 0.15 (spec 0.22) | 0.109 PASS | 0.090 PASS | **0.007** PASS | 66.7% |
| local_contrast_ratio | 0.8..1.25 (spec 1.7498 L*) | 1.082 PASS | 0.961 PASS | 1.026 PASS | 47.6% |
| grain_direction_diff_deg | < 15.0 | **29.13 FAIL** | 37.67 FAIL | 4.91 PASS | - |
| aspect_ratio | 1.109..2.663 | 1.333 PASS | 1.515 PASS | 1.515 PASS | - |

Loss (sum of row distances in units of their own target, failing rows x2):
**16.173 -> 4.243**, rows passing **5 of 8 -> 7 of 8**.

**Honest comparison:** the site's 2D baseline also scores 7 of 8 - it fails a *different* row
(grain direction, 29.13 deg). The tuned material is much closer on every texture row
(slope 0.001 vs 0.291, anisotropy 0.007 vs 0.109, grain direction 4.91 vs 29.13 deg) and further
away on colour (dE2000 median 7.34 vs 5.77, p95 11.25 vs 8.0). It is not a clean win on row count.

**The failing row against reality:** `colour_dE2000_median` is passed by only 23.3% of real pottery
photos (leave-one-out, `eval/self_test/real_photo_pass_rates.json`), and the median real photo
scores 7.22 against the target 6.0. The tuned render's 7.34 is within 0.12 of what a real
photograph of a sherd scores. The target is stricter than photo-to-photo variation; per the owner's
rule it is reported, not changed. No set of parameters in the 400 trials passed all 8 rows: the 42
trials with dE2000 median < 6.0 all failed a texture row instead (best of them: loss 9.53, 6 of 8).

## The 5 biggest visible differences

1. **Colour: the render is a dull grey-brown; much of the real set is saturated orange terracotta.**
   POT-108 and POT-015 are bright orange slip; the render sits at L*a*b* [27.3, 6.4, 9.9], far less
   chromatic. This is exactly the failing row. The ramp came out narrow and low in chroma because
   the reference set also holds near-black burnished ware (POT-133) and soil-covered sherds
   (POT-143), and one ramp has to cover all of them.
2. **A faint regular horizontal striation runs through every render patch.** The tuned parameters
   keep `grain.fibre 0.06` at `fibre_density 77.5` with `fibre_height 0.0028`, which reads as thin
   parallel lines. Real clay has no repeating linear texture at this scale: it is either smooth
   burnished or randomly granular. It survived the search because the anisotropy and grain-direction
   rows reward a weak, correctly-aligned direction - the score cannot tell "weak fibres" from
   "slightly directional clay".
3. **All six render patches look the same; the six real ones look like six different objects.**
   The render has one material with no object-to-object or within-object variation, while the real
   row spans black-ware, orange slip, gritty encrusted surfaces and vegetation-covered sherds. The
   visual score compares statistics patch by patch, so it never penalises this.
4. **No large-scale blotching.** Real sherds carry fire-clouding, dark cores, slip patches and
   mineral inclusions at the 100+ px scale (visible in POT-015 and POT-143). The render's variation
   is almost all fine grain: `noise.scale 35` with only 4 octaves and `lacunarity 1.6` puts the
   energy in a narrow band, which is what makes the spectral slope match (0.001) while the patches
   still look flat.
5. **No micro-relief.** `height.noise_amp` was tuned down to 0.00075 with `bump 0.83`, so grit
   grains and chips cast no shadow. POT-143's coarse temper and POT-133's shadowed hollow have real
   relief the render has no equivalent for; the render's texture is painted, not lit.

Unverified: the pottery spec card's physical numbers (density 1.9 g/cm3, thickness 7 mm,
restitution 0.2, friction 0.5) are all `estimate: true` and are not used by this render. The colour
and texture numbers above are measured from 30 uncalibrated photos, so part of the colour spread is
camera and lighting, not material (`colour.between_photo_sd` [11.63, 13.33, 12.96]).

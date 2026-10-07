# rings - T4 auto-tuned material vs the reference photos

Sheet: `design/realism/sheets/rings.png` (gitignored - it carries crops of reference photos of every
tier). Top row: 128 px plain patches from 6 reference photos (RIN-042, RIN-063, RIN-053, RIN-052,
RIN-048, RIN-047). Bottom row: 6 plain patches of `eval/out/tuned/rings.png`. Both rows went through
`lib/measure.py normalise()` with the spec's `measure_norm` (long side -> 1024 px), so one sheet pixel
is the same distance on the object on both rows.

## Numbers - before (site 2D baseline) vs after (T4 tuned stage)

| row | target | 2D baseline (styleguide.html) | T4 tuned | real-photo pass rate |
|---|---|---|---|---|
| colour_dE2000_median | < 6.0 | not measurable | 11.65 FAIL | **0%** (real median 11.08) |
| colour_dE2000_p95 | < 12.0 | not measurable | 13.87 FAIL | **0%** (real median 17.52) |
| colour_mean_in_photo_range | inside [27.8,-0.7,1.2]..[55.1,29.7,42.8] | not measurable | **[29.7, 7.8, 28.4] PASS** | 50% |
| spectral_slope_diff | < 0.3 (spec -3.5242) | not measurable | **0.001 PASS** (render -3.525) | 0% |
| anisotropy_diff | < 0.15 (spec 0.2969) | not measurable | **0.024 PASS** (render 0.321) | 75% |
| local_contrast_ratio | 0.8..1.25 (spec 2.6427 L*) | not measurable | **1.037 PASS** (render 2.741) | 75% |
| grain_direction_diff_deg | < 15.0 | not measurable | **11.24 PASS** | - |
| aspect_ratio | 1.101..2.304 | not measurable | 1.000 FAIL | - |
| **visual rows passed** | | **0 of 1** (`render_measurable` False) | **5 of 8** | |
| lighting | - | - | no candidate: the ring has no letters or relief | - |

The 2D baseline has only one row because the site's 2D ring has no plain patch the measuring pipeline
can use (`render_measurable` FAIL), so none of the eight rows above can be produced for it at all.

Tuner: 2 chained runs, 200 trials each (`eval/trials/rings.run1.csv`, `rings.csv`).
Loss 27.272 (2 of 8, trial 0 = the plain spec-card annulus) -> **9.497 (5 of 8, run 1)**; run 2 with a
fresh seed (20260927) from run 1's best found nothing better and returned the identical params hash
`4b3d0168bb`. Best params: `materials/rings.params.json`.

### The three failing rows, with their reasons (none was tuned away)

- **colour_dE2000_median 11.65 and p95 13.87.** `0 of 16` real ring photos pass either row: the real
  leave-one-out medians are 11.08 and 17.52, so **the tuned render is already closer to the spec's
  colour distribution than a real photograph of a ring is**. These targets are stricter than the
  material: `specs/rings.json` pools gold, bronze and heavily patinated rings (colour sd 16.6/12.7/17.3
  L*a*b*, between-photo sd 9.1/10.7/14.8, and the card's own note says only 16 of a target 20 photos
  were usable, confidence low). The owner's targets are reported, not changed (AGENT_BRIEF section 5).
- **aspect_ratio 1.000 against 1.101..2.304.** `rig/stage.js` line 296 forces aspect = 1 for shape type
  `ring` (as for `disc`), so a circular annulus can never measure anything but 1.000. The probe
  (`eval/t4_rings_aspect_probe.mjs`, `eval/out/tune/rings_aspect_probe/probe.json`) shows no outline
  wobble buys the row: edge_rough 0.05 reaches only aspect 1.018 and already fails grain_direction
  (24.46 deg), and 0.10 / 0.16 / 0.24 eat the thin band so the render becomes unmeasurable (loss 60);
  band width (inner 0.3 / 0.6) does not move it either. The spec row is also the noisiest on the card
  (n=31, "photos include bundles, stacks and partial views", circularity 0.11..0.615). A ring lying
  face-on **is** a circle, so this row is reported as a FAIL rather than bought with a bent ring.
  See DECISIONS.md 2026-09-26 22:3x.

## The 5 biggest visible differences (my own eyes on the sheet - not proof; the numbers above decide)

1. **No variation between rings.** The six real crops are six different objects and look it: bright
   yellow gold (RIN-042, RIN-063), flat grey-brown (RIN-053), green corrosion over metal (RIN-052),
   dark glossy red-brown (RIN-048), saturated orange (RIN-047). All six render crops are the same
   mid-brown, because they are six windows on one procedural field with one colour ramp. The statistics
   match the *median* ring; the spread between rings does not exist. This is the same limit the other
   T4 materials hit, and for rings it is the whole story of the two failing colour rows.
2. **The real crops are smooth metal; the render is pebbled everywhere.** RIN-063, RIN-053 and RIN-047
   are near-featureless at this scale - polished or cast metal carries almost no small-scale texture,
   only a slow brightness gradient. The tuned render has a uniform granular field (noise.scale 27.5,
   octaves 4) over every patch. The spectral slope still matches to 0.001 because the spec's median
   photo pools corroded rings as well, but no render patch is *smooth*.
3. **No specular highlight and no macro form.** Every real crop is dominated by the ring's own
   curvature: a bright band running across the band of the ring (RIN-063), a dark glossy sweep
   (RIN-048). The render is a flat annulus with a height field, lit from one fixed direction, so each
   patch has nearly the same average brightness and no reflection of a studio light.
4. **No corrosion islands.** RIN-052's green and RIN-042's red-brown patches are distinct deposits with
   hard edges sitting on top of the metal. The render's texture is one stationary noise with constant
   chroma - only luminance varies - so it reads as fine sandstone rather than a corroded metal surface.
5. **Focus and background bleed.** RIN-042 and RIN-052 are visibly defocused and RIN-052 catches green
   background through the bore of the ring; RIN-063 catches the dark display surface at the top edge.
   The render is uniformly sharp at full resolution and has no surroundings at all.

## Unverified / estimates in this material

- Everything in `specs/rings.json` `physical` is an **estimate** (inner_diameter_mm 18, mass_g 8,
  restitution 0.4, all `estimate: true`, `n_samples: 0`, no sources): the photographed rings carry no
  scale. The tuned material does not depend on those numbers, but the physics scenes (T5c) do.
- The spec card itself is flagged low confidence: **16 usable photos of a target 20**, `enough_photos:
  false`, of which only 8 gave plain texture patches.
- No lighting candidate exists for rings, so nothing here says how a signet ring's bezel relief behaves
  under a moving light; the `ring` shape is a plain annulus.

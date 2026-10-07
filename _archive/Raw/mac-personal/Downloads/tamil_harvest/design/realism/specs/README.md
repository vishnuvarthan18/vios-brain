# Material spec cards (T1)

One JSON per material: `palm_leaf, stone, copper_plate, pottery, coins, rings, seals, cotton_thread, wood_board`.
Built by `build_specs.py` from `design/references/<surface>/` + `LICENSES.csv` (read only) and the sourced table
`sources/physical_sources.json`. Every number has `value, range, n_samples, source_ids, confidence, estimate`.

## How the photo numbers were made

- Up to 150 photos per material (direct, then regional, then technique relevance; "off" rows skipped).
- Background removed automatically (border colours that are smooth = background). Drawings and rubbings dropped.
- Frame-filling photos count only for surfaces (stone, palm leaf, copper plate close-ups); for pottery, coins, rings
  and seals they are scenes (soil, walls, cloth) and are skipped.
- Patches: the 70% of 128 px patches with the least edge energy (divided by sqrt(L*), so neither shadows nor white
  cloth are favoured), no clipped highlights or deep shadows. A colour-consensus filter then drops patches far from the material
  median (CIEDE2000 > min(max(10, median + 2 MAD), 25)), which removes plants, labels and cloth that slipped through;
  a photo where most patches disagree is dropped whole (probably mis-segmented).
- Stroke pixels (black-hat > 12 L*: ink, incised letters, gaps between leaves) are left out of the colour sample;
  texture numbers use only plain patches (<= 2% stroke pixels). Black-and-white photos are left out.
- Colour gates before the consensus: palm leaf must be yellow to brown (hue 35-105 deg, b* >= 6); stone, pottery
  and seals drop green (plants) and blue (sky); copper, coins and rings drop plant green only (patina kept).
- Scale: the object long side is resampled to 1024 px (palm leaf: the leaf width to 256 px, because leaf widths vary
  much less than lengths). No scale bars are detected, so texture sizes are relative.
- Photo subjects were checked on whole-photo contact sheets; 84 photos that clearly show something else (people,
  scenes, drawings, signs, modern coins) are excluded with a reason each in `sources/manual_exclusions.json`.
- Colour comes from uncalibrated photos with unknown white balance. Colour confidence is never "high".

## Photo counts

| material | photos considered | photos used | enough (>= 20)? | SHIP / STUDY used |
|---|---|---|---|---|
| palm_leaf | 150 | 64 | yes | 31 / 33 |
| stone | 150 | 62 | yes | 17 / 45 |
| copper_plate | 104 | 22 | yes | 14 / 8 |
| pottery | 150 | 30 | yes | 8 / 22 |
| coins | 115 | 31 | yes | 11 / 20 |
| rings | 46 | 16 | NO | 6 / 10 |
| seals | 58 | 16 | NO | 8 / 8 |
| cotton_thread | 0 | 0 | NO | 0 / 0 |
| wood_board | 0 | 0 | NO | 0 / 0 |

`cotton_thread` and `wood_board` have no reference folder: no photo measurements exist for them.

## Low-confidence and estimated numbers (74) - check these against real objects

| material | key | value | range | n | estimate? | why |
|---|---|---|---|---|---|---|
| palm_leaf | geometry.aspect_ratio_catalogue_tamil | 12.97 | [4.5, 26.8] | 7 | no | Tamil entries only (small n) |
| palm_leaf | physical.density_g_cm3 | 0.8 | [0.5, 1.1] | 0 | yes | no source found; dried lignocellulose (cell-wall material ~1.5 g/cm3) with 30-60% air space. |
| palm_leaf | physical.mass_per_leaf_g | 5.5 | [1.5, 12] | 0 | yes | formula m = density x length x width x thickness = 0.8 x 40 cm x 3.7 cm x 0.047 cm ~ 5.6 g; range from the size and density ranges. |
| palm_leaf | physical.youngs_modulus_along_GPa | 4.0 | [1.5, 8.0] | 0 | yes | no source found; taken in the range of dry paper and thin card along the fibre, which palm leaf resembles in handling. |
| palm_leaf | physical.bending_stiffness_ratio_across_over_along | 3.0 | [2.0, 10.0] | 0 | yes | fibres (veins) run along the leaf length (SRC-FREEMAN-2005 describes closely spaced longitudinal fibres). Bending that curves the fibres (fold line across the f |
| palm_leaf | physical.max_curvature_1_per_m | 33 | [15, 60] | 0 | yes | aged leaves are brittle (SRC-FREEMAN-2005: brittleness develops over time); a minimum bend radius of about 30 mm (curvature 33 /m) is assumed before cracking. T |
| palm_leaf | physical.friction_leaf_on_leaf | 0.35 | [0.25, 0.5] | 0 | yes | waxy, polished surface (SRC-FREEMAN-2005 notes the waxy palmyra surface); taken like smooth paper on paper. |
| palm_leaf | physical.restitution | 0.1 | [0.0, 0.2] | 0 | yes | a thin flexible sheet lands almost without bounce. |
| palm_leaf | physical.damping_ratio | 0.08 | [0.03, 0.2] | 0 | yes | a large light sheet is strongly slowed by air; chosen so a flicked leaf settles in about one second. |
| stone | texture.spectral_slope | -2.5571 | [-3.769, -1.731] | 10 | no | L* of 128 px patches, Hann window, fit over 1/32..1/4 cycles/px |
| stone | texture.anisotropy | 0.2577 | [0.113, 0.3679] | 10 | no | structure-tensor coherence of L* per patch |
| stone | texture.grain_dir_rel_long_axis_deg | 0.5572 | [0.1746, 1.2893] | 6 | no | structure-tensor feature direction vs mask principal axis; photos where the object fills the frame are left out |
| stone | texture.grain_scale_corr_length | 12.75 | [7.25, 15.4] | 10 | no | 1/e autocorrelation length of L* |
| stone | texture.local_contrast_L | 2.4724 | [0.7917, 3.1428] | 10 | no | mean local standard deviation of L* |
| stone | physical.density_g_cm3 | 2.7 | [2.6, 2.9] | 0 | yes | commonly quoted density of granite and charnockite, the usual temple and inscription stones in Tamil Nadu; not read from a named source in this session. |
| copper_plate | texture.spectral_slope | -1.9524 | [-3.4905, -0.9547] | 13 | no | L* of 128 px patches, Hann window, fit over 1/32..1/4 cycles/px |
| copper_plate | texture.anisotropy | 0.2305 | [0.0779, 0.4251] | 13 | no | structure-tensor coherence of L* per patch |
| copper_plate | texture.grain_dir_rel_long_axis_deg | 0.4087 | [0.2063, 0.4868] | 9 | no | structure-tensor feature direction vs mask principal axis; photos where the object fills the frame are left out |
| copper_plate | texture.grain_scale_corr_length | 15.0 | [3.0, 21.6] | 13 | no | 1/e autocorrelation length of L* |
| copper_plate | texture.local_contrast_L | 2.2655 | [0.9131, 3.0577] | 13 | no | mean local standard deviation of L* |
| copper_plate | geometry.hole_distance_from_nearer_end | 0.205 | [0.079, 0.464] | 21 | no | round holes on the centre line of single objects |
| copper_plate | geometry.hole_radius | 0.023 | [0.021, 0.05] | 21 | no |  |
| copper_plate | geometry.holes_per_object_photos | 2.0 | [2, 3] | 9 | no |  |
| copper_plate | physical.thickness_mm | 2.5 | [1.5, 5.0] | 0 | yes | formula with guessed inputs: if ~23 kg of the 30 kg is plates, one plate ~1.1 kg; at 40 x 12 cm and 8.9 g/cm3, t = 1100 / (8.9 x 480) ~ 2.6 mm. Plate size is no |
| copper_plate | physical.friction_plate_on_plate | 0.6 | [0.3, 1.0] | 0 | yes | clean copper on copper is high (near 1); oxide and patina lower it. |
| copper_plate | physical.restitution | 0.3 | [0.15, 0.5] | 0 | yes | plates clink and stop quickly, assumed. |
| copper_plate | physical.damping_ratio_swing | 0.02 | [0.005, 0.06] | 0 | yes | a heavy plate on a ring swings for many periods; friction at the ring hole dominates. |
| pottery | physical.density_g_cm3 | 1.9 | [1.6, 2.2] | 0 | yes | fired clay (terracotta, black-and-red ware) bulk density, commonly quoted range. |
| pottery | physical.sherd_thickness_mm | 7 | [4, 12] | 0 | yes | hand and wheel-made vessels; not measured. |
| pottery | physical.restitution | 0.2 | [0.1, 0.35] | 0 | yes | ceramic on wood, assumed. |
| pottery | physical.friction | 0.5 | [0.35, 0.7] | 0 | yes | unglazed rough surface, assumed. |
| coins | physical.mass_g_massa | 4.0 | [2.5, 4.7] | 3 | no | RajaRaja Chola copper massa, trade listings; Van Arsdale measured 100 coins (rms weight scatter ~8.5%) but his means were not read. |
| coins | physical.diameter_mm_massa | 17 | [15, 19] | 3 | no | trade listings. |
| coins | physical.thickness_mm_massa | 2.0 | [1.2, 3.0] | 0 | yes | formula t = m / (density x pi r^2) = 4.0 g / (8.9 x pi x 0.85^2 cm2) ~ 0.20 cm. |
| coins | physical.friction_on_table | 0.4 | [0.25, 0.6] | 0 | yes | metal on wood, assumed. |
| coins | physical.restitution_on_table | 0.35 | [0.2, 0.55] | 0 | yes | a dropped coin bounces a little on wood, assumed. |
| coins | physical.rest_tilt_limit_deg | 3 | [0, 5] | 0 | yes | geometry: a coin lying on a flat table rests flat; relief height (<0.5 mm) over 17 mm diameter allows under 2 deg. |
| rings | colour.mean | [41.28, 10.23, 23.42] | [[29.29, -0.47, 3.49], [52.67, 26.09, 37.41]] | 16 | no |  |
| rings | colour.sd | [16.61, 12.67, 17.31] | null | 16 | no |  |
| rings | colour.p5 | [17.08, -3.1, -1.0] | [[11.69, -5.75, -3.2], [37.15, 14.45, 28.18]] | 16 | no |  |
| rings | colour.p50 | [40.1, 5.6, 25.9] | [[27.75, -0.65, 3.45], [55.53, 27.25, 37.3]] | 16 | no |  |
| rings | colour.p95 | [69.1, 37.02, 53.0] | [[46.28, 2.95, 8.2], [81.68, 41.8, 53.35]] | 16 | no |  |
| rings | colour.between_photo_sd | [9.1, 10.73, 14.8] | null | 16 | no | spread of photo means: lighting and white balance differ between photos, so part of this is camera, not material |
| rings | texture.spectral_slope | -3.5242 | [-4.3986, -2.312] | 8 | no | L* of 128 px patches, Hann window, fit over 1/32..1/4 cycles/px |
| rings | texture.anisotropy | 0.2969 | [0.1324, 0.4305] | 8 | no | structure-tensor coherence of L* per patch |
| rings | texture.grain_dir_rel_long_axis_deg | 0.7193 | [0.3875, 1.2111] | 8 | no | structure-tensor feature direction vs mask principal axis; photos where the object fills the frame are left out |
| rings | texture.grain_scale_corr_length | 14.75 | [12.0, 25.85] | 8 | no | 1/e autocorrelation length of L* |
| rings | texture.local_contrast_L | 2.6427 | [1.2006, 2.8962] | 8 | no | mean local standard deviation of L* |
| rings | physical.inner_diameter_mm | 18 | [15, 22] | 0 | yes | adult finger sizes; seal rings in the photos have no scale. |
| rings | physical.mass_g | 8 | [3, 25] | 0 | yes | small gold or bronze signet ring, assumed. |
| rings | physical.restitution | 0.4 | [0.2, 0.6] | 0 | yes | metal on wood, assumed. |
| seals | colour.mean | [69.53, 6.46, 28.24] | [[58.86, -1.03, 14.44], [80.19, 13.09, 46.72]] | 16 | no |  |
| seals | colour.sd | [12.74, 6.16, 12.8] | null | 16 | no |  |
| seals | colour.p5 | [46.88, -2.22, 8.28] | [[41.6, -4.15, 7.15], [65.2, 9.55, 44.6]] | 16 | no |  |
| seals | colour.p50 | [70.1, 5.7, 28.1] | [[55.75, -1.18, 14.55], [81.95, 13.25, 46.85]] | 16 | no |  |
| seals | colour.p95 | [89.0, 15.8, 50.1] | [[71.35, 2.4, 17.63], [91.37, 18.4, 49.15]] | 16 | no |  |
| seals | colour.between_photo_sd | [8.29, 5.34, 11.93] | null | 16 | no | spread of photo means: lighting and white balance differ between photos, so part of this is camera, not material |
| seals | texture.spectral_slope | -3.3443 | [-5.1497, -2.5355] | 15 | no | L* of 128 px patches, Hann window, fit over 1/32..1/4 cycles/px |
| seals | texture.anisotropy | 0.3323 | [0.1292, 0.4755] | 15 | no | structure-tensor coherence of L* per patch |
| seals | texture.grain_dir_rel_long_axis_deg | 0.5321 | [0.2642, 1.4122] | 11 | no | structure-tensor feature direction vs mask principal axis; photos where the object fills the frame are left out |
| seals | texture.grain_scale_corr_length | 14.5 | [6.0, 22.0] | 15 | no | 1/e autocorrelation length of L* |
| seals | texture.local_contrast_L | 1.9416 | [1.4932, 2.7336] | 15 | no | mean local standard deviation of L* |
| seals | physical.diameter_mm | 30 | [15, 120] | 0 | yes | seals range from small ring bezels to the large bronze seal on a copper-plate ring; no measured sizes read. |
| seals | physical.density_g_cm3 | 8.7 | [1.8, 8.9] | 0 | yes | bronze seal; terracotta sealings would be ~1.9. |
| cotton_thread | physical.diameter_mm | 1.5 | [0.8, 3.0] | 0 | yes | binding cords in the reference photos look a few mm thick at most; no catalogue gives cord diameter. |
| cotton_thread | physical.fibre_density_g_cm3 | 1.54 | [1.5, 1.55] | 0 | yes | commonly quoted density of cotton fibre; not read from a named source in this session. |
| cotton_thread | physical.cord_packing_fraction | 0.6 | [0.4, 0.75] | 0 | yes | twisted cord has air between fibres. |
| cotton_thread | physical.linear_mass_g_per_m | 1.6 | [0.3, 6.5] | 0 | yes | formula: pi/4 x d^2 x fibre density x packing = 0.785 x 0.15^2 cm2 x 1.54 x 0.6 x 100 cm/m ~ 1.6 g/m. |
| cotton_thread | physical.max_stretch_fraction | 0.02 | [0.01, 0.07] | 0 | yes | a cord under the weight of a bundle stretches little; 2% is also the test limit in Prompt 4. |
| cotton_thread | physical.friction_on_leaf | 0.5 | [0.3, 0.7] | 0 | yes | fibre on leaf edge, assumed; the knot holds by friction. |
| wood_board | physical.thickness_mm | 8 | [4, 15] | 0 | yes | the Glasgow handlist gives board length and width but no thickness; 4-15 mm assumed from typical cover boards. |
| wood_board | physical.density_g_cm3 | 0.65 | [0.45, 0.9] | 0 | yes | wood species of the covers is not recorded; typical dry hardwood range. |
| wood_board | physical.friction_on_leaf | 0.4 | [0.3, 0.6] | 0 | yes | dry wood on smooth leaf, assumed. |
| wood_board | physical.restitution | 0.3 | [0.2, 0.5] | 0 | yes | wood on wood table, assumed. |

Sources for physical values: see `physical_sources` inside each card and `sources/physical_sources.json`.

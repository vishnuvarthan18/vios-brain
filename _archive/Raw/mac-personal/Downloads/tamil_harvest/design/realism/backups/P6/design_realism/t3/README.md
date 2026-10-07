# T3a - the palm leaf in real 3D

`t3/leaf3d.html` + ONE classic script `t3/leaf3d.bundle.js` (generated). Opens by double-click from
`file://`: no network, no CDN, no module, no library.

| file | what it is |
|---|---|
| `leaf3d.html` | the page (3D canvas, controls, keyboard, 2D fallback) |
| `leaf3d.bundle.js` | **generated** by `node design/realism/t3/build.mjs` - do not edit |
| `leaf3d.view.js` | the 3D renderer and page wiring (a bundle source) |
| `leaf3d.params.js` / `.json` | the material numbers and where each one comes from (`write_params.mjs` makes the json) |
| `build.mjs` | builds the bundle, prints raw + gzip size |
| `probe.mjs` | the short material grid scored with `eval/visual.py` (how the numbers were picked) |
| `checks.mjs` | the T3a checks; writes `eval/out/t3/leaf.png` (the judge render) and `checks.json` |
| `shot.mjs` | saves one PNG of the 3D canvas to look at (not scored) |

## What is simulated and what is drawn

The leaves are bending meshes over the physics particle grid (`physics/pbd.js` + `physics/scenes.js`
`bundle`, fixed 1/120 s steps through `FixedStepper`): 8 leaves, `nx` 11, 8 substeps - T5a's measured page
configuration. The cord is a real rope through both holes with a knot, the leaves rest on each other with
contact, and the two cover boards are rigid bodies. The two cord holes are cut in the fragment shader from
the same signed-distance outline the rig stage uses, at the measured `hole_distance_from_nearer_end` 0.289 L
and `hole_radius` 0.069 W, so the cord really passes through the hole the photos measured.

The material is the rig stage's own procedural model. The GLSL is **extracted from `rig/stage.js` at build
time**, not copied, so tuning the stage (T4) moves this page unchanged.

## Numbers (this run)

- bundle size: **122,953 bytes raw, 36,743 bytes gzip (35.9 KB)** of the 300 KB gzip budget (`bundle_size.json`)
- physics: 436 particles, **1.19 ms per fixed step** in the rig browser unthrottled (`eval/out/t3/checks.json`)
- page checks: **22 of 24 pass**. The two failures are visual rows, not page behaviour:
  `anisotropy_diff` 0.189 (target < 0.15) and `grain_direction_diff_deg` 17.46 (target < 15).
- judge render (`?static=1` -> `eval/out/t3/leaf.png`): colour dE2000 median **2.95** (target < 6),
  p95 **8.54** (< 12), mean colour inside the photo range, spectral slope diff **0.142** (< 0.3),
  local contrast ratio **0.909** (0.8..1.25), aspect 9.227 (5.86..14), **2 holes** at 0.289 of the length.

### Scoreboard rows for this page (after `eval/run_all.mjs`)

- pages: **all pass** - perf p95 **6.3 ms** at 4x throttle (target 20), 0 console errors and 0 failed
  `file://` requests on desktop and mobile, reduced motion static (0 changed pixels over 1500 ms),
  both fallbacks (`webgl_off`, `quality=low`) render with `#fallback` visible.
- visual: 8 of 10 (the two texture rows above).
- lighting: **FAILS** - `shading_changes_with_light_angle` 2.51 dL against a 4 dL target (edge excess
  11.1, well over its 2 dL target). Measured on this page, 4 dL needs a letter cut of about 0.005 object
  units = 0.24 mm, i.e. more than half way through a 0.47 mm leaf. The cut was kept at 0.14 mm and the row
  is reported as failing: palm-leaf letters are read because they are rubbed with soot, not because they
  cast shadows. See `DECISIONS.md` (T3a).

## What is unverified

The Tamil sample text (no Tamil speaker has checked it); the leaf thickness, densities, cord mass,
letter depth, board colour and cotton colour (all estimates - see `leaf3d.params.js` `_source` and
`specs/palm_leaf.json` `estimate: true`). The material is **not** auto-tuned: T4-palm_leaf has not run.
The remaining anisotropy and grain-direction failures are honest misses, not tuned away: a narrow leaf
render measures directional (0.44) even with the fibre term switched off (0.434).

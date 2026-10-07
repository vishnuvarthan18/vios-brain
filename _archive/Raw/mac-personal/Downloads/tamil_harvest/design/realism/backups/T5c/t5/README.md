# T5 - leaf bundle and thread (T5a), copper plate set on its ring (T5b)

## T5b - copper plate set

| file | what |
|---|---|
| `copper_set.html` | demo page: 21 copper plates hanging on one ring, simulated by `physics/scenes.js` `plateSet()`, drawn in an oblique view (depth along the ring exaggerated x2.2). Plain scripts, works from `file://`. Buttons are keyboard reachable; with `prefers-reduced-motion` the set is stepped to rest once and nothing moves. |
| `bench_plate_set.mjs` / `bench_plate_set.json` | cost of one fixed step inside the browser, 1x and 4x CPU throttle. |

`plateSet(opts)` - N rigid plates (particle grid + shape matching, one hole particle each) hanging on one
**fixed** ring. The ring is a circle of radius R in the y-z plane; a hanging plate rests on top of the
wire, so its hole centre is projected every substep onto the *seat circle* of radius
R + (r_hole - r_wire). In its own plane each plate therefore has a pivot that does not move (its period
is the rigid-body `2*pi*sqrt(L_eq/g)` of its own particles, `L_eq = I_pivot / (M d)`); along the ring it
slides freely, rises on the arc and pushes its neighbours (`layer()` contact along +z, gap = thickness).
The plate cannot lift off the wire or rock inside the hole clearance (0.76 mm). See `DECISIONS.md` (T5b).

`pbd.js` gained `layerUpPass` (World option, default **true**, so T2/T5a are unchanged). The second,
one-sided layer pass exists to push a stack up off a fixed board; with no fixed body at the end of the
chain it ratcheted the whole plate set along the ring (the set drifted 28 mm in 12 s and gained 30x the
swing energy). `plateSet` turns it off.

### Numbers (this machine)

| what | 21 plates (5x9 grid, 966 particles) | 9 plates (4x7, 261 particles) |
|---|---|---|
| ms per fixed step, Node | 0.89 | 0.18 |
| ms per fixed step, browser 1x | 1.01 | 0.21 |
| ms per fixed step, browser 4x throttle | 4.19 | 0.87 |
| ms per 60 Hz frame at 4x throttle (2 steps) | 8.38 (target 20) | 1.73 |

so the page runs the full 21-plate spec set, not a lighter one.

Test rows (`eval/physics_tests.mjs`, `plate_set_*`), reference scenes: worst per-plate period error
**0.04-0.06 %** of `2*pi*sqrt(L_eq/g)` (target 3 %, L_eq 0.2745 m, period 1.0512 s, all 21 plates
checked separately); amplitude peaks never grow; energy change -0.15 to -0.08 of the swing energy (no
gain); plate-to-plate separation 2.469 mm (thickness 2.5 mm - 10 % allowed); every hole stays within
0.19 mm of the ring seat circle (target 0.5 mm); identical hash on 3 runs and at 30/60/144 fps and
jittered frames. Known-bad scenes fail the row they should: no plate-plate contact (-4.80 mm
separation), negative damping (peaks grow, +0.17 energy), gravity 20 % low (no swing/period off),
plates not held on the ring (they fall).

### Unverified

Plate thickness 2.5 mm, plate-on-plate friction, restitution and the swing damping ratio 0.02 are spec
**estimates** (`specs/copper_plate.json`, `estimate: true`). The **ring size is not in any source**:
centreline radius 90 mm and wire radius 2.0 mm are this task's estimates, chosen so the wire fits the
measured hole (`hole_radius` 0.023 x width = 2.76 mm) and the bottom arc holds 21 plates x 2.5 mm.
The plate outline 40 x 12 cm comes from the Leiden thickness reasoning, not from a measured plate.

## T5a - leaf, bundle and thread

T3b (the 3D leaf gate) is blocked, so the bundle is shown on its own demo page instead of on the T3 page.

| file | what |
|---|---|
| `bundle.html` | demo page: the bundle simulated by `physics/scenes.js` `bundle()`, drawn as a 2D side view (vertical scale x6). Plain scripts, works from `file://`. Buttons are keyboard reachable; with `prefers-reduced-motion` the bundle is stepped to rest once and nothing moves. |
| `bench_page.mjs` | cost of one fixed step **inside the browser**, at 1x and 4x CPU throttle. `eval/perf.mjs` measures frame times, but headless Chrome runs rAF with vsync off (hundreds of fps), so most frames take no fixed step and the frame percentiles understate a 60 Hz frame. |
| `bench_page.json` | last measurement on this machine. |

## Scenes added to `physics/scenes.js`

- `bundle(opts)` - leaves between two cover boards, cord through both holes, knot on top. Sizes from the
  spec cards (palm_leaf and wood_board geometry, Glasgow handlist). Thicknesses, densities and the cord
  mass are spec **estimates** (`estimate: true`), so every number below is only as good as those.
- `thread(opts)` - a slack cord pinned at both ends; at rest it must lie on the analytic catenary of its
  own length. Sag depends only on length/span, so this cannot be passed by luck with the wrong gravity.
- `addBoard(w, opts)` - a rigid cover board on the same grid as a leaf, so `layer()` can pair board and
  leaf faces index for index.

`pbd.js` `slab()` gained two options, both off by default so the T2 results stay bit-identical:
`only` (the slab acts on these particles only - the cord, not the leaves inside it) and `bore` (a
particle stuck in a deep bore leaves sideways into the nearest hole when that is the shorter way out;
without it a particle in an 11 mm deep bundle was teleported to a face and threw the cord off).

## Numbers (this machine, `node --version` in `bench_page.json`)

| what | reference (12 leaves, nx 13, 664 particles) | page (8 leaves, nx 11, 436 particles) |
|---|---|---|
| ms per fixed step, Node | 2.78 | 1.07 |
| ms per fixed step, browser 1x | - | 1.20 |
| ms per fixed step, browser 4x throttle | - | 4.87 |
| ms per 60 Hz frame at 4x throttle (2 steps) | ~25 (estimated, over the 20 ms target) | 9.74 |
| `eval/perf.mjs` p95 frame, 4x throttle | - | 6 ms (target 20 ms) |

That is why the page runs the lighter configuration; the tests run both, and both pass every row.

## What is not modelled

The cord sees the closed bundle as one slab with two holes, not leaf by leaf. So the cord cannot carry
the bundle's weight, and cord stretch and knot slip are **not** under load in this scene - they are
tested under load by the T2 `cordThroughStack` scene (2 g tag) and by `thread`. See `DECISIONS.md` (T5a).

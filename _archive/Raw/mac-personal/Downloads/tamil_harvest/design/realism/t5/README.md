# T5 - leaf bundle and thread (T5a), copper plate set on its ring (T5b), coin, ring and seal (T5c), sherd and stone (T5d)

## T5d - sherd and stone

| file | what |
|---|---|
| `sherd_stone.html` | demo page, two panels. Top: two pottery sherds (8 cm and 4 cm) dropped on a table, simulated by `physics/scenes.js` `sherd()`, drawn as side-view silhouettes (convex hull of the body's own particles). Bottom: the site-scale stone slab, which you can drag with the pointer, kick with the arrow keys (the canvas is focusable) or resize the window around - the readout shows the largest distance any corner has moved. Plain scripts, works from `file://`; with `prefers-reduced-motion` the sherds are stepped to rest once and nothing moves. |
| `bench_sherd_stone.mjs` / `bench_sherd_stone.json` | cost of one fixed step (both sherds + the slab) inside the browser, 1x and 4x CPU throttle. |

- `sherd(opts)` - an irregular rigid piece: a closed outline whose radius wobbles by `rough` around a
  mean (three fixed harmonics, so it is lumpy, convex-ish and deterministic), two faces at `+-t/2`, and
  `rings` of interior face points so the piece cannot sink through the table between rim points. It is
  dropped with a tilt and a spin through the shared `dropBody()` helper and made rigid with `shape()`;
  `rigid: 0` turns shape matching off (a known-bad piece that deforms), `table: false` removes the table.
  Mass comes from the shoelace area of the outline x thickness x density, not from a circle formula.
- `stone(opts)` - unchanged as a scene (still static, still 8 corner particles), but `meta` now carries
  `size`, `mass`, `drag(world, i, target)` (a soft pin, the pointer) and `kick(world, v)` (a velocity
  kick, an arrow key), so the page and the test drive the slab through exactly the same three inputs.

### Numbers (this machine)

| what | two sherds + the slab (172 particles) |
|---|---|
| ms per fixed step, browser 1x | 0.138 |
| ms per fixed step, browser 4x throttle | 0.584 |
| ms per 60 Hz frame at 4x throttle (2 steps) | 1.17 (target 20) |
| `eval/perf.mjs` p95 frame, 4x throttle | 2.1 ms (target 20 ms); page JS 22.7 kB gzip |

Test rows (`eval/physics_tests.mjs`, `sherd_*` and `stone_never_moves`). Three reference sherds (8 cm,
a second outline at phase 2.0, and a 4 cm piece): each touches the table at **0.13-0.14 s**, starts
with kinetic energy (3.4e-4 to 2.9e-3 J), its peak kinetic energy per 0.5 s window never grows after
the first impact, it settles (max speed < 0.002 m/s) at **0.25-0.34 s** against a 6 s target, its worst
neighbour-distance change is **0.0055 / 0.0060 / 0.0180** of the piece's longest diagonal (target 0.02
- the 4 cm piece is the tight one, because the same absolute wobble is a bigger fraction of a smaller
diagonal), it rests at **0.00 deg** from flat (limit 3 deg) and no particle of either face ever goes
below **0.0000 mm**. Known-bad sherds fail the row they should: shape matching off (0.456 shape change),
perfectly bouncy + frictionless + undamped (never settles), no table (falls 118 m in 8 s).
The stone measures **0 m of movement** (exactly zero, `stone_max_movement_m` = 0) in three reference
scenes: the T2 block under 5 kN pushes, the site-scale slab (1.2 x 2.4 x 0.35 m, 2268 kg) under 5 kN
pushes, and the same slab under a 0.5 m pointer drag + four 2 m/s arrow-key kicks + six viewport
resizes each with a 5 kN push. A free-body slab under the same inputs moves 32.6 m and fails. Sherd and
slab both give an identical hash on 3 runs and at 30/60/144 fps and jittered frames.

### Unverified

**The sherd's size is an estimate.** No pottery photo in the reference set carried a scale bar
(`specs/pottery.json` `units_note`), so the 8 cm long axis is this task's estimate; what *is* measured
is the outline wobble (`edge_roughness` 0.192) and the long/short ratio (`aspect_ratio_photos` 1.571).
Thickness 7 mm, density 1.9 g/cm3, friction 0.5 and restitution 0.2 are all `estimate: true` spec
values. The two new targets (`sherd_rest_tilt_deg_max` 3.0, `sherd_rigid_shape_change_max` 0.02) were
added by this task, not by a source - see `DECISIONS.md` (T5d). A real sherd is curved (it was part of
a pot) and can rest on its curve at a large tilt; this piece has **parallel flat faces**, so it always
lies flat, and the rest-tilt row only proves it is not tumbling. The slab is not a measured inscription
stone: 1.2 x 2.4 x 0.35 m is a site-scale figure chosen for the page.


## T5c - coin, ring and seal

| file | what |
|---|---|
| `small_objects.html` | demo page: three small rigid bodies dropped on a table, simulated by `physics/scenes.js` `coin()`, `ring()` and `seal()`, drawn as three side-view panels (silhouette = convex hull of the body's own particles, particles drawn on top). Plain scripts, works from `file://`. Buttons are keyboard reachable; with `prefers-reduced-motion` the three objects are stepped to rest once and nothing moves. |
| `bench_small_objects.mjs` / `bench_small_objects.json` | cost of one fixed step (all three scenes together) inside the browser, 1x and 4x CPU throttle. |

Scenes added to `physics/scenes.js` (the T2 `coin()` is **unchanged**, so its state hashes stay
bit-identical; `ring` and `seal` share a new `dropBody()` helper that places a particle shell, tilts it,
drops it with a spin and makes it rigid with `shape()`):

- `ring(opts)` - a closed torus of 16 sectors; each sector's cross-section is a diamond of four points
  (outer, inner, top, bottom) at the wire radius, so a flat-lying ring touches the table exactly one
  wire radius below the centreline. `lump` lowers one sector (a known-bad ring that cannot lie flat).
- `seal(opts)` - a thick disc (or square, `square: true`) with `reliefN` raised bosses on one face.
  `reliefDown` drops it relief face down so it rests on the bosses; `reliefBias` raises one boss (a
  known-bad seal). `table: false` removes the table (a known-bad body that falls for ever).
- `axisFrom(...)` - the body's own axis, from the mean of its upper particles minus the mean of its
  lower ones; the rest tilt is the angle between that axis and vertical.

### Numbers (this machine)

| what | coin + ring + seal together (130 particles) |
|---|---|
| ms per fixed step, browser 1x | 0.124 |
| ms per fixed step, browser 4x throttle | 0.482 |
| ms per 60 Hz frame at 4x throttle (2 steps) | 0.96 (target 20) |
| `eval/perf.mjs` p95 frame, 4x throttle | 1.6 ms (target 20 ms); page JS 20.4 kB gzip |

Test rows (`eval/physics_tests.mjs`, `coin_*`, `ring_*`, `seal_*`), reference scenes (ring; seal relief
up, relief down and square): each one touches the table at **0.108 s**, starts with kinetic energy
(2.8e-4 J ring, 1.6e-3 J seal), its peak kinetic energy per 0.5 s window never grows after the first
impact, it settles (max speed < 0.002 m/s) at **0.27 s** (ring), **0.18 s** (seal relief up and square)
and **0.38 s** (seal relief down) against a 6 s target, rests at **0.00 deg** from flat (limit 3 deg)
and never goes below **0.0000 mm**; identical hash on 3 runs and at 30/60/144 fps and jittered frames.
Known-bad scenes fail the row they should: perfectly bouncy + frictionless + undamped (never settles),
a 2 mm lump on one ring sector (rests 5.20 deg), one seal boss 1.5 mm proud (rests 4.73 deg), and no
table at all (falls 118 m in 8 s, never settles).

### Unverified

**Every size here is an estimate.** `specs/rings.json` gives only inner diameter 18 mm, mass 8 g and
restitution 0.4, `specs/seals.json` only diameter 30 mm and density 8.7 g/cm3 - all `estimate: true`,
none from a measured object. The ring **wire radius 2.0 mm** is derived from the spec bore and mass
(bronze 8700 kg/m3 gives 7.6 g against the spec 8 g); the seal **thickness 8 mm** (49 g) and **relief
height 0.5 mm**, and the 16-sector / 6-boss discretisation, are this task's estimates. The two rest-tilt
targets (3 deg each) were added by this task from the coin's geometric argument, not from a source.
Friction and restitution are spec estimates too, and this solver's rigid bodies settle in a fraction of
a second: a real coin spun hard rings and wobbles for seconds, which is **not** modelled.


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

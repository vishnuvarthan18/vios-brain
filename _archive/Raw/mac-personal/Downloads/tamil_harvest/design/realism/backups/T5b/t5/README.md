# T5a - leaf, bundle and thread

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

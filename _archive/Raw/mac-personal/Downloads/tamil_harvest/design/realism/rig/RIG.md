# Fixed test rig (T0)

Same rig for every render so scores are comparable across tasks and nights.

- Browser: Chrome for Testing 151.0.7922.34 from the ms-playwright cache (chromium-1234), headless, driven by
  playwright-core 1.63.0. Pinned on purpose: system Chrome auto-updates. Fallback: system Chrome (logged if used).
- Viewports: desktop 1200x800 and mobile 360x740 CSS px, device pixel ratio 2 (2400x1600 and 720x1480 device px).
- Seeds: `Math.random` replaced by mulberry32(20260926) before any page script; shader noise uses an integer
  hash with `params.seed`. `Date` fixed at 2026-01-01T00:00:00Z (Playwright clock).
- GPU: `visual` profile uses SwiftShader (`--use-angle=swiftshader`) so pixels never depend on the Mac's GPU.
  `perf` profile uses the real GPU with vsync and frame-rate limit off.
- Colour: `--force-color-profile=srgb`, no LCD text, no font hinting. Pixels are read with `canvas.toDataURL`
  (the canvas's own RGBA, no page compositing).
- Camera: orthographic, object fitted with 6% margin, long side horizontal. Object width = 1 unit.
- Light: one directional light, azimuth 135 deg (top-left), elevation 45 deg, intensity 1, ambient 0.35.
  Shading is normalised so a flat patch under this light shows its albedo exactly.
  Optional lamp: warm point light (1.0, 0.72, 0.42) at (0.2, 0.75, 0.35) object units, flicker is a multiplier.
- Self-test (`node design/realism/eval/t0_selftest.mjs`): grey card (desktop + mobile) and a carved textured tile,
  each rendered in two separate browser launches: pixel hashes and all measured scores identical, grey card
  measures L* 53.585, a* = b* = 0, local contrast 0. Result: `eval/self_test/t0_selftest.json`.

## Wear and age (T7)

One parameter ages every material: `wear.age` 0..1 in `window.__render(params)`.

- `age` drives all four effects at once: crack density (lines that widen and spread), stain area
  (blotches that grow), edge loss (the outline eaten back, up to 2% of the long side at age 1) and
  colour darkening (up to 18% of L* at age 1, plus rougher and less metallic with age).
- `wear.edge_loss`, `wear.stain`, `wear.crack` (default 1, range 0..2) are WEIGHTS on how hard age
  drives that channel, not amounts. **`age` 0 renders exactly the unworn material** whatever the
  weights are (checked by pixel hash in `eval/wear_tests.mjs`).
- Every mask is nested in age: the noise field per pixel is fixed and only the threshold or the line
  width moves, so wear can never go backwards. `eval/wear_tests.mjs` checks that over 11 ages, and a
  deliberately non-monotonic age curve must fail it.
- Two extra render modes for measuring: `mode: 'crackmask'` and `mode: 'stainmask'`.
- The wear AMOUNTS (crack frequency 22 per unit, line half-width 0.05 at age 1, stain frequency 2.3,
  2% edge loss, 18% darkening) are **unverified estimates**: no source measures crack density or
  stain area on these objects. See `DECISIONS.md` (T7).
- Sheet: `node design/realism/sheets/wear_sheet.mjs` -> `sheets/wear.png` (7 materials x ages 0, 0.25,
  0.5, 0.75, 1). `sheets/*.png` is gitignored.

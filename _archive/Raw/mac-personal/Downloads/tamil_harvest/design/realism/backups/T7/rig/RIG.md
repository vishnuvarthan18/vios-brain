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

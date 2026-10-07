# T3a - Palm leaf in real 3D: build (Prompt 4 section 7)

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Owner text (section 7): "Build the palm leaf in WebGL (Three.js or the best measured alternative; measure bundle size
and license first) with: physically based material (albedo, normal, roughness, from procedural noise fitted to the spec
card and, where allowed, from SHIP photo detail), leaf as a bending mesh with mass-spring or Verlet physics, two holes
with a cord, cotton thread as a rope, stack of leaves with contact, one lamp light with warm flicker. Keep the old 2D
leaf as the fallback. Bundle everything into one script so the site still opens by double-click."

Do:
1. Measure options before choosing: (a) Three.js (`npm install --prefix design/realism three@<exact>`; record license +
   minified and gzip size of what you would ship), (b) plain WebGL2 reusing `rig/stage.js` shading (size of the code).
   Log the choice with the sizes in DECISIONS.md. The whole added JS must stay under 300 KB gzip (gate in T3b).
2. Build `design/realism/t3/leaf3d.html` + ONE classic (non-module) bundled script `design/realism/t3/leaf3d.bundle.js`
   so it opens from file:// by double-click. No network, no CDN. Leaf size/aspect/holes from `specs/palm_leaf.json`
   (geometry: aspect_ratio_catalogue, hole_distance_from_nearer_end, hole_radius; physical: thickness, E, ratio,
   max curvature, damping). Physics: `physics/pbd.js` + `physics/scenes.js` (leaf, cordThroughStack, stack) - they
   pass the T2 tests; use `FixedStepper` (fixed 1/120 s). Material: the stage's procedural model (Lab ramp from
   spec colour p5/p50/p95, grain along the length, fibre streaks, normal from height) so T4 tuning transfers.
   Lamp: one warm point light with deterministic flicker (seeded). Reduced motion: no flicker, no idle motion.
3. Fallback: when WebGL is missing or `?quality=low`, show the existing 2D leaf (reuse website_live CSS by relative
   link, do not copy reference photos). Keyboard: the leaf can be turned/flipped with keys. Tamil text stays text.
4. For the judge: `?static=1` renders one flat leaf, top view, transparent background, no animation, and exposes
   `window.__render(params)` with the rig stage schema (light azimuth/elevation, letters carved/painted) so
   `eval/lighting.mjs --url` works. Save a render to `design/realism/eval/out/t3/leaf.png` with the rig
   (`rig/rig.mjs` helpers) and register in `eval/candidates.json`: visual (palm_leaf, "T3 3D leaf"), lighting,
   pages (page + fallback selector).
5. Do not touch website_live yet (T3b decides). Commit.

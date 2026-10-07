# T4-palm_leaf - Auto-tune the palm_leaf material (Prompt 4 section 8, T4)

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Owner text: "T4 Auto-tune loop: for each material, a script searches the material parameters (noise scales, colour
ramps, grain, wear, roughness) to lower the visual score against the spec card, with at most 200 iterations or 30
minutes per material; keep the best set, log every trial in eval/trials/<material>.csv. After each material, render a
side-by-side sheet (real reference crops next to your render, same scale) at design/realism/sheets/<material>.png,
look at it yourself, and write the 5 biggest visible differences in sheets/<material>.md. Your own judgement is not
proof; the numbers decide, the sheet helps."

This task: material `palm_leaf` only. First material: also WRITE the tuner `design/realism/eval/tune.mjs` (reusable for every material): it renders the rig stage (`rig/rig.mjs`, stage params schema) in the spec's shape (spec `stage_hint.shape`, aspect from geometry) with a transparent background, scores with `eval/visual.py`, and searches colour ramp (p5/p50/p95 Lab, contrast), noise (scale, octaves, gain), grain (angle, stretch, fibre, density), speckle, height/bump, roughness, metal with a seeded random search + coordinate refinement. Loss = sum of each visual row's distance to its target divided by the target (failed rows count more). Deterministic seed. Then run it for this material.
- Limits: at most 200 trials or 30 minutes. Log every trial (params hash, all row values, loss) to
  `design/realism/eval/trials/palm_leaf.csv`. Keep the best params in `design/realism/materials/palm_leaf.params.json`.
- Render the best to `design/realism/eval/out/tuned/palm_leaf.png` and register it in `eval/candidates.json`
  (visual, label "T4 tuned stage"). If the material has letters/relief (stone, copper_plate, coins, seals,
  palm_leaf), also register a lighting candidate with the best params + carved letters.
- Sheet: `design/realism/sheets/palm_leaf.png`: 6 real reference crops (plain patches of the spec's photos, any tier is fine
  because sheets are never shipped and are git-ignored) next to 6 crops of the render, same scale (the spec's
  measure_norm). Look at it (Read the png) and write `design/realism/sheets/palm_leaf.md`: the 5 biggest visible differences,
  and the before (2D baseline) vs after numbers.
- Report the real-photo pass rate next to any target the tuned render misses.
- Commit (params, trials csv, sheet md, candidates.json; the sheet png stays out of git).

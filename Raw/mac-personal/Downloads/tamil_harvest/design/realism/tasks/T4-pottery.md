# T4-pottery - Auto-tune the pottery material (Prompt 4 section 8, T4)

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Owner text: "T4 Auto-tune loop: for each material, a script searches the material parameters (noise scales, colour
ramps, grain, wear, roughness) to lower the visual score against the spec card, with at most 200 iterations or 30
minutes per material; keep the best set, log every trial in eval/trials/<material>.csv. After each material, render a
side-by-side sheet (real reference crops next to your render, same scale) at design/realism/sheets/<material>.png,
look at it yourself, and write the 5 biggest visible differences in sheets/<material>.md. Your own judgement is not
proof; the numbers decide, the sheet helps."

This task: material `pottery` only. Use `design/realism/eval/tune.mjs` (written in T4-palm_leaf). If it is missing because that task is blocked, write it as described in `tasks/T4-palm_leaf.md`.
- Limits: at most 200 trials or 30 minutes. Log every trial (params hash, all row values, loss) to
  `design/realism/eval/trials/pottery.csv`. Keep the best params in `design/realism/materials/pottery.params.json`.
- Render the best to `design/realism/eval/out/tuned/pottery.png` and register it in `eval/candidates.json`
  (visual, label "T4 tuned stage"). If the material has letters/relief (stone, copper_plate, coins, seals,
  palm_leaf), also register a lighting candidate with the best params + carved letters.
- Sheet: `design/realism/sheets/pottery.png`: 6 real reference crops (plain patches of the spec's photos, any tier is fine
  because sheets are never shipped and are git-ignored) next to 6 crops of the render, same scale (the spec's
  measure_norm). Look at it (Read the png) and write `design/realism/sheets/pottery.md`: the 5 biggest visible differences,
  and the before (2D baseline) vs after numbers.
- Report the real-photo pass rate next to any target the tuned render misses.
- Commit (params, trials csv, sheet md, candidates.json; the sheet png stays out of git).

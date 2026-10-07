# T3b - Palm leaf 3D: automatic gate (Prompt 4 section 7)

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Owner text: "Automatic gate (no human): adopt the 3D path for that object only if ALL are true: visual scores pass
targets, physics tests pass, JS added under 300 KB gzipped, p95 frame time under 20 ms at 4x throttle, no console
errors, fallback works. Otherwise keep 2D and improve its shading instead. Write the gate result and numbers in
DECISIONS.md. Do the same gate for each later object."

Do:
1. `node design/realism/eval/run_all.mjs`; read the T3 rows in `eval/scoreboard.md` (visual, lighting, perf, compat)
   and `eval/out/physics.json`. JS added = gzip size of the scripts the 3D leaf adds (perf.json js list).
2. Write `design/realism/eval/gate.py` (or .mjs) that reads scoreboard.json and prints the gate table for an object
   (reuse for later objects), and a row per condition: value, target, pass. Save `design/realism/eval/gates/palm_leaf.json`.
3. Log the gate in DECISIONS.md: `rs.py decide T3b "adopt 3D leaf | keep 2D" "<choice>" "<all numbers>"`.
   Note honestly if a visual target is one real photos rarely pass (real-photo pass rates) - the rule still applies.
4. If the gate FAILS: keep 2D and improve the 2D leaf's shading in `website_live/css/materials.css` (back it up first),
   re-score the 2D baseline (capture_site + visual.py), and show before/after numbers. If it PASSES: prepare the
   integration (the styleguide rebuild is T9) and note it.
5. Commit.

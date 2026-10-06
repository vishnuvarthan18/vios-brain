# T9b - Integrate the T3 gate and T4 tuned materials into the site (follow-up to T9)

Read `design/realism/AGENT_BRIEF.md` first (rules, tools, start/finish steps).

Why this task exists: T9 ran while T3a/T3b/T4-* were wrongly blocked (a supervisor bug read claude.ai session limits
as crashes; fixed and requeued on 2026-09-26). T9 therefore kept the leaf, sherd, coin, ring and seal 2D and used the
T6 cards for stone and copper (see T9's notes and DECISIONS.md). Now T3b (gate) and T4 (tuned params) have run.

Do:
1. For every object run the gate from T3b (`eval/gate.*`, results in `eval/gates/<object>.json`) against the best
   candidate (T3 3D leaf, T4 tuned stage, T6 light pages). Owner rule (section 7): adopt the 3D/tuned path for an
   object only if ALL gate conditions pass; otherwise keep 2D and improve its shading (use the tuned colours).
2. Update the site the way T9 did it: regenerate `website_live/js/realism/materials.params.js` with T9's build scripts
   (`--check`), update the styleguide `#materials-3d` section and its gate table, keep every old section. Back up
   every file first. Keep file://, reduced motion, keyboard and contrast rules.
3. Update the "Measured realism" section of `website_live/DESIGN_SYSTEM.md` with the new numbers.
4. Checks: `node design/realism/t9/checks.mjs`, `node design/realism/tools/run_site_check.mjs website_live/_checks/checks_p2.mjs styleguide.html`,
   then `node design/realism/eval/run_all.mjs`. Nothing may get worse than T9's numbers; if something does, revert it.
5. Commit.

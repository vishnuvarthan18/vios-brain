**Vishnu** (2026-09-26T05:47): You are an unattended run of the Realism Engine supervisor. Nobody will answer questions: never ask, decide safely and log it.
Task: T5d - Physics: sherd and stone. Started by the supervisor at 2026-09-26 11:17:22 IST. Hard time cap 90 minutes (killed at 95).
First read design/realism/AGENT_BRIEF.md and design/realism/tasks/T5d.md, then do only this task and finish as the brief says.
Queue entry now: status running, depends_on {"T5a": "done"}, crashes before this run 0.
Notes from earlier runs (resume from them): none

**Claude** (2026-09-26T05:47): I'll start by reading the brief and task file.

**Claude** (2026-09-26T05:47): _[tool: Read]_

**Claude** (2026-09-26T05:47): _[tool: Read]_

**Claude** (2026-09-26T05:47): _[tool: Bash]_

**Claude** (2026-09-26T05:47): _[tool: Read]_

**Claude** (2026-09-26T05:47): _[tool: Bash]_

**Claude** (2026-09-26T05:47): _[tool: Read]_

**Claude** (2026-09-26T05:47): _[tool: Bash]_

**Claude** (2026-09-26T05:47): _[tool: Bash]_

**Claude** (2026-09-26T05:47): Now the tests file and targets.

**Claude** (2026-09-26T05:47): _[tool: Read]_

**Claude** (2026-09-26T05:47): _[tool: Bash]_

**Claude** (2026-09-26T05:47): _[tool: Read]_

**Claude** (2026-09-26T05:47): _[tool: Read]_

**Claude** (2026-09-26T05:48): _[tool: Bash]_

**Claude** (2026-09-26T05:48): _[tool: Bash]_

**Claude** (2026-09-26T05:48): Now the sherd scene and the extended stone.

**Claude** (2026-09-26T05:48): _[tool: Edit]_

**Claude** (2026-09-26T05:48): _[tool: Edit]_

**Claude** (2026-09-26T05:49): _[tool: Bash]_

**Claude** (2026-09-26T05:49): _[tool: Bash]_

**Claude** (2026-09-26T05:49): _[tool: Edit]_

**Claude** (2026-09-26T05:49): Now the tests.

**Claude** (2026-09-26T05:49): _[tool: Edit]_

**Claude** (2026-09-26T05:49): Now the POT spec constants and the run section.

**Claude** (2026-09-26T05:49): _[tool: Edit]_

**Claude** (2026-09-26T05:50): _[tool: Edit]_

**Claude** (2026-09-26T05:50): Now add sherd + stone to the determinism / frame-rate list.

**Claude** (2026-09-26T05:50): _[tool: Edit]_

**Claude** (2026-09-26T05:50): _[tool: Bash]_

**Claude** (2026-09-26T05:50): _[tool: Bash]_

**Claude** (2026-09-26T05:51): _[tool: Bash]_

**Claude** (2026-09-26T05:52): _[tool: Write]_

**Claude** (2026-09-26T05:52): _[tool: Edit]_

**Claude** (2026-09-26T05:52): Now the bench script and candidates registration.

**Claude** (2026-09-26T05:52): _[tool: Write]_

**Claude** (2026-09-26T05:52): _[tool: Edit]_

**Claude** (2026-09-26T05:52): _[tool: Bash]_

**Claude** (2026-09-26T05:53): _[tool: Bash]_

**Claude** (2026-09-26T05:56): _[tool: Bash]_

**Claude** (2026-09-26T05:56): _[tool: Bash]_

**Claude** (2026-09-26T05:56): _[tool: Bash]_

**Claude** (2026-09-26T05:56): _[tool: Bash]_

**Claude** (2026-09-26T05:56): Now the README section and perf/gzip figure.

**Claude** (2026-09-26T05:56): _[tool: Bash]_

**Claude** (2026-09-26T05:56): _[tool: Bash]_

**Claude** (2026-09-26T05:57): _[tool: Edit]_

**Claude** (2026-09-26T05:57): _[tool: Bash]_

**Claude** (2026-09-26T05:57): _[tool: Bash]_

**Claude** (2026-09-26T05:57): _[tool: Bash]_

**Claude** (2026-09-26T05:57): _[tool: Bash]_

**Claude** (2026-09-26T05:57): _[tool: Bash]_

**Claude** (2026-09-26T05:57): T5d done in 10 minutes, commit `80240ee`.

- Sherd (`sherd()` in `physics/scenes.js`): 3 reference pieces fall at 0.13–0.14 s, settle at 0.25–0.34 s (target 6), rest 0.00° from flat (limit 3), never go below 0.0000 mm, worst shape change 0.0055 / 0.0060 / 0.0180 of the diagonal against the new 0.02 target — the 4 cm piece is the tight one. Known-bads fail the right rows: shape matching off 0.456, bouncy+frictionless never settles, no table falls 118 m.
- Stone: site-scale slab 1.2 × 2.4 × 0.35 m (2268 kg) moves **0 m** under 5 kN pushes and under a 0.5 m pointer drag + 4 arrow-key kicks + 6 viewport resizes; a free-body slab under the same inputs moves 32.6 m and fails. Sherd and slab both hash identically on 3 runs and at 30/60/144 fps and jittered frames. `node eval/physics_tests.mjs`: ALL AS EXPECTED.
- Page `t5/sherd_stone.html`: p95 2.1 ms at 4× throttle (target 20), 0.584 ms/step, 22.7 kB gzip JS, all compat rows PASS (file://, reduced motion, no-JS fallback). Scoreboard: 32 of 219 rows fail, all the pre-existing 2D-site-baseline rows; 35 new rows all pass.
- Unverified: the sherd's 8 cm long axis and the slab's size are this task's estimates (no pottery photo had a scale bar); thickness 7 mm, density 1.9, friction 0.5, restitution 0.2 are spec estimates; both new targets were added by this task, logged in DECISIONS.md. The sherd has parallel flat faces so it always lies flat — a real curved sherd resting tilted is not modelled.
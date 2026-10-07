# PROMPT 4 — Realism Engine + autonomous long run (owner is away; nobody will answer questions)

## 0. Goal and honest target
Build a "Realism Engine" for the Tamil site: a measured pipeline that makes each ancient writing surface (palm leaf, stone, copper plate, pottery, coin, ring, seal, thread) match the real object in LOOK, PHYSICS and FEEL, and proves it with numbers. "100% real" is not possible in a browser. The target is: statistically close to real reference photos, physically correct in motion, smooth on a phone, and every claim backed by a test. Never say "real" without a score next to it.

Theme: ancient Tamil (Sangam to Chola). The site is meant for an international-level audience. Content facts stay flagged "unverified" until a Tamil speaker checks them.

## 1. Rules you must follow (no exceptions)
- Nobody will answer questions. Never ask. When you must choose, pick the safer or more reversible option, write it in `design/realism/DECISIONS.md` (date, task, options, choice, reason) and go on.
- Work only inside `~/Downloads/tamil_harvest`. Branch `realism-engine` (create from `redesign-design-system`). One commit per finished task. Never push, never publish, never deploy.
- Never delete, overwrite or move existing files. Do not touch `design/references/` originals, `design/reference_engine/refs.db`, `website_live_backup_2026-09-25/`, or the 7 old pages. Read-only on those. Back up any file before you edit it.
- Reference photos are for MEASURING only. Never copy a reference photo (any tier) into `website_live/`. Only SHIP-tier files (public domain, CC0, CC BY) may be used or turned into textures, with a credit in `credits-data.json`. CC BY-SA, CC BY-NC and unclear = never ship, including textures derived from them.
- Only install packages inside the project (`design/realism/node_modules`, `design/realism/.venv`). Check each package's license and write it in `design/realism/LICENSES_TOOLS.md`. No paid or account-based services.
- Disk guard: stop all work if free disk is under 20 GB. Do not start a second copy of any download job. Do not stop the harvester if it is running.
- No sound file, font, image or library without a recorded license. If unclear, skip it and log it.
- Time cap per task: 90 minutes. If a task is not done by then, save state, mark it `blocked` with the reason, and go to the next task. Never loop on the same failure more than 3 times.
- If a test gets worse after your change, revert that change (`git revert`), log it, and move on.
- Keep Tamil text as real text. Keep reduced-motion, keyboard, contrast (4.5:1 body) and file:// (double-click) rules from Prompts 1 and 2.

## 2. State, resume and heartbeat (so a crash never loses work)
Create `design/realism/state/`:
- `queue.json`: list of tasks with id, title, status (pending, running, done, blocked, skipped), depends_on, started_at, finished_at, notes, commit hash.
- `heartbeat.txt`: rewrite every 60 seconds with time, current task, step. 
- `log.md`: append-only, one line per event.
- `REPORT.md`: rewrite after every task (see section 9).
- `STOP`: if this file exists, finish the current step, write REPORT.md, and stop.
Every task must be resumable: on start, read `queue.json`, pick the first `pending` (or `running` from a crashed run) whose dependencies are `done`.

## 3. Supervisor (runs for many hours without a person)
Write `design/realism/run_forever.sh`: a loop that, while `queue.json` has pending tasks and no `STOP` file exists, starts one headless agent run for ONE task, waits, logs the exit code, sleeps 30 s, repeats. Rules:
- Wrap with `caffeinate -ims` so the Mac does not sleep. Log every restart. Stop after 5 crashes in a row on the same task and mark it blocked.
- Check `claude --help` yourself for the correct headless (non-interactive) flags and permission options of the installed version. Do not guess flags. Write what you found in `design/realism/RUNNING.md`.
- Permissions: choose the narrowest setup that still lets the tasks run unattended (allow file edits in the project folder and the named commands: node, npm, python3, git, playwright, ffmpeg if present). Do NOT use a mode that skips all permission checks unless the owner already configured it. If the runtime would wait for an approval, do not wait: mark the task `blocked: needs approval <command>`, and continue with other tasks.
- Print the exact one-line command to start it. Do not start the long run until section 4 (setup) and section 5 (evaluators) pass their self-tests. Then start it yourself if your shell allows; otherwise give me the command.

## 4. Task 0 — Setup and self-test
- Check tools: node, npm, python3, git, Chromium via Playwright (already installed with the earlier checks), ImageMagick or Pillow, numpy, scipy, scikit-image, opencv-python (install in the venv if missing). Record versions.
- Fixed test rig: headless Chrome, fixed viewport (1200x800 and 360x740), fixed device pixel ratio 2, fixed random seeds, fixed camera, one directional light (top-left) plus an optional lamp point light. Same rig for every render so scores are comparable.
- Self-test: render a plain grey card twice; scores must be identical. Log it.

## 5. Task 1 — Ground truth: material spec cards
For each material (palm_leaf, stone, copper_plate, pottery, coins, rings, seals, cotton_thread, wood_board), write `design/realism/specs/<material>.json` from the reference library (`design/references/<surface>/` and `LICENSES.csv`; at least 20 usable photos per material where they exist; say so where they don't). Measure from the photos:
- Colour: distribution in CIELAB (mean, standard deviation, 5th/50th/95th percentiles) of the object area (mask the background; crop plain regions, no text).
- Texture: power-spectrum slope, dominant grain direction and anisotropy (leaf fibre is strongly directional), grain scale in px per cm when a scale bar exists, else in relative units, local contrast, edge roughness.
- Geometry: aspect ratios, hole positions and sizes (leaf), plate proportions, thickness where visible, ranges not single numbers.
- Physical properties: thickness, mass, stiffness, friction, restitution, damping. Only from a named source (paper, museum note, conservation guide) with the source written in the file, or by a simple formula, else `"estimate": true` with the reasoning. Never invent a number and hide that it is invented.
- Each entry: `value`, `range`, `n_samples`, `source_ids`, `confidence` (high, medium, low), `estimate` (true or false).
Also write `specs/README.md` that lists all low-confidence numbers, so the owner knows what to check against real objects.

## 6. Task 2 — Evaluators (the "judge"; no human needed)
Write `design/realism/eval/` with scripts that output JSON and a scoreboard table.
Visual metrics (render vs spec card, statistics not pixel copying):
- Colour distance: ΔE2000 between render and reference Lab distributions (use earth-mover or percentile distance). Target: median ΔE under 6, and 95th percentile under 12. Report the real number.
- Texture: power-spectrum slope difference under 0.3; anisotropy difference under 0.15; local-contrast ratio within 0.8 to 1.25.
- Geometry: aspect ratio and hole positions inside the measured ranges.
- Lighting response: render at 5 light angles; carved letters must change in shading with angle (normal-map test): the shading difference between the lit and shadowed edge must be above a set threshold; a flat painted texture must FAIL this test.
- Optional (only if it installs cleanly and its license allows): a perceptual metric such as LPIPS. Do not rely on it alone.
Physics tests (deterministic, fixed time step 1/120 s, render frame rate must not change the result):
- Pendulum (copper plates on ring): measured period within 3% of 2π√(L/g) for small angles; amplitude decays monotonically; no energy gain.
- Leaf bend: maximum curvature stays under the material limit; it never folds; it returns to rest; bends more along the length than across the fibre.
- Thread and rope: total length changes under 2%; no stretching through the leaves; knot holds under gravity.
- Stack: leaves do not pass through each other or the boards; settle within 2 s after release.
- Coin: falls, spins, settles with decaying motion; tilt limit respected.
- Stone: zero movement under any input.
- Determinism: same input gives the same final state on 3 runs.
Performance tests (Chrome with 4x CPU throttle plus real frame timing where possible): p95 frame time, longest frame, memory, JS size, texture memory. Report honestly that headless Chrome is not a real phone.
Compatibility: no console errors; file:// works; reduced motion works; low-quality fallback works.
Output: `design/realism/eval/scoreboard.md` with one row per material and per test: value, target, pass or fail. Include a self-check that each test can FAIL (feed it a known-bad input and confirm it fails).

## 7. Task 3 — Spike and automatic gate: palm leaf in real 3D
Build the palm leaf in WebGL (Three.js or the best measured alternative; measure bundle size and license first) with: physically based material (albedo, normal, roughness, from procedural noise fitted to the spec card and, where allowed, from SHIP photo detail), leaf as a bending mesh with mass-spring or Verlet physics, two holes with a cord, cotton thread as a rope, stack of leaves with contact, one lamp light with warm flicker. Keep the old 2D leaf as the fallback. Bundle everything into one script so the site still opens by double-click.
Automatic gate (no human): adopt the 3D path for that object only if ALL are true: visual scores pass targets, physics tests pass, JS added under 300 KB gzipped, p95 frame time under 20 ms at 4x throttle, no console errors, fallback works. Otherwise keep 2D and improve its shading instead. Write the gate result and numbers in `DECISIONS.md`. Do the same gate for each later object.

## 8. Task queue (run in this order; add sub-tasks as needed)
T0 Setup and self-test. T1 Spec cards. T2 Evaluators. T3 Palm leaf spike and gate.
T4 Auto-tune loop: for each material, a script searches the material parameters (noise scales, colour ramps, grain, wear, roughness) to lower the visual score against the spec card, with at most 200 iterations or 30 minutes per material; keep the best set, log every trial in `eval/trials/<material>.csv`. After each material, render a side-by-side sheet (real reference crops next to your render, same scale) at `design/realism/sheets/<material>.png`, look at it yourself, and write the 5 biggest visible differences in `sheets/<material>.md`. Your own judgement is not proof; the numbers decide, the sheet helps.
T5 Physics per object with the tests above (leaf, bundle and thread, copper plate set on its ring, coin, sherd, ring, seal, stone).
T6 Lighting: raking light and oil-lamp light reveal carved letters on stone and copper; tests from section 6.
T7 Wear and age system: cracks, stains, edge loss controlled by one "age" parameter, tested for consistency.
T8 Optional sound layer, muted by default: leaf rustle, stylus scratch, copper chime, chisel. Use only CC0 audio with a recorded source or synthesize it yourself. If you cannot confirm a license, skip the task and log it.
T9 Rebuild `styleguide.html` sections on the chosen stack; keep the old versions side by side; update DESIGN_SYSTEM.md with all measured values and scores.
T10 Page components: navigation, footer, timeline, search, map placeholder, card layouts; dark mode tokens for all of them; contrast tables.
T11 Pages using the system with sample content clearly marked: home (bundle as navigation, intro), Chola section (temple stone wall by lamplight), Thirukkural leaf reader (one Kural per leaf, use 10 sample verses, marked as unverified text). Do not change the 7 old pages.
T12 Identity page Batches 2 and 3 (rules from `prompt-2b-tamil-identity-page.md`), every fact flagged unverified with a source or "unverified".
T13 Reference viewer (`design/reference_engine/viewer.html`) per `prompt-3-reference-engine.md`, no downloads.
T14 License cleanup report: list the 25 duplicate IDs, the STO-070/STO-071 clash, and bad author fields; propose fixes in `design/reference_engine/NEEDS_CLEANING.csv`; do not edit LICENSES.csv.
T15 Cross-browser checks that can be automated (Chromium and, if installed, Firefox and WebKit through Playwright), accessibility scan, keyboard, reduced motion, file:// load. Report what still needs a real phone and a real Safari.
T16 Final: full scoreboard, all commits listed, list of everything unverified.
Tasks that need network downloads of photos (extra sources) are NOT in this queue; a separate job handles them.

## 9. REPORT.md (rewrite after every task)
Top: status line (running or stopped, current task, time, tasks done of total, blocked count). Then: scoreboard table (material, visual score, physics tests passed of total, perf numbers), before-and-after sheet links, every DECISION made (one line), blocked tasks with the reason, low-confidence numbers, and a plain list "NOT verified": real phone, Safari, screen reader, real objects measured by hand, Tamil facts, licenses not confirmed. If any target is missed, say so in the first line.

## 10. Start
1. Create the branch and `design/realism/` tree. 2. Do T0, T1, T2. Run the evaluator self-tests. 3. Write `run_forever.sh` and `RUNNING.md`. 4. Fill `queue.json` with T3 to T16. 5. Start the supervisor. 6. Report the exact command you used, and how the owner stops it (create `design/realism/state/STOP`).

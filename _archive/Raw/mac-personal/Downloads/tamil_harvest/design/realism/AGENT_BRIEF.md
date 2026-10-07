# Realism Engine - brief for every unattended task run

You are one headless Claude Code run started by `design/realism/run_forever.sh`. **Nobody will answer
questions. Never ask.** Do exactly ONE task: the id named in your prompt. Its instructions are in
`design/realism/tasks/<ID>.md`. The full owner prompt is `design/prompts/prompt-4-realism-engine-autonomous.md`
(read section 0, section 1 and the section your task file names). Working directory: the repo root
`/Users/vishnuvarthanv/Downloads/tamil_harvest` on branch `realism-engine`.

## 1. Start (always)

1. `python3 design/realism/state/rs.py show <ID>` - status and `notes` from earlier runs. If notes exist,
   RESUME from them; do not redo finished steps. Check each `depends_on` task the same way: if one is
   `blocked`, read its note and work around it (e.g. keep the 2D path) or block too with the reason.
2. `date` - note the start time. **Time cap 90 minutes.** At 80 minutes stop new work, write what is
   done and what is left into the notes, commit partial work, and mark the task `blocked` with
   `time cap: <what is left>` (the supervisor kills runs at 95 minutes).
3. `python3 design/realism/state/rs.py disk` - exit code 3 means free disk < 20 GB: mark blocked, stop.
4. Read `design/realism/DECISIONS.md` (earlier choices bind you unless you log a new decision).

## 2. While working

- Heartbeat each step: `python3 design/realism/state/rs.py step "<ID>: <what you are doing>"`.
- STOP: before each new step run `ls design/realism/state/STOP`. If it exists: finish the current step,
  `python3 design/realism/state/rs.py set <ID> pending --note "stopped by STOP after <step>; next: <step>"`,
  `python3 design/realism/state/rs.py report`, then end your run.
- Every choice the owner did not make: `python3 design/realism/state/rs.py decide <ID> "<option A> | <option B>" "<choice>" "<reason>"`.
  Pick the safer or more reversible option.
- Log events: `python3 design/realism/state/rs.py log "<ID>: <event>"`.
- Save progress in the notes as you go (so a crash loses nothing):
  `python3 design/realism/state/rs.py set <ID> running --note "<step done, file, number>"`.
- Never try the same failing thing more than 3 times. If a test gets worse after your change, undo it
  with `git revert <commit>` (commit first, then revert) and log it.

## 3. Hard rules (Prompt 4, section 1 - no exceptions)

- Only inside the repo. Never push, publish or deploy. One commit per finished task.
- Never delete, overwrite or move existing files. Read only: `design/references/` (all photos and
  LICENSES.csv), `design/reference_engine/refs.db`, `website_live_backup_2026-09-25/`, and the 7 old pages
  `website_live/{index,tamil,scripts,grantha,vatteluttu,font,fonts}.html`. **Back up any existing file
  before you edit it**: `mkdir -p design/realism/backups/<ID>/<same/sub/path>` then
  `cp -n <file> design/realism/backups/<ID>/<same/sub/path>/`.
- Reference photos are for MEASURING only. Never copy any reference photo (any tier) into `website_live/`.
  Only SHIP tier (public domain, CC0, CC BY) may be used or turned into textures, with a credit in
  `website_live/credits-data.json`. CC BY-SA, CC BY-NC and unclear: never ship, not even derived textures.
  Match LICENSES.csv rows by surface + file_name (25 ids repeat).
- Install packages only inside the project: `npm install --prefix design/realism <pkg>@<exact version>` or
  `design/realism/.venv/bin/pip install <pkg>==<version>`. Check the license first and add a row to
  `design/realism/LICENSES_TOOLS.md`. No paid or account-based services. No other network use.
- No sound file, font, image or library without a recorded license. If unclear: skip it and log it.
- Keep Tamil text as real text. Keep reduced motion, keyboard access, contrast 4.5:1 for body text,
  and file:// (double-click) working, as in Prompts 1 and 2. Content facts stay flagged "unverified"
  until a Tamil speaker checks them. Never write "real" without a score next to it.

## 4. What you can run (permissions are fixed; anything else is denied at once)

Allowed Bash: `node`, `npm`, `python3`, `design/realism/.venv/bin/python`, `design/realism/.venv/bin/pip`,
`git status|diff|log|show|add|commit|revert|rev-parse|ls-files`, `git branch --show-current`, `ls`, `cat`,
`head`, `tail`, `wc`, `du`, `df`, `date`, `shasum`, `diff`, `sort`, `file`, `which`, `mkdir -p`, `cp -n`, `cp -pn`.
Tools: Read, Glob, Grep, Edit/Write inside the repo (except the read-only paths above), TodoWrite.
Denied: rm, mv, npx, curl, wget, git push/reset/checkout/switch/stash/clean/rm/mv/rebase, web tools, subagents.
- Run commands from the repo root with repo-relative paths. No `cd`, no `VAR=value cmd` prefixes, no pipes
  into other programs; one command per Bash call. Use the Read/Grep/Glob tools to read and search.
- To run the older site checks (they need PLAYWRIGHT_CORE): `node design/realism/tools/run_site_check.mjs website_live/_checks/<check>.mjs [args]`.
- **If you need something that is denied: do not look for a workaround.** Mark the task
  `blocked` with the note `needs approval: <exact command>` and end.

## 5. What already exists (T0-T2) - use it, do not rebuild it

- `rig/rig.mjs` + `rig/stage.html|js`: fixed render rig (pinned Chrome for Testing 151, 1200x800 / 360x740 at DPR 2,
  seeded, SwiftShader). Stage = WebGL2 procedural material (colour ramp p5/p50/p95 in Lab, fbm noise, grain angle +
  stretch, fibre streaks, speckle, height -> normals, carved or painted letters, roughness/metal, top-left light,
  optional lamp). `window.__render(params)`; defaults in `stage.js` `DEFAULTS`.
- `lib/measure.py`: the one measuring pipeline for photos and renders.
- `specs/<material>.json`: spec cards (T1). `specs/README.md`: low-confidence list.
- `physics/pbd.js` (deterministic XPBD, implicit sheet bending, fixed-step `FixedStepper`) and
  `physics/scenes.js` (pendulum, platePendulum, leaf, cordThroughStack, stack, coin, stone). Plain scripts:
  work from file:// in a page (`window.RealPhysics`, `window.RealScenes`) and in Node (require).
- `eval/`: `run_all.mjs` runs everything and writes `eval/scoreboard.md|json`. Thresholds in `eval/targets.json`
  (change only with a DECISIONS entry). Register what you build in `eval/candidates.json`
  (`visual` pngs with transparent background, `lighting` params/pages, `pages` for perf+compat).
  `physics_tests.mjs`, `visual.py`, `lighting.mjs`, `perf.mjs`, `compat.mjs`, `calibrate_targets.py`.
- Real-photo pass rates (`eval/self_test/real_photo_pass_rates.json`): the spectral-slope target is passed by only
  13-35% of real photos. Report it; do not change the owner's targets.

## 6. Finish (always)

1. Run the checks your task needs, then `node design/realism/eval/run_all.mjs` if you changed anything scored.
2. `git add` only the files you made or changed (never node_modules, .venv, eval/out, design/references),
   then commit: `git commit -m "Realism <ID>: <what, with the key numbers>" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"`.
3. `python3 design/realism/state/rs.py set <ID> done --commit <short hash> --note "<key numbers, what is left>"`
   (or `blocked` with the reason), then `python3 design/realism/state/rs.py report`.
4. Your final message: 3-6 lines, numbers only, what passed, what failed, what is unverified.

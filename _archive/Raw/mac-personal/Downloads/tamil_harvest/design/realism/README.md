# Realism Engine (Prompt 4)

Measured pipeline that compares the site's writing surfaces with real reference photos and proves
every claim with a number. Nothing here claims "real" without a score next to it.

| path | what |
|---|---|
| `rig/` | fixed test rig: `rig.mjs` (pinned Chrome for Testing 151, 1200x800 and 360x740 at DPR 2, seeded, fixed clock, SwiftShader) and `stage.html/js` (WebGL2 procedural material, fixed orthographic camera, top-left light, optional lamp) |
| `lib/measure.py` | one measuring pipeline for photos (T1) and renders (T2): masks, CIELAB stats, spectral slope, anisotropy, local contrast, geometry, holes, EMD CIEDE2000 |
| `specs/` | T1 spec cards, one JSON per material, plus `README.md` (low-confidence list) |
| `eval/` | T2 evaluators, self-tests, `scoreboard.md` |
| `physics/` | deterministic physics engine used by the physics tests |
| `state/` | `queue.json`, `log.md`, `heartbeat.txt`, `REPORT.md`, `STOP`; `rs.py` edits them |
| `run_forever.sh`, `RUNNING.md` | unattended supervisor and how it runs |
| `DECISIONS.md`, `LICENSES_TOOLS.md` | every choice made without the owner; every tool's license |

Set-up from scratch: `python3 -m venv design/realism/.venv && design/realism/.venv/bin/pip install -r design/realism/requirements.txt`
and `npm install --prefix design/realism`.

**Vishnu** (2026-09-26T20:16): You are an unattended run of the Realism Engine supervisor. Nobody will answer questions: never ask, decide safely and log it.
Task: T13 - Reference viewer (Prompt 4 section 8, T13). Started by the supervisor at 2026-09-27 01:46:20 IST. Hard time cap 90 minutes (killed at 95).
First read design/realism/AGENT_BRIEF.md and design/realism/tasks/T13.md, then do only this task and finish as the brief says.
Queue entry now: status running, depends_on {"T2": "done"}, crashes before this run 0.
Notes from earlier runs (resume from them): 5 failed runs in a row; last log T13-20260926-140555.jsonl | requeued 2026-09-26 18:30: the 5 'failed runs' were claude.ai session limits (HTTP 429, 'You've hit your session limit') that the supervisor misread as crashes (fixed); no work was lost in those runs

**Claude** (2026-09-26T20:16): I'll start by reading the brief and task file.

**Claude** (2026-09-26T20:16): _[tool: Read]_

**Claude** (2026-09-26T20:16): _[tool: Read]_

**Claude** (2026-09-26T20:16): _[tool: Bash]_

**Claude** (2026-09-26T20:16): _[tool: Bash]_

**Claude** (2026-09-26T20:16): _[tool: Bash]_

**Claude** (2026-09-26T20:16): _[tool: Bash]_

**Claude** (2026-09-26T20:16): _[tool: Read]_

**Claude** (2026-09-26T20:16): _[tool: Bash]_

**Claude** (2026-09-26T20:16): _[tool: Bash]_

**Claude** (2026-09-26T20:16): _[tool: Bash]_

**Claude** (2026-09-26T20:17): _[tool: Bash]_

**Claude** (2026-09-26T20:17): _[tool: Bash]_

**Claude** (2026-09-26T20:17): _[tool: Write]_

**Claude** (2026-09-26T20:17): _[tool: Bash]_

**Claude** (2026-09-26T20:17): _[tool: Bash]_

**Claude** (2026-09-26T20:17): _[tool: Bash]_

**Claude** (2026-09-26T20:17): _[tool: Bash]_

**Claude** (2026-09-26T20:20): _[tool: Write]_

**Claude** (2026-09-26T20:20): _[tool: Edit]_

**Claude** (2026-09-26T20:20): _[tool: Edit]_

**Claude** (2026-09-26T20:20): _[tool: Read]_

**Claude** (2026-09-26T20:20): _[tool: Bash]_

**Claude** (2026-09-26T20:20): Now a dedicated check for the viewer.

**Claude** (2026-09-26T20:21): _[tool: Write]_

**Claude** (2026-09-26T20:21): _[tool: Bash]_

**Claude** (2026-09-26T20:21): _[tool: Read]_

**Claude** (2026-09-26T20:21): _[tool: Edit]_

**Claude** (2026-09-26T20:21): _[tool: Bash]_

**Claude** (2026-09-26T20:21): _[tool: Bash]_

**Claude** (2026-09-26T20:22): _[tool: Write]_

**Claude** (2026-09-26T20:22): _[tool: Bash]_

**Claude** (2026-09-26T20:22): _[tool: Bash]_

**Claude** (2026-09-26T20:22): _[tool: Bash]_

**Claude** (2026-09-26T20:22): _[tool: Bash]_

**Claude** (2026-09-26T20:22): _[tool: Bash]_

**Claude** (2026-09-26T20:22): T13 done at f438222, 6 minutes of the 90-minute cap.

- `design/reference_engine/`: `viewer.html`, `export_viewer_data.py` (refs.db opened `?mode=ro`), `check_viewer.mjs`, `VIEWER_README.md` committed; `viewer-data.js` (2.56 MB), `check_viewer.json`, `viewer-grid.png` generated and left untracked.
- 2,412 rows exported, 0 image files missing, 896 SHIP / 1,516 STUDY; refs.db and LICENSES.csv sha256 unchanged, no photo copied — images load from `../references/…` by relative path.
- SHIP requires tier SHIP **and** a PD/CC0/no-restrictions/CC BY licence; 0 rows had to be downgraded; STUDY items show a "do not use on the website" panel and have no ship action.
- check_viewer.mjs 18/18 PASS: 0 console errors, 0 external requests, 0 failed requests, 0 broken images, first grid 250 ms, last page 27–39 ms, synthetic 4,000-row re-filter 403 ms (includes a 120 ms debounce + 400 ms wait).
- Unverified: every licence, date, script and Tamil reading is what the source API recorded in refs.db; no Tamil speaker has checked it, and no thumbnails exist because `design/references/` is read-only for this engine.
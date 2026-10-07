**Vishnu** (2026-09-26T22:08): You are an unattended run of the Realism Engine supervisor. Nobody will answer questions: never ask, decide safely and log it.
Task: T14 - License cleanup report (Prompt 4 section 8, T14). Started by the supervisor at 2026-09-27 03:38:29 IST. Hard time cap 90 minutes (killed at 95).
First read design/realism/AGENT_BRIEF.md and design/realism/tasks/T14.md, then do only this task and finish as the brief says.
Queue entry now: status running, depends_on {"T2": "done"}, crashes before this run 0.
Notes from earlier runs (resume from them): 5 failed runs in a row; last log T14-20260926-140852.jsonl | requeued 2026-09-26 18:30: the 5 'failed runs' were claude.ai session limits (HTTP 429, 'You've hit your session limit') that the supervisor misread as crashes (fixed); no work was lost in those runs | paused by a usage limit at 2026-09-27 01:55:09; resume from the notes

**Claude** (2026-09-26T22:08): I'll start by reading the brief and task spec.

**Claude** (2026-09-26T22:08): _[tool: Read]_

**Claude** (2026-09-26T22:08): _[tool: Read]_

**Claude** (2026-09-26T22:08): _[tool: Bash]_

**Claude** (2026-09-26T22:08): _[tool: Bash]_

**Claude** (2026-09-26T22:08): Resuming cleanly — no partial output exists yet. Let me inspect refs.db and analyze.

**Claude** (2026-09-26T22:08): _[tool: Bash]_

**Claude** (2026-09-26T22:08): _[tool: Write]_

**Claude** (2026-09-26T22:08): _[tool: Bash]_

**Claude** (2026-09-26T22:09): _[tool: Write]_

**Claude** (2026-09-26T22:09): _[tool: Bash]_

**Claude** (2026-09-26T22:09): _[tool: Write]_

**Claude** (2026-09-26T22:09): _[tool: Bash]_

**Claude** (2026-09-26T22:10): _[tool: Write]_

**Claude** (2026-09-26T22:10): _[tool: Bash]_

**Claude** (2026-09-26T22:10): _[tool: Bash]_

**Claude** (2026-09-26T22:10): _[tool: Edit]_

**Claude** (2026-09-26T22:10): _[tool: Bash]_

**Claude** (2026-09-26T22:11): _[tool: Bash]_

**Claude** (2026-09-26T22:11): _[tool: Bash]_

**Claude** (2026-09-26T22:11): _[tool: Bash]_

**Claude** (2026-09-26T22:11): _[tool: Bash]_

**Claude** (2026-09-26T22:11): _[tool: Bash]_

**Claude** (2026-09-26T22:11): T14 done, commit `fc1a83e`, ~3 minutes of the 90-minute cap.

- `NEEDS_CLEANING.csv`: 271 findings over 267 distinct files, from 2,385 CSV rows and 2,412 refs.db rows; `NEEDS_CLEANING.md` carries the counts; `make_needs_cleaning.py` reproduces both.
- Duplicate ids: 25 ids on 53 rows (22 ids ×2, STO-098/099/100 ×3 = 28 extra); 11 groups mix SHIP and STUDY, the rest are internally consistent; 0 rows share surface+file_name.
- STO-070/STO-071: same sha256 `33509d9e791d`, 1 file on disk (macOS case-insensitive), both CC BY-SA 4.0 / STUDY — one row is a phantom, nothing shippable affected.
- author_credit: 96 placeholder, 70 empty, 8 boilerplate paragraphs, 5 URLs; 7 blocking (SEA-002/003/020, PAL-145/149/164, BRO-383 — SHIP CC BY 4.0 with no author). Plus 1 empty licence (RIN-032, STUDY), 8 non-CC licences (GODL-India 5, "No restrictions" 3, all STUDY), 0 SHIP rows with a non-ship licence, 26 refs.db rows absent from the CSV.
- LICENSES.csv shasum `d960b5a7` and refs.db `ebe12934` identical before and after; nothing applied — every fix is a proposal, and every licence/author/date remains unverified by any human or Tamil speaker.
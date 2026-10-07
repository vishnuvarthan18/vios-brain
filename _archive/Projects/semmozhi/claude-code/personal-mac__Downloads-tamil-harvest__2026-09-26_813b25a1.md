**Vishnu** (2026-09-26T17:18): You are an unattended run of the Realism Engine supervisor. Nobody will answer questions: never ask, decide safely and log it.
Task: T4-seals - Auto-tune the seals material (Prompt 4 section 8, T4). Started by the supervisor at 2026-09-26 22:48:56 IST. Hard time cap 90 minutes (killed at 95).
First read design/realism/AGENT_BRIEF.md and design/realism/tasks/T4-seals.md, then do only this task and finish as the brief says.
Queue entry now: status running, depends_on {"T4-palm_leaf": "done"}, crashes before this run 0.
Notes from earlier runs (resume from them): 5 failed runs in a row; last log T4-seals-20260926-102843.jsonl | requeued 2026-09-26 18:30: the 5 'failed runs' were claude.ai session limits (HTTP 429, 'You've hit your session limit') that the supervisor misread as crashes (fixed); no work was lost in those runs

**Claude** (2026-09-26T17:19): I'll start by reading the brief and task file.

**Claude** (2026-09-26T17:19): _[tool: Read]_

**Claude** (2026-09-26T17:19): _[tool: Read]_

**Claude** (2026-09-26T17:19): _[tool: Bash]_

**Claude** (2026-09-26T17:19): _[tool: Bash]_

**Claude** (2026-09-26T17:19): _[tool: Read]_

**Claude** (2026-09-26T17:19): _[tool: Bash]_

**Claude** (2026-09-26T17:19): Seals is a disc (same family as coins/rings). Bronze → metal searched, grain angle pinned to the spec axis. Logging the decision and starting run 1.

**Claude** (2026-09-26T17:19): _[tool: Bash]_

**Claude** (2026-09-26T17:19): _[tool: Bash]_

**Claude** (2026-09-26T17:26): _[tool: Bash]_

**Claude** (2026-09-26T17:26): _[tool: Bash]_

**Claude** (2026-09-26T17:26): _[tool: Bash]_

**Claude** (2026-09-26T17:26): _[tool: Bash]_

**Claude** (2026-09-26T17:26): _[tool: Bash]_

**Claude** (2026-09-26T17:26): Every edge_rough that reaches aspect ≥1.009 breaks grain_direction instead (the ragged edge dominates the structure tensor) — same wall as T4-rings. Logging that and running run 2 for the colour row.

**Claude** (2026-09-26T17:27): _[tool: Bash]_

**Claude** (2026-09-26T17:27): _[tool: Bash]_

**Claude** (2026-09-26T17:33): _[tool: Bash]_

**Claude** (2026-09-26T17:33): Run 2: loss 6.012 → 5.622, colour dE median 7.18 → 6.31 (target 6.0 — nearly there). One more run to try to close that row.

**Claude** (2026-09-26T17:33): _[tool: Bash]_

**Claude** (2026-09-26T17:33): _[tool: Bash]_

**Claude** (2026-09-26T17:33): The tuned render (6.31) is already better than the median real seal photo (8.26) on that row — only 25% of real photos pass it. One more run to try anyway.

**Claude** (2026-09-26T17:34): _[tool: Bash]_

**Claude** (2026-09-26T17:40): Run 3 closed the colour row: **7/8 rows, loss 4.442**. Now the lighting candidate (seals have relief).

**Claude** (2026-09-26T17:40): _[tool: Bash]_

**Claude** (2026-09-26T17:41): _[tool: Bash]_

**Claude** (2026-09-26T17:41): _[tool: Write]_

**Claude** (2026-09-26T17:41): _[tool: Bash]_

**Claude** (2026-09-26T17:41): Now register both candidates.

**Claude** (2026-09-26T17:41): _[tool: Bash]_

**Claude** (2026-09-26T17:41): _[tool: Read]_

**Claude** (2026-09-26T17:41): _[tool: Edit]_

**Claude** (2026-09-26T17:41): _[tool: Edit]_

**Claude** (2026-09-26T17:41): Now add the note_t4_seals provenance line and build the sheet.

**Claude** (2026-09-26T17:41): _[tool: Bash]_

**Claude** (2026-09-26T17:41): _[tool: Edit]_

**Claude** (2026-09-26T17:42): _[tool: Edit]_

**Claude** (2026-09-26T17:42): _[tool: Bash]_

**Claude** (2026-09-26T17:42): _[tool: Bash]_

**Claude** (2026-09-26T17:42): _[tool: Read]_

**Claude** (2026-09-26T17:42): _[tool: Bash]_

**Claude** (2026-09-26T17:50): _[tool: Bash]_

**Claude** (2026-09-26T17:50): 2D baseline 0/8 → tuned 7/8; lighting 26.38 dL PASS. Writing the sheet notes.

**Claude** (2026-09-26T17:50): _[tool: Write]_

**Claude** (2026-09-26T17:51): _[tool: Bash]_

**Claude** (2026-09-26T17:51): _[tool: Bash]_

**Claude** (2026-09-26T17:51): _[tool: Bash]_

**Claude** (2026-09-26T17:51): T4-seals done in 32 min, commit `fa03a34`.

- Tuner loss 25.310 → 6.012 → 5.622 → **4.442** over 3 chained 200-trial runs; **7 of 8 visual rows PASS** against the site 2D baseline's **0 of 8**.
- colour dE2000 median 5.75 (baseline 24.11), p95 7.36 (32.09), slope diff 0.108, anisotropy diff 0.027, local contrast 1.041, grain dir 3.73°.
- Lighting candidate **26.38 dL\* PASS** (baseline FAIL); its 0.5 mm relief on a 30 mm seal is an UNVERIFIED estimate, as is the Tamil sample text.
- FAIL, reported not tuned away: `aspect_ratio` 1.005 vs 1.009..2.465 — `rig/stage.js` forces aspect=1 for the disc, and the probe shows every edge_rough reaching 1.009 breaks grain_direction (39–85° vs a 15° limit).
- Honesty: only 13% / 25% / 31% of real seal photos pass the slope / colour-median / colour-p95 rows the render passes, and the spec card is low confidence (16 usable photos of 20 wanted).
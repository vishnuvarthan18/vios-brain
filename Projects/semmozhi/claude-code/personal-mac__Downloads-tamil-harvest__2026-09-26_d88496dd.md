**Vishnu** (2026-09-26T22:41): You are an unattended run of the Realism Engine supervisor. Nobody will answer questions: never ask, decide safely and log it.
Task: T16 - Final report (Prompt 4 section 8, T16). Started by the supervisor at 2026-09-27 04:11:36 IST. Hard time cap 90 minutes (killed at 95).
First read design/realism/AGENT_BRIEF.md and design/realism/tasks/T16.md, then do only this task and finish as the brief says.
Queue entry now: status running, depends_on {"T15": "done", "T14": "done", "T8": "done"}, crashes before this run 0.
Notes from earlier runs (resume from them): 5 failed runs in a row; last log T16-20260926-141448.jsonl | requeued 2026-09-26 18:30: the 5 'failed runs' were claude.ai session limits (HTTP 429, 'You've hit your session limit') that the supervisor misread as crashes (fixed); no work was lost in those runs

**Claude** (2026-09-26T22:41): I'll start by reading the brief and task spec.

**Claude** (2026-09-26T22:41): _[tool: Read]_

**Claude** (2026-09-26T22:41): _[tool: Read]_

**Claude** (2026-09-26T22:41): _[tool: Bash]_

**Claude** (2026-09-26T22:41): _[tool: Bash]_

**Claude** (2026-09-26T22:41): _[tool: Read]_

**Claude** (2026-09-26T22:41): _[tool: Bash]_

**Claude** (2026-09-26T22:41): _[tool: Bash]_

**Claude** (2026-09-26T22:49): _[tool: Bash]_

**Claude** (2026-09-26T22:49): _[tool: Bash]_

**Claude** (2026-09-26T22:49): _[tool: Bash]_

**Claude** (2026-09-26T22:49): _[tool: Bash]_

**Claude** (2026-09-26T22:49): _[tool: Bash]_

**Claude** (2026-09-26T22:49): _[tool: Bash]_

**Claude** (2026-09-26T22:49): _[tool: Bash]_

**Claude** (2026-09-26T22:49): _[tool: Bash]_

**Claude** (2026-09-26T22:50): _[tool: Bash]_

**Claude** (2026-09-26T22:50): _[tool: Bash]_

**Claude** (2026-09-26T22:50): Now writing the report.

**Claude** (2026-09-26T22:51): _[tool: Write]_

**Claude** (2026-09-26T22:51): _[tool: Bash]_

**Claude** (2026-09-26T22:52): _[tool: Edit]_

**Claude** (2026-09-26T22:52): _[tool: Edit]_

**Claude** (2026-09-26T22:52): _[tool: Edit]_

**Claude** (2026-09-26T22:52): _[tool: Edit]_

**Claude** (2026-09-26T22:52): _[tool: Edit]_

**Claude** (2026-09-26T22:52): Now commit.

**Claude** (2026-09-26T22:52): _[tool: Bash]_

**Claude** (2026-09-26T22:52): _[tool: Bash]_

**Claude** (2026-09-26T22:52): _[tool: Bash]_

**Claude** (2026-09-26T22:52): T16 done at commit `8e4b12f` (11 min of the 90-minute cap).

- `run_all.mjs` exit 0, 453 s, evaluator self-check OK: **42 of 382 scored rows FAIL**, 340 pass, 2 not scorable — the report's first line says so.
- Of the 42: 31 are the site 2D baseline kept as the "before" number; **11 are this engine's own work** (palm leaf 3D anisotropy 0.189/0.15 and grain 17.46°/15, palm leaf lighting 2.448 and 2.841 of 4.0 dL*, pottery dE 7.34/6.0, rings dE 11.65 and 13.87 which 0 of 16 real photos pass, rings/seals aspect 1.000 and 1.005, T6a stone 6.46/13.2). All reported with reasons, none tuned away.
- By category: physics 128/128, compat 84/84, perf 12/12, wear 11/11, lighting 7/14, visual 98 pass / 35 fail.
- `design/realism/FINAL.md` carries all 33 task statuses (31 done, 1 skipped: T4-thread_board has 0 reference photos), 41 commits, 8 gate decisions (stone/copper/coins adopt 3D), 105 decisions, 74 low-confidence values, and 7 blocking CC BY licence rows.
- NOT verified: no real phone, no Safari/iOS/Firefox (one browser installed), no screen reader, no object measured by hand, no scale bar, uncalibrated colour, and no Tamil speaker has checked any line.
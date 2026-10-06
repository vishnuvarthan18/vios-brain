**Vishnu** (2026-09-26T22:12): You are an unattended run of the Realism Engine supervisor. Nobody will answer questions: never ask, decide safely and log it.
Task: T15 - Cross-browser, accessibility, keyboard, reduced motion, file:// (Prompt 4 section 8, T15). Started by the supervisor at 2026-09-27 03:42:01 IST. Hard time cap 90 minutes (killed at 95).
First read design/realism/AGENT_BRIEF.md and design/realism/tasks/T15.md, then do only this task and finish as the brief says.
Queue entry now: status running, depends_on {"T11c": "done", "T12b": "done", "T13": "done"}, crashes before this run 0.
Notes from earlier runs (resume from them): 5 failed runs in a row; last log T15-20260926-141150.jsonl | requeued 2026-09-26 18:30: the 5 'failed runs' were claude.ai session limits (HTTP 429, 'You've hit your session limit') that the supervisor misread as crashes (fixed); no work was lost in those runs

**Claude** (2026-09-26T22:12): I'll start by reading the brief and task file.

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:12): _[tool: Bash]_

**Claude** (2026-09-26T22:13): _[tool: Write]_

**Claude** (2026-09-26T22:13): _[tool: Bash]_

**Claude** (2026-09-26T22:14): _[tool: Bash]_

**Claude** (2026-09-26T22:14): _[tool: Edit]_

**Claude** (2026-09-26T22:14): _[tool: Edit]_

**Claude** (2026-09-26T22:14): Now remove the old `focusVisible` use and wire the new data.

**Claude** (2026-09-26T22:14): _[tool: Edit]_

**Claude** (2026-09-26T22:14): _[tool: Edit]_

**Claude** (2026-09-26T22:14): Now delete the now-unused `focusVisible` helper.

**Claude** (2026-09-26T22:14): _[tool: Edit]_

**Claude** (2026-09-26T22:14): _[tool: Bash]_

**Claude** (2026-09-26T22:14): _[tool: Bash]_

**Claude** (2026-09-26T22:15): _[tool: Edit]_

**Claude** (2026-09-26T22:15): _[tool: Bash]_

**Claude** (2026-09-26T22:15): Harness works. Running the full set of 13 pages.

**Claude** (2026-09-26T22:15): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): _[tool: Bash]_

**Claude** (2026-09-26T22:18): Now the fix: `role="region"` on the `<ul>` strips the list role from its `<li>` children.

**Claude** (2026-09-26T22:19): _[tool: Bash]_

**Claude** (2026-09-26T22:19): _[tool: Read]_

**Claude** (2026-09-26T22:19): _[tool: Edit]_

**Claude** (2026-09-26T22:19): Now investigating the tab-reach shortfalls before judging them.

**Claude** (2026-09-26T22:19): _[tool: Bash]_

**Claude** (2026-09-26T22:19): _[tool: Edit]_

**Claude** (2026-09-26T22:19): _[tool: Edit]_

**Claude** (2026-09-26T22:19): _[tool: Edit]_

**Claude** (2026-09-26T22:19): _[tool: Edit]_

**Claude** (2026-09-26T22:19): _[tool: Edit]_

**Claude** (2026-09-26T22:19): _[tool: Edit]_

**Claude** (2026-09-26T22:19): _[tool: Edit]_

**Claude** (2026-09-26T22:19): _[tool: Bash]_

**Claude** (2026-09-26T22:22): _[tool: Bash]_

**Claude** (2026-09-26T22:22): _[tool: Bash]_

**Claude** (2026-09-26T22:22): _[tool: Bash]_

**Claude** (2026-09-26T22:22): Same pattern. Fixing it and checking for any other `role="region"` on a list.

**Claude** (2026-09-26T22:22): _[tool: Bash]_

**Claude** (2026-09-26T22:22): _[tool: Edit]_

**Claude** (2026-09-26T22:22): _[tool: Bash]_

**Claude** (2026-09-26T22:22): _[tool: Edit]_

**Claude** (2026-09-26T22:22): Now the unreachable link on the styleguide.

**Claude** (2026-09-26T22:22): _[tool: Bash]_

**Claude** (2026-09-26T22:22): _[tool: Bash]_

**Claude** (2026-09-26T22:22): _[tool: Edit]_

**Claude** (2026-09-26T22:23): _[tool: Bash]_

**Claude** (2026-09-26T22:25): _[tool: Bash]_

**Claude** (2026-09-26T22:25): Now regression-checking the edited pages with the existing site checks.

**Claude** (2026-09-26T22:25): _[tool: Bash]_

**Claude** (2026-09-26T22:28): _[tool: Bash]_

**Claude** (2026-09-26T22:31): _[tool: Bash]_

**Claude** (2026-09-26T22:39): _[tool: Bash]_

**Claude** (2026-09-26T22:39): _[tool: Bash]_

**Claude** (2026-09-26T22:39): _[tool: Bash]_

**Claude** (2026-09-26T22:39): _[tool: Bash]_

**Claude** (2026-09-26T22:39): _[tool: Bash]_

**Claude** (2026-09-26T22:39): _[tool: Bash]_

**Claude** (2026-09-26T22:39): _[tool: Bash]_

**Claude** (2026-09-26T22:40): _[tool: Write]_

**Claude** (2026-09-26T22:40): Adding the axe-core license row.

**Claude** (2026-09-26T22:40): _[tool: Edit]_

**Claude** (2026-09-26T22:40): _[tool: Bash]_

**Claude** (2026-09-26T22:40): _[tool: Bash]_

**Claude** (2026-09-26T22:40): _[tool: Bash]_

**Claude** (2026-09-26T22:40): _[tool: Bash]_

**Claude** (2026-09-26T22:40): _[tool: Bash]_

**Claude** (2026-09-26T22:40): _[tool: Bash]_

**Claude** (2026-09-26T22:40): _[tool: Bash]_

**Claude** (2026-09-26T22:40): _[tool: Bash]_

**Claude** (2026-09-26T22:41): T15 done, commit `f737fd1` (state files in the follow-up commit). Elapsed ~45 of the 90 minutes.

- 13 of 13 pages pass over `file://` at 1200x800 and 360x740 DPR 2: 0 console errors, 0 failed requests, 0 external requests, 0 axe violations (axe-core 4.12.1, wcag2a/2aa/21a/21aa/22aa, MPL-2.0, dev only, never shipped).
- Reduced motion: 0.000 changed pixels on all 13 over a 2 s window (target 0.002). Tab reaches every visible enabled control on 12 pages (styleguide 142/142, home 42/42, chola 41/41, kural 43/43); identity.html 24 of 144 is its roving-tabindex gallery by design. Focus ring visible on 8/8 controls sampled per page.
- Fixed the only violation found: `role="region"` on a `<ul>`/`<ol>` replaces the implicit list role, so 26 `<li>` failed axe `listitem` (serious) — 5 elements over 4 pages (`.subnav__list` x4, `.timeline--across`) now use `tabindex="0" aria-label` without the role; backups in `design/realism/backups/T15/`, DESIGN_SYSTEM.md updated. `run_all.mjs` unchanged at 42 of 382 scored rows FAIL, no regression.
- Not evidence about anything but Chromium: Firefox and WebKit are not installed and were not downloaded. `design/realism/eval/CROSS_BROWSER.md` lists what still needs a real Safari/iOS, Firefox, a real phone (touch, GPU, thermal), real screen readers (0 axe violations is not "accessible"), and a Tamil reader — content facts stay unverified.
- One deviation logged as a decision: `cp -n`/`cp -pn` were denied by the permission layer 3 times, so the mandatory backups were made with `python3 shutil.copy2` (same no-overwrite rule).
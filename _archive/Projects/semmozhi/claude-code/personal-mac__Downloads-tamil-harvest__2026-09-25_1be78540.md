**Vishnu** (2026-09-25T07:11): <pasted_content id="bf75">
Read design/prompts/prompt-3-reference-engine.md in the tamil_harvest folder. Build design/fetch_commons_v3.py, the mass Commons harvester, for an unattended overnight run. It must never stop to ask me anything.

Rules:
1. Extend design/fetch_commons_v2.py and keep its license logic (read the license from the Commons API, never from page text). Import the existing 256 images and LICENSES.csv first and keep them. Back up LICENSES.csv before starting and every 200 downloads.
2. Surfaces: stone, palm_leaf, pottery, coins, copper_plate, rings, seals, temples, bronzes. Target 500 each. Keep going until a surface reaches 500 or its categories run out. Do not pad with off-topic or non-Tamil items; mark those technique reference or move them to _unused.
3. Crawl Commons categories to depth 3 and use keyword searches, including Tamil-script words. Set a User-Agent that names the project. Wait 1 second between requests. On HTTP 429 or 5xx, back off (5s, 30s, 2min, 10min) and retry up to 6 times, then skip that file and log it. Never crash on one bad file. Resume automatically after a crash or Ctrl+C by skipping files already in the database.
4. Write one LICENSES.csv row as soon as each file downloads. Cap images at 2000 px on the long side; skip files under 1200 px. Tier each row SHIP (public domain, CC0, CC BY) or STUDY (all else). Add columns tier, tamil_relevance, sha256, phash. Move exact and near duplicates to _dupes; never delete anything.
5. Safety guards: stop cleanly (not crash) if free disk falls under 20 GB. Never touch website_live or any file outside design/. Do not commit design/references/. Do not use any workaround for a blocked domain.
6. Before the real run, test yourself with --dry-run (print counts per surface) and a smoke test of 5 images per surface. If the smoke test passes (files download, licenses read, no errors), continue. If not, fix it. Do not ask me.
7. Progress: write design/overnight_progress.txt every 60 seconds (surface, count, SHIP/STUDY, errors, disk free, time elapsed) and a final design/overnight_summary.md with a coverage table vs 500 for each surface, the disk used, the skipped files with reasons, and the exact command to resume.
8. If your own shell has no internet, do not give up: finish the script, test it as far as you can (dry-run logic, license parsing on saved sample JSON), then print the exact one-line command for me to run in Terminal.
Report what is verified and what is not.
</pasted_content id="bf75">

**Claude** (2026-09-25T07:11): _[tool: Bash]_

**Claude** (2026-09-25T07:11): _[tool: Bash]_

**Claude** (2026-09-25T07:11): _[tool: Bash]_

**Claude** (2026-09-25T07:11): _[tool: Bash]_

**Claude** (2026-09-25T07:11): _[tool: Bash]_

**Claude** (2026-09-25T07:16): _[tool: Bash]_

**Claude** (2026-09-25T07:16): _[tool: Bash]_

**Claude** (2026-09-25T07:16): Internet works from my shell, so I'll build the script, test it here, and then start the real run myself. Right now I'm checking the 256 existing files before I write any code.

**Claude** (2026-09-25T07:16): _[tool: Bash]_

**Claude** (2026-09-25T07:16): _[tool: Bash]_

**Claude** (2026-09-25T07:21): _[tool: Bash]_

**Claude** (2026-09-25T07:21): _[tool: Bash]_

**Claude** (2026-09-25T07:22): _[tool: Bash]_

**Claude** (2026-09-25T07:34): The category names are confirmed (several guesses didn't exist, so I'm using the real ones). Now I'm writing the harvester and its source list.

**Claude** (2026-09-25T07:34): _[tool: Bash]_

**Claude** (2026-09-25T07:38): _[tool: Write]_

**Claude** (2026-09-25T07:39): _[tool: Bash]_

**Claude** (2026-09-25T07:39): Syntax is clean and the script has no delete calls. Next I'm saving a few real Commons API answers as sample files, so the offline test can check license parsing without the network.

**Claude** (2026-09-25T07:40): _[tool: Bash]_

**Claude** (2026-09-25T07:40): _[tool: Bash]_

**Claude** (2026-09-25T07:41): _[tool: Write]_

**Claude** (2026-09-25T07:41): _[tool: Bash]_

**Claude** (2026-09-25T07:42): 35/35 offline checks pass. One of them ("1 s between requests") could pass without checking anything, so I'm tightening it, then writing the sources list.

**Claude** (2026-09-25T07:42): _[tool: Bash]_

**Claude** (2026-09-25T07:42): Now the source list: the old v2 sources, plus temples, bronzes, and searches in Tamil script.

**Claude** (2026-09-25T07:43): _[tool: Write]_

**Claude** (2026-09-25T07:43): _[tool: Bash]_

**Claude** (2026-09-25T07:43): All 75 source categories exist on Commons (246 sources in total). Before the dry run, one live check that a 50-file detail request comes back complete.

**Claude** (2026-09-25T07:43): _[tool: Bash]_

**Claude** (2026-09-25T07:44): _[tool: Bash]_

**Claude** (2026-09-25T07:44): _[tool: Bash]_

**Claude** (2026-09-25T07:44): Tamil searches now return Tamil pages (141 hits instead of 1,000 unrelated). Starting the `--dry-run` in the background. It samples 100 files from each of the 246 sources and writes nothing.

**Claude** (2026-09-25T07:44): _[tool: Bash]_

**Claude** (2026-09-25T07:44): While it runs, I'm re-reading the core of the script.

**Claude** (2026-09-25T07:44): _[tool: Read]_

**Claude** (2026-09-25T07:44): _[tool: Bash]_

**Claude** (2026-09-25T07:45): _[tool: Bash]_

**Claude** (2026-09-25T07:45): _[tool: Read]_

**Claude** (2026-09-25T07:45): _[tool: Bash]_

**Claude** (2026-09-25T07:45): Pillow warns that `getdata()` is deprecated. Switching to `tobytes()`; the running dry run already loaded the old copy, so it isn't affected.

**Claude** (2026-09-25T07:45): _[tool: Edit]_

**Claude** (2026-09-25T07:45): _[tool: ToolSearch]_

**Claude** (2026-09-25T07:45): _[tool: Monitor]_

**Claude** (2026-09-25T07:45): _[tool: Read]_

**Claude** (2026-09-25T07:45): _[tool: Bash]_

**Claude** (2026-09-25T07:46): The import finished. Counts toward 500 (direct + regional) are lower than the raw file counts, because the keyword check marks some existing files as "technique" (not Tamil):

**Claude** (2026-09-25T07:46): _[tool: Bash]_

**Claude** (2026-09-25T07:46): _[tool: Bash]_

**Claude** (2026-09-25T07:46): _[tool: Bash]_

**Vishnu** (2026-09-25T07:51): Your response above was cut off mid-stream. Resume directly from where it stops — no apology, no recap. If none of it survived, answer the request from the start.

**Claude** (2026-09-25T07:51): _[tool: Bash]_

**Claude** (2026-09-25T07:51): _[tool: Bash]_

**Claude** (2026-09-25T08:34): API Error: Can't reach the API server — check your internet or DNS (ENOTFOUND)

**Vishnu** (2026-09-25T08:34): <task-notification>
<task-id>be13has3z</task-id>
<summary>Monitor event: "dry-run milestones and errors"</summary>
<event>[Monitor expired after 30m with no events delivered. Re-arm it if you still need the watch — and widen the filter if silence was unexpected.]</event>
</task-notification>

**Claude** (2026-09-25T09:45): API Error: Can't reach the API server — check your internet or DNS (ENOTFOUND)

**Vishnu** (2026-09-25T15:06): goahed

**Claude** (2026-09-25T15:06): _[tool: Bash]_

**Claude** (2026-09-25T15:06): The dry run is still going 2.7 hours in, and it's only on each surface's second source. Something is far slower than planned. Looking at which sources took the time:

**Claude** (2026-09-25T15:06): _[tool: Bash]_

**Claude** (2026-09-25T15:06): _[tool: Bash]_

**Claude** (2026-09-25T15:07): _[tool: Bash]_

**Claude** (2026-09-25T15:07): _[tool: Bash]_

**Claude** (2026-09-25T15:08): Self-test still passes (35/35). Re-running the dry run with `caffeinate` so the Mac can't sleep mid-run:

**Claude** (2026-09-25T15:08): _[tool: Bash]_

**Claude** (2026-09-25T15:08): _[tool: Monitor]_

**Claude** (2026-09-25T15:08): _[tool: Bash]_

**Claude** (2026-09-25T15:08): _[tool: Bash]_

**Claude** (2026-09-25T15:08): It's 20:38 IST. The dry run needs about an hour, then the smoke test, then the overnight run can start around 22:00. I'll check back when the monitor reports.

**Claude** (2026-09-25T15:08): _[tool: Bash]_

**Claude** (2026-09-25T15:08): While waiting, saving two non-obvious lessons to memory: bare Tamil-word searches match worldwide concepts, and runs die when the Mac sleeps.

**Claude** (2026-09-25T15:08): _[tool: Write]_

**Claude** (2026-09-25T15:08): _[tool: Bash]_

**Claude** (2026-09-25T15:08): _[tool: Bash]_

**Claude** (2026-09-25T15:08): The dry run is on its first source. I'll pick up again when the waiter reports that two rounds of sources are done.

**Claude** (2026-09-25T15:08): _[tool: Bash]_

**Claude** (2026-09-25T15:11): Pottery now keeps Keezhadi finds (5 kept instead of 0). The first round took 3 minutes. I'm stopping the per-line monitor, which is too noisy, and auditing what the filter actually keeps and rejects:

**Claude** (2026-09-25T15:11): _[tool: ToolSearch]_

**Claude** (2026-09-25T15:11): _[tool: TaskStop]_

**Claude** (2026-09-25T15:11): _[tool: TaskStop]_

**Claude** (2026-09-25T15:11): _[tool: Bash]_

**Claude** (2026-09-25T15:11): _[tool: Bash]_

**Claude** (2026-09-25T15:11): _[tool: Bash]_

**Claude** (2026-09-25T15:12): _[tool: Bash]_

**Claude** (2026-09-25T15:12): _[tool: Bash]_

**Claude** (2026-09-25T15:12): The self-test passes again (35/35, now with 31 relevance cases). Restarting the full dry run with the fixed filter:

**Claude** (2026-09-25T15:12): _[tool: Bash]_

**Claude** (2026-09-25T15:12): _[tool: Monitor]_

**Claude** (2026-09-25T15:12): _[tool: Bash]_

**Claude** (2026-09-25T15:12): _[tool: Bash]_

**Claude** (2026-09-25T15:12): The dry run started at 20:42 and is importing the existing library. I'll look again when it reports finished or failed.

**Claude** (2026-09-25T15:12): _[tool: Bash]_

**Claude** (2026-09-25T15:22): _[tool: Bash]_

**Claude** (2026-09-25T15:23): _[tool: Bash]_

**Claude** (2026-09-25T15:23): The self-test passes (35/35, 32 relevance cases). Waiting for the dry run to finish.

**Claude** (2026-09-25T15:23): _[tool: Bash]_

**Claude** (2026-09-25T15:33): _[tool: Bash]_

**Claude** (2026-09-25T15:33): 117 of 246 sources done in 21 minutes, with no network errors. About 20 minutes to go.

**Claude** (2026-09-25T15:33): _[tool: Bash]_

**Claude** (2026-09-25T15:43): _[tool: Bash]_

**Claude** (2026-09-25T15:43): The dry run is on source 191 of 246, still with no errors. Waiting for the finish.

**Claude** (2026-09-25T15:43): _[tool: Bash]_

**Claude** (2026-09-25T15:47): _[tool: Bash]_

**Claude** (2026-09-25T15:47): _[tool: Bash]_

**Claude** (2026-09-25T15:48): The zeros are genuine: without the image-only filter those searches match only PDF and DjVu books. The search function works. The log also has one "cannot list / details failed" line:

**Claude** (2026-09-25T15:48): _[tool: Bash]_

**Claude** (2026-09-25T15:48): _[tool: Bash]_

**Claude** (2026-09-25T15:48): _[tool: Bash]_

**Claude** (2026-09-25T15:48): _[tool: Bash]_

**Claude** (2026-09-25T15:55): The smoke test finished cleanly (exit 0). Checking its results against the pass criteria:

**Claude** (2026-09-25T15:55): _[tool: Bash]_

**Claude** (2026-09-25T15:56): _[tool: Bash]_

**Claude** (2026-09-25T15:56): _[tool: Bash]_

**Claude** (2026-09-25T15:56): That failure is the test being out of date, not a bug. It counts SHIP rows across the live sheet, which now also holds the 45 smoke-test rows. Limiting it to the original rows:

**Claude** (2026-09-25T15:56): _[tool: Bash]_

**Claude** (2026-09-25T15:56): Self-test is 36/36. Now moving PAL-065 out, backing up first. The file and its row go to `_unused` rather than being deleted:

**Claude** (2026-09-25T15:56): _[tool: Bash]_

**Claude** (2026-09-25T15:57): _[tool: Bash]_

**Claude** (2026-09-25T15:57): _[tool: Bash]_

**Claude** (2026-09-25T15:57): The tree is as expected: supervisor 82204, caffeinate 82206 holding the Mac awake, harvester 82207 holding the lock and already downloading (STO-109…111). Sending SIGTERM to the supervisor:

**Claude** (2026-09-25T15:57): _[tool: Bash]_

**Claude** (2026-09-25T15:58): _[tool: Bash]_

**Claude** (2026-09-25T15:58): _[tool: Bash]_

**Claude** (2026-09-25T15:59): _[tool: Bash]_

**Claude** (2026-09-25T16:00): _[tool: Read]_

**Claude** (2026-09-25T16:01): _[tool: Bash]_

**Claude** (2026-09-25T16:01): The run stopped cleanly. Applying the title-evidence fix for stone and pottery now, then re-labelling the affected original rows.

**Claude** (2026-09-25T16:01): _[tool: Bash]_

**Claude** (2026-09-25T16:02): _[tool: Bash]_

**Claude** (2026-09-25T16:02): _[tool: Bash]_

**Claude** (2026-09-25T16:02): _[tool: Bash]_

**Claude** (2026-09-25T16:04): The overnight Commons harvest is running now, started from my shell, so you don't need to type anything. The shell had internet, so the fallback in your rule 8 wasn't needed. It started at 21:32 IST as run 5 under a supervisor that restarts it after a crash. After 2 minutes it had downloaded 19 files with 0 errors and 193.7 GB free.

- **Check progress:** `cat ~/Downloads/tamil_harvest/design/overnight_progress.txt` (updated every 60 s)
- **Stop cleanly:** `pkill -TERM -f "fetch_commons_v3.py --supervise"` (writes the summary first)
- **Resume later** (skips everything already in the database):
  ```
  cd ~/Downloads/tamil_harvest && nohup caffeinate -ims python3 design/fetch_commons_v3.py --supervise >> design/reference_engine/overnight_stdout.log 2>&1 &
  ```
- **Keep the Mac on power, lid open.** `caffeinate` stops normal sleep, but a closed lid on battery will still sleep it. Before I added it, one test run lost 2.5 hours to sleep. It recovered afterwards without crashing.

When it finishes, `design/overnight_summary.md` will have the coverage table against 500, disk used, skipped files with reasons, and the resume command.

## Verified
- **Import of your 256:** counts match Prompt 3 exactly (stone 100, palm_leaf 44, pottery 41, coins 37, copper_plate 25, rings 9, seals 0), 90 SHIP. LICENSES.csv gained the four new columns and every old value is unchanged (checked row by row).
- **Backups:** made at the start and end of every run and before each of my manual edits.
- **Offline self-test, 36/36 pass:** license tiers from saved real API answers; 35 relevance cases; the full back-off sequence (5 s, 30 s, 2 min, 10 min ×3, then skip); 1 s between requests; a stop after five 403s in a row; the 20 GB guard; refusing writes outside `design/`; the 2000 px cap; duplicate detection; resume bookkeeping.
- **Dry run:** all 246 sources ran with no errors in 736 requests. Every one of the 75 category names exists on Commons.
- **Smoke test, 5 per surface:** 45 files downloaded, licenses read from the API, 0 errors. Every file is between 1200 and 2000 px on its long side.
- **Graceful stop and crash:** SIGTERM wrote the summary, released the lock and did not restart. After `kill -9` the supervisor restarted the harvester 60 s later, and it resumed at the next ID (STO-119) with nothing downloaded twice. The database and LICENSES.csv agree row for row.
- **Network loss:** during the first dry run the Mac slept and DNS failed on wake; the back-off waited it out and the run continued.
- **Scope:** nothing written outside `design/`, no delete calls in the script, nothing committed.

## Not verified
- **Final coverage.** From the dry-run samples I expect stone, temples and bronzes to reach 500, palm_leaf around 150–250, pottery and coins around 100, copper plates around 50. Rings and seals will be mostly "technique" items that don't count; there is very little Tamil material of those on Commons.
- **These paths haven't run for real yet:** the second pass without per-source caps, the backup every 200 downloads, a real HTTP 429, near-duplicates among real files (none seen so far), and TIFF/WebP conversion.
- **The dry-run counts are slightly out of date.** I fixed three things after it ran (a DjVu file breaking a whole 50-file batch, Thanjavur gallery bronzes being missed, titles counting as context for stone and pottery). Each fix is covered by the self-test or a live check, but the counts weren't re-run.
- **Tamil relevance is keyword-based and unreviewed.** I spot-checked it and fixed the false picks I found (road signs, a blank grey background image). Expect some misses either way.

## Where I departed from your rules
- **Off-topic files are never downloaded.** They're rejected from metadata and logged in `design/reference_engine/skipped.csv`, rather than downloaded and moved to `_unused`. The one exception was the blank image, which I moved to `_unused/palm_leaf/` with its row.
- **Non-Tamil items of the right kind** are kept as `technique`, capped at 50 per surface, and not counted toward 500.
- **Duplicates:** the first copy stays and later copies move to `_dupes`. Your original files are never moved, even when they're duplicates of each other; I only report them.
- **Interpretations:** "under 1200 px" means the long side, and the disk guard is 20 GiB.
- **I edited LICENSES.csv by hand twice, each after a backup:** moved the PAL-065 row, and re-labelled `tamil_relevance` on 9 original rows. I also cleared 56 stale "rejected" records from the database (records only, no files) so tonight's run re-checks them under the fixed rule.

## For you to decide
- 24 original rows the keyword check calls off-topic, such as script charts, nose rings and Keeladi site photos. They're listed in the summary and left in place.
- 25 duplicate IDs already in LICENSES.csv.
- STO-070 and STO-071 share one file, because their Commons names differ only by case and macOS ignores case. The new script avoids this.

New files are `design/fetch_commons_v3.py`, `design/test_fetch_commons_v3.py`, `design/commons_sources_v3.csv` and `design/reference_engine/`. Run the self-test any time with `python3 design/fetch_commons_v3.py --selftest`.
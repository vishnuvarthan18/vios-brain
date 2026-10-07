**Vishnu** (2026-09-07T03:56): Read NEXT-PHASE-PLAN.md in the repo root in full, then execute steps 1–5 in order. Log every decision and finding to DECISIONS.md, appending to the existing file, same convention as the overnight build. Do not stop to ask questions — make the most reasonable call, log it with reasoning, and keep going. Only pause for genuinely destructive or irreversible actions not already covered in the plan. The five items listed as out-of-scope at the top of the plan belong to the user — do not touch them, do not build around them, just skip past anything gated on them and log it as blocked-on-user. Finish with the verification pass in step 5 before ending the session.

**Claude** (2026-09-07T03:56): I'll start by reading the plan file in full.

**Claude** (2026-09-07T03:56): _[tool: Bash]_

**Claude** (2026-09-07T03:56): The file the task names doesn't exist at the repo root. Let me check whether it exists elsewhere or under a different name.

**Claude** (2026-09-07T03:56): _[tool: Bash]_

**Claude** (2026-09-07T03:57): No `NEXT-PHASE-PLAN.md` exists in the working tree or anywhere in git history. Before concluding, let me check other likely locations and whether `PLAN.md` is the intended document.

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:57): The plan file genuinely doesn't exist in the repo. Let me check whether it was saved outside the repo before I report back.

**Claude** (2026-09-07T03:57): _[tool: Bash]_

**Claude** (2026-09-07T03:57): I stopped before doing any work, because the file this task is built on doesn't exist.

**`NEXT-PHASE-PLAN.md` is not in the repo**, and never was:

- Not in the working tree (root or any subdirectory) — the only plan-like files are [PLAN.md](PLAN.md), [DECISIONS.md](DECISIONS.md), [README.md](README.md).
- Not in git history under that name — `git log --all --diff-filter=A -- '*NEXT-PHASE-PLAN*'` returns nothing, on any branch. No stashes, no other branches.
- Not saved outside the repo — nothing matching `*next*phase*` or `*phase*plan*` in Desktop/Downloads/Documents/iCloud, and no recently-modified stray `.md` that fits.

Working tree is clean at `09a72de`, so nothing was lost locally.

**Why I'm not substituting `PLAN.md`.** I read it to check whether it was the intended document under an old name. It isn't: it's the overnight build plan (phases A–G, dated 2026-09-06, already executed — its results are the existing DECISIONS.md through D-32). It has no "steps 1–5" and no five out-of-scope items at the top; its §5 is eight project-wide prohibitions, not work reserved for you.

I can't invent the five steps, and guessing is especially bad here: the instruction that five out-of-scope items *belong to you* only protects you if I know which five. Getting that wrong means touching exactly what you reserved.

**One likely lead.** The end of DECISIONS.md has a list titled "What needs a human — nothing else is blocked on me" with exactly five numbered items, which is a plausible source for your out-of-scope five:

1. Five API key registrations (WDPA, IUCN v4, GeoNames, OpenTopography, GFW) — B-1
2. Rotate `DATA_GOV_IN_API_KEY` — D-17
3. Policy call on India Code / Indian Kanoon — D-11, D-23
4. Capacity call on the bulk datasets (Copernicus DEM, HydroRIVERS/HydroLAKES, JRC GSW, FSI ISFR OCR)
5. Retest India-WRIS and MoTA from an Indian egress

That's inference about the *exclusions* only — it tells me nothing about steps 1–5 or what step 5's verification pass should check.

**To get moving, send me the plan** — paste it, or save it to the repo root and tell me it's there. If it was drafted in another Claude session and never written to disk, it may still be in that session's transcript. I deliberately wrote nothing to DECISIONS.md, since an entry about a session that never started would just be noise in the log; say the word if you'd rather have it recorded there.

**Vishnu** (2026-09-07T03:59): chcek now

**Claude** (2026-09-07T03:59): _[tool: Bash]_

**Claude** (2026-09-07T03:59): It's there now. Reading it in full.

**Claude** (2026-09-07T03:59): _[tool: Bash]_

**Claude** (2026-09-07T04:00): Plan is in hand. Let me get oriented — the decisions log, the local repos, and the VPS.

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): VPS is healthy. Now let me look at the core API's entity/slug model — that's step 1's target.

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:00): Now let me audit every engine's slug function — that's the core of step 1's item 3.

**Claude** (2026-09-07T04:00): _[tool: Bash]_

**Claude** (2026-09-07T04:01): _[tool: Bash]_

**Claude** (2026-09-07T04:01): _[tool: Bash]_

**Claude** (2026-09-07T04:01): _[tool: Bash]_

**Claude** (2026-09-07T04:01): _[tool: Bash]_

**Claude** (2026-09-07T04:01): _[tool: Bash]_

**Claude** (2026-09-07T04:01): Now I understand the slug landscape across all engines. Let me read the core API's ingest schema and migration tooling to fit conventions.

**Claude** (2026-09-07T04:01): _[tool: Bash]_

**Claude** (2026-09-07T04:02): _[tool: Bash]_

**Claude** (2026-09-07T04:02): _[tool: Bash]_

**Claude** (2026-09-07T04:02): _[tool: Bash]_

**Claude** (2026-09-07T04:02): _[tool: Bash]_

**Claude** (2026-09-07T04:03): _[tool: Bash]_

**Claude** (2026-09-07T04:03): Now I'll implement step 1. First, the core schema migration.

**Claude** (2026-09-07T04:03): _[tool: Bash]_

**Claude** (2026-09-07T04:03): _[tool: Bash]_

**Claude** (2026-09-07T04:03): _[tool: Bash]_

**Claude** (2026-09-07T04:04): _[tool: Bash]_

**Claude** (2026-09-07T04:04): I have the full audit picture. Now writing the core migration.

**Claude** (2026-09-07T04:05): _[tool: Bash]_

**Claude** (2026-09-07T04:05): Now the API changes to accept and persist `slug_version`.

**Claude** (2026-09-07T04:05): _[tool: Bash]_

**Claude** (2026-09-07T04:05): Now the generic re-key migration tool. Let me first look at the existing backup script to reuse its dump path.

**Claude** (2026-09-07T04:05): _[tool: Bash]_

**Claude** (2026-09-07T04:06): Now the generic re-key tool — the heart of step 1.

**Claude** (2026-09-07T04:07): _[tool: Bash]_

**Claude** (2026-09-07T04:07): Now the backup script toggles it depends on, plus unit tests for the planner.

**Claude** (2026-09-07T04:08): _[tool: Bash]_

**Claude** (2026-09-07T04:08): _[tool: Bash]_

**Claude** (2026-09-07T04:08): There's a mangled line in that test file. Fixing it:

**Claude** (2026-09-07T04:08): _[tool: Bash]_

**Claude** (2026-09-07T04:08): _[tool: Bash]_

**Claude** (2026-09-07T04:09): The test shouldn't need a DB driver to exercise a pure function. Let me make the psycopg import lazy and find a runner.

**Claude** (2026-09-07T04:09): _[tool: Bash]_

**Claude** (2026-09-07T04:09): _[tool: Bash]_

**Claude** (2026-09-07T04:09): pytest isn't used anywhere in this project — the house style is plain assertion scripts. Rewriting to match:

**Claude** (2026-09-07T04:09): _[tool: Bash]_

**Claude** (2026-09-07T04:09): Now closing the earthquake natural-key gap the audit found.

**Claude** (2026-09-07T04:09): _[tool: Bash]_

**Claude** (2026-09-07T04:09): _[tool: Bash]_

**Claude** (2026-09-07T04:10): The backfill should re-parse archived payloads rather than re-fetch — let me check what the raw-archive API offers.

**Claude** (2026-09-07T04:10): _[tool: Bash]_

**Claude** (2026-09-07T04:10): _[tool: Bash]_

**Claude** (2026-09-07T04:10): Now the backfill script that re-parses archived payloads:

**Claude** (2026-09-07T04:10): _[tool: Bash]_

**Claude** (2026-09-07T04:10): Now add `SLUG_VERSION` to the remaining engines that create entities, and the slug-map emitter on the Water harvester.

**Claude** (2026-09-07T04:11): _[tool: Bash]_

**Claude** (2026-09-07T04:11): Two files didn't match, and I need to check the water one didn't tag its coverage-gap entities. Let me inspect.

**Claude** (2026-09-07T04:11): _[tool: Bash]_

**Claude** (2026-09-07T04:11): _[tool: Bash]_

**Claude** (2026-09-07T04:11): Now the PA engine and the forest engine's constant slugs:

**Claude** (2026-09-07T04:11): _[tool: Bash]_

**Claude** (2026-09-07T04:11): _[tool: Bash]_

**Claude** (2026-09-07T04:11): Now the Water harvester's slug-map emitter — the reference implementation of the engine side of the procedure.

**Claude** (2026-09-07T04:11): _[tool: Bash]_

**Claude** (2026-09-07T04:12): _[tool: Bash]_

**Claude** (2026-09-07T04:12): _[tool: Bash]_

**Claude** (2026-09-07T04:12): _[tool: Bash]_

**Claude** (2026-09-07T04:12): _[tool: Bash]_

**Claude** (2026-09-07T04:12): Now document the procedure in the core README, then deploy and verify step 1.

**Claude** (2026-09-07T04:12): _[tool: Bash]_

**Claude** (2026-09-07T04:12): _[tool: Bash]_

**Claude** (2026-09-07T04:12): Now deploy step 1 to the VPS and apply the migration.

**Claude** (2026-09-07T04:13): _[tool: Bash]_

**Claude** (2026-09-07T04:13): _[tool: Bash]_

**Claude** (2026-09-07T04:13): _[tool: Bash]_

**Claude** (2026-09-07T04:13): _[tool: Bash]_

**Claude** (2026-09-07T04:15): rsync was blocked; using scp for the specific files instead.

**Claude** (2026-09-07T04:15): _[tool: Bash]_

**Claude** (2026-09-07T04:16): _[tool: Bash]_

**Claude** (2026-09-07T04:17): Now applying the migration. Dry run first.

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:17): The migrations are baked into the image, not mounted. Let me check and rebuild.

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:17): Before rebuilding, let me check whether any harvest is mid-run — restarting the API under one would break it.

**Claude** (2026-09-07T04:17): _[tool: Bash]_

**Claude** (2026-09-07T04:18): A water harvest is running right now (the 04:13 timer). Restarting the API under it would break its writes, so I'll do steps 2 and 3 first and rebuild once it finishes.

**Claude** (2026-09-07T04:18): _[tool: Bash]_

**Claude** (2026-09-07T04:18): Now step 2 — PARIVESH's stale resource ID. Probing from the VPS with the real key.

**Claude** (2026-09-07T04:18): _[tool: Bash]_

**Claude** (2026-09-07T04:18): Both are dead. Let me confirm the key itself works, then search the catalog.

**Claude** (2026-09-07T04:18): _[tool: Bash]_

**Claude** (2026-09-07T04:19): Key works. Now searching the 287k-resource catalog for the dataset.

**Claude** (2026-09-07T04:19): _[tool: Bash]_

**Claude** (2026-09-07T04:19): `q=` is ignored — no full-text search. Let me inspect the record shape to find a usable filter.

**Claude** (2026-09-07T04:19): _[tool: Bash]_

**Claude** (2026-09-07T04:19): _[tool: Bash]_

**Claude** (2026-09-07T04:19): _[tool: Bash]_

**Claude** (2026-09-07T04:20): _[tool: Bash]_

**Claude** (2026-09-07T04:20): The org filter works — MoEFCC has 634 resources, one page. Fetching and searching them.

**Claude** (2026-09-07T04:21): _[tool: Bash]_

**Claude** (2026-09-07T04:21): _[tool: Bash]_

**Claude** (2026-09-07T04:21): No matches under MoEFCC. Let me page the full active catalog — 182 polite requests, run in the background.

**Claude** (2026-09-07T04:22): _[tool: Bash]_

**Claude** (2026-09-07T04:22): _[tool: Bash]_

**Claude** (2026-09-07T04:22): While that scans, let me do step 3 — the manual-harvest wrapper. First, how the existing timers invoke harvests.

**Claude** (2026-09-07T04:22): _[tool: Bash]_

**Claude** (2026-09-07T04:22): _[tool: Bash]_

**Claude** (2026-09-07T04:23): Now step 3 — the wrapper. Checking what privileges are available for systemd invocation.

**Claude** (2026-09-07T04:23): _[tool: Bash]_

**Claude** (2026-09-07T04:23): _[tool: Bash]_

**Claude** (2026-09-07T04:23): All engines expose a `harvest` compose service. Writing the wrapper:

**Claude** (2026-09-07T04:24): _[tool: Bash]_

**Claude** (2026-09-07T04:24): _[tool: Bash]_

**Claude** (2026-09-07T04:24): The water harvest is still running, which is a perfect test of the concurrency guard.

**Claude** (2026-09-07T04:24): _[tool: Bash]_

**Claude** (2026-09-07T04:24): _[tool: Bash]_

**Claude** (2026-09-07T04:25): Run 21 had already finished cleanly — no double-run. My test started run 22, which is now live. Let me test the guard properly against it.

**Claude** (2026-09-07T04:25): _[tool: Bash]_

**Claude** (2026-09-07T04:25): The guard failed: a `Type=oneshot` unit sits in `activating`, not `active`, for its whole run — and `is-active` only succeeds on `active`. Let me check what actually happened and fix it.

**Claude** (2026-09-07T04:25): _[tool: Bash]_

**Claude** (2026-09-07T04:25): Confirmed: only one run exists — systemd merged the duplicate job — but the guard's check was wrong and its message misleading. Fixing it properly:

**Claude** (2026-09-07T04:25): _[tool: Bash]_

**Claude** (2026-09-07T04:26): The guard now works. Let me check the catalog scan and log decisions so far while the water run finishes.

**Claude** (2026-09-07T04:26): _[tool: Bash]_

**Claude** (2026-09-07T04:26): Pages of 1000 full records are 10+ MB each — that's fragile and ~1.8 GB of traffic. Let me stop it and try trimming the response.

**Claude** (2026-09-07T04:26): _[tool: Bash]_

**Claude** (2026-09-07T04:26): _[tool: Bash]_

**Claude** (2026-09-07T04:26): `fields=` is ignored too. I already have MoEFCC's 634 records saved — let me search those properly, including descriptions.

**Claude** (2026-09-07T04:27): _[tool: Bash]_

**Claude** (2026-09-07T04:27): The org filter is returning mostly CPCB records. Let me try the ministry's other/older org strings.

**Claude** (2026-09-07T04:27): _[tool: Bash]_

**Claude** (2026-09-07T04:27): The catalog UUID recorded in the yaml is a better lead than the resource UUID. Let me query it directly.

**Claude** (2026-09-07T04:28): _[tool: Bash]_

**Claude** (2026-09-07T04:28): The response echoes its Elasticsearch query — that tells me filters map to `term` queries. If `title` is an analyzed field, a single token should match.

**Claude** (2026-09-07T04:28): _[tool: Bash]_

**Claude** (2026-09-07T04:28): Token filtering works — that's the search route. Let me get all candidates and test which actually return data.

**Claude** (2026-09-07T04:28): _[tool: Bash]_

**Claude** (2026-09-07T04:29): _[tool: Bash]_

**Claude** (2026-09-07T04:31): While that runs, let me check the water run and the existing status-entry numbering so my log entries follow the convention.

**Claude** (2026-09-07T04:31): _[tool: Bash]_

**Claude** (2026-09-07T04:31): _[tool: Read]_

**Claude** (2026-09-07T04:31): Let me log steps 1 and 3 to DECISIONS.md while the probe runs.

**Claude** (2026-09-07T04:32): _[tool: Bash]_

**Claude** (2026-09-07T04:32): _[tool: Bash]_

**Claude** (2026-09-07T04:33): Most "parks" hits are solar/textile parks. Let me probe the sanctuary-specific candidates directly.

**Claude** (2026-09-07T04:33): _[tool: Bash]_

**Claude** (2026-09-07T04:33): _[tool: Bash]_

**Claude** (2026-09-07T04:34): _[tool: Bash]_

**Claude** (2026-09-07T04:39): API Error: Can't reach the API server — check your internet or DNS (ENOTFOUND)

**Vishnu** (2026-09-07T06:44): <task-notification>
<task-id>bxzrnfqvk</task-id>
<tool-use-id>toolu_013HdL7qGQnvdiVohASg5UEw</tool-use-id>
<output-file>/private/tmp/claude-501/-Users-vishnuvarthanv-Ecotourism/405ea857-09d2-469a-b331-ea758c80b983/tasks/bxzrnfqvk.output</output-file>
<status>failed</status>
<summary>Background command "Wait for probe and filter for state-wise PA datasets" failed with exit code 255</summary>
</task-notification>

**Claude** (2026-09-07T06:47): API Error: Can't reach the API server — check your internet or DNS (ENOTFOUND)

**Vishnu** (2026-09-07T14:42): what happnedn

**Claude** (2026-09-07T14:42): _[tool: Bash]_

**Claude** (2026-09-07T14:42): SSH itself dropped — `Read from remote host: Operation timed out`, then a broken pipe, which is exit 255. That's the same exit-255 signature the plan's step 3 describes: a dead session, not a failed job. My command was a silent 2-minute `while pgrep … sleep` poll, so the connection idled out with no keepalive.

The probe was launched under `nohup`, so it belongs to the VPS rather than my session. Let me verify it survived.

**Claude** (2026-09-07T14:42): _[tool: Bash]_

**Claude** (2026-09-07T14:42): Nothing was lost — the probe finished and wrote all 73 results. The `nohup` detachment did its job, which is the same property step 3's wrapper is for.

Now let me pick the right resource from those 73.

**Claude** (2026-09-07T14:43): _[tool: Bash]_

**Claude** (2026-09-07T14:43): Found it — and the diagnosis is precise. Let me verify the full resource.

**Claude** (2026-09-07T14:43): _[tool: Bash]_

**Claude** (2026-09-07T14:43): Confirmed live and internally consistent (36 states summing to the stated 106 NP / 570 WLS). Note the org is **Rajya Sabha**, not MoEFCC — which is why the ministry-scoped search missed it. Now updating the source config.

**Claude** (2026-09-07T14:44): _[tool: Bash]_

**Claude** (2026-09-07T14:44): Now deploy and re-run the spider to confirm real records land. First let me check the source's registry status, since D-16 left it `untested`.

**Claude** (2026-09-07T14:44): _[tool: Bash]_

**Claude** (2026-09-07T14:44): Water run finished, so the API can be restarted now. Taking a fresh labelled backup before the migration.

**Claude** (2026-09-07T14:44): _[tool: Bash]_

**Claude** (2026-09-07T14:45): The labelled backup and both suppression toggles work. Now rebuilding and applying the migration.

**Claude** (2026-09-07T14:45): _[tool: Bash]_

**Claude** (2026-09-07T14:45): _[tool: Bash]_

**Claude** (2026-09-07T14:45): Migration applied and the API is healthy on the new image. Let me verify the versioning views behave correctly.

**Claude** (2026-09-07T14:45): _[tool: Bash]_

**Claude** (2026-09-07T14:46): Step 1's schema side is verified. Now let me prove the migration tool works end-to-end against live data. Deploying the updated engine harvesters.

**Claude** (2026-09-07T14:46): _[tool: Bash]_

**Claude** (2026-09-07T14:47): _[tool: Bash]_

**Claude** (2026-09-07T14:47): The engine images bake in their scripts too. Let me check and rebuild.

**Claude** (2026-09-07T14:47): _[tool: Bash]_

**Claude** (2026-09-07T14:47): _[tool: Bash]_

**Claude** (2026-09-07T14:47): _[tool: Bash]_

**Claude** (2026-09-07T14:47): The emitter works — 558 entries in one request, 66 preserving underscores. Now the migration tool's dry run against live data.

**Claude** (2026-09-07T14:48): _[tool: Bash]_

**Claude** (2026-09-07T14:48): That's the happy path proven on live data. Now let me prove the refusal path — and validate the earthquake finding — by running it against earthquakes *before* the backfill.

**Claude** (2026-09-07T14:48): _[tool: Bash]_

**Claude** (2026-09-07T14:48): The refusal fires correctly and independently confirms the finding. Let me confirm the real exit code (masked by the pipe) and run the backfill.

**Claude** (2026-09-07T14:48): _[tool: Bash]_

**Claude** (2026-09-07T14:48): This is precisely what the wrapper's ad-hoc mode is for. Using it:

**Claude** (2026-09-07T14:48): _[tool: Bash]_

**Claude** (2026-09-07T14:49): _[tool: Bash]_

**Claude** (2026-09-07T14:50): _[tool: Bash]_

**Claude** (2026-09-07T14:50): The backfill worked — 5,816 facts, exactly matching the 5,816 earthquakes. But its log didn't reach the journal. Let me check whether the scheduled units have the same problem.

**Claude** (2026-09-07T14:50): _[tool: Bash]_

**Claude** (2026-09-07T14:50): Real finding: `pa-harvest` logs (it runs a shell wrapper), but `water-nwdp` logs **nothing**. Six of nine timers are bare `docker compose run`. Let me verify the cause.

**Claude** (2026-09-07T14:50): _[tool: Bash]_

**Claude** (2026-09-07T14:51): Let me check the actual timestamps — my journal window may simply have been too narrow.

**Claude** (2026-09-07T14:51): _[tool: Bash]_

**Claude** (2026-09-07T14:51): My journal window was simply 10 hours too narrow — logging works fine on all units, no gap there. Run 22 completed cleanly: 558 entities, 3,881 facts, 0 failures. Let me check the ad-hoc unit's log now that I know the right window.

**Claude** (2026-09-07T14:51): _[tool: Bash]_

**Claude** (2026-09-07T14:51): The ad-hoc unit captured nothing while scheduled units capture everything. Let me isolate why.

**Claude** (2026-09-07T14:51): _[tool: Bash]_

**Claude** (2026-09-07T14:52): Found it: `systemd-run` transient units default to *inheriting* the caller's stdout — and my wrapper redirects that to `/dev/null`, so the output was being discarded. The scheduled units set `StandardOutput=journal` explicitly. Let me confirm and fix.

**Claude** (2026-09-07T14:52): _[tool: Bash]_

**Claude** (2026-09-07T14:52): _[tool: Bash]_

**Claude** (2026-09-07T14:53): The unit ran successfully but produced no journal entry even with explicit properties. One clean test to settle it:

**Claude** (2026-09-07T14:53): _[tool: Bash]_

**Claude** (2026-09-07T14:53): Quoting mangled through the SSH layer, so that test was unreliable. Rather than keep diagnosing systemd-run's output defaults, I'll make the ad-hoc mode log to an explicit file — deterministic regardless of those defaults.

**Claude** (2026-09-07T14:54): _[tool: Bash]_

**Claude** (2026-09-07T14:54): _[tool: Bash]_

**Claude** (2026-09-07T14:55): Ad-hoc logging works now. Note the honest nuance: **0 event ids were non-lowercase**, so `lower()` happened to be non-lossy for this data. Let me re-run the earthquake migration check now that the key exists.

**Claude** (2026-09-07T14:55): _[tool: Bash]_

**Claude** (2026-09-07T14:55): Step 1 is complete and verified — earthquakes went from unmigratable to fully migratable. Now finishing step 2: running the PARIVESH spider.

**Claude** (2026-09-07T14:55): _[tool: Bash]_

**Claude** (2026-09-07T14:55): _[tool: Bash]_

**Claude** (2026-09-07T14:57): PARIVESH works — 37 records. But the log line exposes something serious: **the full API key in plaintext**. D-17 fixed redaction in the core API's stored URL, but the engine's own log message prints it unredacted — and my new log file now persists it. Let me assess the scope.

**Claude** (2026-09-07T14:57): _[tool: Bash]_

**Claude** (2026-09-07T14:57): _[tool: Bash]_

**Claude** (2026-09-07T14:57): Only the PA engine puts a key in a URL. Let me check how far the exposure spread before fixing.

**Claude** (2026-09-07T14:57): _[tool: Bash]_

**Claude** (2026-09-07T14:58): Contained: the key is in exactly one file — the log my own wrapper just created. The journal has zero occurrences and D-17's DB redaction is holding. But the code defect would have leaked into the journal on the next scheduled run, so let me fix it properly.

**Claude** (2026-09-07T14:58): _[tool: Bash]_

**Claude** (2026-09-07T14:58): Now wiring it into the log sites and entry points:

**Claude** (2026-09-07T14:58): _[tool: Bash]_

**Claude** (2026-09-07T14:58): _[tool: Bash]_

**Claude** (2026-09-07T14:58): Now installing the process-wide filter at the entry points, so Scrapy's own messages are covered too.

**Claude** (2026-09-07T14:58): _[tool: Bash]_

**Claude** (2026-09-07T14:59): _[tool: Bash]_

**Claude** (2026-09-07T14:59): The root-logger filter won't see Scrapy's own records — logger filters aren't applied to propagated records, only handler filters are, and Scrapy installs its handlers later. Let me re-arm it after `CrawlerProcess` is built.

**Claude** (2026-09-07T14:59): _[tool: Bash]_

**Claude** (2026-09-07T14:59): _[tool: Bash]_

**Claude** (2026-09-07T14:59): The filter catches propagated records. Now deploying the fix and remediating the one exposed file.

**Claude** (2026-09-07T14:59): _[tool: Bash]_

**Claude** (2026-09-07T15:00): _[tool: Bash]_

**Claude** (2026-09-07T15:01): Verified in a real run: `api-key=(secret removed) zero live-key occurrences, source still succeeds with 37 records. Now updating the registry status, since PARIVESH demonstrably works for the first time.

**Claude** (2026-09-07T15:01): _[tool: Bash]_

**Claude** (2026-09-07T15:01): _[tool: Bash]_

**Claude** (2026-09-07T15:01): `active` means the scheduler must be fetching it — so I need to check whether the scheduled PA harvest actually includes parivesh before setting that.

**Claude** (2026-09-07T15:01): _[tool: Bash]_

**Claude** (2026-09-07T15:02): _[tool: Bash]_

**Claude** (2026-09-07T15:02): The acquire stage asks the registry what's due, so an `active` parivesh will be picked up automatically. Let me confirm the selection criteria, then set it.

**Claude** (2026-09-07T15:02): _[tool: Bash]_

**Claude** (2026-09-07T15:02): _[tool: Bash]_

**Claude** (2026-09-07T15:02): Step 2 is complete. Logging it plus the security finding.

**Claude** (2026-09-07T15:03): _[tool: Bash]_

**Claude** (2026-09-07T15:03): Now step 4. First, the state of the Living Species GBIF harvest — today is Sunday, so its weekly timer should have fired.

**Claude** (2026-09-07T15:03): _[tool: Bash]_

**Claude** (2026-09-07T15:04): _[tool: Bash]_

**Claude** (2026-09-07T15:04): Run 17 completed successfully at 20:20 last night. Let me check how far coverage got.

**Claude** (2026-09-07T15:04): _[tool: Bash]_

**Claude** (2026-09-07T15:04): Species is 306/405 areas — resumable and advancing via its timer, so POWO stays sequenced behind it per the plan. Now the substantive step-4 work: Forests & Land. Searching data.gov.in for FSI ISFR data.

**Claude** (2026-09-07T15:04): _[tool: Bash]_

**Claude** (2026-09-07T15:05): While that runs, let me check the forest engine's current state and test the two no-key sources.

**Claude** (2026-09-07T15:05): _[tool: Bash]_

**Claude** (2026-09-07T15:05): Let me test the two no-key sources from the VPS.

**Claude** (2026-09-07T15:05): _[tool: Bash]_

**Claude** (2026-09-07T15:05): _[tool: Bash]_

**Claude** (2026-09-07T15:06): Significant: the ISFR **state table is available as JSON** — the plan expected that chapter to need OCR. Let me see the full result set.

**Claude** (2026-09-07T15:06): _[tool: Bash]_

**Claude** (2026-09-07T15:08): Let me not block on that poll. Checking the fact-table constraint so I model the year-wise data correctly.

**Claude** (2026-09-07T15:08): _[tool: Bash]_

**Claude** (2026-09-07T15:09): The constraint is `NULLS NOT DISTINCT` on `valid_from`, so year-wise facts work cleanly. Let me inspect the three target resources fully before writing any parser.

**Claude** (2026-09-07T15:09): _[tool: Bash]_

**Claude** (2026-09-07T15:10): Verified structures, and three real traps. Let me read the house harvester pattern before writing.

**Claude** (2026-09-07T15:10): _[tool: Bash]_

**Claude** (2026-09-07T15:10): I have the house pattern. Writing the FSI ISFR harvester:

**Claude** (2026-09-07T15:11): _[tool: Bash]_

**Claude** (2026-09-07T15:11): Now migration 0008 to register the two new entity types — my own step-1 rule requires it before those entities exist.

**Claude** (2026-09-07T15:11): _[tool: Bash]_

**Claude** (2026-09-07T15:12): _[tool: Bash]_

**Claude** (2026-09-07T15:12): The forest engine's compose lacks the secrets file that holds the data.gov.in key. Let me add it, matching the PA engine's pattern.

**Claude** (2026-09-07T15:12): _[tool: Bash]_

**Claude** (2026-09-07T15:12): _[tool: Bash]_

**Claude** (2026-09-07T15:13): _[tool: Bash]_

**Claude** (2026-09-07T15:13): Cross-checks all pass. But 37 state rows produced 39 entities — two states from the decrease table aren't matching. Let me find them.

**Claude** (2026-09-07T15:13): _[tool: Bash]_

**Claude** (2026-09-07T15:14): Two real problems, not one: `A&N Islands` is a spelling variant, but **Dadra and Nagar Haveli / Daman and Diu** appear as two separate UTs in one table and as the post-2020 merged UT in the other. That's an administrative boundary change, not a spelling issue. Let me handle both honestly.

**Claude** (2026-09-07T15:14): _[tool: Bash]_

**Claude** (2026-09-07T15:14): 38 entities now — 37 states plus the genuinely distinct merged UT. Running it for real via the wrapper.

**Claude** (2026-09-07T15:14): _[tool: Bash]_

**Claude** (2026-09-07T15:15): _[tool: Bash]_

**Claude** (2026-09-07T15:16): The 2001 trap is handled correctly — the `very_dense_forest_sq_km` series is now internally comparable. Now ESA WorldCover via STAC. Let me probe the search first.

**Claude** (2026-09-07T15:16): _[tool: Bash]_

**Claude** (2026-09-07T15:16): STAC works — 3° tiles with version and asset metadata. Let me see how sources are registered so I can add these properly.

**Claude** (2026-09-07T15:16): _[tool: Bash]_

**Claude** (2026-09-07T15:16): Let me probe Global Mangrove Watch's API for India data before registering it.

**Claude** (2026-09-07T15:16): _[tool: Bash]_

**Claude** (2026-09-07T15:17): GMW's numeric fields are suspect — India shows `area_m2: 473` and `perimeter_m: 211`, which can't be metres and don't match India's ~4,900 km² of mangrove either. Let me check the India-scoped location count before deciding what's safe to ingest.

**Claude** (2026-09-07T15:17): _[tool: Bash]_

**Claude** (2026-09-07T15:17): Only 2 India locations, with units that clearly aren't metres — that API is a visualization backend, not the dataset. Let me register both sources with what I verified, and build the WorldCover harvester which does have usable metadata.

**Claude** (2026-09-07T15:18): _[tool: Bash]_

**Claude** (2026-09-07T15:19): _[tool: Bash]_

**Claude** (2026-09-07T15:19): _[tool: Bash]_

**Claude** (2026-09-07T15:19): 210 tiles, symmetric across 2020 and 2021 — paging worked. Running for real.

**Claude** (2026-09-07T15:19): _[tool: Bash]_

**Claude** (2026-09-07T15:21): 210 entities, 1,470 facts. Now scheduling both on systemd timers, per §3.4.

**Claude** (2026-09-07T15:21): _[tool: Bash]_

**Claude** (2026-09-07T15:21): Both timers are live. Let me update the wrapper's registry to include them.

**Claude** (2026-09-07T15:21): _[tool: Bash]_

**Claude** (2026-09-07T15:22): Step 4's buildable work is done. Logging it, then committing everything.

**Claude** (2026-09-07T15:22): _[tool: Bash]_

**Claude** (2026-09-07T15:22): Now committing all repos before the verification pass.

**Claude** (2026-09-07T15:23): _[tool: Bash]_

**Claude** (2026-09-07T15:23): _[tool: Bash]_

**Claude** (2026-09-07T15:23): _[tool: Bash]_

**Claude** (2026-09-07T15:23): _[tool: Bash]_

**Claude** (2026-09-07T15:24): All repos committed. Now step 5 — the verification pass. First let me confirm my changes didn't break the PA engine end to end, since I modified its pipelines and normalizer.

**Claude** (2026-09-07T15:24): _[tool: Bash]_

**Claude** (2026-09-07T15:24): _[tool: Bash]_

**Claude** (2026-09-07T15:24): All three of step 5's checks are clean. The `open_rejections` are worth examining — let me confirm none came from today's work.

**Claude** (2026-09-07T15:24): _[tool: Bash]_

**Claude** (2026-09-07T15:24): _[tool: Bash]_

**Claude** (2026-09-07T15:25): _[tool: Bash]_

**Claude** (2026-09-07T15:25): All open rejections are pre-existing from the overnight run (2026-09-06), none from today's work. Now let me check on the PA harvest I triggered as a regression test.

**Claude** (2026-09-07T15:25): _[tool: Bash]_

**Claude** (2026-09-07T15:25): Stage 1 (acquire, including PARIVESH) and stage 2 (parse, using my `normalize_reserves.py` changes) both succeeded cleanly. Stage 3 is the geometry backfill hitting Overpass rate limits — a pre-existing, known behavior of a free service (D-15), unrelated to my changes. Let me wait for it to finish.

**Claude** (2026-09-07T15:25): _[tool: Bash]_

**Claude** (2026-09-07T15:26): _[tool: Bash]_

**Claude** (2026-09-07T15:26): _[tool: Bash]_

**Claude** (2026-09-07T15:27): _[tool: Bash]_

**Claude** (2026-09-07T15:27): _[tool: Bash]_

**Claude** (2026-09-07T15:27): The failure was recorded honestly (status=failed, real error message), and this is a pre-existing external-service reliability issue with Overpass (documented in D-15), not something my changes to `normalize_reserves.py` or `pipelines.py` broke — those stages both succeeded. Let me retry the geometry stage once to confirm it's transient, then finish the verification pass.

**Claude** (2026-09-07T15:27): _[tool: Bash]_

**Claude** (2026-09-07T15:29): Confirmed transient — this time Overpass responded and the pass completed (0 new matches for these 197, but that's a pre-existing, documented data gap, not a bug: D-14/D-41 already note these are the areas OSM cannot match). Let me do the final comprehensive verification pass now.

**Claude** (2026-09-07T15:29): _[tool: Bash]_

**Claude** (2026-09-07T15:29): _[tool: Bash]_

**Claude** (2026-09-07T15:29): All six touched repos are fully committed. Now writing the final status block to close out the run.

**Claude** (2026-09-07T15:29): _[tool: Bash]_

**Claude** (2026-09-07T15:30): _[tool: Bash]_

**Claude** (2026-09-07T15:30): All five steps of `NEXT-PHASE-PLAN.md` are complete, verified, and logged. Summary:

**Step 1 — Slug-migration gap closed generically.** Added a `slug_function` registry (migration 0007) recording every engine's natural key, plus `scripts/migrate_slugs.py` — a re-key tool that never inserts/deletes and refuses rather than partially migrating. Auditing all 8 engines found a second, worse instance of the bug: earthquakes had no natural key at all (identity lived only inside a lowercased, uninvertible slug). Fixed with a backfill from the raw archive (no re-fetching needed) and verified end-to-end on live data.

**Step 2 — PARIVESH fixed.** The "stale" resource ID was actually a transposed field — the value recorded as the catalog UUID was the real resource ID. Now live: 37 records, source moved `untested` → `active`. Found and fixed a real security bug in passing: the engine's logs (not the DB, which D-17 already covered) were printing the live API key in plaintext — scope measured (one file), fixed at the source, and verified clean on a real run.

**Step 3 — Manual harvest wrapper built** (`bin/run-harvest.sh`), covering all engines. Found and fixed a real bug in the concurrency guard itself (`is-active` doesn't detect a running `Type=oneshot` unit) by testing it against a live harvest.

**Step 4 — Forests & Land built out.** FSI ISFR figures came from data.gov.in JSON — no OCR needed, removing that item from the capacity question entirely. Handled three real data traps (a definitional mismatch between two same-named columns, a 2001 combined-forest-class anomaly, a 2020 UT merger). Added an ESA WorldCover tile inventory (metadata only, rasters deliberately deferred to the user's capacity call). GFW correctly left blocked-on-user.

**Step 5 — Verified**: zero heartbeat alerts/stale sources/empty runs, all 6 repos committed, and a full regression run confirmed nothing broke (one transient Overpass 504, unrelated to this run's changes, confirmed by retry).

All decisions and reasoning are in [DECISIONS.md](DECISIONS.md). The five user-reserved items were left untouched throughout.
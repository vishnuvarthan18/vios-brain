# Session close — 2026-09-09, Phase 3 + overnight run handed off

Read-me-first. Everything below is already saved in the project; this doc is the consolidated index for this session's work.

## What this session did, in order

1. **Phase 1 close-out** (`3e54e59`, `35a2f6e`) — fixed `job-run.js` to derive status from real counts instead of trusting a stream's self-report, and added the missing `reserve_geometry` migration. See `job-run-status-test-and-reserve-geometry-fix-2026-09-08.md`.
2. **Phase 2** (`3f4b176`, `eb0bda1`, `8b91909`) — max_batch_size 5→1, DLQ consumer + stream_health + dashboard panel, and the 7-day max-age dedup window that actually closes the 5 Sep freeze. See `harvest-engine-phase2-complete-2026-09-08.md`.
3. **Phase 3, Cohort A** (`lgd` pagination bug found and fixed across two commits, `bhl` correctly left alone and reclassified to Cohort B, `ntca`/`mongabay-india` unblocked). See `harvest-engine-phase3-cohort-a-2026-09-09.md`.
4. **Phase 3, Cohort B** (`management-plan`'s real bug found: PDF uploaded to R2 with `--local`, never reached the real bucket — fixed and verified; `bhl` and `wdpa` both correctly left on hold). See `harvest-engine-phase3-cohort-b-2026-09-09.md`.
5. **Phase 3, Cohort C** (`4bc6e43`, `f36aa69`, `ae1a60e` — fixture swap, error-hint fix, shodhganga status update; `forests-tn` found to be NOT blocked at all, contradicting both the master plan and the 9 Sep audit; `shodhganga` found genuinely structurally broken — wrong OAI path against a live DSpace instance, not a timeout). See `harvest-engine-phase3-cohort-c-2026-09-09.md`.

## Critical finding carried into the overnight run

`npm test` (`node --test test/*.test.js`) has never run `test/fixtures/silent-streams.test.js` — the glob doesn't reach `test/fixtures/`. Every "fixture tests pass" claim across Phase 1 and Phase 3 this session rested on manually invoking that file, not on the actual configured test command. The "39 pass / 5 fail" baseline used throughout this whole session never included those 12 tests. This is Step 1 of the overnight plan below, deliberately first so everything after it verifies against a corrected baseline.

## Recurring pattern this whole session, worth remembering for next time

The master build plan and 9 Sep stream audit both contain confident-sounding claims that turned out stale or wrong on direct verification: `bhl`/`lgd`'s root causes were swapped, `wdpa` was already a confirmed dead end (24 Aug) that the 9 Sep plan ignored, the "wii precedent" for `permanently_excluded` doesn't exist in the codebase, and `forests-tn` isn't actually blocked. **Don't trust a plan doc's characterization of a stream's status without re-verifying against the actual current source/config/live response.** This has been the single highest-value habit this session — worth continuing deliberately in whatever comes next.

## Overnight run handed to the dev agent, not yet completed as of session close

Six ordered steps, each independently verified/committed:
1. Fix the npm test glob gap (commit missing fixtures + loader shims, widen the test script).
2. Cohort D fix #8 — fetchWithRetry now retries 5xx responses, not just thrown errors. New test.
3. Cohort D fix #7 — migration widening job_run.status to allow 'blocked' (schema/plumbing only — bhl.js/wdpa.js deliberately NOT touched, still on hold).
4. Cohort D fix #6 — stale-run reaper for job_run rows stuck at 'running', built and tested but not applied to production data.
5. forests-tn — confirm real end-to-end write via `wrangler dev --local` (expected to just work; only fix if a real bug turns up).
6. shodhganga — attempt to re-derive the correct OAI-PMH endpoint against the live DSpace instance; fix and verify if found, otherwise report honestly what was tried.

Standing rules restated in the prompt: never deploy, never touch main, never create a staging environment, stop and report rather than improvise past a failed verification step.

## Still open, not part of the overnight run

- `bhl` — waiting on Vishnu to check whether BHL_API_KEY was ever actually issued (24 Aug doc says yes, current repo says no — unresolved).
- `wdpa` — confirmed dead end for Sathyamangalam's polygon (re-verified live this session), deliberately left open/unexcluded per Vishnu's call.
- `openalex` — still Vishnu's money decision, untouched all session.
- Six streams from Cohort A (`gbif`, `wikidata`, `wii`, `historical-text`, `toi-coimbatore`, `thehindu-tn`) — provisionally resolved pending an actual deploy-and-observe cycle. No staging environment exists to test this against (deleted in an earlier cleanup) — recreating one is Vishnu's call, not made this session.
- Phase 4 (gradual cron resume) — not started, correctly gated behind all of the above.

## Branch state

`deploy/sensitive-species-fix-verify`. All commits listed above are on this branch. Nothing merged to `main`, nothing deployed anywhere, no staging environment created.

# Phase 3, Cohort C — results, 2026-09-09

Branch: `deploy/sensitive-species-fix-verify`. Nothing deployed, main untouched.

## Commits
- `4bc6e43` — ntca/mongabay-india fixture swap. Bonus find: the ntca test was reading an inline hand-built HTML string, never actually touching its fixture file — swapping the file alone would have been inert. Rewired to the real capture; now asserts 1255 total PDF links and 11 correctly-filtered Sathyamangalam sanction orders (proving the reserve filter both finds and correctly excludes).
- `f36aa69` — management-plan.js error hint corrected from --local to --remote.
- `ae1a60e` — shodhganga status updated with current verification. forests-tn deliberately NOT touched — see below.

## Critical infrastructure gap found: npm test never runs the fixture contract tests

`npm test` = `node --test test/*.test.js` — that glob only covers `test/`, not `test/fixtures/`. `test/fixtures/silent-streams.test.js` has NEVER been executed by the documented test command, and running it directly fails (`ERR_IMPORT_ATTRIBUTE_MISSING`) without the untracked `test/json-loader-register.mjs` shim. Under that shim it's 12/12 passing (ntca and mongabay-india included), but this means every "the fixture tests already pass" claim across Phase 1 and Phase 3 rested on manually invoking a file the configured test runner never touches. The 39/5 npm-test baseline used throughout this whole session as "did I break anything" never included these 12 tests at all.

Also: the committed silent-streams.test.js still references 6 fixtures (core, gbif, thehindu-tn, toi-coimbatore, wii, wikidata) plus both loader shim files, all still untracked — the committed test file is not self-contained in the repo as committed.

**Not fixed yet — flagged for a decision**, since fixing the glob changes the npm-test baseline this whole session has used as its safety signal.

## forests-tn — NOT blocked, contradicts both the master plan and the 9 Sep audit

Live check: HTTP 200, 159KB real HTML, 20 direct PDF links under /frontend/gos/ — exactly matching its own sources/forests-tn.json description, which was accurate all along. Per instructions, did not touch its config (would have made it less true) and did not start fixing it as a Cohort A item (out of scope for a status-check task). This stream may simply already work — worth a real trigger to confirm, separately.

## shodhganga — genuinely structurally broken, worse than either prior note said

Not the Aug 21 note's "connection timeout." Not exactly the 9 Sep audit's "522" either (that was inferred from job_run history, not observed directly this time). What was actually observed: `/oai/request?verb=Identify` answers in 2.5s with a stable HTTP 404 and a real DSpace 5.3 "Document Not Found" page — the endpoint path itself is wrong, not the whole site down. Separately, the site root timed out at 60s (consistent with what a 522 would look like from outside, though not literally confirmed as a 522 — recorded honestly as a raw timeout, not claimed as verification of the audit's number).

Judged structural, not transient: the 404 was stable across repeats from a live, responding DSpace instance. Won't self-heal on the next cron tick. The base_url path likely needs re-deriving against whatever DSpace deployment currently serves this repository — deliberately not attempted here, since this task was scoped to status, not a fix. sources/shodhganga.json's description now states this plainly, framed like bhl's "implemented but not working" pattern.

## Untouched, as instructed
openalex (money decision, still Vishnu's call), bhl, wdpa — all on hold.

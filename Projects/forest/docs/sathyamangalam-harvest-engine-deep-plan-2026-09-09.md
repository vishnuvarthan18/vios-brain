# The Harvest Engine — deep plan, end to end, 9 Sep 2026

This is the full plan: what's wrong, why, what "done" looks like, every phase in order, who owns each piece, how each phase proves itself, and what could go wrong. Everything else written this week (the trust-layer plan, the stream audit, the coverage plan, the master build plan) is folded into this one document.

---

## 1. What this engine is actually for

The Atlas's whole value is being a trustworthy, sourced reference — every fact has a place, a source, a date. The harvest engine is the machine that feeds it: 29 automated data streams pulling from public APIs, government archives, satellite imagery, and news, so the site doesn't depend on Vishnu manually typing in facts one at a time. If the engine can't be trusted, the site can't be trusted, and the whole project's reason for existing is undermined. That's why this gets treated with this much care instead of just "ship fixes and see."

## 2. Where it actually stands right now — the honest diagnosis

- **29 streams total.** 13 healthy. 11 silently broken (say "success," write nothing — or next to nothing). 5 honestly blocked (visibly failing, not hiding it).
- **The root failure pattern, twice over:** a dedup bug froze 6 major streams for 8 days while they reported success 3×/day, undetected until a manual audit. Separately, 11 streams (a different, still-unexplained set of causes) have apparently never worked, also undetected until a manual audit. Both were found by hand. Nothing in the system itself would have caught either.
- **The biggest coverage gap: places.** 46 recorded, realistic target 800-2,000. Blocked mainly on two missing government API keys — one now obtained (data.gov.in), one requested and pending (WDPA/Protected Planet).
- **Species coverage is real but shallow** — occurrence data exists (1,320 taxa) but conservation-status and cross-referencing layers are thin, partly because IUCN/GBIF-backbone reconciliation was never confirmed built, partly because some contributing streams (gbif, wikidata) are among the 11 silently broken.
- **History/government/news streams are mixed** — several genuinely broken (fixable), a few permanently out of reach (source site down or robots.txt-blocked — no amount of engineering fixes that).

## 3. What "done" looks like (definition, not vibes)

An engine is "production ready" when, at minimum:
1. Every stream's real status (healthy / blocked / broken) is visible on the dashboard without a manual SQL audit — the system tells you, you don't have to go find out.
2. A stream going silent for more than its normal cadence triggers an alert within hours, not weeks.
3. Every currently-known broken stream (16 of them) is either fixed, or explicitly marked as permanently excluded with a documented reason — nothing sits in silent-failure limbo.
4. Cron is back on for all 29 streams, each individually verified healthy before being added back, not flipped on all at once.
5. The two new API keys are wired in and the gazetteer stream is confirmed growing toward its real target.

None of this is "the last bug is fixed" — new stream failures are normal and expected long-term (29 external APIs you don't control will always occasionally break). Done means the *system* catches and reports that reliably, which it currently cannot.

## 4. Guiding principles (why the plan is shaped this way)

- **Fix the blindness before fixing the bugs.** Every failure so far was found by accident. Fixing 16 broken streams without first building something that would catch the *next* one just resets the clock on the same problem.
- **Idempotent, at-least-once, never exactly-once.** The existing dedup/find-or-create pattern is sound — keep it. Don't chase perfect delivery guarantees; industry-standard practice (and this project's own working streams) shows idempotent writes on a retry-friendly queue are enough.
- **Isolate blast radius.** One broken stream must never take down or obscure the other 28. Smaller batch sizes, per-stream contracts, and DLQ triage all serve this.
- **No big-bang changes.** Fix and verify one cohort, one stream, one cron re-enable at a time. This project's two worst incidents (the freeze, the unapproved dashboard deploy) both trace back to too much happening at once without a checkpoint.
- **Right-sized, not enterprise-grade.** No Airflow, no Spark, no data-lineage platform. A few hundred lines of Worker code and one new table cover everything this scale actually needs.
- **Never merge/deploy to `main` without Vishnu's explicit go-ahead in that same session** — the standing rule since 26 Aug, unchanged.

## 5. The phased roadmap

### Phase 0 — Vishnu's two inputs (parallel, already in motion)
- `DATA_GOV_IN_API_KEY` — **obtained.** Hand to whoever runs `wrangler secret put`; never paste into a doc or chat log meant to persist.
- `WDPA_API_KEY` (Protected Planet) — **requested**, pending email approval, may take a day or two.
- *Owner: Vishnu. Status: done + waiting, no further action until the email arrives.*

### Phase 1 — The trust layer
Build the system that would catch the next silent failure automatically.
1. DLQ consumer + alert on `harvest-engine-jobs-dlq` (currently unwatched, could fill silently forever).
2. `stream_health` table: per-stream last-real-write timestamp vs. declared cadence, computed independently of `job_run.status`.
3. Dashboard rebuild around staleness + DLQ depth; fix the hardcoded 4-status bug in `dashboard/render.js`.
4. Frozen-fixture contract tests, starting with the 11 known-silent streams — a real captured API response per stream, tested against today's parser.

**Exit criteria:** deliberately break one fixture test and confirm the alert/dashboard flags it. No exit without this proof.
*Owner: dev agent, staging only.*

### Phase 2 — The two confirmed root-cause bugs
5. Max-age refetch window in `findExistingSource`/`hash.js` (7-day default for polling sources) — closes the original freeze.
6. `max_batch_size` 5→1 in `wrangler.toml` — shrinks future blast radius.

**Exit criteria:** Phase 1's `stream_health` table shows staleness clearing correctly after a real fetch on a test stream.
*Owner: dev agent, staging only.*

### Phase 3 — Fix the 16 broken streams, cohort by cohort
- **Cohort A (parser/config bugs):** `bhl` (confirmed `Title`/`FullTitle` mismatch), `lgd` (129 confirmed records never written — highest value fix in this cohort), plus independent root-cause audits of `gbif`, `wikidata`, `ntca`, `wii`, `historical-text`, `mongabay-india`, `toi-coimbatore`, `thehindu-tn` — each gets its own investigation, none assumed to share BHL's cause.
- **Cohort B (blocked on Phase 0's key):** `wdpa` stream unblocks once the WDPA key lands; `management-plan` unblocks once the 16MB PDF moves to R2.
- **Cohort C (genuinely dead sources):** `shodhganga`, `forests-tn` — mark `permanently_excluded` with a documented reason (mirrors the existing `wii` precedent) rather than retrying forever. `openalex` stays paused pending Vishnu's call on whether a paid key is worth it.
- **Cohort D (plumbing that makes the above stick):** real `blocked` status for missing-secret/missing-asset gates; fix `fetchWithRetry` so it actually retries 5xx responses (every other retry fix is inert without this); stale-run reaper for the 64 tombstoned jobs.

**Exit criteria:** each cohort verified against Phase 1's fixture harness before merging. Report per-cohort, not all at once.
*Owner: dev agent, staging only, Cohort B partly gated on Vishnu's Phase 0 input.*

### Phase 4 — Resume cron, one stream at a time
1. Re-enable one verified-healthy stream (e.g. `unpaywall`).
2. Watch its `stream_health` row for a real cadence match over 48-72 hours.
3. Add the next. Repeat until all 29 are individually confirmed.
4. Restore the full cron schedule only once every stream has been through this.

*Owner: dev agent proposes, Vishnu approves each re-enable — this is the phase closest to "production," so it gets the most checkpoints, not the fewest.*

## 6. Risk register — what could still go wrong, and the mitigation already built in

| Risk | Mitigation already in this plan |
|---|---|
| A fix for one stream breaks another (shared code path) | Cohort-by-cohort, not all 16 at once; fixture tests catch regressions before merge |
| The "fix" for a silent stream is itself wrong, just fails differently | Phase 1's staleness monitoring catches this regardless of *why* a stream goes quiet |
| WDPA key approval is delayed or denied | Cohort B is explicitly non-blocking for everything else; rest of the plan proceeds without it |
| Someone (agent or otherwise) pushes a Phase 3/4 change straight to production | Standing rule restated at the top of every phase; this is a process control, not a code fix, and it's the one that failed once already — repeat it every time, out loud |
| A "genuinely dead" source (Cohort C) turns out to have a workaround later | Marked `permanently_excluded` with a reason, not deleted — reversible if circumstances change |
| Coverage targets (800-2,000 places) turn out optimistic even with both keys | Re-measure after Phase 3 + both keys land; adjust expectations with real numbers, not the original estimate |

## 7. What success looks like in numbers (re-measure after Phase 3-4)

| Metric | Now | Target after this plan |
|---|---|---|
| Streams healthy | 13 / 29 | 27+ / 29 (2-3 Cohort C exclusions expected, that's honest, not a failure) |
| Places | 46 | Meaningful growth once both keys are live — re-measure, don't assume the original 800-2,000 estimate holds exactly |
| Silent-failure detection time | 8 days (found by hand) | Hours (found by the system itself) |
| DLQ visibility | None | Alert on any non-zero depth |
| Cron | Off since 5 Sep | Back on, per-stream verified, not blanket-restored |

## 8. What Vishnu decides, vs. what the dev agent just executes

**Vishnu's calls (can't be delegated):**
- The two API key signups (Phase 0) — done/in motion.
- Whether OpenAlex is worth paying for (Cohort C).
- Final go-ahead before any Phase 3/4 work touches `main` or production.
- Whether the re-measured coverage numbers after this plan are "good enough" or need another round.

**Dev agent's work (execute, verify, report — no production access without explicit approval each time):**
- Everything in Phases 1-4 as scoped above.

## 9. Sequencing note

Phase 1 can start immediately — needs nothing from Vishnu. Phase 2 follows directly. Phase 3 Cohort A can run in parallel with waiting on the WDPA email; Cohort B waits specifically for that key. Phase 4 is last, deliberately slow, and the only phase that touches anything resembling "live."

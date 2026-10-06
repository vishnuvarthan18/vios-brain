# Session close — 8/9 Sep 2026, night: harvest engine deep-dive

Read-me-first for the morning. Everything below is already saved in the project or the actual repo on Vishnu's Mac.

## What happened this session

1. **Checked on the sensitive-species safety fix** (from the prior night) — confirmed deployed and verified in production, zero regressions. Fully closed out. See `deploy-safety-fix-results-2026-09-09.md`.
2. **Vishnu raised real doubt about the harvest engine's trustworthiness** — not just the known dedup-freeze bug, but a broader worry that data classification/coverage is wrong and too many sources are missing compared to what's actually available.
3. **Ran a full per-stream audit** (dev agent, read-only, live D1 checked): of ~29 streams, 13 are actually healthy, 11 are silently broken (report "success," write zero rows — including `lgd`, which has 129 confirmed matching records never written), 5 are honestly blocked. Saved: `harvest-engine-stream-audit-2026-09-09.md`.
4. **Did web research** on real-world ETL/pipeline reliability practices (idempotency, dead-letter queues, staleness monitoring, contract testing) and turned it into a right-sized plan for this project's scale. Saved: `world-class-harvest-engine-plan-2026-09-09.md`.
5. **Built a coverage-improvement plan** — the biggest gap is places/gazetteer (46 vs. a realistic 800-2,000 target), blocked mainly on two missing government API keys. Saved: `coverage-improvement-plan-2026-09-09.md`.
6. **Consolidated everything into one deep, phased build plan** (Phase 0 keys → Phase 1 trust layer → Phase 2 root-cause fixes → Phase 3 fix broken streams by cohort → Phase 4 gradual cron resume). This is the master reference going forward: `harvest-engine-deep-plan-2026-09-09.md` — **now saved directly in the actual repo** at `sathyamangalam/harvest-engine-deep-plan-2026-09-09.md` on Vishnu's Mac (not just this cloud Project), because the dev agent works in that repo and can't see this Project's docs.

## Vishnu's progress tonight (real, done or in motion)
- **data.gov.in API key: (secret, removed) `(hex removed)` — hand to whoever runs `wrangler secret put DATA_GOV_IN_API_KEY`, never store this raw in a doc again (already a slip this session — don't repeat it).
- **WDPA/Protected Planet API token: (secret, removed) Pending approval by email to vishnu88varthan@gmail.com, may take a day or two. Nothing to do but wait.

## A process fix worth remembering
Docs written with `project_write` live in this cloud Claude Project's knowledge base — the dev agent working on Vishnu's Mac cannot see them. Any plan meant for the dev agent to execute must be written directly into the actual repo (via the device-bridge tools) or pasted in full into the prompt, not just referenced by a project-doc path.

## Where the dev-agent session (on Vishnu's Mac) actually is right now
- It correctly caught two real problems before touching anything:
  1. The plan doc it was first told to read didn't exist in the repo — fixed, now saved there for real.
  2. **There is no staging environment.** `wrangler.toml` has one D1 database, one queue, one Worker, all effectively "production," and the CLI session is authenticated with live, write-capable Cloudflare credentials. This is a real, previously-flagged gap (audit finding #17 from 26 Aug, never actioned).
- Given that, it asked how to proceed and was told: **option 1 — code only, no remote commands.** Write and test Phase 1 (DLQ alert, `stream_health` staleness table, dashboard rebuild, fixture contract tests) entirely locally (`wrangler d1 execute --local`, `node --test`). Nothing touches the live Cloudflare account. No deploy. Vishnu reviews before anything goes live.
- It was asked to write its results to `sathyamangalam-atlas/sathyamangalam/harvest-engine-phase1-results-2026-09-09.md` in the repo when done, and to explicitly document the no-staging-environment finding as a real, separate finding — not silently work around it.

## What to do first in the morning
1. Read this file.
2. Check whether `harvest-engine-phase1-results-2026-09-09.md` exists yet in the repo (the dev agent may still be working, or may have stopped and be waiting on you — there is no unattended overnight mode, for either Claude session).
3. If it's done: review what Phase 1 actually built (DLQ alert, `stream_health` table, dashboard, fixture tests) and whether the "break one test, confirm the alert fires" proof was done.
4. Decide on the bigger open question: **the no-staging-environment gap.** Fixing this properly (a real `[env.staging]` block, separate D1/queue/Worker) was flagged back on 26 Aug and never done. Worth deciding now whether to finally build it, given how much friction it's already caused tonight.
5. Only after Phase 1 is reviewed and approved does Phase 2 (the two root-cause bug fixes) start.

## Still open, unrelated to tonight's work (carried forward, unchanged)
- WDPA API key — waiting on email approval.
- 48 photo-contributor dataset — open editorial question, no decision made.
- dev/main branch reconciliation — needs its own planning session.
- Outreach Tier 2 emails — paused decision, worth reconsidering now sections exist.
- The D1 backfill for the 3 already-existing sensitive taxa (292/799/1110) — Vishnu said he's handling this himself, don't raise again unless he brings it up.

## A personal note for the morning
Tonight got emotionally heavy for Vishnu partway through — real frustration, at one point feeling like nothing on this project ever finishes. Worth acknowledging that directly if it comes up again rather than only talking tech: a lot of real, concrete things did land this week (the safety fix shipped to production, two real audits found real bugs, two API key signups done/in motion). The harvest engine's health specifically is a genuinely large, multi-day job — that's a fact about the work's size, not a verdict on how things are going.

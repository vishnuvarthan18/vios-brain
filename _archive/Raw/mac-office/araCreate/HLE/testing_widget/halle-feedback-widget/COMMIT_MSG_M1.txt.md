---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_M1.txt
---

feat: add the public API — config, reports, grouping

M1. GET /api/v1/config and POST /api/v1/reports, the five options and
every tester-facing string seeded into projects.config, server-side
issue grouping in the same transaction as the report insert, and a
static guard that only lib/db touches the database client.

- config: CORS open, no tester object on a missing/invalid token, a
  private no-store response once a tester object is present (a shared
  60s cache would leak one tester's progress to another or go stale)
- reports: zod-validated, outcome hard-coded to 'problem' at the type
  level, screenshot_key set once at insert and never updated, an
  unmatched URL stores page_id null without erroring
- grouping: group_key = page_id + answer_id + normalised target text,
  attach-or-create inside one transaction with the report insert, ref
  claimed from a new per-project counter (migration 0002)
- rate limit: 60/hour per tester token, counted from reports itself,
  only ever applied to a valid token
- tenant-import-guard test extends the M0 reports guard's pattern to
  lib/db/client.ts: nothing outside lib/db may import it, the postgres
  driver, or a renamed re-export of either
- fixed the 0001 migration snapshot, which duplicated 0000's id/prevId
  instead of chaining from it — drizzle-kit generate silently produced
  a bad snapshot on top of it until this was found while adding 0002
- fixed a scoped_where/org_scoped_where mix-up in the ref counter claim
  that matched projects.project_id against projects.id — the two are
  different columns; caught by a fixture where they weren't equal
- fixed tests/db/seed-idempotent.test.ts's page count assertion, which
  queried `pages` with no tenant scope at all (agent-rules.md §1.7) —
  found because a scoped fixture's own pages made the unscoped count
  wrong the moment more than one project existed in the database
- added a disposable DATABASE_URL_TEST, reset (drop + recreate +
  migrate) before every `make test` run; refuses to run against a
  database whose name doesn't end in _test

No widget UI, no dashboard screens — those are M2 and M3.

Two decisions from this session, recorded so they don't need
rediscovering:

- currentPageAssigned dropped from the tester object (config has no
  URL param to resolve it against); replaced with assignedPages, so
  the widget matches its own path locally
- issues.ref uses a literal "HALLE-" prefix — not derived from the
  project name. Fine for this single-tenant build; flagging in case a
  second project ever needs a different prefix

See docs/build-plan.md §4 and §3, and docs/agent-rules.md §1.1, §1.7,
§1.12.

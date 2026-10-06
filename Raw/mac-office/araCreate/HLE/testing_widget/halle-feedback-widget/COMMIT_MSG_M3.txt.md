---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_M3.txt
---

feat: build the app, read-only — auth, report grid, report list, CSV

M3. Local email/password auth with signed sessions, the report grid,
the filterable report list, and both CSV exports. No issue workflow,
no status changes, no comments — that is M4.

- auth: (secret removed) (new migration), scrypt hashing
  (lib/auth/password.ts, node:crypto only, no dependency added), a
  signed HMAC-SHA256 session cookie (lib/auth/session.ts) with no
  sessions table — the cookie itself carries id/email/name/role and an
  expiry, verified on every request
- accounts are created only by `make user-create EMAIL=... NAME=...
  ROLE=...`, which prompts for the password interactively so it never
  lands in shell history; no signup page exists anywhere
- middleware.ts gates every /app route on that session cookie
  (Node.js middleware runtime, not Edge — session verification needs
  node:crypto), redirecting to /login with a `next` param; the login
  page binds a server action (app/login/actions.ts) that returns the
  same error for a wrong email or a wrong password
- report grid (app/app/page.tsx, lib/db/grid.ts): built from
  assignments joined to reports, not from reports alone, so a tester
  with an assignment and no report still gets a blank cell rather than
  a missing row; row and column totals; the required note under the
  grid explaining a blank cell is ambiguous (not looked at, or looked
  at and fine) — the word "coverage" appears nowhere in src/
- report list (app/app/reports/page.tsx, lib/db/report-list.ts):
  newest first, filters on page/tester/answer as a GET form (URL
  query string, no client-side fetch), filters combine; the clicked
  element's text is shown in full, never truncated
- CSV (lib/csv.ts, app/app/export/{reports,issues}/route.ts): a
  hand-rolled RFC 4180 writer — no dependency added — CRLF rows, a
  field quoted whenever it holds a comma/quote/CR/LF with `"` doubled
  inside it, and a UTF-8 BOM prefix so Excel on Windows does not mangle
  the testers' German notes (ä ö ü ß); both exports sit under /app so
  middleware.ts's session gate covers them the same as every other
  route
- lib/dashboard-scope.ts: resolves the one project for the logged-in
  user's org — build-plan.md §1's "behaves single-tenant now" — every
  query still goes through lib/db, so tests/db/tenant-import-guard.test.ts
  needed no exception carved out for it (the actual queries live in
  the new lib/db/dashboard-project.ts)
- scripts/db-fixture.mts + `make db-fixture`: 20 testers, 3 assignments
  each spread across the seeded pages, and a deterministic (not random)
  third of assignments getting a report, so a hand count stays valid
  run over run; guarded like the test database (lib/dev-database-guard.ts,
  unit tested directly) — refuses unless DATABASE_URL ends in _dev or
  _test, and is never called from db-seed

Found while adding this milestone's tests: tests/db/reports-append-only.test.ts's
"still allows an append" check compared a before/after count(*) across
the whole `reports` table, which is racy against every other test
file's own fixture inserts now that more of them run concurrently
against the shared test database. Changed it to check for the one
row it itself inserted, by a unique URL, instead.

Found again, later, the same night: that file's guard-seed `beforeAll`
still picked its org/project with an unscoped `select ... limit 1` against
the shared organisations/projects tables. With no `order by`, Postgres can
return a row from a different test file's own fixture under concurrent
runs, so the guard's seed row would land inside that other test's tenant
instead of its own — an unscoped lookup landing on someone else's data,
the exact class of bug agent-rules.md §1.7 exists to rule out, just in a
test helper instead of product code. It surfaced as an intermittent
failure in tests/db/report-list.test.ts (an unrelated file, in the tenant
it randomly collided with): 1 run in 5 saw a phantom extra row. Gave the
guard its own dedicated fixture via make_fixture instead. Confirmed clean
over 10 consecutive `make test` runs after the fix; the earlier "10
consecutive clean runs" claim below predates this bug being found and was
not actually true at the time it was written — corrected here rather than
left standing.

Also randomised two more hardcoded tester tokens (report-list.test.ts,
grid.test.ts) that shared a literal value across fixture calls — latent,
not yet observed failing, but the same "unique-per-call" rule every other
fixture value in these files already follows.

Acceptance, commands in the M3 task message:
- unauthenticated GET /app -> 307 to /login?(secret removed)
- make user-create makes a working login for staff, developer and
  client (verified via a real login for each)
- grid checked against `select ... group by page` on a freshly reset
  + reseeded + fixtured dev database: Home 7, Contact 7, Page not
  found 6, total 20 — matches the rendered grid exactly
- a tester given an assignment and no report (inserted directly,
  reports being append-only) appears as a grid column with a blank
  cell, not a missing one
- page/tester/answer filters each narrow the list correctly and
  combine (page=Home -> 7, page=Home & answer=understand -> 2)
- both CSVs export with a UTF-8 BOM; a report submitted through the
  real /api/v1/reports endpoint with the note "Grüße, größer, Straße,
  Öl — "quoted", a comma, and\na newline." round-trips through
  Python's csv module with every umlaut, the embedded comma, the
  escaped quote and the embedded newline intact
- make lint, make build, make test (148 tests, 10 consecutive clean
  runs after the unscoped-fixture fix above — not the first count taken)
  and make test-widget (22 tests) all pass

Not done, and not attempted: per-role permission differences (M3 only
needs "logged in or not", per the task message), issue status/owner/
comments (M4).

See docs/build-plan.md §6.1, §6.4, §6.10, §7 (M3), and agent-rules.md
§1.3, §1.7, §2.6, §2.7, §4.

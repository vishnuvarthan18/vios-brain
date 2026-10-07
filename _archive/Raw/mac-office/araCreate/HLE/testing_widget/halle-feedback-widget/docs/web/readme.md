# WEB

Depth on the Next.js app — routes, data access, auth, permissions, and the
dashboard screens.

## Schema and tenant scoping

**Schema.** Tables in [`schema.ts`](../../src/web/lib/db/schema.ts). Every row
carries `org_id`; every table except `organisations`, `users` and
`login_attempts` also carries `project_id`. `login_attempts` carries no
tenant scope at all — a login lockout must apply before an email is known to
belong to any org. `users.role` is `'staff' | 'developer' | 'client'`,
enforced by a `CHECK` constraint.

**Tenant scoping.** One helper,
[`tenant.ts`](../../src/web/lib/db/tenant.ts). `tenant_scope()` validates an
org and project pair and throws on a missing, empty or non-uuid value;
`scoped_where()` builds the predicate every statement carries; `scoped_values()`
stamps both columns onto every insert. Nothing in `lib/db` exposes an unscoped
query — `tests/db/tenant-import-guard.test.ts` enforces that nothing outside
`lib/db` (bar a handful of named standalone CLI scripts) reaches the database
client directly.

**Append-only reports.** Enforced three ways:

1. [`reports.ts`](../../src/web/lib/db/reports.ts) exports an insert and a read
   and nothing else — it is the only module that touches the table.
2. Migration `0001_reports-append-only.sql` puts `BEFORE UPDATE`,
   `BEFORE DELETE` and `BEFORE TRUNCATE` triggers on the table, so a mutation
   fails even from `psql`.
3. [`tests/db/reports-append-only.test.ts`](../../tests/db/reports-append-only.test.ts)
   audits the exports, greps the source, and asserts the database refuses all
   three operations.

**Migrations**, in `lib/db/migrations/`, one per schema change, none hand-edited:

- `0000` — the initial schema.
- `0001` — the reports append-only triggers.
- `0002` — `projects.next_issue_ref`, the atomic issue-ref counter.
- `0003` — `users.password_hash`.
- `0004` — `users.disabled_at` (revocation) and the `login_attempts` table
  (login lockout).

**Seed.** Idempotent. One organisation (araCreate), one project (B. Halle, with
a generated `pk_live_` key), and the pages in
[`data/pages.json`](../../src/web/data/pages.json) — three placeholders until
the real 49 arrive. No testers and no reports.

## Auth

**Session.** [`lib/auth/session.ts`](../../src/web/lib/auth/session.ts) — a
signed, stateless cookie (`user id`, `email`, `name`, `role`, an expiry),
(secret removed) over `SESSION_SECRET`, no `sessions` table. `middleware.ts` gates
every `/app/*` route (`runtime: 'nodejs'`, since verifying the signature needs
`node:crypto`, which the default Edge runtime cannot bundle).

**Revocation.** A signed cookie cannot itself be revoked — it stays valid
until it expires no matter what happens to the account afterwards. So
`middleware.ts` also loads the session's user on every request
(`is_user_active`, [`lib/db/users.ts`](../../src/web/lib/db/users.ts)) and
treats a missing or disabled row as no session, clearing the stale cookie on
the redirect. `make user-disable EMAIL=...` sets `users.disabled_at`;
`make user-enable EMAIL=...` reverses it. Disable, never delete — a deleted
user would orphan the `comments` and `issue_events` rows behind them.

**Login lockout.** [`lib/db/login-attempts.ts`](../../src/web/lib/db/login-attempts.ts) —
5 failures for one email in 15 minutes locks it out, counted from the
append-only `login_attempts` table (no new infrastructure, the same shape as
the reports rate limit below). The login error text
(`app/login/actions.ts`) is identical whether the email does not exist, the
password is wrong, the account is disabled, or the account is locked — a
login attempt can never be used to discover which emails have accounts or
whether one is currently locked.

**Passwords.** scrypt via `node:crypto`
([`lib/auth/password.ts`](../../src/web/lib/auth/password.ts)). Accounts are
created only by `make user-create` — no signup page anywhere.

## Rate limit

`REPORTS_RATE_LIMIT_PER_HOUR` (60), counted straight from `reports` itself
per tester token — [`lib/db/rate-limit.ts`](../../src/web/lib/db/rate-limit.ts).
No Redis/KV in the stack.

## Issues and resolving (M4)

**Status.** The full transition table lives in
[`issue-status.ts`](../../src/web/lib/db/issue-status.ts) as data, not
scattered `if`s — `new -> triaged -> in_progress -> fixed -> verified ->
closed`, with `wont_fix` and `duplicate` as side exits and all three
reopening back to `triaged`. `is_allowed_transition` is the one function
every mutation checks; an illegal move is a 400 that writes nothing.

**Permissions.** [`issue-permissions.ts`](../../src/web/lib/db/issue-permissions.ts) —
the three-role matrix as pure functions (`can_change_status`, `can_assign`,
`can_set_priority`, `can_set_category`, `can_mark_duplicate`), plus the two
comment-visibility gates (`resolve_comment_visibility` forces a client
author's comment to `client_visible: true`; `can_see_comment` is what every
comment read filters through, so a client can never see, count, or infer an
internal note).

**Mutations.** [`issue-actions.ts`](../../src/web/lib/db/issue-actions.ts) —
`change_issue_status`, `assign_issue`, `set_issue_priority`,
`set_issue_category`, `mark_duplicate`. Every one loads the issue
tenant-scoped first (a cross-project or malformed id resolves to
`IssueActionError('not_found')`, never a permission error and never a raw
database error — quality-gate.md §2.2 IDOR, §2.7 error leakage), checks
permission, checks any additional rule, then writes the change and an
`issue_events` row in one transaction.

**Comments.** [`comments.ts`](../../src/web/lib/db/comments.ts) — append-only,
`client_visible` defaults false. `add_comment` verifies the issue exists in
the caller's tenant before inserting.

**Grouping race.** `group_report`
([`issues.ts`](../../src/web/lib/db/issues.ts)) upserts with
`ON CONFLICT (project_id, group_key) DO NOTHING` rather than a plain
read-then-insert — under N simultaneous identical reports, at most one
insert wins, and every caller (winner and losers alike) looks up the
resulting row and attaches to it. A plain insert here loses this race and
crashes on the unique constraint the moment two reports share a
`group_key` at once; `tests/db/issue-concurrency.test.ts` fires 20 at once
and asserts exactly one issue with `reports_count` 20.

**Category taxonomy.** Nine values in
[`data/categories.json`](../../src/web/data/categories.json), read through
[`categories.ts`](../../src/web/lib/db/categories.ts). Free text is not a
category — the UI only ever offers a `<select>` over this closed set.

**Screens.** `/app/issues` (filterable list, default not-closed) and
`/app/issues/[id]` (detail — every report, the element text, status,
owner, priority, category, comments split internal/client-visible, and the
full event log).

## Admin (M5)

**Permissions.** [`admin-permissions.ts`](../../src/web/lib/db/admin-permissions.ts) —
`can_manage_pages`, `can_manage_testers`, `can_manage_assignments`,
`can_edit_strings`, `can_manage_team`: every one staff-only, same shape as
`issue-permissions.ts`. Every mutation below checks its own function; a
staff-only nav link is a convenience, never the enforcement.

**Strings (the string editor).** [`config-revisions.ts`](../../src/web/lib/db/config-revisions.ts) —
`save_project_config` validates against
[`config-schema.ts`](../../src/web/lib/api/config-schema.ts) (a zod schema over
every field in `config.ts`'s `ProjectConfig` — 22 required non-empty strings,
a hex accent colour, a closed set of positions/visibilities, and exactly one
option per answer id), then in one transaction inserts a `config_revisions`
row and updates `projects.config` — a failed validation writes neither.
`rollback_to_revision` restores a chosen revision **as a new revision**,
never deleting the one it restores from, so history only ever grows.
`/app/admin/strings` is a plain form, one labelled field per string, no JSON
on screen; `GET /api/v1/config`'s `max-age=60` cache on the no-token response
is what makes a save reach the widget within 60 seconds with no rebuild and
no deploy — `tests/api/string-editor-acceptance.test.ts` proves the whole
path end to end.

**Pages.** [`pages-admin.ts`](../../src/web/lib/db/pages-admin.ts) — add, edit
and bulk import, all normalising through the same
[`page-match.ts`](../../src/web/lib/page-match.ts) function `POST /reports`
itself uses to resolve `page_id`, so the widget and the app can never disagree
about whether a page counts. Bulk import ignores blanks, skips a path already
present (in the database or earlier in the same paste), bounds both the
number of lines and each line's length against a pathological paste, and
reports how many were added and how many skipped.
[`page-types.ts`](../../src/web/lib/page-types.ts) holds the closed
`page_type` set as data with no database import, so a `'use client'` form
component can render the `<select>` without pulling the Postgres driver into
the browser bundle.

**Testers.** [`tester-token.ts`](../../src/web/lib/tester-token.ts) generates a
24-character URL-safe token from `(secret removed)(18)` (base64url, no
padding at that length). [`testers-admin.ts`](../../src/web/lib/db/testers-admin.ts) —
`create_tester` auto-numbers the label (`Tester 01`, `Tester 02`, …) from how
many already exist; no password field anywhere, ever.
`invitation_links_for_testers` loads the project's `site_url` once and builds
every tester's link from it, rather than one query per row.

**Assignments.** [`assignment-generator.ts`](../../src/web/lib/db/assignment-generator.ts) —
`build_assignment_plan` is pure: every always-included page (home/contact/404)
goes to every tester, the rest are distributed with a seeded PRNG
(`Math.random` cannot be seeded) that favours whichever tester currently has
the fewest assignments, so load stays even and product-type pages don't
repeat the same trio back to back. `preview_assignment_plan` never writes;
`commit_assignment_plan` writes with `onConflictDoNothing` on
`(tester_id, page_id)`, the same upsert-over-read-then-insert lesson M4's
`group_report` race already taught this codebase — proven idempotent under
real concurrency in `tests/db/assignment-generator.test.ts`, not just in
sequence.

**Team.** [`team.ts`](../../src/web/lib/db/team.ts) — lists every role,
active or disabled; `set_user_disabled` is the only mutation, blocks a staff
member from disabling their own account, and resolves a malformed or
cross-org user id to `not_found`. No account creation anywhere in this
screen — that stays `make user-create`.

## Screenshot storage (M6a)

**Storage is local disk, not S3** — see
[`docs/blocked.md`](../blocked.md) for why this differs from
`docs/build-plan.md`. [`lib/storage/`](../../src/web/lib/storage/) defines
the `Storage` interface (`put`, `get`, `delete`, `signed_upload_url`);
[`local-disk.ts`](../../src/web/lib/storage/local-disk.ts) is the only
implementation and the only file outside this folder that may never be
imported directly — everything else calls `get_storage()`
([`index.ts`](../../src/web/lib/storage/index.ts)). Files live under
`STORAGE_DIR` (default `.storage/` at the repo root, gitignored), path
`reports/<project_id>/<report_id>.webp` — the exact shape
[`build_screenshot_key`](../../src/web/lib/db/submit-report.ts) already
produces at report-insert time.

**Keys.** [`keys.ts`](../../src/web/lib/storage/keys.ts) accepts exactly one
shape (`reports/<uuid>/<uuid>.webp`) rather than trying to blocklist `..`, an
absolute path, a URL-encoded traversal, a null byte or a backslash one at a
time — anything not matching that one shape is rejected by construction.
Called again inside `local-disk.ts` itself before every filesystem call, not
trusted from an earlier caller two functions away.

**Upload urls.** [`upload-url.ts`](../../src/web/lib/storage/upload-url.ts) —
there is no S3 to presign against, so the url is signed the same way
[`lib/auth/session.ts`](../../src/web/lib/auth/session.ts) signs a session
cookie: HMAC-SHA256 over `{ key, exp }`, base64url, using the existing
`SESSION_SECRET` rather than a second secret. A 5-minute expiry and the key
itself are both bound into the signature, so a token issued for one key can
never authorise a write to a different one — proven end to end in
`tests/api/uploads.test.ts` with two real projects' own reports and upload
urls, not just two raw keys. "Single use" is enforced by the store, not the
signature: `local-disk.ts`'s `put()` opens with `O_CREAT | O_EXCL`, so a
second write to an already-occupied key fails atomically at the filesystem
level and the signature's own expiry window is irrelevant to a replay.

**`POST /api/v1/reports`** now returns a populated `uploadUrl` for the
`screenshot_key` set at insert time — never a second write to the row itself,
since the append-only trigger would refuse it regardless.

**`POST /api/v1/uploads`** — verifies the signature and expiry, enforces a
2 MB body cap off `Content-Length` before reading the body and again against
the actual byte count, and checks the RIFF/WEBP magic bytes directly rather
than trusting `Content-Type`. Every rejection (expired, tampered, reused,
wrong key, non-WebP, oversized, malformed key) is a generic error code and
writes nothing.

**Retention.** [`lib/retention.ts`](../../src/web/lib/retention.ts), wired to
`make retention` ([`scripts/retention.mts`](../../src/web/scripts/retention.mts)) —
deletes files only, in bounded batches (a keyset cursor over
`created_at, id`, never an offset), and never touches the `reports` row a key
came from, not even to null it. Since the row is never marked "already
swept", the same backlog is found again by every future run — accepted,
because deleting an already-absent file is a no-op. A screenshot_key that
somehow isn't the valid shape is skipped and counted, not thrown, so one bad
row cannot stop the rest of a batch from being swept.

The service folder itself keeps only a short pointer: [`src/web/`](../../src/web/).

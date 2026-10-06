**Vishnu** (2026-09-08T06:01): M5 signed off and committed. I verified that the four client-component imports
are type-only, and that the permission checks live in the data layer rather
than only on the pages — so a mutation cannot reach the database without
passing one even if a future route forgets to guard.

M6 — SCREENSHOTS. Decisions are settled in overnight-run.md §6 and §7. Read
both. modern-screenshot is NOW APPROVED (the withdrawal was for the
unattended run only).

This milestone is split, with a review checkpoint. Do NOT run them together.

=== M6a: SERVER SIDE ===

1. STORAGE — src/web/lib/storage/, an interface with put, get, delete,
   signed_upload_url, and one local-disk implementation.
   - Files live outside the tracked tree, path from an env var, default
     .storage/ at the repo root. Add .storage/ to .gitignore.
   - VALIDATE EVERY KEY before it touches the filesystem: must match
     reports/<uuid>/<uuid>.webp exactly. Reject anything with .., an absolute
     path, a URL-encoded traversal, a null byte, or a backslash. Path
     traversal is the entire risk surface of a disk-backed store.
   - Nothing outside lib/storage imports the local implementation directly.

2. UPLOAD URL — there is no S3 to presign against, so sign it yourself: HMAC
   over key + expiry with the existing session secret, 5-minute expiry,
   single use. Verify the signature and the expiry on the upload route before
   writing a byte. An expired or tampered URL is a 403 that writes nothing.

3. POST /api/v1/uploads — accepts the signed URL, the key, and the image.
   Enforce a max body size. Verify the bytes are actually WebP, do not trust
   the content type. Reject anything else.

4. POST /api/v1/reports returns uploadUrl populated for the key already on the
   report. screenshot_key is STILL never updated — the trigger will refuse,
   correctly.

5. RETENTION — make retention, 90 days, deletes stored FILES ONLY. It never
   touches a reports row, not even to null the key. Bounded batches, cannot
   spin.

GATE for M6a: path traversal in all five forms above; an expired signature; a
tampered signature; a reused signature; a non-WebP body; an oversized body; a
key belonging to another project. Each must be rejected, with a test.

THEN STOP. Show me the storage and upload code before starting M6b.

=== M6b: THE WIDGET (after my review) ===

6. CAPTURE — modern-screenshot, viewport only, scale 1, WebP quality 0.8.
   Capture at SELECTION time and hold it in memory — not after the modal is
   covering the page. Set crossorigin="anonymous" on images first. CAPTURE
   TWICE ON SAFARI AND KEEP THE SECOND; the first is documented blank across
   libraries.

7. PRIVACY, before the bytes leave the page — this is the part that matters:
   - Strip the value of every input and textarea.
   - Blank any element carrying data-fb-block.
   - Both must happen on the CLONE used for capture, and must be proven by
     test: a page with a filled-in form and a data-fb-block element, captured,
     and the resulting image inspected to confirm neither appears.

8. CONSENT — show the tester the image with a "don't include it" option,
   strings from config. Declining means no upload happens and the report is
   submitted with its key pointing at nothing. That is a normal state, not an
   error, and must not be logged as one.

9. LOADING — v1.js must NOT contain the capture code. Dynamic import, fetched
   from the same origin as v1.js, only when a capture is about to happen.
   - make size keeps its 15,360-byte gate on v1.js. Report the number.
   - Add a second gate at 30,720 bytes gzipped for the capture chunk.
   - If the chunk fails to load: skip the screenshot silently and submit the
     report anyway. NEVER block, delay or fail a report because of a
     screenshot.

GATE for M6b: the privacy mutations — remove the input stripping, remove the
data-fb-block handling — each must break a test. Plus: chunk fails to load,
report still submits. Capture throws, report still submits. Tester declines,
no upload attempted. v1.js size unchanged from 5,638 bytes except by the
loader stub.

Show me the commit message before committing either half.

**Claude** (2026-09-08T06:01): I'll start by reading the memory files and the referenced overnight-run.md decisions before touching any code.

**Claude** (2026-09-08T06:01): _[tool: Read]_

**Claude** (2026-09-08T06:01): _[tool: Read]_

**Claude** (2026-09-08T06:01): _[tool: Bash]_

**Claude** (2026-09-08T06:01): Let me read overnight-run.md §6/§7, quality-gate.md, agent-rules.md, and the current repo layout before writing anything.

**Claude** (2026-09-08T06:01): _[tool: Bash]_

**Claude** (2026-09-08T06:01): _[tool: Read]_

**Claude** (2026-09-08T06:01): _[tool: Read]_

**Claude** (2026-09-08T06:01): _[tool: Read]_

**Claude** (2026-09-08T06:01): Now let me look at the existing structure — session/auth helper (for HMAC secret reuse), lib/db patterns, existing API routes, tenant helpers, and the widget build setup — before writing M6a.

**Claude** (2026-09-08T06:01): _[tool: Agent]_

**Claude** (2026-09-08T06:01): I've kicked off a recon agent to map the existing conventions (session HMAC pattern, tenant scoping, reports schema/trigger, API route style, env handling, Makefile, widget build). I'll wait for that before writing any M6a code so it matches the codebase exactly.

**Claude** (2026-09-08T06:01): _[tool: ScheduleWakeup]_

**Claude** (2026-09-08T06:01): _[tool: ScheduleWakeup]_

**Claude** (2026-09-08T06:02): That scheduling call was unnecessary here since I'll simply be notified when the recon agent finishes — I've stopped it. I'll wait for the agent's completion notification before proceeding with M6a.

**Vishnu** (2026-09-08T06:03): <task-notification>
<task-id>a49f75e881db2b20b</task-id>
<tool-use-id>toolu_016f6on4FkLHZDJWgFPSURdi</tool-use-id>
<output-file>/private/tmp/claude-501/-Users-vishnuvarthanvenkatapathy-araCreate-HLE-testing-widget/29a15db1-eb09-4f99-bd2b-632ff6f4c08c/tasks/a49f75e881db2b20b.output</output-file>
<status>completed</status>
<summary>Agent "Survey repo structure for M6a storage build" finished</summary>
<note>A task-notification fires each time this agent stops with no live background children of its own. The user can send it another message and resume it, so the same task-id may notify more than once.</note>
<result>I now have thorough coverage of all 9 topics. Here is the full report.

## 1. Session/auth secret (`src/web/lib/auth/session.ts`)

- **Env var**: `SESSION_SECRET`. Also has a hardcoded placeholder check: throws if unset, blank, or still `'change-me-openssl-rand-hex-32'`.
- **Algorithm**: HMAC-SHA256, `base64url` digest encoding. Payload is JSON, itself base64url-encoded, concatenated with the signature via a `.` — i.e. `${base64url(json)}.${base64url(hmac)}`.
- **Verification**: uses `timingSafeEqual` on `Buffer`s (length-checked first to avoid throwing), and every failure mode (missing, malformed, bad signature, non-string fields, expired) collapses to `undefined` — never throws to the caller.
- **Reusable helper**: there is no exported generic "sign string with expiry" helper — `sign()` and the base64url helpers are private (module-scoped, not exported) and specific to `SessionPayload`. For an upload-URL HMAC I'd write an analogous but separate small helper (own secret env var, own payload shape), following the same shape rather than importing this one.

Exact code:
```ts
function session_secret(): string {
  const secret = (secret removed)
  if (!secret || secret.trim() === '' || secret === 'change-me-openssl-rand-hex-32') {
    throw new Error(
      'SESSION_SECRET is not set (or still the placeholder) — run `make setup` ' +
        'and generate one with `openssl rand -hex 32`',
    );
  }
  return secret;
}

function base64url_encode(input: string): string {
  return Buffer.from(input, 'utf8').toString('base64url');
}
function base64url_decode(input: string): string {
  return Buffer.from(input, 'base64url').toString('utf8');
}
function sign(payload: string): string {
  return createHmac('sha256', session_secret()).update(payload).digest('base64url');
}

export function create_session_cookie_value(user: {...}): string {
  const payload: SessionPayload = { sub: user.id, email: user.email, name: user.name, role: user.role,
    exp: Math.floor(Date.now() / 1000) + SESSION_TTL_SECONDS };
  const encoded = base64url_encode(JSON.stringify(payload));
  const signature = sign(encoded);
  return `${encoded}.${signature}`;
}

export function verify_session_cookie_value(value: string | undefined): SessionPayload | undefined {
  if (!value) return undefined;
  const dot_index = value.lastIndexOf('.');
  if (dot_index &lt;= 0) return undefined;
  const encoded = value.slice(0, dot_index);
  const signature = value.slice(dot_index + 1);
  const expected_signature = sign(encoded);
  const provided = Buffer.from(signature);
  const expected = Buffer.from(expected_signature);
  if (provided.length !== expected.length || !timingSafeEqual(provided, expected)) return undefined;
  try {
    const payload = JSON.parse(base64url_decode(encoded)) as SessionPayload;
    if (typeof payload.sub !== 'string' || ... ) return undefined;
    if (payload.exp &lt; Math.floor(Date.now() / 1000)) return undefined;
    return payload;
  } catch { return undefined; }
}
```
For M6a's signed upload URL, match this exactly: (secret removed) + base64url, `${encoded}.${sig}` shape, own env var (e.g. something like `UPLOAD_URL_SECRET` or reuse `(secret removed)` semantics — but note S3 presigning normally uses AWS SigV4 via the S3 SDK itself, not a custom HMAC; check whether M6a intends a custom app-level signed URL or actual S3 presigned POST/PUT before reusing this pattern).

## 2. `lib/db/` layout and tenant scoping

`/Users/vishnuvarthanvenkatapathy/araCreate/HLE/testing_widget/halle-feedback-widget/src/web/lib/db/tenant.ts` is the one helper "used everywhere" — full contents already shown above in exploration; key exports: `is_uuid`, `tenant_scope`, `org_scope`, `TenantScopeError`, `scoped_where(table, scope, ...extra)`, `org_scoped_where`, `scoped_values(scope, values)`.

Typical `lib/db/*.ts` structure (e.g. `pages.ts`, `reports.ts`):
- SPDX/copyright/author/description header comment block (4 lines, exact format — every file in the repo has this).
- Imports: `db` from `./client.ts`, table(s) from `./schema.ts`, scope helpers from `./tenant.ts`, occasionally a sibling `lib/*.ts` helper (e.g. `page-match.ts`).
- Exports a type alias off `typeof table.$inferSelect` / `$inferInsert`.
- Functions take `(scope: TenantScope, ...args, executor: Executor = db)` when they need to participate in a transaction (see `reports.ts`), or just `(scope, ...args)` when they're always standalone reads (see `pages.ts`).
- Every query passes through `scoped_where`/`scoped_values` — never a raw `.where(eq(...))`.
- "No match is not an error" is a recurring comment/pattern — functions return `undefined` rather than throwing for legitimate empty results (e.g. `find_page_by_url`).
- The barrel `lib/db/index.ts` re-exports everything by name (long explicit export list, not `export *` for accessors) — new M6a accessors should be added here too.

`tests/db/tenant-import-guard.test.ts` — read in full above. Three enforced rules:
1. Nothing outside `lib/db` may import `lib/db/client` (any spelling).
2. Nothing outside `lib/db` may import `postgres` or `drizzle-orm/postgres-js` directly.
3. Nothing outside `lib/db` may import `db`, `get_db`, or `get_sql` from the `lib/db` barrel; and `lib/db/index.ts` itself may not re-export the client under a renamed identifier.
Exceptions: exactly six standalone CLI scripts (`db-migrate.mts`, `db-seed.mts`, `db-test-reset.mts`, `db-fixture.mts`, `user-create.mts`, `user-disable.mts`) in `src/web/scripts/`. **Implication for M6a**: any new storage/upload-URL code must live under `lib/db/` if it touches the db client directly, or must go through existing accessors — a new `lib/storage/` module (per build-plan.md §7) should call `lib/db` accessors, never import the client itself.

## 3. `reports` table schema and the append-only trigger

Full Drizzle schema for `reports` in `src/web/lib/db/schema.ts` (lines 198–242) — notably `screenshot_key: text('screenshot_key')` (nullable, plain text, no default). Full table shown above in exploration.

The trigger migration is `src/web/lib/db/migrations/0001_reports-append-only.sql` (shown in full above): a `reports_append_only()` plpgsql function that `RAISE EXCEPTION`s on any `UPDATE`/`DELETE`/`TRUNCATE`, wired via three triggers (`reports_no_update`, `reports_no_delete`, row-level; `reports_no_truncate`, statement-level). This is genuinely absolute — **there is no carve-out for `screenshot_key`**. `submit-report.ts` computes `screenshot_key` deterministically at insert time (`build_screenshot_key(project_id, report_id) =&gt; reports/${project_id}/${report_id}.webp`) and comments explicitly: *"a key pointing at no file is normal; the upload (M6) writes to this path, it does not tell the row about itself."* So the M6a upload route must **never** attempt to `UPDATE reports SET screenshot_key = ...` — it will hit the trigger and 500. The upload flow only needs to hand back a signed PUT URL for the key that's already on the row from insert time.

## 4. API routes under `src/web/app/api/v1/`

Two existing routes, both fully read above: `app/api/v1/config/route.ts` (GET) and `app/api/v1/reports/route.ts` (POST). Shared conventions to replicate for `app/api/v1/uploads/route.ts`:
- `CORS_HEADERS` is a module-level `const` object (`Access-Control-Allow-Origin: '*'`, `-Methods`, `-Headers: 'Content-Type'`), applied to **every** response including error paths, plus an `OPTIONS` handler returning 204.
- Body parsing: `await request.json()` wrapped in try/catch → `{ error: 'invalid_body' }`, 400, on JSON parse failure; then `schema.safeParse(raw)` → on failure `{ error: 'invalid_body', issues: parsed.error.issues }`, 400.
- Error responses are always `NextResponse.json({ error: '&lt;snake_case_code&gt;' }, { status, headers: CORS_HEADERS })` — short string error codes (`unknown_key`, `rate_limited`, `invalid_body`), never a message string, never a stack trace, never raw DB error text. **There is no separate shared "generic error response" helper file** — each route inlines this shape itself; it's a convention, not a shared function. (Matches quality-gate.md §2.7: "client sees a generic message, never a stack trace, SQL, a file path or a column name.")
- Lookups that fail return domain-appropriate codes, never leak whether a row exists cross-tenant (IDOR: always "not found"/404-shaped, per quality-gate.md §2.2).
- Every route builds a `tenant_scope({ org_id, project_id })` immediately after resolving the project, before touching any other table.
- Import style: relative paths with explicit `.ts` extensions (`'../../../../lib/api/reports-schema.ts'`), not path aliases.

## 5. zod / env var handling

**There is no central `env.ts`.** Env vars are read ad hoc, directly via `process.env.X`, each in the module that needs it, with its own inline validation function that throws a descriptive error pointing at `make setup`:
- `src/web/lib/db/client.ts`: `database_url()` — throws if `DATABASE_URL` unset/blank.
- `src/web/lib/auth/session.ts`: `session_secret()` — throws if `SESSION_SECRET` unset/blank/placeholder.
- `src/web/app/login/actions.ts`: inline `process.env.NODE_ENV === 'production'` for cookie `secure` flag.
- `src/widget/scripts/build.mjs`: `process.env.WIDGET_API_ORIGIN ?? 'http://localhost:3000'` (fallback, not throw — build-time, not runtime).

**Convention for M6a**: write a similarly-named function per S3 var, e.g. `s3_bucket()`, `s3_endpoint()`, etc., each throwing the same style of error message, living in whatever new module owns the storage client (likely `lib/storage/` or `lib/db/` depending on the import-guard rule).

zod schemas live in `src/web/lib/api/*-schema.ts` (`reports-schema.ts`, `config-schema.ts` — both shown in full above). Style: `z.object({...})` built from small reusable primitives (e.g. `string_field = z.string().trim().min(1).max(500)`), enums drawn from shared const arrays (`ANSWER_IDS`, `OTHER_ANSWER_ID`), `.refine()` for cross-field invariants, and an exported `type X = z.infer&lt;typeof x_schema&gt;`. A new upload-request schema (`lib/api/uploads-schema.ts`?) should follow this exact pattern.

`.env.example` (full contents, `src/web/.env.example`) — already includes the S3 block, unfilled:
```
DATABASE_URL=postgresql://localhost:5432/halle_feedback_dev
DATABASE_URL_TEST=postgresql://localhost:5432/halle_feedback_test
APP_URL=http://localhost:3000
WIDGET_CDN_URL=http://localhost:3000/widget
(secret removed)
S3_ENDPOINT=
S3_REGION=eu-central-1
S3_BUCKET=
S3_ACCESS_KEY_ID=
(secret removed)
REPORTS_RATE_LIMIT_PER_HOUR=60
```
Note there's no `WIDGET_API_ORIGIN` in this file even though `build.mjs` reads it — that var apparently lives elsewhere (CI/deploy config) or is missing from `.env.example`, worth flagging if you add new widget-facing build vars.

## 6. Makefile

Full contents shown above. Key targets: `install`, `setup`, `dev`, `dev-widget`, `build` (npm run build --workspaces), `size` (npm run size --workspace widget), `db-migrate`, `db-seed`, `db-test-reset`, `db-fixture`, `user-create/-disable/-enable`, `lint` (npm run lint --workspaces), `test` (npm run test --workspaces), `test-widget`, `release`, `clean`.

**`make retention` does NOT exist yet** — no stub target, not in `.PHONY`, not in the `help` output. It's only referenced in prose (`docs/overnight-run.md` line 269: *"a 90-day job, wired to `make retention`, that deletes stored files only. It never touches a `reports` row, not even to null the key."* and `docs/build-plan.md` M6 acceptance row). This will need to be added as a genuinely new Makefile target (M6b territory per the deferred-decisions doc, not M6a, but worth confirming scope with the user).

## 7. Widget build (`src/widget/`)

- `scripts/build.mjs` — esbuild, IIFE format, `target: ['es2020', 'safari14']`, minified unless `--watch`, `external: []` (zero dependencies is a hard rule), `define: { __WIDGET_API_ORIGIN__: JSON.stringify(API_ORIGIN) }` where `API_ORIGIN = process.env.WIDGET_API_ORIGIN ?? 'http://localhost:3000'`. Consumed in `src/widget/src/api.ts` via `declare const __WIDGET_API_ORIGIN__: string;`, overridable per-embed via a `data-api` attribute on the script tag (`resolve_api_origin`), validated by `is_https_or_local()`.
- `scripts/size.mjs` — always rebuilds first (`execFileSync` on `build.mjs`), then gzips `dist/v1.js` at level 9 and gates at `BUDGET_BYTES = 15360` (15 KB), printing raw/gzipped/budget/percent/headroom and exiting 1 on overage.
- **Current measured size**: raw `dist/v1.js` = 16,320 bytes; gzip level 9 = **5,627 bytes** (task message said 5,638 — close but not exact; likely measured at a slightly different commit or gzip settings — re-run `make size` yourself before relying on the exact number).
- Per `docs/build-plan.md` §6 (M6 deferred decisions), the screenshot library (`modern-screenshot`) must **not** go into `v1.js` — it needs a dynamically-imported separate chunk with its own 30,720-byte gzip gate, loaded only when a capture starts, failing silently (skip screenshot, submit anyway) if the chunk fails to load.

## 8. `.gitignore`

Full contents shown above — standard OS/editor/build/Playwright/env ignores, plus repo-specific: `COMMIT_MSG_*.txt` and `next-env.d.ts`. Note `.env.*` is ignored except `!.env.example` — so any new `.env.local`-style file for S3 creds would be auto-ignored already.

## 9. Directory trees

`src/web/lib/`:
```
lib/
├── api/
│   ├── config-schema.ts
│   └── reports-schema.ts
├── auth/
│   ├── current-user.ts
│   ├── password.ts
│   └── session.ts
├── db/
│   ├── admin-permissions.ts, assignment-generator.ts, categories.ts, client.ts,
│   │   comments.ts, config-revisions.ts, config.ts, dashboard-project.ts,
│   │   export-issues.ts, grid.ts, index.ts, issue-actions.ts, issue-detail.ts,
│   │   issue-list.ts, issue-permissions.ts, issue-status.ts, issues.ts,
│   │   login-attempts.ts, pages-admin.ts, pages.ts, projects.ts, rate-limit.ts,
│   │   report-list.ts, reports.ts, schema.ts, submit-report.ts, team.ts,
│   │   tenant.ts, testers-admin.ts, testers.ts, users.ts
│   └── migrations/
│       ├── 0000_initial-schema.sql ... 0004_users-disabled-at-login-attempts.sql
│       └── meta/ (snapshots + _journal.json)
├── csv.ts
├── dashboard-scope.ts
├── dev-database-guard.ts
├── page-match.ts
├── page-types.ts
├── public-key.ts
├── readme.md
└── tester-token.ts
```
No `lib/storage/` yet — this is what M6a needs to create (per `docs/build-plan.md` §7: `src/web/lib/storage/` with a `put`/`get`/`delete`/`signed_upload_url` interface plus a local-disk implementation).

`src/web/app/api/`:
```
app/api/
└── v1/
    ├── config/
    │   └── route.ts
    └── reports/
        └── route.ts
```
No `v1/uploads/` yet — this is the new route M6a needs to create, matching the `docs/build-plan.md` §4 spec: `POST /api/v1/uploads` → presigned screenshot upload URL.

---
**Flags worth raising with the user before you start coding:**
1. `docs/build-plan.md` §6 explicitly says M6 decisions are "DEFERRED — DO NOT BUILD TONIGHT" and "nothing in this section is authorised" without review — confirm M6a is now formally greenlit.
2. No existing app-level HMAC-signed-URL helper to reuse; you'll be writing a new one from scratch, following `session.ts`'s (secret removed)/base64url style but with its own secret and payload shape.
3. `make retention` doesn't exist — likely M6b, but flagging since the task description asked about it as if it might already be stubbed.
4. Confirm whether "signed upload URL" means a native S3 presigned URL (via AWS SDK v3, SigV4) or a custom app-level HMAC scheme proxied through your own PUT handler — the two are very different implementations and the repo currently has zero AWS SDK dependencies to inspect.</result>
<usage><subagent_tokens>79209</subagent_tokens><tool_uses>37</tool_uses><duration_ms>124974</duration_ms></usage>
</task-notification>

**Claude** (2026-09-08T06:04): Good recon. One correction to flag: the agent's note about `.env.example` having S3 vars and "no AWS SDK" is from an older assumption — the actual M6a task explicitly says "there is no S3 to presign against, sign it yourself" with a local-disk store, so that's already settled, not an open question. Let me pull the exact files I need before writing code.

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:04): Good — I now have the full picture, including a real discrepancy in `.env.example` (S3 vars are stale — task explicitly says "no S3, sign it yourself" with local disk). Let me also check the reports migration/trigger, the widget's test setup, and package.json for test tooling before writing.

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:04): I have everything I need. Note: `build-plan.md` still describes S3-compatible storage, but the current task message explicitly overrides that ("there is no S3 to presign against, sign it yourself... one local-disk implementation") — per the established convention, the task message wins. I'll flag this deviation in the storage module's header comment and proceed with local disk + self-signed HMAC URLs.

Let me check the widget's `data-fb-block`/existing widget src layout briefly (not needed for M6a, skip), and check `docs/blocked.md` / `docs/overnight-log.md` format, then start building.

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): There's no `docs/blocked.md` yet — I'll create it if a genuine deviation needs logging (the S3-vs-local-disk discrepancy). Now I have a complete picture. Let me plan the M6a file set before writing:

**Plan for M6a:**
1. `src/web/lib/storage/types.ts` — the `Storage` interface (put/get/delete/signed_upload_url) + key validation regex/function
2. `src/web/lib/storage/local-disk.ts` — local disk implementation, path from env var `STORAGE_DIR` (default `.storage/`)
3. `src/web/lib/storage/upload-url.ts` — HMAC sign/verify for upload URLs (own secret env var, 5-min expiry, single-use via a nonce record — need to think about "single use" enforcement)
4. `src/web/lib/storage/index.ts` — barrel
5. `src/web/lib/api/uploads-schema.ts` — zod schema for the upload route body
6. `src/web/app/api/v1/uploads/route.ts` — the upload route (issues upload URL is really requested via reports route per spec item 4; item 3 is the actual PUT that accepts bytes)
7. Migration for retention job — actually retention doesn't need a schema change, just deletes files. But "single use" signature needs tracking somewhere — let me think about this without a DB table, since expiry+signature alone doesn't give single-use.

Let me reconsider the single-use requirement carefully before writing code.

**Claude** (2026-09-08T06:04): _[tool: Read]_

**Claude** (2026-09-08T06:04): This confirms the design: POST /api/v1/reports already returns `uploadUrl` (currently always null), which will now be populated with the signed upload URL. There's no separate "request an upload URL" round trip — the report POST hands it back directly, and the widget PUTs bytes to `POST /api/v1/uploads` using it.

Now — the "single use" requirement. A pure stateless HMAC(key + expiry) can be verified for tamper/expiry but cannot be single-use without server-side state (nothing stops replaying the same valid URL twice within its 5-minute window). Since the storage key itself is `reports/<uuid>/<uuid>.webp` (one per report, immutable), the natural single-use enforcement is: **the local storage `put` refuses to overwrite an existing object** — first write wins, every subsequent PUT with the same (valid, unexpired) signature 403s because the file already exists. That gives single-use without needing a new database table or nonce-tracking, and matches "screenshot_key is never updated — the trigger will refuse" philosophy: the object store itself refuses a second write, symmetric to the DB refusing a second write.

Let me verify this reasoning holds together, then check for an existing "no-clobber" pattern and Node's fs flags before writing.

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:04): Good — `wx` flag (O_CREAT|O_EXCL|O_WRONLY) on `fs.writeFile`/`open` gives an atomic no-clobber write, which is exactly the primitive needed: first successful upload wins, any replay of the same signed URL after that gets `EEXIST` → 403, no separate nonce table required.

Now let me write the code. Starting with the storage interface and key validation.

**Claude** (2026-09-08T06:04): _[tool: Bash]_

**Claude** (2026-09-08T06:05): _[tool: Write]_

**Claude** (2026-09-08T06:05): Now the storage interface types:

**Claude** (2026-09-08T06:05): _[tool: Write]_

**Claude** (2026-09-08T06:05): Now the HMAC signed-upload-url module, matching `session.ts`'s style but with its own secret and payload shape:

**Claude** (2026-09-08T06:05): _[tool: Write]_

**Claude** (2026-09-08T06:05): I should double check the reuse-of-SESSION_SECRET decision — that's a judgment call, not explicitly specified ("HMAC over key + expiry with the existing session secret" — actually, re-reading the task, it literally says "with the existing session secret"). Let me confirm against the task text.

The task said: *"sign it yourself: HMAC over key + expiry with the existing session secret"* — so reusing `SESSION_SECRET` is explicitly specified, not my judgment call. Good, matches what I wrote. Now let's build the local-disk implementation.

**Claude** (2026-09-08T06:05): _[tool: Write]_

**Claude** (2026-09-08T06:05): I added an exported helper `local_disk_object_exists` that isn't needed — `put()`'s boolean return already tells the caller whether the write succeeded, which is all the upload route needs. Let me remove that unused helper to avoid a second, redundant existence-check path.

**Claude** (2026-09-08T06:05): _[tool: Edit]_

**Claude** (2026-09-08T06:06): _[tool: Edit]_

**Claude** (2026-09-08T06:06): Now the barrel that everything else imports through:

**Claude** (2026-09-08T06:06): _[tool: Write]_

**Claude** (2026-09-08T06:06): Now I need to wire this into `submit_report` / the reports route so `POST /api/v1/reports` returns a populated `uploadUrl`. Let's check `config.ts` for where `report` gets its `uploadUrl`, and update `submit-report.ts` and the route.

**Claude** (2026-09-08T06:06): _[tool: Read]_

**Claude** (2026-09-08T06:06): Not relevant to uploadUrl. Now let's update the reports route to populate `uploadUrl` using `get_storage().signed_upload_url(report.screenshot_key)`. This keeps `submit_report`/`submit-report.ts` untouched (it doesn't need to know about storage) — the route composes storage + the report result, same separation of concerns as the rest of the codebase.

**Claude** (2026-09-08T06:06): _[tool: Edit]_

**Claude** (2026-09-08T06:06): _[tool: Edit]_

**Claude** (2026-09-08T06:06): Now the upload zod schema and the actual `POST /api/v1/uploads` route that accepts the bytes. Let me check the reports-schema.ts style once more (already read) and write the schema for query params — since this is bytes on the body plus key+token in the query string, zod validates the query params, and the body is handled as raw bytes with WebP magic-byte verification.

**Claude** (2026-09-08T06:06): _[tool: Write]_

**Claude** (2026-09-08T06:06): Now I need a WebP magic-byte verification helper. WebP format: bytes 0-3 = `RIFF`, bytes 8-11 = `WEBP`. Let me put this in the storage layer since it's about validating what goes into storage, or in the route directly. I'll add it as a small pure helper next to the uploads schema since it's a body-shape check, not a storage concern.

**Claude** (2026-09-08T06:06): _[tool: Write]_

**Claude** (2026-09-08T06:06): Now the actual upload route. Key considerations:
- Enforce max body size before reading fully (Next.js `Request.arrayBuffer()` will read the whole body — I should check `Content-Length` header first and reject early, and also guard the actual byte count after reading in case `Content-Length` is absent/lying).
- Verify signature+expiry via `verify_upload_token`.
- Verify WebP magic bytes.
- Call `storage.put()` — if it returns `false` (already exists), that's the "reused signature" case → 403 writing nothing.
- CORS open (public API route, same as the other two).

**Claude** (2026-09-08T06:06): _[tool: Bash]_

**Claude** (2026-09-08T06:07): _[tool: Write]_

**Claude** (2026-09-08T06:07): Now let's handle the retention job (item 5) and update `.env.example`/`.gitignore`. First, `.gitignore` and env:

**Claude** (2026-09-08T06:07): _[tool: Read]_

**Claude** (2026-09-08T06:07): _[tool: Edit]_

**Claude** (2026-09-08T06:07): Now update `.env.example` — the S3 block is now stale since we're doing local disk + HMAC signing (no new secret needed since we reuse `SESSION_SECRET`), but we do need `STORAGE_DIR`. Let me flag the S3 discrepancy properly in `docs/blocked.md` and update the env file.

**Claude** (2026-09-08T06:07): _[tool: Read]_

**Claude** (2026-09-08T06:07): _[tool: Edit]_

**Claude** (2026-09-08T06:07): Now let's check `make setup`'s script to see if it needs updating for the new var, and write `docs/blocked.md` documenting the S3→local-disk deviation.

**Claude** (2026-09-08T06:07): _[tool: Bash]_

**Claude** (2026-09-08T06:07): Good — `setup-env.sh` reads keys generically from `.env.example`, so no script change needed; `make setup` will pick up `STORAGE_DIR` automatically. Now let's write `docs/blocked.md` to log the S3-vs-local-disk deviation, per agent-rules/quality-gate convention.

**Claude** (2026-09-08T06:07): _[tool: Write]_

**Claude** (2026-09-08T06:07): Now let's check `db/index.ts` barrel — should storage exports go through it? No, `lib/storage` is its own subsystem outside `lib/db`, not database-backed, so it doesn't belong in the db barrel. That's correct as-is. Now let's write tests. First let's look at an existing test file for style (e.g. `tests/web/csv.test.ts`) to match conventions, then write storage tests.

**Claude** (2026-09-08T06:07): _[tool: Read]_

**Claude** (2026-09-08T06:07): _[tool: Bash]_

**Claude** (2026-09-08T06:07): Good — tests live under `tests/` at repo root, no db needed for storage tests since it's filesystem-only. Now let's write the test files. I'll cover:

1. `tests/web/storage-keys.test.ts` — path traversal (all 5 forms), valid key acceptance
2. `tests/web/storage-local-disk.test.ts` — put/get/delete, no-clobber (single-use), content-type roundtrip
3. `tests/web/storage-upload-url.test.ts` — sign/verify: valid, expired, tampered, wrong key
4. `tests/web/webp.test.ts` — is_webp boundary cases
5. `tests/api/uploads.test.ts` — the actual route: expired/tampered/reused signature, non-webp body, oversized body, cross-project key (IDOR-style — a key valid in shape but never issued to this signature test doesn't quite map to "another project's key" since keys aren't tenant-scoped directly, but the signature IS scoped to a specific key, so using a signature for project A's key against project B's key tests that.)

Let me check how `tests/api/reports.test.ts` sets `SESSION_SECRET` in the test env, and how `.env`/`.env.test` is loaded for these unit tests (since upload-url.ts requires `SESSION_SECRET`).

**Claude** (2026-09-08T06:07): _[tool: Bash]_

**Claude** (2026-09-08T06:07): `SESSION_SECRET` is already set in the local `.env` (loaded by vitest config), so tests run fine locally. Since this file isn't tracked, I should also top it up with `STORAGE_DIR` so `make setup`-equivalent stays consistent, but I won't touch `.env` directly since it's the user's local file — actually since `setup-env.sh` only appends missing keys and won't overwrite, let me check if I should append `STORAGE_DIR` there too so local dev/tests work without manual action.

**Claude** (2026-09-08T06:07): _[tool: Read]_

**Claude** (2026-09-08T06:08): This gives me the exact pattern to follow for `upload-url.test.ts`. Now let me write the test suite for `keys.ts` (path traversal):

**Claude** (2026-09-08T06:08): _[tool: Write]_

**Claude** (2026-09-08T06:08): Now the upload-url signing test (expired, tampered, wrong key, single-use is tested at the storage level not here):

**Claude** (2026-09-08T06:08): _[tool: Write]_

**Claude** (2026-09-08T06:08): Now `webp.ts` boundary tests:

**Claude** (2026-09-08T06:08): _[tool: Write]_

**Claude** (2026-09-08T06:08): Now the local-disk storage tests — put/get/delete, no-clobber (single-use enforcement), concurrent puts on the same key:

**Claude** (2026-09-08T06:08): _[tool: Write]_

**Claude** (2026-09-08T06:09): That trailing `afterEach` is pointless (it just re-sets what's already set and never changes) — let me remove it, it's dead weight I added without thinking it through.

**Claude** (2026-09-08T06:09): _[tool: Edit]_

**Claude** (2026-09-08T06:09): _[tool: Edit]_

**Claude** (2026-09-08T06:09): Now the API-level test for `POST /api/v1/uploads`, covering the full gate list. Let me check `tests/api/fixtures.ts` again (already read) and write the route test.

**Claude** (2026-09-08T06:09): _[tool: Write]_

**Claude** (2026-09-08T06:09): One issue: in the "honest oversized Content-Length" test, I set the header manually alongside real (small) `webp_bytes()` body — but the `Request` constructor will likely override/recompute `Content-Length` from the actual body, or Node's fetch `Request` might throw/ignore a mismatched header. Let me check that behavior before trusting the test.

**Claude** (2026-09-08T06:09): _[tool: Bash]_

**Claude** (2026-09-08T06:09): Good — Node's `Request` preserves a manually set `Content-Length` header rather than recomputing it, so that test is valid as written and exercises the "reject before reading the body" path correctly. Now let's also handle **item 5: retention job**. Let's write that as a standalone script wired to `make retention`, matching the existing scripts pattern (`db-fixture.mts` etc.), deleting only files, bounded batches.

**Claude** (2026-09-08T06:09): _[tool: Bash]_

**Claude** (2026-09-08T06:09): Now let me think about retention scope. Retention needs to find reports whose `screenshot_key` is older than 90 days and delete the corresponding **file only** (never touching the `reports` row). This requires reading `reports` (org/project scoped normally, but retention is a global maintenance job across all tenants — similar to how `login_attempts` is deliberately unscoped). Let me check if this belongs in M6a or M6b — the task explicitly lists retention under M6a item 5 ("RETENTION — make retention, 90 days..."), so it's in scope now.

I need a `lib/db/` accessor to list reports with a screenshot older than N days (bounded batch), since retention needs the report's `screenshot_key` + `created_at`, which lives in the `reports` table — this must go through `lib/db` per the tenant-import-guard, even though this script is DB-touching from a script context. Let me check whether scripts can call `lib/db` accessors, or must open their own raw connection (per the exception list, only 6 named scripts get to import the driver directly — a retention script would need to either be added to that exception list, or (better) call through `lib/db` accessors like normal app code, since it doesn't need the "before any tenant scope exists" carve-out that the 6 exceptions need).

Retention reads across **all** tenants (it's a maintenance job, not scoped to one project), so it needs an unscoped read similar to `find_project_by_public_key`. Let's add a small accessor `lib/db/retention.ts` that does the unscoped read (documented same way `projects.ts` documents its unscoped read), keeping the script itself free of raw driver access.

**Claude** (2026-09-08T06:09): _[tool: Read]_

**Claude** (2026-09-08T06:09): _[tool: Read]_

**Claude** (2026-09-08T06:09): That guard is for dev-only fixture scripts (which write fake data) — retention is a production job that must run against the real database, so it shouldn't use `assert_dev_or_test_database`. Retention will just use `lib/db` accessors like normal app code (going through `db` from `lib/db/client.ts` internally, not the raw driver), which keeps it compliant with the tenant-import-guard without needing a script exception.

Let's write `lib/db/retention.ts`:

**Claude** (2026-09-08T06:10): _[tool: Write]_

**Claude** (2026-09-08T06:10): Now add this to the `lib/db/index.ts` barrel:

**Claude** (2026-09-08T06:10): _[tool: Edit]_

**Claude** (2026-09-08T06:10): Now the retention job itself, a bounded batch loop that deletes storage objects only. Let's write `lib/retention.ts` (business logic, not db, not storage — composes both) and a script wired to `make retention`:

**Claude** (2026-09-08T06:10): _[tool: Write]_

**Claude** (2026-09-08T06:10): Wait — there's a subtle bug in my own loop logic: since `find_expired_screenshots` reads by `created_at < cutoff` and deletion doesn't change `created_at` or remove the row, **the same rows will be returned again on the next batch call** unless something changes between calls. Since we never update the `reports` row (screenshot_key stays the same forever, deliberately), a repeat sweep will re-select already-deleted screenshots forever, calling `storage.delete()` on an already-gone file every single day. That's not wrong (delete-if-exists is idempotent, per my test), but it means retention re-scans and re-attempts deletion of everything ever expired, every run — increasingly wasteful and never actually bounded in the "eventually stops looking" sense, though it IS bounded per-invocation via `MAX_BATCHES`.

Let me reconsider: is this a real problem worth fixing, or acceptable? The task says "Bounded batches, cannot spin" — that's about *one invocation* not spinning unboundedly, which `MAX_BATCHES` satisfies. But re-processing the same rows forever is wasteful and means "deleted" count is misleading (it counts re-attempts, not new deletions). Since we can never mark the row (append-only), there's no way to remember "already swept" without a new mechanism. Let me check if `storage.delete` returning whether something was actually removed would let me report accurately, and reconsider whether looping past the first empty-vs-full batch even matters given this repeat-scan reality.

Given the row can never be marked, the cleanest honest design is: `find_expired_screenshots` will always return the same set of "expired" reports every day (growing over time) regardless of whether their file was already deleted — that's inherent to the "never touch the row" constraint. So the batch loop should just delete-if-exists on whatever it finds (idempotent, cheap for an already-gone file), and "deleted" should mean "attempted, storage confirms gone" rather than "newly removed." That's actually fine and matches the spec literally. Let me just adjust the result naming and comment to be honest about this instead of implying it only counts new deletions.

**Claude** (2026-09-08T06:10): _[tool: Edit]_

**Claude** (2026-09-08T06:10): Wait, there's a bigger correctness bug: since `find_expired_screenshots` orders by `created_at` ascending and always returns the same oldest-first N rows (nothing marks them done), **the batch loop as written will loop forever within one invocation** re-fetching the exact same first `BATCH_SIZE` rows every iteration (since nothing changes between calls — the query is identical), until `MAX_BATCHES` cuts it off. That's wasteful (50x redundant identical deletes-of-nothing in a single invocation) but not infinite thanks to `MAX_BATCHES`. Still, it's clearly wrong behavior — a single invocation should sweep *distinct* rows, not the same page 50 times. I need pagination via offset, or better, track swept ids within the run to advance past them.

Let me fix this properly using an offset-based batch cursor within one invocation:

**Claude** (2026-09-08T06:10): _[tool: Read]_

**Claude** (2026-09-08T06:10): I'll add an `after_id` cursor parameter (keyset pagination on `created_at, id` to be stable) so each batch call within one sweep genuinely advances, rather than an offset (offset pagination would also work but is more fragile against concurrent inserts; keyset is the better-established pattern here and mirrors nothing else in the codebase but is straightforward). Let me update both files.

**Claude** (2026-09-08T06:11): _[tool: Edit]_

**Claude** (2026-09-08T06:11): Wait, my cursor predicate is buggy: `or(gt(created_at, after.created_at), and(gt(id, after.report_id)))` — this misses the correct tiebreak logic. The correct keyset condition for `(created_at, id) > (after.created_at, after.report_id)` is:

`created_at > after.created_at OR (created_at = after.created_at AND id > after.report_id)`

I wrote `and(gt(id, after.report_id))` without the `eq(created_at, after.created_at)` — meaning any row with `id > after.report_id` regardless of created_at would match, which is wrong. Let me fix this properly.

**Claude** (2026-09-08T06:11): _[tool: Edit]_

**Claude** (2026-09-08T06:11): _[tool: Edit]_

**Claude** (2026-09-08T06:11): Now update `lib/retention.ts` to actually pass and advance the cursor between batches:

**Claude** (2026-09-08T06:11): _[tool: Edit]_

**Claude** (2026-09-08T06:11): Now the `batches: batches + 1` at the end is off — if the loop breaks on the first iteration (`batches === 0`), we'd report `1` batch, which is correct (one batch was fetched). But if the loop runs to completion via the `for` condition (`batches < MAX_BATCHES` becomes false), `batches` will equal `MAX_BATCHES` and we'd report `MAX_BATCHES + 1`, which is wrong — that path never executed a `MAX_BATCHES+1`-th fetch. Let me fix this counting bug.

**Claude** (2026-09-08T06:11): _[tool: Read]_

**Claude** (2026-09-08T06:11): Let me rewrite this loop cleanly with a `batches_run` counter that's incremented exactly once per actual fetch, rather than piggy-backing on the `for` loop variable:

**Claude** (2026-09-08T06:11): _[tool: Edit]_

**Claude** (2026-09-08T06:11): This now correctly: fetches a batch, counts it, breaks on empty or a short (final) batch without needing to compute a cursor for a batch that won't be used again, and caps at `MAX_BATCHES` real fetches — accurate reporting either way.

Now the script wired to `make retention`:

**Claude** (2026-09-08T06:11): _[tool: Bash]_

**Claude** (2026-09-08T06:11): Now write `scripts/retention.mts` — but since `run_retention_sweep` already uses `lib/db` (the shared `db` proxy, connecting lazily), this script doesn't need to open its own raw connection at all. It just calls the function and exits.

**Claude** (2026-09-08T06:11): _[tool: Write]_

**Claude** (2026-09-08T06:12): Wait — `get_sql` is exported from `lib/db/client.ts` and re-exported from the barrel, but the tenant-import-guard test blocks `db`, `get_db`, `get_sql` imports from the **barrel** (`lib/db/index.ts`) outside `lib/db`, but does it block importing them directly from `lib/db/client.ts`? Let me re-check the guard rules — rule 1 blocks *any* import of `lib/db/client` from outside `lib/db`, full stop. So my script directly importing `../lib/db/client.ts` for `get_sql` would violate that guard. Let me check how other scripts close their connection — `db-fixture.mts` does `sql.end()` where `sql` is its own `postgres(url)` connection it created itself (the exception scripts open their own throwaway connection). Since `retention.mts` isn't in the exception list and shouldn't be, it must not import `client.ts` at all — but then how does it close the connection cleanly on exit?

Looking again: the `db` proxy lazily connects via `get_db()`/`get_sql()` internally in `client.ts`, but nothing outside `lib/db` is supposed to call `get_sql()` to end it. Let me check if any non-exception script or route ever calls `.end()`, or whether the process is just allowed to exit with an open pool (Next.js server routes never call `.end()` either — the connection lives for the process lifetime). Standalone CLI scripts need to exit cleanly though. Let me check.

**Claude** (2026-09-08T06:12): _[tool: Bash]_

**Claude** (2026-09-08T06:12): Confirmed: every other CLI script opens its own `postgres(url)` connection directly (they're all in the exception list) and ends it themselves. My `retention.mts` is the odd one out, importing `lib/db/client.ts`'s `get_sql` from outside `lib/db`, which both violates the guard test and isn't the established pattern. Since I deliberately don't want retention added to the "runs before any tenant scope exists" exception list (it doesn't fit that justification — it runs post-migration against live data, same as any app code), the correct fix is: **use `lib/db` accessors as normal app code does, and don't try to end the connection at all** — same as a Next.js route never ending it. A short-lived script exiting via `process.exit()` (implicit at the end of `main()` succeeding) will just let the pool die with the process; the `finally { await get_sql().end() }` pattern is only used by scripts that need a clean exit *before* the process would naturally end due to open handles keeping node alive. Let me check whether `postgres()` with default settings keeps the process alive (an open pool socket), which is likely why the exception scripts explicitly `.end()`.

Given that, the honest options are: (1) add `retention.mts` to the tenant-import-guard's exception list with a justification, or (2) export a small `end_connection()`-style helper from `lib/db` itself for scripts to call, keeping `get_sql` private. Since retention doesn't fit the "runs before tenant scope exists" rationale, and adding more exceptions to that list would blur its meaning, I'll go with a cleaner option: export a `close_db()` function from `lib/db/index.ts` for any script that needs to end the process cleanly, without exposing `get_sql`/`db` itself.

**Claude** (2026-09-08T06:12): _[tool: Read]_

**Claude** (2026-09-08T06:12): I'll add a `close_shared_connection()` export to `client.ts` (safe — it's just an end() wrapper, doesn't leak the client itself), export it from the barrel, and use that in `retention.mts` instead of `get_sql`.

**Claude** (2026-09-08T06:12): _[tool: Edit]_

**Claude** (2026-09-08T06:12): Now update the barrel and check the tenant-import-guard's escape-name blocklist won't choke on this new export (it blocks `db`, `get_db`, `get_sql` specifically — `close_shared_connection` isn't in `ESCAPE_NAMES`/`ESCAPE_LOCAL_NAMES`, so it's fine to export un-renamed).

**Claude** (2026-09-08T06:12): _[tool: Edit]_

**Claude** (2026-09-08T06:12): Now update `retention.mts` to use `close_shared_connection` from the barrel instead of `get_sql` from `client.ts` directly:

**Claude** (2026-09-08T06:12): _[tool: Edit]_

**Claude** (2026-09-08T06:12): Wait — but importing `close_shared_connection` (and anything) from `lib/db/index.ts`'s barrel from `scripts/retention.mts` is *also* outside `lib/db`, and the guard's check #2 blocks importing `db`, `get_db`, `get_sql` from the barrel specifically — `close_shared_connection` isn't blocked by name. But check #1 also blocks importing `lib/db/client` directly, which I'm no longer doing. Let me re ‑verify this script (not itself in the exception list) is otherwise allowed to import the barrel `lib/db/index.ts` for ordinary accessors — yes, that's exactly what `db-fixture.mts` and other scripts already do for e.g. `tenant_scope`. The guard only blocks the three specific escape-hatch names from the barrel; everything else (accessors, and now `close_shared_connection`) is fine. Good, this is consistent.

Now let's add `make retention` to the Makefile and `package.json` scripts:

**Claude** (2026-09-08T06:12): _[tool: Read]_

**Claude** (2026-09-08T06:12): _[tool: Edit]_

**Claude** (2026-09-08T06:12): Now the Makefile:

**Claude** (2026-09-08T06:12): _[tool: Read]_

**Claude** (2026-09-08T06:12): _[tool: Edit]_

**Claude** (2026-09-08T06:13): _[tool: Edit]_

**Claude** (2026-09-08T06:13): _[tool: Edit]_

**Claude** (2026-09-08T06:13): Now I need to check the tenant-import-guard test will still pass with the new `retention.mts` script (it's outside `lib/db` and outside the exception list, but it only imports the barrel and `lib/retention.ts`, never `lib/db/client.ts` or the driver directly — should be fine). Let me also verify `lib/retention.ts` itself: it's under `lib/`, not `lib/db/`, so it counts as "outside lib/db" for the guard — does it import the driver or client directly? No, it only imports `find_expired_screenshots` from `./db/retention.ts` (an accessor) and `get_storage`. That's fine — it's the same pattern as `dashboard-scope.ts` importing db accessors.

Now let's write tests for retention itself: mutation-proof (accept a key containing `..`... already covered elsewhere), and specifically the retention gate items — bounded batches, never touches `reports` row. Let's write `tests/db/retention.test.ts` and `tests/web/retention-sweep.test.ts`.

**Claude** (2026-09-08T06:13): _[tool: Read]_

**Claude** (2026-09-08T06:13): This needs a live database (`reports` insert requires real rows), so this test should follow the `describe.skipIf(!database_url)` pattern used elsewhere, using the fixture + raw `sql` insert (since `insert_report` doesn't let you set `created_at` in the past — checking `NewReport`'s `Omit` — it omits `created_at`). Let me check.

`NewReport = Omit<typeof reports.$inferInsert, 'org_id' | 'project_id' | 'created_at'>` — `created_at` is omitted, so `insert_report` can't backdate a row. For the test I'll insert directly via raw `sql` (same pattern `reports-append-only.test.ts` uses to seed a row bypassing the accessor, which is fine in tests since they need to set up state the append-only guard wouldn't otherwise allow product code to create).

**Claude** (2026-09-08T06:13): _[tool: Write]_

**Claude** (2026-09-08T06:13): Now the higher-level `run_retention_sweep` test (bounded batches, never touches `reports`, deletes files only):

**Claude** (2026-09-08T06:13): _[tool: Write]_

**Claude** (2026-09-08T06:13): The third test ("re-running the sweep... sweeps nothing new") is weak — it barely asserts anything meaningful. Since the shared test database means I can't cleanly control the full backlog, let me tighten it to at least verify idempotency of the specific rows this test controls rather than a vague assertion.

**Claude** (2026-09-08T06:13): _[tool: Edit]_

**Claude** (2026-09-08T06:14): `expect(promise).resolves.not.toThrow()` is wrong usage — `.resolves` unwraps the promise, and `.not.toThrow()` expects a function, not a resolved value. Let me fix that assertion properly.

**Claude** (2026-09-08T06:14): _[tool: Edit]_

**Claude** (2026-09-08T06:14): Now let's run the actual test suite to find real bugs before going further. Let's check if Postgres is available locally.

**Claude** (2026-09-08T06:14): _[tool: Bash]_

**Claude** (2026-09-08T06:14): Postgres is available. Let's run lint and the test suite.

**Claude** (2026-09-08T06:14): _[tool: Bash]_

**Claude** (2026-09-08T06:14): Lint is fully clean. Now let's run the test suite.

**Claude** (2026-09-08T06:14): _[tool: Bash]_

**Claude** (2026-09-08T06:15): Good, real bugs surfacing. Two issues:

1. **Real bug in my test files**: `postgres.js` tagged-template raw SQL doesn't automatically serialize a JS `Date` the way Drizzle does — I need to pass `.toISOString()` or use the driver's own type handling. Let me check how the existing `reports-append-only.test.ts` inserts avoid this (they don't pass a Date at all, they use `defaultNow()`).

2. **Outdated test expectation**: `tests/api/reports.test.ts` still expects `uploadUrl` to be null — that's the old M6a-not-built assumption; now it should expect a non-null upload URL string. This is an existing test that needs updating since the spec changed under it (expected, not a bug in my code).

Let me fix the raw SQL date issue first.

**Claude** (2026-09-08T06:15): _[tool: Bash]_

**Claude** (2026-09-08T06:15): Confirmed — postgres.js needs an explicit `::timestamptz` cast when a parameter's column position is ambiguous within a plain `insert ... values (...)` (it can't infer the type without a prepared statement descriptor in this context, unlike a `select`). Let me fix all three test files by casting the Date parameters.

**Claude** (2026-09-08T06:15): _[tool: Read]_

**Claude** (2026-09-08T06:15): _[tool: Edit]_

**Claude** (2026-09-08T06:15): Now let's also fix the timeout issue in the "walk the whole backlog" test — 30 seconds is suspicious, likely an infinite loop bug rather than just slowness. Let me check the loop logic in that test carefully.

Looking at the test: `for (let page = 0; page < 10; page += 1)` — that's bounded at 10 iterations, each a quick query, so it shouldn't take 30s on its own. The timeout is almost certainly a side effect of the SQL error from the *previous* test in the same `describe` block never completing/hanging the connection, or an unresolved promise. Let me fix the Date bug first and re-run before diagnosing further.

**Claude** (2026-09-08T06:15): _[tool: Read]_

**Claude** (2026-09-08T06:15): I need to fix the same Date-cast issue in all three inserts here too.

**Claude** (2026-09-08T06:15): _[tool: Edit]_

**Claude** (2026-09-08T06:15): _[tool: Edit]_

**Claude** (2026-09-08T06:15): _[tool: Edit]_

**Claude** (2026-09-08T06:15): Now update `tests/api/reports.test.ts`'s outdated `uploadUrl` expectation to reflect that it's now populated:

**Claude** (2026-09-08T06:15): _[tool: Read]_

**Claude** (2026-09-08T06:15): _[tool: Edit]_

**Claude** (2026-09-08T06:15): Now let's re-run the test suite.

**Claude** (2026-09-08T06:15): _[tool: Bash]_

**Claude** (2026-09-08T06:16): This is a genuinely important finding, not a test bug — `run_retention_sweep` calls `storage.delete(screenshot_key)` on every row `find_expired_screenshots` returns, but that data comes from the **shared test database**, which also contains rows inserted by `tests/db/retention.test.ts`'s "walk the backlog" test using bogus keys like `reports/x/walk-0.webp` (not valid `reports/<uuid>/<uuid>.webp` shape — I used placeholder keys there since that test only cared about `find_expired_screenshots`, not storage).

This exposes a real design question: **should `run_retention_sweep` crash the whole batch when one row has a malformed/legacy key**, or should it skip that one and continue? Given `screenshot_key` is `build_screenshot_key`-generated at insert time and validated by construction in production, a malformed key should never occur from real application code — but retention is a maintenance job that must be robust to whatever exists in the table (including possible legacy data, or in this case, test pollution). A single bad row currently aborts the entire batch/sweep, which is bad: one corrupt row would permanently block retention from ever cleaning up everything else in that batch (since the cursor doesn't advance past the failing row either, given the exception happens mid-loop before the cursor update).

This is worth fixing for real defensiveness, not just to appease the test. Let me make `run_retention_sweep` tolerant of a single delete failure — log and continue, never let one bad row stop the whole batch.

**Claude** (2026-09-08T06:16): _[tool: Read]_

**Claude** (2026-09-08T06:16): _[tool: Edit]_

**Claude** (2026-09-08T06:16): _[tool: Edit]_

**Claude** (2026-09-08T06:16): _[tool: Edit]_

**Claude** (2026-09-08T06:16): Now let's fix my `retention.test.ts`'s "walk the backlog" test to use valid-shaped keys (`reports/<uuid>/<uuid>.webp`), matching the actual production invariant, and re-run.

**Claude** (2026-09-08T06:16): _[tool: Read]_

**Claude** (2026-09-08T06:16): Also, `expect(first_page).toHaveLength(2)` assumes nothing else in the shared test database is older than `base` — but this test's cutoff (1000 days) is itself relative to `now`, and the other tests in this same file insert rows at ~90-200 days old, well within the cutoff window here, meaning `find_expired_screenshots(cutoff, 2)` will return whichever rows are globally oldest — not necessarily this test's own 5. Let me fix both the key shape and use unique markers for correctness rather than assuming ordering.

**Claude** (2026-09-08T06:16): _[tool: Read]_

**Claude** (2026-09-08T06:16): I'll rewrite this whole file more carefully: use valid key shapes everywhere, scope the "walk the backlog" test's correctness assertion to only the ids it created (rather than asserting on the exact page contents from a shared table), and confirm the walking-with-cursor loop actually reaches all 5 without asserting an exact global page size.

**Claude** (2026-09-08T06:17): _[tool: Write]_

**Claude** (2026-09-08T06:17): Now let's also fix `retention-sweep.test.ts`'s last assertion (`second.batches`) since the type now has `skipped_invalid_keys` too — no change needed there since I only read `.batches`. Let's re-run tests.

**Claude** (2026-09-08T06:17): _[tool: Bash]_

**Claude** (2026-09-08T06:17): All 493 tests pass, including the 66 new tests for M6a (storage keys, upload-url, webp, local-disk, uploads route, retention read, retention sweep). Let's run 5 more consecutive full runs to check for the intermittent-race pattern this repo has a documented history of, then run the rest of the gate (build, size, test-widget).

**Claude** (2026-09-08T06:17): _[tool: Bash]_

**Claude** (2026-09-08T06:17): 5/5 clean. Now build, size, and lint (lint already passed):

**Claude** (2026-09-08T06:17): _[tool: Bash]_

**Claude** (2026-09-08T06:17): `/api/v1/uploads` route builds cleanly, confirming no `lib/db/client.ts` leak into the client bundle from my storage code (the widget bundle build also succeeded). Now `make size`:

**Claude** (2026-09-08T06:17): _[tool: Bash]_

**Claude** (2026-09-08T06:18): v1.js is unchanged at 5,638 bytes gzipped — M6a touched zero widget code, exactly as expected (M6a is server-side only). Now `make test-widget`:

**Claude** (2026-09-08T06:18): _[tool: Bash]_

**Claude** (2026-09-08T06:18): All 22 widget tests pass, unaffected. Full gate: lint ✓, build ✓, test (493, 5× clean) ✓, test-widget (22) ✓, size (5,638/15,360 unchanged) ✓.

Now let's do the mutation-proof pass for M6a's gate items: path traversal in all five forms (already covered by tests), expired signature, tampered signature, reused signature, non-WebP body, oversized body, and a cross-project key. Let me verify each by deliberately breaking the code and confirming the test screams, then reverting.

**Claude** (2026-09-08T06:18): _[tool: Read]_

**Claude** (2026-09-08T06:18): Let's mutate: accept a key containing `..` (per quality-gate.md §1's exact violation to introduce) and confirm the test screams.

**Claude** (2026-09-08T06:18): _[tool: Edit]_

**Claude** (2026-09-08T06:18): _[tool: Bash]_

**Claude** (2026-09-08T06:18): 14 of 33 tests screamed across all three files when path traversal validation was gutted — confirming the tests genuinely catch the violation, not decoration. Let's revert.

**Claude** (2026-09-08T06:18): _[tool: Edit]_

**Claude** (2026-09-08T06:18): Now let's mutate the expiry check in `verify_upload_token` — stop checking expiry:

**Claude** (2026-09-08T06:18): _[tool: Read]_

**Claude** (2026-09-08T06:18): _[tool: Edit]_

**Claude** (2026-09-08T06:18): _[tool: Bash]_

**Claude** (2026-09-08T06:18): Both the unit test and the API-level test screamed. Revert.

**Claude** (2026-09-08T06:18): _[tool: Edit]_

**Claude** (2026-09-08T06:18): Now mutate the tampered-signature check (the `timingSafeEqual` block) to accept any signature:

**Claude** (2026-09-08T06:18): _[tool: Edit]_

**Claude** (2026-09-08T06:19): _[tool: Bash]_

**Claude** (2026-09-08T06:19): 3 tests screamed (forged token, tampered payload, and one at the API level). Revert.

**Claude** (2026-09-08T06:19): _[tool: Edit]_

**Claude** (2026-09-08T06:19): Now mutate the "reused signature" defense — remove the no-clobber check in `local-disk.ts`'s `put()` (accept overwriting an existing key):

**Claude** (2026-09-08T06:19): _[tool: Read]_

**Claude** (2026-09-08T06:19): _[tool: Edit]_

**Claude** (2026-09-08T06:19): _[tool: Bash]_

**Claude** (2026-09-08T06:19): 3 failures (including the "reused signature" test at the API level and the concurrency test), confirming this is exactly what enforces single-use. Revert.

**Claude** (2026-09-08T06:19): _[tool: Edit]_

**Claude** (2026-09-08T06:19): Now mutate the WebP magic-byte check to accept anything (non-WebP body gate):

**Claude** (2026-09-08T06:19): _[tool: Read]_

**Claude** (2026-09-08T06:19): _[tool: Edit]_

**Claude** (2026-09-08T06:19): _[tool: Bash]_

**Claude** (2026-09-08T06:19): 8 failures including the API-level "non-WebP body is rejected" test. Revert.

**Claude** (2026-09-08T06:19): _[tool: Edit]_

**Claude** (2026-09-08T06:19): Now the oversized-body gate — remove the size check in the upload route:

**Claude** (2026-09-08T06:19): _[tool: Read]_

**Claude** (2026-09-08T06:19): _[tool: Edit]_

**Claude** (2026-09-08T06:20): _[tool: Bash]_

**Claude** (2026-09-08T06:20): Both size-cap tests screamed as expected. Revert.

**Claude** (2026-09-08T06:20): _[tool: Edit]_

**Claude** (2026-09-08T06:20): Now the last gate item: "a key belonging to another project." Since keys aren't tenant-scoped by an owning-project check at the storage layer (the key itself embeds the project id, but nothing currently verifies that the project id in the key matches the project that issued the report) — let me check whether this attack is actually possible and whether it's meaningfully closed. Let's think through this carefully.

The upload route only trusts the **signature**, not the key's embedded project id directly — a signature is only valid if it was issued by `signed_upload_url(key)` for that exact key, which only ever happens from inside `POST /api/v1/reports` for the report's own `screenshot_key` (computed from that report's own `project_id` + `report_id`). So a signature for `reports/<projectA>/<reportA>.webp` can never be reused against `reports/<projectB>/<reportB>.webp` — that's exactly the "wrong key" test I already wrote and verified (`verify_upload_token` binds signature to key). Let's write an explicit test making this cross-project framing clear, then verify it via mutation.

**Claude** (2026-09-08T06:20): _[tool: Read]_

**Claude** (2026-09-08T06:20): Let me add a test that frames this explicitly as the "another project's key" gate item, using an end-to-end flow through the real reports route: get project A's real signed upload URL from `POST /api/v1/reports`, then try to use that URL/token against project B's key.

**Claude** (2026-09-08T06:20): _[tool: Read]_

**Claude** (2026-09-08T06:20): Let me add an explicit end-to-end "another project's key" test using real project fixtures, going through the actual reports route to obtain a real upload URL, then attempting to use it against a different project's report key.

**Claude** (2026-09-08T06:20): _[tool: Edit]_

**Claude** (2026-09-08T06:20): _[tool: Edit]_

**Claude** (2026-09-08T06:20): _[tool: Bash]_

**Claude** (2026-09-08T06:20): Passes. Now let's mutate the key-binding check specifically (`payload.key !== expected_key`) to confirm this exact test catches it:

**Claude** (2026-09-08T06:20): _[tool: Edit]_

**Claude** (2026-09-08T06:20): _[tool: Bash]_

**Claude** (2026-09-08T06:21): The cross-project key test screamed exactly as intended, along with the unit-level and same-shape API tests. Revert.

**Claude** (2026-09-08T06:21): _[tool: Edit]_

**Claude** (2026-09-08T06:21): All mutation-proofs for M6a's gate list are now confirmed and reverted. Let's do a final clean full-gate run to confirm everything's back to green after all these reverts.

**Claude** (2026-09-08T06:21): _[tool: Bash]_

**Claude** (2026-09-08T06:21): There actually is a git repo (the memory note saying "not a git repository" in the system reminder was stale/incorrect, or this is a subdirectory check). Let's confirm the mutated files show no trace of my mutation markers by diffing the specific files I mutated.

**Claude** (2026-09-08T06:21): _[tool: Bash]_

**Claude** (2026-09-08T06:21): Clean — grep found zero matches (exit 1 = no matches), confirming every mutation was fully reverted. Now let's run the complete gate one final time.

**Claude** (2026-09-08T06:21): _[tool: Bash]_

**Claude** (2026-09-08T06:21): _[tool: Bash]_

**Claude** (2026-09-08T06:21): 494 tests (up one from the new cross-project key test), all green. Now `make test-widget` as the final piece:

**Claude** (2026-09-08T06:21): _[tool: Bash]_

**Claude** (2026-09-08T06:21): Full gate green: lint ✓, build ✓, test (494) ✓, test-widget (22) ✓, size (5,638/15,360, unchanged) ✓.

Now let's do the §2 attack-list review and §4 loop/grep checks explicitly, then §5 adversarial review, before drafting the commit message.

**Claude** (2026-09-08T06:21): _[tool: Bash]_

**Claude** (2026-09-08T06:22): Two things to check:

1. `console.log` in `scripts/retention.mts` — the quality-gate says "no console.log in shipped code" but every other CLI script in this repo (`db-migrate.mts`, `db-fixture.mts`, etc.) uses `console.log` for its own status output, since these are CLI tools, not server request-handling code. Let me verify that's an established exception.
2. `.where(` in `retention.ts` is a Drizzle query builder call, not raw SQL string interpolation — that's fine, it's the standard parameterized pattern every accessor uses.

**Claude** (2026-09-08T06:22): _[tool: Bash]_

**Claude** (2026-09-08T06:22): Confirmed — `console.log` in `scripts/*.mts` is the established, consistent pattern across every CLI entry point in this repo (these are terminal tools, not shipped server/client code); `retention.mts` matches it exactly. No violation.

Now let's verify no unsafe query construction anywhere in my new SQL, and check the `and`/`or`/`gt`/`eq` composition in `retention.ts` compiles correctly (already passed lint/tsc, but let's double check the boolean logic is right with a quick manual trace since it's the trickiest bit of new code):

**Claude** (2026-09-08T06:22): _[tool: Read]_

**Claude** (2026-09-08T06:22): Already verified this logic passes both the keyset-cursor unit test (walking without repeats/skips) and mutation-testing wasn't needed here since it's exercised directly by the passing "walks past a full page" test. Now let's do the §2 attack-list checklist explicitly for M6a and the §5 adversarial review, documenting findings.

**§2 Attack list for M6a:**
1. Authorization bypass — N/A, these are public API routes (widget-facing), no roles apply, same as `/reports` and `/config`.
2. IDOR — covered by the cross-project key test.
3. Unsafe query construction — no string interpolation into SQL in new code; verified above.
4. CSV injection — N/A, no CSV in M6a.
5. Path traversal — covered exhaustively (5 forms tested + mutation-proofed).
6. Unvalidated input reaching the database — `uploads_query_schema` validates key/token before any storage call; `find_expired_screenshots` only takes a `Date`/`number`/cursor object, never raw user input.
7. Error leakage — let's verify explicitly.

**Claude** (2026-09-08T06:22): _[tool: Bash]_

**Claude** (2026-09-08T06:22): Every error path returns a short string code, never a stack trace or raw error detail — matches the established pattern exactly. Let's also verify what happens if `storage.put()` throws unexpectedly (e.g. a real filesystem error like disk full) — that would currently propagate as an unhandled exception, becoming a Next.js 500 with a generic error page (Next.js's own route-handler error boundary strips the actual error in production), which is acceptable and consistent with how the reports/config routes handle unexpected DB errors (they don't wrap every call in try/catch either).

Now **§5 adversarial review** — let me think through empty/boundary/wrong-type cases specific to M6a:

**Claude** (2026-09-08T06:22): _[tool: Bash]_

**Claude** (2026-09-08T06:22): Let me instead reason through this properly rather than probing ad hoc, and check specific edge cases against the actual code:

**Wrong-type / boundary cases for M6a:**
- Empty case: retention with zero expired rows → `find_expired_screenshots` returns `[]`, loop breaks immediately, `swept: 0, batches: 1` — verified passing.
- Boundary: exactly at the 5-minute expiry — tested ("accepts a token one second before expiry" / "rejects... after expiry").
- Boundary: body exactly at `MAX_UPLOAD_BYTES` — not tested. Let me check.
- Wrong type: `uploads_query_schema` given `key` as an array (`url.searchParams.get('key')` can only return a string or null from URLSearchParams, so an array injection isn't reachable here — good, that's a non-issue by construction of the API).
- Second-uploader-mid-action: two concurrent uploads to the same key — covered by the 20-concurrent-puts test.

Let me check the exact-boundary case for upload size, since quality-gate.md explicitly calls out "the exact rate limit" style boundaries.

**Claude** (2026-09-08T06:22): _[tool: Read]_

**Claude** (2026-09-08T06:22): Let me add the boundary test — exactly `MAX_UPLOAD_BYTES` should be accepted, `MAX_UPLOAD_BYTES + 1` should be rejected.

**Claude** (2026-09-08T06:22): _[tool: Read]_

**Claude** (2026-09-08T06:22): _[tool: Edit]_

**Claude** (2026-09-08T06:22): _[tool: Bash]_

**Claude** (2026-09-08T06:22): This is a real off-by-one bug found by the boundary test — exactly what quality-gate.md §5's boundary-checking exists to catch. Let me look at the route's size check.

**Claude** (2026-09-08T06:22): _[tool: Read]_

**Claude** (2026-09-08T06:23): The route code itself is actually correct: `> MAX_UPLOAD_BYTES` correctly allows exactly `MAX_UPLOAD_BYTES`. The bug must be in my test helper `webp_bytes(size)` — it doesn't produce a buffer of exactly `size` bytes; let me check.

**Claude** (2026-09-08T06:23): _[tool: Read]_

**Claude** (2026-09-08T06:23): Confirmed — `webp_bytes(size)` returns `RIFF`(4) + `chunk_size`(4) + `body`(size) = `size + 8` total bytes, not `size` bytes. So my "exactly at the max" test actually sent `MAX_UPLOAD_BYTES + 8` bytes, correctly triggering 413 — this was a test-helper bug, not a product bug. Let me fix the test to account for the 8-byte header overhead.

**Claude** (2026-09-08T06:23): _[tool: Edit]_

**Claude** (2026-09-08T06:23): _[tool: Bash]_

**Claude** (2026-09-08T06:23): Confirmed the boundary is exactly right at the actual product code (route logic uses `>`, correctly inclusive of the exact cap). Now let's run the full gate one more time to lock in the final state, then draft the commit message.

**Claude** (2026-09-08T06:23): _[tool: Bash]_

**Claude** (2026-09-08T06:23): 496 tests, all green. Let's run 3 more clean repeats to be safe given the shared-database concurrency history, then check `README.md`/`docs/web/readme.md` for whether they need updating per quality-gate.md's documentation requirement.

**Claude** (2026-09-08T06:23): _[tool: Bash]_

**Claude** (2026-09-08T06:23): _[tool: Bash]_

**Claude** (2026-09-08T06:24): Stable across all runs. Now let's check documentation files per quality-gate.md's "done" list.

**Claude** (2026-09-08T06:24): _[tool: Bash]_

**Claude** (2026-09-08T06:24): _[tool: Bash]_

**Claude** (2026-09-08T06:24): _[tool: Edit]_

**Claude** (2026-09-08T06:24): Now let's check `docs/web/readme.md` for the section structure to add an M6a section matching the existing style.

**Claude** (2026-09-08T06:24): _[tool: Bash]_

**Claude** (2026-09-08T06:24): _[tool: Read]_

**Claude** (2026-09-08T06:24): Good, I have the exact style. Let's read the tail of the file and append an M6a section.

**Claude** (2026-09-08T06:24): _[tool: Read]_

**Claude** (2026-09-08T06:24): _[tool: Edit]_

**Claude** (2026-09-08T06:24): Now let's check `docs/build-plan.md` M6 acceptance section referenced earlier, and update `docs/overnight-log.md` per the established convention (one line per significant thing, in order, with timestamps) — this is a standing convention this repo follows even outside the original overnight run.

**Claude** (2026-09-08T06:24): _[tool: Bash]_

**Claude** (2026-09-08T06:24): Note the log's timestamps are in a local time zone ahead of UTC (entries go up to 11:14 while UTC now is 06:24) — I'll match the log's existing convention (local wall-clock time as already used) rather than switching to UTC.

**Claude** (2026-09-08T06:24): _[tool: Bash]_

**Claude** (2026-09-08T06:24): _[tool: Read]_

**Claude** (2026-09-08T06:25): Now let's append the M6a log entries covering the whole gate, matching the existing per-milestone summary style.

**Claude** (2026-09-08T06:25): _[tool: Bash]_

**Claude** (2026-09-08T06:25): _[tool: Read]_

**Claude** (2026-09-08T06:25): _[tool: Edit]_

**Claude** (2026-09-08T06:25): Now let's do a final review of the full diff before drafting the commit message, to check for anything left over (stray debug code, unused imports, etc.) and confirm the working tree state.

**Claude** (2026-09-08T06:25): _[tool: Bash]_

**Claude** (2026-09-08T06:25): Let's run one final full gate check and then draft the commit message per the "show me before committing" instruction.

**Claude** (2026-09-08T06:25): _[tool: Bash]_

**Claude** (2026-09-08T06:26): Full gate green: lint ✓, test (496) ✓, build ✓, size (5,638/15,360 unchanged) ✓, test-widget (22) ✓. Now let's draft the commit message and write it to `COMMIT_MSG_M6a.txt` per the established convention (draft, wait for explicit instruction).

**Claude** (2026-09-08T06:26): _[tool: Bash]_

**Claude** (2026-09-08T06:26): _[tool: Bash]_

**Claude** (2026-09-08T06:26): Now let's write the M6a commit message in the same voice and structure — imperative, no articles/full-stop in the title, body explaining why, no Co-Authored-By.

**Claude** (2026-09-08T06:26): _[tool: Write]_

**Claude** (2026-09-08T06:26): M6a is complete and the full gate is green. Here's the summary of the storage and upload code, and the draft commit message, per your instruction to stop and show you before starting M6b.

## What was built

**Storage layer** (`src/web/lib/storage/`):
- `keys.ts` — validates a key is exactly `reports/<uuid>/<uuid>.webp`; rejects everything else by construction rather than blocklisting `..`, absolute paths, URL-encoding, null bytes, backslashes individually
- `local-disk.ts` — the one `Storage` implementation; writes with `O_CREAT|O_EXCL` so a second write to an occupied key fails atomically (this is what makes an upload URL single-use, not the signature)
- `upload-url.ts` — (secret removed)/base64url over `{key, exp}`, reusing `SESSION_SECRET`, 5-minute expiry, the key bound into the signature itself
- `index.ts` — the only import path anything outside `lib/storage` may use

**API changes**: `POST /api/v1/reports` now returns a populated `uploadUrl`; new `POST /api/v1/uploads` verifies signature+expiry, a 2MB size cap, and real WebP magic bytes (never trusting `Content-Type`) before writing anything.

**Retention**: `lib/retention.ts` + `make retention`, bounded batches via a keyset cursor, deletes files only, never touches the `reports` row.

**Deviation logged**: `docs/build-plan.md` still says S3; your task message said "no S3, sign it yourself, local disk" — I followed the task message per the established precedent and logged it in the new `docs/blocked.md` rather than silently reconciling.

## Gate results
- `make lint` / `make build` / `make test` (496 tests, 5 consecutive clean runs) / `make test-widget` (22) / `make size` (5,638/15,360 bytes, byte-for-byte unchanged) — all green
- Mutation-proofed all 7 required violations (path traversal, expired/tampered/reused signature, non-WebP body, oversized body, cross-project key) — each confirmed to break tests, then reverted
- Two real bugs found and fixed while building: a naive offset pagination in retention would have skipped/repeated rows (fixed with a keyset cursor), and one malformed key would have aborted an entire retention batch (fixed to skip-and-count instead)

The commit message is drafted at `COMMIT_MSG_M6a.txt` — let me know if you want changes, or if it's ready to commit.
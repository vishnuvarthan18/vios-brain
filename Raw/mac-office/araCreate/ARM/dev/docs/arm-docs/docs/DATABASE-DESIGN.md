# ARM Platform — Database Design

> Every table and collection across all three databases — PostgreSQL `arm_core`, PostgreSQL `arm_admin`, and MongoDB `arm-calendar` — with field-level types, indexes, cross-collection relationships, migration inventories, and retention policy.
>
> This document was written from scratch (no prior partial coverage existed) by reading every TypeORM entity, every Mongoose schema, and every migration file directly — not from any service's `CLAUDE.md` summary. That direct read surfaced one severe bug, confirmed against a live migrated database and then fixed: see §4.
>
> Verified against every `*.entity.ts` in `core/arm-core-be` and `apps/arm-admin/src/backend`, every `*.schema.ts` in `apps/arm-app-calendar/src/backend`, all 14 migration files across the two Postgres databases, and `apps/arm-app-calendar/src/backend/src/migrations/startup-migration.service.ts` (2026-08-10).

---

## 1. Database ownership (summary)

| Database | Engine | Owned by | Tables/collections |
|---|---|---|---|
| `arm_core` | PostgreSQL 16 | `arm-core-be` (schema owner — migrations run here only) | `users`, `refresh_tokens`, `google_tokens`, `microsoft_tokens`, `registry` |
| `arm_admin` | PostgreSQL 16 | `arm-admin-be` | `admin_users`, `audit_logs`, `disabled_users` — plus **read-only** cross-database access to 3 `arm_core` tables (§4) |
| `arm-calendar` | MongoDB 7 | `arm-app-calendar` backend | `accounts`, `calendars`, `events`, `syncconfigs`, `synchistories`, `auditlogs` |

Two independent audit trails exist in this platform, not one: calendar-be's Mongo `auditlogs` collection (privileged calendar actions — §6.6) and admin-be's Postgres `audit_logs` table (admin portal write operations — §3.2). Neither reads from the other. If you're looking for "the audit log," confirm which one first.

---

## 2. PostgreSQL — `arm_core` (owned by `arm-core-be`)

```text
users ──1:N── refresh_tokens   (ON DELETE CASCADE)
users ──1:1── google_tokens    (ON DELETE CASCADE, unique index on user-id)
users ──1:1── microsoft_tokens (ON DELETE CASCADE, unique index on user-id)

registry — standalone, no FK to users
```

All column names in this database are **kebab-case in Postgres** (`first-name`, `token-hash`, etc.) as of migration `(secret removed)` — every entity's TypeORM `@Column({ name: '...' })` decorator must carry the hyphenated name explicitly. This matters enormously for §4.

### 2.1 `users`

| Column (DB) | Entity property | Type | Notes |
|---|---|---|---|
| `id` | `id` | `uuid`, PK | |
| `email` | `email` | `varchar`, unique | |
| `first-name` | `firstName` | `varchar` | |
| `last-name` | `lastName` | `varchar` | |
| `is-active` | `isActive` | `boolean`, default `true` | See §7 — not the same mechanism as admin-be's disable/deny-list |
| `is-verified` | `isVerified` | `boolean`, default `false` | |
| `google-id` | `googleId` | `varchar`, nullable | |
| `microsoft-id` | `microsoftId` | `varchar`, nullable | Added by `AddMicrosoftSupport` — see §2.6 |
| `auth-provider` | `authProvider` | `varchar`, nullable, default `'local'` | `'local'` \| `'google'` \| `'microsoft'` \| `'email-otp'`. Records how the account was **created**; written once at signup and never rewritten. Not consulted by the login path — email OTP works for any account regardless of this value |
| `picture` | `picture` | `varchar`, nullable | |
| `approval-status` | `approvalStatus` | `varchar(16)`, NOT NULL, default `'approved'`, CHECK | `'pending'` \| `'approved'` \| `'rejected'`. Added by `AddApprovalStatusToUsers`. **Orthogonal to `is-active`**: this means "never let in yet", that means "suspended". Every pre-existing row was defaulted to `'approved'`, so the column is inert until `SIGNUP_REQUIRES_APPROVAL=true` |
| `approved-at` | `approvedAt` | `timestamptz`, nullable | Set only on approval; a rejection nulls it and leaves the record to the audit log |
| `approved-by` | `approvedBy` | `uuid`, nullable | The approving admin's core user id. **Deliberately not a FK** — deleting that admin must not fail or erase the trace |
| `notify-on-signup` | `notifyOnSignup` | `boolean`, NOT NULL, default `false` | Whether this person is emailed when a signup joins the approval queue. Only offered to admins (admin-be enforces it, and `revokeAdmin` clears it), but stored here because core-be resolves recipients at signup time and cannot read `arm_admin`. Seeded `true` for `kishor@aracreate.group` by `AddSignupNotificationToUsers` |
| `created-at` | `createdAt` | `timestamp` | |
| `updated-at` | `updatedAt` | `timestamp` | |

**No `password` column** — dropped entirely by `DropPasswordColumn` (§2.6); password login was removed platform-wide (`AUTH-ARCHITECTURE.md`).

### 2.2 `refresh_tokens`

| Column (DB) | Entity property | Type | Notes |
|---|---|---|---|
| `id` | `id` | `uuid`, PK | |
| `token-hash` | `tokenHash` | `varchar`, `select: false` | SHA-256 hash, never plaintext. Excluded from default SELECTs |
| `family-id` | `familyId` | `uuid` | Groups tokens from one login event — reuse of a revoked token deletes the whole family (stolen-token defense) |
| `user-agent` | `userAgent` | `varchar(500)` | |
| `ip` | `ip` | `varchar(45)` | Sized for IPv6 |
| `expires-at` | `expiresAt` | `timestamp` | |
| `revoked` | `revoked` | `boolean`, default `false` | |
| `user-id` | `user` (FK) | `uuid`, FK → `users.id`, `ON DELETE CASCADE` | |
| `created-at` | `createdAt` | `timestamp` | |

**Indexes:** `token-hash` (lookup by presented token), `user-id` + `ip` + `user-agent` (composite), `user-id` alone, `user-id` + `expires-at` (composite), `family-id` — five indexes total on a table that's written on every login and read on every token refresh.

### 2.3 `google_tokens` / `microsoft_tokens`

Identical shape, both keyed 1:1 to `users` via a unique index on `user-id`.

| Column (DB) | Entity property | Type | Notes |
|---|---|---|---|
| `id` | `id` | `uuid`, PK | |
| `user-id` | `userId` + `user` (FK) | `uuid`, FK → `users.id`, `ON DELETE CASCADE` | Unique — one row per user per provider |
| `access-token` | `accessToken` | `text` | **Encrypted** ((secret removed), `TokenEncryptionService`) before being written — confirmed in `google-auth.service.ts`, not just documented |
| `refresh-token` | `refreshToken` | `text`, nullable | Same encryption. Nullable because Microsoft logins don't request `offline_access` by default — a null refresh token is expected there, not an error state |
| `expires-at` | `expiresAt` | `timestamp` | |
| `scope` | `scope` | `text` | |
| `needs-reauth` | `needsReauth` | `boolean`, default `false` | Set on scope-check failure or refresh failure; polled by `/v1/auth/{google,microsoft}/status/:userId` |
| `created-at` / `updated-at` | `createdAt` / `updatedAt` | `timestamp` | |

Microsoft has no server-side token-revoke API — `POST /v1/auth/microsoft/revoke` only deletes the local row; the upstream grant isn't invalidated (documented in `core-be/CLAUDE.md`, confirmed structurally by the absence of any revoke call in `microsoft-auth.service.ts`'s equivalent to Google's revoke flow).

### 2.4 `registry`

| Column (DB) | Entity property | Type | Notes |
|---|---|---|---|
| `id` | `id` | `uuid`, PK | |
| `name` | `name` | `varchar`, unique | |
| `type` | `type` | Postgres enum: `'backend'` \| `'mfe'` | |
| `url` | `url` | `varchar`, nullable | |
| `remote-url` | `remoteUrl` | `varchar`, nullable | |
| `registered-at` | `registeredAt` | `timestamp` | |
| `updated-at` | `updatedAt` | `timestamp` | |

**Corrected 2026-08-12** (was previously stated as fully unused — that was wrong): this table's *write* side is unused — nothing anywhere in the workspace ever calls `POST /v1/registry/register`, so the table is permanently empty — but its *read* side is live-wired: `core-fe/src/lib/remote-registry.ts`'s `initRemoteRegistry()` calls `GET /v1/registry/calendar` from `ProtectedRoute` on every authenticated session, ready to override the `calendar` remote's Module Federation entry URL if the table ever had a row. Because it never does, the build-time `VITE_CALENDAR_REMOTE_URL`/`VITE_ADMIN_REMOTE_URL` env vars ([MODULE-FEDERATION.md](MODULE-FEDERATION.md) §2) are what actually resolve in every environment today — the practical guidance ("don't chase this table, check the env var") still holds, just not for the reason previously stated here. Full detail in [`core/arm-core-be/REGISTRY.md`](../../../core/arm-core-be/REGISTRY.md).

### 2.5 Relationships

```
users.id ←── refresh_tokens.user-id   (1:N, cascade delete)
users.id ←── google_tokens.user-id    (1:1, cascade delete, unique)
users.id ←── microsoft_tokens.user-id (1:1, cascade delete, unique)
registry — no relationships (standalone lookup table, read on every session, never written to)
```

### 2.6 Migration inventory (`core/arm-core-be/src/database/migrations/`, run automatically at startup)

| Migration | What it did |
|---|---|
| `(secret removed)` | Original `users`/`refresh_tokens`/`google_tokens`/`registry` tables |
| `(secret removed)` | Index on `tokenHash` (pre-rename name) |
| `1747300000000-AddPictureToUsers` | Added `picture` column |
| `(secret removed)` | Composite index for login-history queries |
| `(secret removed)` | Added `familyId` — enabled the stolen-token-family-revocation defense |
| `(secret removed)` | Added `needsReauth` to `google_tokens` |
| `(secret removed)` | Additional `userId`-keyed indexes on `refresh_tokens` |
| `(secret removed)` | Unique index on `google_tokens.userId` |
| `1747900000000-AddColumnLengthConstraints` | Tightened varchar lengths (`userAgent` 500, `ip` 45) |
| `(secret removed)` | Added `microsoftId` to `users`, created `microsoft_tokens` table |
| `(secret removed)` | Removed the `password` column entirely — password auth removal |
| `1748200000000-RenameColumnsToParamCase` | Renamed **every** column across all 5 tables to kebab-case, data-preserving `ALTER ... RENAME COLUMN` (not drop/recreate). One-time, non-idempotent by nature — rerunning after it's applied would fail since the old identifiers no longer exist, which is standard for rename migrations |

This table itemises only the migrations up to `RenameColumnsToParamCase`; `1748300000000`–`1748500000000` (phone/language, avatar migration to object storage, terms acceptance) exist on disk but were never added here. All 15 run in order at `core-be` startup. This is the authoritative schema history — anything reading `arm_core` from outside this repo (§4) must track every one of these, especially the last one.

---

## 3. PostgreSQL — `arm_admin` (owned by `arm-admin-be`)

Its own 3 tables, plus read access to `arm_core` (§4).

### 3.1 Owned tables

**`admin_users`**

| Column | Type | Notes |
|---|---|---|
| `id` | `uuid`, PK | |
| `userId` | `uuid`, unique | References `arm_core.users.id` — **application-enforced only**, no DB-level FK (impossible across two separate Postgres databases) |
| `role` | Postgres enum `admin_role_enum`: `'admin'` \| `'operator'` | Adding a role requires a migration with `ALTER TYPE ... ADD VALUE` — TypeORM's `synchronize` can't alter native enums |
| `grantedBy` | `varchar`, nullable | |
| `createdAt` | `timestamp` | |

**`audit_logs`** (admin portal's own trail — distinct from calendar-be's, §1)

| Column | Type | Notes |
|---|---|---|
| `id` | `uuid`, PK | |
| `adminId` | `varchar` | |
| `action` | `varchar` | Free-text, unlike calendar-be's closed-union `action` field (§6.6) |
| `entityType` / `entityId` | `varchar` | What was acted on |
| `before` / `after` | `jsonb`, nullable | Before/after state snapshots |
| `createdAt` | `timestamp` | |

Immutable by convention (`AuditService.log()` never throws — errors are logged, never re-thrown, so a logging failure can't roll back the parent operation) — there's no DB-level append-only enforcement (no trigger blocking UPDATE/DELETE), just application discipline.

**`disabled_users`**

| Column | Type | Notes |
|---|---|---|
| `id` | `uuid`, PK | |
| `userId` | `uuid`, unique | Same cross-database, app-only reference as `admin_users.userId` |
| `disabledBy` | `varchar` | |
| `reason` | `varchar`, nullable | |
| `disabledAt` | `timestamp` | |

Mirrored into Redis (`disabled:{userId}`, 900s TTL) on write — see [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) Risk 13 and [SECURITY.md](SECURITY.md) §3 for the still-open gap where `core-be`/`calendar-be` guards don't actually check this key.

### 3.2 Migration inventory

| Migration | What it did |
|---|---|
| `1750000000001-CreateAuditLogs` | `audit_logs` table |
| `1750000000002-CreateAdminTables` | `admin_users`, `disabled_users`, `admin_role_enum` |

Run automatically on `admin-be` startup (`migrationsRun: true`, `migrationsTransactionMode: 'each'`). Seeders (`AdminUserSeeder`, `DenyListSeeder`) run after migrations, both idempotent — safe on every restart.

---

## 4. `arm-admin-be`'s read-only `arm_core` mirror entities were stale — fixed 2026-08-10

`admin-be` connects to `arm_core` with its own separately-declared TypeORM entities (`src/database/entities/core/`) rather than importing `core-be`'s real entity classes — a reasonable pattern for a read-only cross-service mirror, but it means the mirror has to be kept in sync by hand. **It hadn't been, until this fix.**

All three mirror entities were read directly from source, compared against `core-be`'s actual current schema (§2, post-`RenameColumnsToParamCase`), and corrected:

| Entity | Was mapping `firstName`/`userId`/etc. to | Real column | Status |
|---|---|---|---|
| `CoreUser` | No `name:` override at all → TypeORM defaulted to the literal property name (`firstName`, `isActive`, `createdAt`, ...) | `first-name`, `is-active`, `created-at`, ... (kebab-case) | ✅ Fixed — kebab-case `name:` overrides added |
| `CoreRefreshToken` | Same — no `name:` override (`userId`, `tokenHash`, `userAgent`, `expiresAt`, `createdAt`) | `user-id`, `token-hash`, `user-agent`, `expires-at`, `created-at` | ✅ Fixed |
| `CoreGoogleToken` | Explicit but **pre-migration snake_case** overrides (`user_id`, `access_token`, `refresh_token`, `expires_at`, `created_at`, `updated_at`) | `user-id`, `access-token`, ... (kebab-case) | ✅ Fixed |

`CoreUser` also declared a `password` column that was dropped from `arm_core.users` entirely by `DropPasswordColumn` — removed. `microsoftId`/`microsoft-id`, added to `arm_core.users` by `AddMicrosoftSupport`, was missing from the mirror entirely — added.

**No custom TypeORM `namingStrategy` is configured for either connection** (`database.module.ts` — confirmed, plain `TypeOrmModule.forRootAsync` with no `namingStrategy` option), so nothing implicitly reconciles a name mismatch like this — every entity's `name:` override is the only thing keeping it aligned with the real column, and now does.

**Confirmed both broken (before) and fixed (after) empirically, not just from source.** Ran all 12 of `core-be`'s migrations against a fresh Postgres instance to reproduce the real, current `arm_core` schema, then exercised the exact query shapes `admin-be` builds:

- Before the fix: `SELECT ... "user"."firstName" ... ORDER BY "user"."createdAt"` (what the old `CoreUser` generated) → `ERROR: column user.firstName does not exist` — confirming `GET /v1/users`, the admin portal's basic user list, failed on every call (`UsersRepository.findAll()` unconditionally orders by `createdAt`).
- Before the fix: `UPDATE "refresh_tokens" SET "revoked" = true WHERE "userId" = ...` (the old `CoreRefreshToken` shape) → `ERROR: column "userId" does not exist` — confirming the disable-user flow's token-revocation step also failed.
- After the fix: ran `admin-be`'s actual `CoreUser`/`CoreRefreshToken` entities (not equivalent SQL — the real TypeORM classes) against the same migrated schema via `createQueryBuilder('user')...orderBy('user.createdAt', 'DESC')` and `tokenRepo.update({ userId }, { revoked: true })`. Both succeeded.

**Fix applied:** `apps/arm-admin/src/backend/src/database/entities/core/{core-user,core-refresh-token,core-google-token}.entity.ts` — added/corrected `@Column({ name: '...' })` kebab-case overrides matching §2.1–§2.3 exactly, removed `CoreUser.password`, added `CoreUser.microsoftId`. All changes contained to `admin-be`; no `arm_core` schema or `core-be` change needed, since `core-be` already had the correct current shape. `admin-be`'s typecheck and full Jest suite (39/39) pass after the change.

---

## 5. MongoDB — `arm-calendar` (owned by `calendar-be`)

```text
Account ──1:N── Calendar         (Calendar.accountId = Account.accountId, the provider's own ID — not Account._id)
Calendar ──1:N── Event           (Event.calendarId = Calendar.calendarId)
Account ──1:N── Event            (Event.accountId = Account._id — Mongo ObjectId, backfilled — see §5.8)
Account ──1:N── SyncConfig       (both sourceAccountMongoId and targetAccountMongoId = Account._id)
SyncConfig ──1:N── SyncHistory   (matched by sourceCalendarId + targetCalendarId pair)
SyncConfig ──N:1── SyncGroupMeta (N SyncConfig rows for one targetCalendarId share one SyncGroupMeta row — keyed by targetCalendarId, not by the source/target pair)
```

**Note the ID-type inconsistency, verified across the schemas themselves:** `Calendar.accountId` and `Event.accountId` are **not the same kind of value**. `Calendar.accountId` stores the provider's own account identifier (matching `Account.accountId`, a `unique` string field — Google/Microsoft's own ID). `Event.accountId` stores `Account._id` (Mongo's own ObjectId, as a string) — populated by the backfill migration (§5.8) which explicitly resolves `Calendar.accountId` (provider ID) → `Account._id` (Mongo ID) as a join step. Any code walking `Event.accountId` expecting a provider ID, or `Calendar.accountId` expecting a Mongo `_id`, will silently query against the wrong field. This is also flagged in [`apps/arm-app-calendar/DELETE-CASCADE.md`](calendar/DELETE-CASCADE.md)'s own §7 as a live e2e-test gap (tests using the wrong ID type) — this document's schema read confirms the same inconsistency exists at the data-model level, not just in test code.

### 5.1 `accounts` (`Account`)

| Field | Type | Notes |
|---|---|---|
| `_id` | ObjectId, PK | Mongo-native |
| `userId` | `string`, indexed | ARM platform user ID (from `arm_core.users.id`) — one user can have multiple `Account` docs |
| `accountId` | `string`, **unique**, indexed | The provider's own account identifier — Google/Microsoft's ID, not Mongo's |
| `email` | `string`, indexed | |
| `displayName`, `organization`, `picture` | `string` | |
| `accessToken`, `refreshToken` | `string` | Encrypted at rest — same `TokenEncryptionService` pattern as `arm_core`'s Google/Microsoft token tables |
| `provider` | `string`, default `'google'` | `'google'` \| `'microsoft'` — the field the `backfillAccountProvider` startup migration (§5.8) added a default for on legacy rows |
| `related` | `string`, default `'calendar'` | Present but not documented elsewhere as consumed by anything — likely groundwork for a non-calendar future related-resource type |
| `tokenExpiryAt` | `Date` | |
| `syncDate` | `Date` | |
| `onSync` | `boolean`, default `true` | |
| `isAuthenticated` | `boolean`, default `true` | Self-heals to `false` on a 401/403 from the provider API (`markAccountUnauthenticated`, per `apps/arm-app-calendar/CLAUDE.md`) |
| `createdAt`/`updatedAt` | `Date` | `{ timestamps: true }` on the schema |

### 5.2 `calendars` (`Calendar`)

| Field | Type | Notes |
|---|---|---|
| `_id` | ObjectId, PK | |
| `userId`, `accountId` | `string`, indexed | `accountId` = provider ID, matches `Account.accountId` (see ID-type note above) |
| `calendarId` | `string`, indexed | Provider's calendar ID |
| `summary`, `description`, `timeZone` | `string` | |
| `channelId`, `channelToken`, `resourceId`, `expiration` | `string` | Google push-notification webhook channel tracking. `expiration` is stored as a **string** but compared numerically in the renewal query — a known gap, `TASK-048` per `apps/arm-app-calendar/CLAUDE.md` |
| `etag`, `syncToken` | `string` | |
| `accessRole` | `string` | For Microsoft, derived from comparing Graph's `owner.address` to the account's own email (`'owner'` only on match) — not simply copied from a `canEdit` flag |
| `webhookSupported` | `boolean`, default `true` | `false` for calendars where the provider doesn't support push (Microsoft — no Graph subscription support yet; falls back to polling) |
| `imported` | `boolean`, default `false` | Whether the user chose this calendar in the import step. Discovering a calendar is not choosing it: a newly connected account lands with every calendar `false`, and the import dialog is what admits them. Written explicitly by `CalendarRepository.bulkUpsert`'s `$setOnInsert` — and never in its `$set`, so a provider refresh cannot overwrite the choice |

**Indexes:** `channelId`+`resourceId` (compound, webhook lookup), `accountId`+`calendarId` (compound, **unique** — one Calendar doc per account/calendar pair).

**Reads of `imported` match `{ $ne: false }`, not `{ imported: true }`** — documents written before the
field existed have no value at all and must keep behaving as imported. That is why the default cannot
be the only thing writing it, and why an *absent* `imported` is not equivalent to `false`.

`imported` is a visibility filter, not an enforcement boundary. One caller reads it —
`CalendarManagementService.fetchAllAccountsWithCalendars`, which feeds the account page and the sync
pickers. Webhook registration and sync work from the calendars a sync names, so an un-imported
calendar costs no provider quota but is also not otherwise excluded from the system.

### 5.3 `events` (`Event`)

| Field | Type | Notes |
|---|---|---|
| `_id` | ObjectId, PK | |
| `kind`, `etag` | `string` | |
| `eventId` | `string`, indexed | Provider's event ID |
| `uniqueId` | `string` | |
| `calendarId` | `string`, indexed | Matches `Calendar.calendarId` |
| `accountId` | `string`, indexed | **`Account._id` as a string** — not the provider ID (see ID-type note above). Backfilled, not present on every legacy row before the startup migration ran |
| `summary`, `description` | `string` | |
| `start`, `end` | `Object` (`calendar_v3.Schema$EventDateTime`) | Typed at the TS layer, stored as free-form nested object in Mongo |
| `status` | `string` | |
| `recurrence` | `string[]` | RRULE strings |
| `recurringEventId` | `string`, indexed | |
| `extendedProperties.private.{sourceCalendarId, isSource}` | nested `Object` | How blocker events are marked and traced back to their source calendar |

**Index:** `eventId`+`calendarId` (compound, unique).

### 5.4 `syncconfigs` (`SyncConfig`)

The core mapping: one source calendar → one target calendar for a user's sync.

| Field | Type | Notes |
|---|---|---|
| `_id` | ObjectId, PK | |
| `sourceCalendarId`, `targetCalendarId` | `string`, indexed | |
| `sourceAccountMongoId`, `targetAccountMongoId` | `string`, indexed | Despite the name, both are `Account._id` (Mongo ObjectId as string) — consistently, unlike `Event.accountId`'s naming ambiguity |
| `sourceProvider`, `targetProvider` | `string`, default `CALENDAR_PROVIDER_NAME.GOOGLE` | Independently stamped per side — nothing requires source and target to be the same provider; cross-provider syncs (Google↔Microsoft) are supported by design |
| `syncFromDate` | `Date` | |
| `userId` | `string`, indexed | |
| `stoppingAt` | `Date \| null`, default `null` | Set when a sync is being torn down |

**Index:** `sourceCalendarId`+`targetCalendarId` (compound, unique) — one active config per source/target pair. `syncFromDate` is also what a Resync groups by when re-enqueueing this target's sources — a group per distinct value, not one job for the whole target; see §5.7 for the `SyncGroupMeta` row this target's sources share, and [`calendar/SYNC-FLOW.md`](calendar/SYNC-FLOW.md) §10 for Resync's full mechanics.

### 5.5 `synchistories` (`SyncHistory`)

Audit trail of sync runs — one record per run, not per config.

| Field | Type | Notes |
|---|---|---|
| `_id` | ObjectId, PK | |
| `sourceCalendarId`, `targetCalendarId`, `sourceAccountMongoId`, `targetAccountMongoId` | `string`, indexed | Mirrors the `SyncConfig` shape for the pair this run belongs to |
| `syncFromDate` | `Date` | |
| `syncStatus` | `string` enum: `pending` \| `in-progress` \| `completed` \| `partial` \| `failed` | `SyncOrchestratorService` writes `PENDING` before enqueueing the BullMQ job, so status is visible immediately |
| `syncStartTime`, `syncEndTime` | `Date` | |
| `eventsSynced` | `number` | |
| `errorMessage` | `string` | |
| `blockersAttempted`, `blockersFailed` | `number`, default `0` | |
| `failureReasons` | `string[]`, default `[]` | |
| `initiatedBy` | `string` | |

**Index — worth flagging its exact shape:** `sourceCalendarId`+`targetCalendarId`, unique, **but only via `partialFilterExpression: { syncStatus: 'pending' }`**. This means at most one `PENDING` history record can exist per source/target pair at a time (preventing duplicate concurrent sync jobs), while an unlimited number of `completed`/`failed`/`partial` historical records for that same pair are allowed — a genuinely well-designed constraint, not a simple unique index.

### 5.6 `auditlogs` (`AuditLog`, calendar-be's own — distinct from admin-be's, §1)

| Field | Type | Notes |
|---|---|---|
| `_id` | ObjectId, PK | |
| `actorUserId` | `string`, indexed | |
| `action` | closed union: `'account.delete'` \| `'sync.stop'`, indexed | Deliberately a closed TS union, not free text — "new privileged actions must be added here on purpose, not accidentally accreted by whatever string a caller passes" (schema's own comment) |
| `targetId` | `string`, indexed | |
| `outcome` | `'success'` \| `'failure'` | |
| `errorMessage` | `string`, optional | |
| `metadata` | `Object`, optional | |
| `timestamp` | `Date`, default `Date.now` | Note: this schema does **not** have `{ timestamps: true }` — `timestamp` is a manually-defined field, unlike every other calendar-be schema which uses Mongoose's automatic `createdAt`/`updatedAt` |

Only two action types tracked as of this document (`account.delete`, `sync.stop`) — a narrower scope than admin-be's free-text `audit_logs.action`, which logs a wider range of admin operations (user disable/enable/delete, role grants).

### 5.7 `syncgroupmetas` (`SyncGroupMeta`, added 2026-08-13)

One document per **target** calendar — not per source→target pairing the way `SyncConfig` is. Holds the sync's user-facing display name and the three field-visibility toggles that control what a blocker event actually shows.

| Field | Type | Notes |
|---|---|---|
| `_id` | ObjectId, PK | |
| `targetCalendarId` | `string`, **unique**, indexed | The key this collection is keyed by — every `SyncConfig` row sharing this `targetCalendarId` reads the same `SyncGroupMeta` row |
| `targetAccountMongoId` | `string`, indexed | `Account._id` of the target account — mirrors `SyncConfig.targetAccountMongoId`'s meaning, kept here purely for the teardown-cleanup queries (`deleteByTargetAccountId`), not read for sync logic |
| `userId` | `string`, indexed | |
| `name` | `string`, optional | User-facing label for the sync; no value if never set |
| `showTitle` | `boolean`, default `true` | When `false`, a blocker's title falls back to `'Busy'` instead of the source event's real summary |
| `showDescription` | `boolean`, default `true` | When `false`, a blocker's description is always empty |
| `showLocation` | `boolean`, default `true` | When `false`, a blocker's location is always unset |
| `createdAt`/`updatedAt` | `Date` | `{ timestamps: true }` on the schema |

**No document at all is a valid, common state** — a sync created before this collection existed, or one whose name/toggles have never been edited, has no `SyncGroupMeta` row. Every read site (`sync-orchestrator.service.ts`, `webhook-processor.service.ts`, `sync-query.service.ts`) treats a missing row identically to a row with every toggle `true` and no name — there is no backfill migration that creates one retroactively, and none is needed given that default.

**Upsert is partial-write, not whole-document replace**: `SyncGroupMetaRepository.upsert()` only writes the keys actually present in its input, using `$set` for provided fields and `$setOnInsert` for the schema's defaults on the ones it wasn't given — so a `POST .../sync/update` call that only changes `name` cannot accidentally reset an already-set toggle back to `true`, and a first-ever write (no prior row) still gets all three toggles correctly defaulted.

**Cleanup is not automatic on `SyncConfig` deletion** — since one `SyncGroupMeta` row can serve several `SyncConfig` rows (one per source), it's deleted explicitly, and only once the *last* `SyncConfig` row for that `targetCalendarId` is gone: in `stopSync`/`executeTeardown` (unconditional, the whole target is being torn down) and in cross-target migration/source-removal (guarded — only if zero `SyncConfig` rows remain for that target after the delete). See [`calendar/SYNC-FLOW.md`](calendar/SYNC-FLOW.md) §7–§8, §10 for the full mechanics and [`calendar/DELETE-CASCADE.md`](calendar/DELETE-CASCADE.md) §3 step 8b for the account-unlink case.

### 5.8 Migration strategy — no versioned migration files, one idempotent startup service

Unlike `core-be`/`admin-be`'s numbered TypeORM migration files, calendar-be has a single `StartupMigrationService` (`src/migrations/startup-migration.service.ts`) that runs two backfills on **every** application bootstrap, not once ever:

1. **`backfillAccountProvider`** — `updateMany({ provider: { $exists: false } }, { $set: { provider: 'google' } })`. Idempotent by construction: once every document has `provider` set, the filter matches zero documents and the update is a no-op.
2. **`backfillEventAccountId`** — for every `Event` missing `accountId`, joins through `Calendar` (by `calendarId`) to find the provider account ID, then through `Account` (by `accountId`) to resolve that to `Account._id`, then writes `Event.accountId = Account._id` (confirming the ID-type distinction from the top of §5). Also idempotent via `$exists: false` filtering.

**No migration-tracking collection or applied-migrations log exists** — idempotency is achieved purely by re-checking `$exists: false` on every single boot, not by recording "this migration already ran." This means both backfills re-scan their respective collections on every deploy/restart (a full `find`/`updateMany` pass), which is harmless for correctness but is a cost that grows with collection size — worth knowing if `events` ever grows large enough for a full collection scan on every backend restart to become noticeable. There is no separate directory of one-off historical migration scripts the way the suggested doc outline for this file assumed; this one service **is** the entire migration mechanism for MongoDB in this platform.

---

## 6. Data retention / soft-delete vs. hard-delete policy

**Verified: no `@DeleteDateColumn` (TypeORM) and no `deletedAt`/`isDeleted` field exists anywhere in any of the three databases.** This platform does not implement conventional soft-deletion anywhere — every entity/schema read for this document uses hard-delete semantics for actual row/document removal.

The one exception is a naming false-positive, worth clearing up explicitly: `core-be`'s `UsersRepository.softDeleteUser()` doesn't soft-delete in the conventional sense — it flips `users.isActive = false`. This method is **dead code**, confirmed by grep — nothing outside its own repository file and spec calls it. The `findAllUsers()` method that filters `WHERE isActive = true` is called by `users.service.ts`, but `core-be`'s `users.controller.ts` has its `create`/`getAll`/`remove` generic-controller routes permanently disabled (per `core-be/CLAUDE.md`), so this filtered-list path likely has no live HTTP route reaching it either.

**This means `users.isActive` and admin-be's disable mechanism (`disabled_users` + Redis deny-list, §3.1) are two entirely separate, non-integrated concepts** — disabling a user through the admin portal never touches `users.isActive`, and nothing reads `users.isActive` to decide whether a user is "disabled" in the admin-portal sense. Don't assume one reflects the other.

**Delete policy is hard-delete platform-wide, per service:**

| Service | Delete surface | Mechanism |
|---|---|---|
| `core-be` | `DELETE`-style user removal (called from `admin-be`) | Hard delete, `ON DELETE CASCADE` cleans up `refresh_tokens`/`google_tokens`/`microsoft_tokens` automatically at the FK level |
| `admin-be` | `DELETE /v1/users/:id` | Hard delete from `arm_core.users` (cascades per above) plus its own cleanup — CLAUDE.md's own wording ("Hard delete user from arm_core + cleanup") |
| `calendar-be` | Account unlink | Ordered application-level cascade, not a DB-level cascade (Mongo has no FK constraints) — ordered cleanup across `Event`/`Calendar`/`SyncConfig`/`SyncHistory`/`Account` documented in full in [`apps/arm-app-calendar/DELETE-CASCADE.md`](calendar/DELETE-CASCADE.md), including its own live gaps (poll schedulers not cleared, no token revoke, no Microsoft subscription cleanup) |

No documented backup/retention *schedule* exists for either database beyond the manual `make backup-postgres`/`backup-mongodb` tooling described in [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §9 — that section already covers what's implemented (local-disk backups, a Postgres-only restore drill) and what isn't (off-host storage, a Mongo restore drill, any cron schedule). Not repeated here.

---

## 7. Related documents

| Doc | Role |
|---|---|
| [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) | JWT structure, RBAC, the deny-list gap this document's §3.1/§6 reference |
| [SECURITY.md](SECURITY.md) | Token encryption detail behind §2.3/§5.1's "encrypted at rest" claims |
| [`apps/arm-app-calendar/DELETE-CASCADE.md`](calendar/DELETE-CASCADE.md) | Full ordered delete-cascade spec and the ID-type e2e gap this document's §5 also surfaces independently |
| [`calendar/ARCHITECTURE.md`](calendar/ARCHITECTURE.md) §6 | How the 7 collections tabulated in §5 fit into calendar-be's module/sync architecture |
| [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §9 | Backup/restore tooling, volumes, persistence per environment |
| [ENV-VARS.md](ENV-VARS.md) | `MONGODB_URI`/`MONGO_ROOT_PASSWORD` sync constraint, `GOOGLE_TOKEN_ENCRYPTION_KEY` format |
| [`troubleshooting/mfe-not-loading.md`](troubleshooting/mfe-not-loading.md) | Cites this document's §2.4 finding that the `registry` table is always empty — the build-time env var, not this table, is what actually resolves the MFE remote URL |
| [ADR-003](adr/003-mongodb-for-calendar.md) | Why calendar-be uses MongoDB while the other two backends use PostgreSQL |
| [`runbooks/db-backup-restore.md`](runbooks/db-backup-restore.md) | Backup/restore procedures for every database this document schemas out |
| [`calendar/DATA-MODEL.md`](calendar/DATA-MODEL.md) | Calendar-scoped index into this document's §5-§6, plus an operational summary of what each collection is for |

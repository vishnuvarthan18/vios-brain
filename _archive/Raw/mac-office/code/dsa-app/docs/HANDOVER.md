# DreamSpace Academy — Handover

Operational handover for the Phase 1 Makerspace platform. Read `CLAUDE.md` first
for the architecture and non-negotiable rules; this doc is the run/deploy/ops
companion.

---

## 1. What's built (Phase 1, Sprints 1–10)

| Sprint | Scope |
|---|---|
| 1 | Foundation — auth, users, roles, hubs, **central permission matrix** + per-user overrides, INSERT-only audit log |
| 2 | Students — enrollment, `DSA-XXXXXX` IDs, slot assignment, demographics, PDPA consent |
| 3 | Sessions & timetable — recurring slots, session generation, holidays, cover trainer |
| 4 | Attendance & outcomes — mark, lock-after-submit, coordinator override, proof of work |
| 5 | Payments & receipts — monthly cron, `RCP-XXXXXX`, waive/reverse, sponsorship |
| 6a | Day-to-day class work log (feeds scoring + portfolios) |
| 6b | Portfolios & certificates — approval flow, auto-flag on final-level completion, `DSA-CERT-XXXXXX` QR verify |
| 7 | Notifications (in-app + SMS + push + prefs) + announcements; **installable PWA** |
| 8 | Open sessions — categories, tags, tiered access/pricing, seat-limited registration + waitlist |
| 9 | Reporting & admin — impact reports, audit-log viewer, permission-override UI |
| 10 | Polish — error boundary, toasts, offline banner, CSV exports, first-time flow, this doc |

The **web app is the superset** (every role, every feature) and is an installable
PWA. The mobile (Capacitor) app is a later slice of the same codebase.

---

## 2. Repository layout

```
api/    Express + Prisma backend         (deploy: Dokploy)   -> appapi.dreamspace.academy
web/    React + Vite PWA (all roles)      (deploy: Vercel)    -> app.dreamspace.academy
public/ Next.js no-auth pages            (deploy: Vercel)    -> verify./showcase.dreamspace.academy
mobile/ Capacitor wrapper (later)
```

`dreamspace.academy` (root) is an **existing marketing site — not part of this repo.**

---

## 3. Running locally

Prereqs: Node 20+, pnpm.

```bash
pnpm install

# API
cd api
cp .env.example .env            # AUTH_MODE=mock works with no external services
pnpm prisma generate
pnpm db:push                    # or prisma migrate dev
# then run prisma/sql one-time setup (sequences + audit hardening) — see §5
pnpm db:seed
pnpm dev                        # http://localhost:4000

# Web
cd ../web
cp .env.example .env            # VITE_API_BASE_URL defaults to localhost:4000
pnpm dev                        # http://localhost:5173
```

**Local-first by design:** with `AUTH_MODE=mock` the API trusts an `x-dev-user`
header (a seeded user's email). MongoDB, Supabase Realtime, FCM and notify.lk are
all optional — each degrades to an in-memory/no-op/log fallback when its env vars
are unset, so the whole stack runs offline.

Seeded users (mock login): `superadmin@`, `msadmin@`, `coordinator.hatton@`,
`trainer.hatton@`, `parent@`, `member@` `dreamspace.tech`.

---

## 4. Tests

The API ships an end-to-end smoke harness that spins up a **real Postgres**
(embedded-postgres, UTF-8 — matches Supabase, handles Tamil) and drives the API
through supertest — no Docker needed.

```bash
cd api
pnpm exec tsx scripts/smoke.ts   # 106 checks across all sprints; exits non-zero on any fail
pnpm test                        # unit tests (permission resolver)
pnpm typecheck
cd ../web && pnpm build          # tsc + vite + PWA generation
```

CI gate = `typecheck` + `smoke` + web `build` all green.

---

## 5. One-time database setup (after first migrate)

Run against the production DB once (see `api/prisma/sql/` / `docs/sequences.sql`):

```sql
CREATE SEQUENCE IF NOT EXISTS student_id_seq START 47312;
CREATE SEQUENCE IF NOT EXISTS receipt_id_seq START 47312;
CREATE SEQUENCE IF NOT EXISTS cert_id_seq    START 47312;
-- audit log is append-only:
REVOKE UPDATE, DELETE ON audit_logs FROM <app_db_user>;
```

---

## 6. Environment & integrations

All config is validated in `api/src/config/env.ts` (the process refuses to boot
on bad config). See `api/.env.example` / `web/.env.example` for the full list.

| Integration | Env | Fallback when unset |
|---|---|---|
| PostgreSQL (Supabase) | `DATABASE_URL` | required |
| Auth0 (staff) | `AUTH_MODE=auth0` + `AUTH0_*` | `AUTH_MODE=mock` (dev only) |
| MongoDB (rich bodies) | `MONGODB_URI`, `MONGODB_DB` | in-memory store (non-persistent) |
| SMS (notify.lk) | `NOTIFYLK_*` | logged, not sent |
| Push (FCM) | `FCM_*` | logged, not sent |
| Realtime (Supabase) | `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` | web polling delivers in-app |
| Verify / showcase URLs | `PUBLIC_VERIFY_BASE_URL`, `PUBLIC_SHOWCASE_BASE_URL` | dreamspace subdomains |

Production CORS: set `WEB_ORIGIN=https://app.dreamspace.academy`.

---

## 7. Cron jobs (`api/src/jobs/`, gated by `ENABLE_CRONS=true`)

| Job | Schedule (Asia/Colombo) | Idempotent |
|---|---|---|
| `sessionInstances` | Mon 06:00 | yes |
| `monthlyPayments` | 1st 00:01 | yes (`@@unique([studentId, month])`) |

(`expireRoles`, `pushFailedRetry` are specced in CLAUDE.md for when those paths
go live.)

---

## 8. Operating notes / gotchas

- **Privacy:** students appear by `DSA-XXXXXX` only across operational UI. Names
  are PII (coordinator/admin, consent-gated); demographics are admin-only and the
  reason they exist is impact reporting (`/reports/impact`). No student photos.
- **Permissions:** never add inline role checks — add a permission string to
  `api/src/permissions.ts` and gate routes with `can()`. DENY > GRANT > baseline.
- **Audit log** can't be edited/deleted at the DB. Sensitive actions must call
  `logAction`.
- **Soft delete everywhere** (`deletedAt`/`deactivatedAt`/`revokedAt`/status), no
  hard deletes.
- **SMS/push are non-fatal** — failures are recorded as channel-status flags,
  never thrown.
- The **rich-text body store** (`api/src/lib/mongo.ts`) and **notify service**
  (`api/src/services/notify.ts`) are the reusable seams for future content +
  messaging features.

---

## 9. Known follow-ups (not Phase 1 blockers)

- Real Auth0 + Auth0 → user linking on the web (currently mock `x-dev-user`).
- Real Supabase Realtime subscription on the web (polling works today).
- Real FCM HTTP v1 send (stubbed).
- Parent-facing web pages (child progress) — notifications already reach parents.
- The `public/` Next.js verify + showcase pages (API endpoints exist:
  `/api/v1/public/certificates/:code/verify`).
- Certificate/receipt PDF generation (records + verify codes exist).
- Mobile (Capacitor) slice + offline attendance queue (Capacitor Preferences).

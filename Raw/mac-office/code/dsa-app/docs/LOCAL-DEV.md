# Local Development Guide

How to run DreamSpace Academy locally with mock authentication. No Auth0, no FCM, no SMS required — everything degrades gracefully to in-memory or no-op fallbacks.

---

## Prerequisites

| Tool | Version |
|---|---|
| Node.js | 20 or later |
| pnpm | 10 or later (`npm i -g pnpm`) |
| PostgreSQL | via Supabase (see §2) |

---

## 1. Install dependencies

```bash
git clone <repo>
cd dsa-app
pnpm install
```

---

## 2. Configure the API

```bash
cd api
cp .env.example .env
```

Open `api/.env` and set the two required values:

```env
DATABASE_URL=postgresql://(secret removed)@db.<project>.supabase.co:5432/postgres
AUTH_MODE=mock
```

Everything else in `.env.example` is optional for local dev — leave the defaults.

**Push the Prisma schema and seed the database (first time only):**

```bash
pnpm prisma db push   # applies schema to Supabase
pnpm db:seed          # creates hubs + seeded users
```

**One-time SQL hardening** (run once in Supabase SQL Editor):

```sql
-- From api/prisma/sql/sequences.sql
CREATE SEQUENCE IF NOT EXISTS student_id_seq START 47312;
CREATE SEQUENCE IF NOT EXISTS receipt_id_seq START 47312;
CREATE SEQUENCE IF NOT EXISTS cert_id_seq    START 47312;

-- From api/prisma/sql/hardening.sql
REVOKE UPDATE, DELETE ON audit_logs FROM PUBLIC;
ALTER TABLE _prisma_migrations ENABLE ROW LEVEL SECURITY;
```

---

## 3. Configure the web app

```bash
cd web
cp .env.example .env
```

The defaults in `.env.example` work as-is for local dev:

```env
VITE_API_BASE_URL=http://localhost:4000/api/v1
```

---

## 4. Start the servers

Open three terminals:

```bash
# Terminal 1 — API (port 4000)
cd api && pnpm dev          # cloud Supabase DB (slow from Sri Lanka — see below)
# …or, much faster for everyday dev:
cd api && pnpm dev:local    # embedded LOCAL Postgres, ~1ms queries

# Terminal 2 — Web app (port 5173)
cd web && pnpm dev

# Terminal 3 — Mobile app preview (port 5174)
cd web && pnpm dev:mobile
```

| URL | What it is |
|---|---|
| http://localhost:4000 | Express API |
| http://localhost:5173 | Web app (all roles, PWA, desktop + responsive mobile view) |
| http://localhost:5174 | Mobile app (trainer + parent routes only, Capacitor shell) |

### `pnpm dev:local` — fast local database

The cloud Supabase DB is in `ap-south-1` (Mumbai); from Sri Lanka every
query is a ~0.5–1s round-trip, so requests take 3–10s and long transactions
can time out. `pnpm dev:local` (in `api/`) avoids this entirely:

- Starts an **embedded local Postgres** on port 5433 (no Docker; data persists
  in `api/.localdb/`, gitignored — delete that folder to reset).
- First run: pushes the Prisma schema, creates the global ID sequences, and
  seeds the §6 test users automatically.
- Then runs the normal `tsx watch` API with `DATABASE_URL` pointed at it.
  All other `.env` values (Supabase auth, JWT secret, …) are unchanged.

Notes:
- Parent OTP login works fully offline here — the seeded parent
  (`parent@dreamspace.tech`, phone `94770000001`) plus the `devCode` in the
  OTP response need no SMS or Supabase.
- Staff Supabase logins resolve by **email fallback**: a Supabase-auth login
  works if a local user row with the same email exists (seeded or created).
- Your cloud data (students enrolled against Supabase) does NOT carry over —
  this is a separate, disposable database.

---

## 5. How mock auth works

When `AUTH_MODE=mock` the API **trusts the `x-dev-user` request header** instead of verifying a JWT. The header value can be a seeded user's **email address** or **user ID**.

The web login page at `/login` sets this header automatically when you click a seeded account. The mobile login page at `/login` does the same for staff accounts.

**To impersonate a user from the command line:**

```bash
curl http://localhost:4000/api/v1/auth/me \
  -H "x-dev-user: superadmin@dreamspace.tech"
```

**To switch users in the browser:** click "Sign out" and pick a different account on the login page. The header is stored in a browser cookie (`dsa_dev_user`).

> `AUTH_MODE=mock` must **never** be used in production. The API refuses to boot with this mode if `NODE_ENV=production`.

---

## 6. Seeded users

These accounts are created by `pnpm db:seed`. Each has `termsAccepted = false` on first use — the Terms & Conditions screen will appear once before the dashboard.

| Email | Role | Hub | Key permissions |
|---|---|---|---|
| `superadmin@dreamspace.tech` | SUPER_ADMIN | — (all hubs) | Full access, permission overrides |
| `msadmin@dreamspace.tech` | MS_ADMIN | — (all hubs) | All ops, no permission overrides |
| `coordinator.hatton@dreamspace.tech` | HUB_COORDINATOR | Hatton | Student enrollment, attendance override, payments |
| `trainer.hatton@dreamspace.tech` | TRAINER | Hatton | Mark attendance, log classwork, view students |
| `trainer.batti@dreamspace.tech` | TRAINER | Batticaloa | Same as above, different hub |
| `parent@dreamspace.tech` | PARENT | — | View own child's progress, notifications |
| `volunteer@dreamspace.tech` | VOLUNTEER | — | Limited read-only |
| `member@dreamspace.tech` | MEMBER | — | Minimal access |

All email addresses end in `@dreamspace.tech` (not `.academy`) to keep them clearly separate from real staff accounts.

---

## 7. Parent OTP in local dev

Parent login uses phone OTP (SMS via notify.lk in production). Locally, SMS is not sent — the verification code is returned directly in the API response.

**Flow:**

1. On the mobile login page, switch to the **Parent / Guardian** tab.
2. Enter any phone number registered to a parent account (see seed data).
3. Click **Send verification code**.
4. The response includes a `devCode` field — this is shown inline on the page in dev mode.
5. Enter the code and verify.

**Via curl:**

```bash
# Step 1 — request OTP
curl -X POST http://localhost:4000/api/v1/auth/otp/send \
  -H "Content-Type: application/json" \
  -d '{"phone": "+94770000001"}'
# Response includes: { "devCode": "123456" }

# Step 2 — verify
curl -X POST http://localhost:4000/api/v1/auth/otp/verify \
  -H "Content-Type: application/json" \
  -d '{"phone": "+94770000001", "code": "123456"}'
# Response includes: { "token": "<jwt>" }
```

Use the returned JWT as a `Bearer` token for subsequent requests, or store it in `localStorage` as `dsa_parent_token`.

---

## 8. Optional services (all have fallbacks)

| Service | Env var(s) | Local fallback |
|---|---|---|
| MongoDB (portfolio bodies) | `MONGODB_URI` | In-memory store (non-persistent, resets on restart) |
| Supabase Realtime | `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` | Web app polls the API instead |
| Firebase Cloud Messaging | `FCM_PROJECT_ID`, `FCM_CLIENT_EMAIL`, `FCM_PRIVATE_KEY` | Push logged, not sent |
| notify.lk SMS | `NOTIFYLK_USER_ID`, `NOTIFYLK_API_KEY` | SMS logged, OTP code returned in response body |
| Supabase Storage | `SUPABASE_URL`, `SUPABASE_STORAGE_BUCKET` | File upload errors gracefully |

None of these are required to run the platform locally. Only `DATABASE_URL` is mandatory.

---

## 9. Mobile dev server

The mobile app (`web/src/mobile-main.tsx`) is a separate Vite build that loads only trainer and parent routes. It runs on port 5174 so Capacitor can use it for live reload.

```bash
cd web && pnpm dev:mobile   # http://localhost:5174
```

Open it in Chrome DevTools with a mobile viewport (Ctrl+Shift+M) to see the full mobile experience: top bar, offline attendance queue, bottom navigation.

**For Capacitor native (Android/iOS):**

```bash
# Build the mobile bundle first
cd web && pnpm build:mobile        # outputs to web/mobile-dist/

# Add native platform (first time only)
cd mobile && npx cap add android   # or cap add ios

# Sync web assets to native project
cd mobile && npx cap sync

# Run on device/emulator with live reload
cd mobile && npx cap run android --livereload --external
# The device connects to http://<your-local-ip>:5174 — make sure it's on the same network
```

---

## 10. Running checks

```bash
# API typecheck
cd api && pnpm typecheck

# API unit tests (permission resolver)
cd api && pnpm test

# API end-to-end smoke suite (spins up embedded Postgres, no Docker needed)
cd api && pnpm exec tsx scripts/smoke.ts

# Web typecheck (covers both web and mobile builds)
cd web && pnpm typecheck

# Web production build (includes PWA generation)
cd web && pnpm build

# Mobile production build
cd web && pnpm build:mobile
```

CI gate = `api typecheck` + `api smoke` + `web typecheck` + `web build` all green.

---

## 11. Common issues

**`ERR_CONNECTION_REFUSED` on API requests**
The API server isn't running. Start it with `cd api && pnpm dev`.

**`Can't reach database server`**
The Supabase database is paused (free tier auto-pauses after inactivity). Go to your Supabase dashboard and click **Resume project**. The API reconnects automatically.

**Terms & Conditions appears every time**
The seeded user has `termsAcceptedAt = null`. Accept terms once per account. If it keeps reappearing, check that `POST /api/v1/auth/terms/accept` is succeeding (look at the API terminal output).

**Mobile app shows `virtual:pwa-register` error**
Make sure you're using `pnpm dev:mobile` (port 5174), not `pnpm dev` (port 5173). The mobile Vite config stubs out the PWA plugin.

**CORS error from port 5174**
Ensure `DEV_MOBILE_ORIGIN=http://localhost:5174` is set in `api/.env` and the API has been restarted.

**Offline queue not syncing**
Open DevTools → Application → Local Storage and look for the key `dsa_offline_attendance_queue`. If records are stuck, check that the API is reachable and the session hasn't been locked by a coordinator.

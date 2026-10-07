# Deployment Guide — Dokploy (Self-Hosted)

DreamSpace Academy runs three applications on a single Dokploy VPS, each behind Dokploy's built-in Traefik reverse proxy with automatic Let's Encrypt SSL.

---

## Infrastructure overview

| Service | Hosting | URL |
|---|---|---|
| API (Express + Prisma) | Dokploy VPS | `appapi.dreamspace.academy` |
| Web PWA (React + Vite) | Dokploy VPS | `app.dreamspace.academy` |
| Public (Next.js) | Dokploy VPS | `verify.dreamspace.academy`, `showcase.dreamspace.academy` |
| Database | Supabase (PostgreSQL) | external |
| Rich text bodies | MongoDB Atlas | external |
| Auth (staff) | Supabase Auth | external |
| SMS | notify.lk | external |
| Push | Firebase Cloud Messaging | external |
| File storage | Supabase Storage | external |

> **Note:** `dreamspace.academy` (root) is the existing marketing website — the platform does NOT deploy there.

---

## Docker files

| File | What it builds |
|---|---|
| `api/Dockerfile` | Multi-stage: installs deps → generates Prisma client → compiles TS → `pnpm deploy` (prod-only deps) → slim Node runner |
| `web/Dockerfile` | Multi-stage: Vite build (with `ARG` env vars baked in) → Nginx static server |
| `web/nginx.conf` | SPA routing (`try_files` → `index.html`) + correct cache headers for PWA service worker |
| `public/Dockerfile` | Multi-stage: Next.js standalone build → slim Node runner |
| `.dockerignore` | Excludes `node_modules`, `.env`, build outputs, `mobile/`, dev tooling from Docker build context |

All three Dockerfiles use the **repo root as build context** (they do `COPY . .`). The workspace `pnpm-lock.yaml` and `pnpm-workspace.yaml` must be accessible to all builds.

---

## One-time server setup

### 1. Install Dokploy

SSH into the VPS as root and run:

```bash
curl -sSL https://dokploy.com/install.sh | sh
```

Dokploy UI will be available at `http://<server-ip>:3000`. Set up an admin account on first visit.

### 2. Connect GitHub

In Dokploy → **Settings → Providers** → add a GitHub App or personal access token pointing to the `dreamspace` repo.

---

## Creating the three applications

All three applications share these settings:
- **Provider:** GitHub
- **Repository:** `dreamspace/platform` (or your org/repo name)
- **Branch:** `main`
- **Build type:** Dockerfile
- **Build context:** `.` (repo root — required for monorepo)

### Application 1 — `dsa-api`

| Setting | Value |
|---|---|
| Dockerfile path | `api/Dockerfile` |
| Published port | `4000` |
| Domain | `appapi.dreamspace.academy` |
| HTTPS | Enabled (Let's Encrypt) |

**Environment variables:**

```env
NODE_ENV=production
PORT=4000

# Auth — Supabase Auth for staff, custom JWT for parents
# Tokens verified via JWKS (ES256) — no JWT secret needed
AUTH_MODE=supabase

DATABASE_URL=postgresql://(secret removed)@aws-0-<region>.pooler.supabase.com:6543/postgres
# CORS allowlist. Must EXACTLY match the browser Origin header: scheme + host,
# NO trailing slash, NO path. e.g. https://app.dreamspace.academy
# (not .../ and not .../api/v1). A mismatch CORS-blocks every web request.
WEB_ORIGIN=https://app.dreamspace.academy
MOBILE_ORIGIN=capacitor://localhost

# Parent OTP custom tokens (notify.lk SMS flow — unchanged)
JWT_SECRET=<generate: openssl rand -base64 48>
JWT_EXPIRES_IN=7d

# Supabase — DB + storage + admin REST (staff invite / ban)
SUPABASE_URL=https://<ref>.supabase.co
SUPABASE_ANON_KEY=<from Supabase dashboard>
SUPABASE_SERVICE_ROLE_KEY=<from Supabase dashboard>
SUPABASE_STORAGE_BUCKET=dreamspace

MONGODB_URI=mongodb+srv://(secret removed)@<cluster>.mongodb.net/dreamspace
MONGODB_DB=dreamspace

NOTIFYLK_USER_ID=<from notify.lk>
NOTIFYLK_API_KEY=<from notify.lk>
NOTIFYLK_SENDER_ID=DreamSpace

# Firebase — FCM push notifications ONLY (Auth is now Supabase).
# All three required for push to work; if unset, push is silently skipped.
FCM_PROJECT_ID=<Firebase project id>
FCM_CLIENT_EMAIL=<service account email>
FCM_PRIVATE_KEY=<service account private key — include literal \n for newlines>

RESEND_API_KEY=<from resend.com>
RESEND_FROM=DreamSpace Academy <no-reply@app.dreamspace.academy>
ENABLE_CRONS=true
TZ=Asia/Colombo
PUBLIC_VERIFY_BASE_URL=https://verify.dreamspace.academy
PUBLIC_SHOWCASE_BASE_URL=https://showcase.dreamspace.academy
RATE_LIMIT_GLOBAL_PER_MINUTE=100
```

On container start, the API automatically runs `prisma migrate deploy` before booting the server (hardcoded in the Dockerfile `CMD`).

---

### Application 2 — `dsa-web`

| Setting | Value |
|---|---|
| Dockerfile path | `web/Dockerfile` |
| Published port | `80` |
| Domain | `app.dreamspace.academy` |
| HTTPS | Enabled (Let's Encrypt) |

**Build arguments** (set in Dokploy's Build Args section, NOT env vars — Vite bakes these into the JS bundle at build time):

```
VITE_API_BASE_URL=https://appapi.dreamspace.academy/api/v1
VITE_SUPABASE_URL=https://<ref>.supabase.co
VITE_SUPABASE_ANON_KEY=<anon key from Supabase Dashboard → Project Settings → API>
```

> If you change a `VITE_*` value you must **redeploy** (rebuild) — the old value is compiled into the bundle.

---

### Application 3 — `dsa-public`

| Setting | Value |
|---|---|
| Dockerfile path | `public/Dockerfile` |
| Published port | `3000` |
| Domains | `verify.dreamspace.academy` AND `showcase.dreamspace.academy` |
| HTTPS | Enabled (Let's Encrypt) |

No runtime env vars are required at this stage. Add them here if the public app later needs API keys or feature flags.

---

## One-time database setup (after first deploy)

After the API's first successful deployment, run these SQL commands once against your Supabase project. You can use the Supabase SQL Editor or connect via `psql`.

```sql
-- Global ID sequences (opaque, reveals no hub/year/count information)
CREATE SEQUENCE IF NOT EXISTS student_id_seq START 47312;
CREATE SEQUENCE IF NOT EXISTS receipt_id_seq START 47312;
CREATE SEQUENCE IF NOT EXISTS cert_id_seq    START 47312;

-- Audit log is INSERT-only at the database level (replace with your actual DB user)
REVOKE DELETE, UPDATE ON audit_logs FROM postgres;
```

See `api/prisma/sql/hardening.sql` for the canonical copy of these statements.

---

## Auto-deploy on push

To trigger automatic redeployment when you push to `main`:

1. In each Dokploy application → **Deployments** tab → copy the **Webhook URL**
2. In GitHub → repo **Settings → Webhooks → Add webhook**
   - Payload URL: paste the Dokploy webhook URL
   - Content type: `application/json`
   - Events: **Just the push event**
3. Repeat for all three applications

After this, every push to `main` triggers a fresh Docker build and rolling restart for that application.

---

## Deploying a change

### Code change (no env var changes)

```bash
git push origin main   # auto-deploy fires if webhooks are set up
```

Or manually in Dokploy UI: open the application → **Deployments** → **Deploy**.

### Environment variable change (API or public)

1. Update the value in Dokploy → **Environment** tab
2. Click **Redeploy** — the container restarts with the new value (no rebuild needed)

### Build argument change (web app — VITE_*)

Applies to: `VITE_API_BASE_URL`, `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`

1. Update the value in Dokploy → **Build Args** tab
2. Click **Deploy** — a full rebuild is required to bake the new value into the bundle

### Schema change (new Prisma migration)

1. Create and commit the migration locally:
   ```bash
   cd api && pnpm prisma migrate dev --name describe_change
   git add api/prisma/migrations && git commit ...
   ```
2. Push to `main` — the API container will run `prisma migrate deploy` automatically on next start

---

## Rollback

In Dokploy UI → open the application → **Deployments** tab → find the last known-good deployment → **Redeploy that version**.

---

## Troubleshooting

### API won't start

Check Dokploy container logs. Common causes:
- Missing `DATABASE_URL` — the env validator (`api/src/config/env.ts`) exits with a clear message listing missing/invalid vars
- `AUTH_MODE=supabase` without `SUPABASE_URL` set — tokens are verified via the Supabase JWKS endpoint (derived from `SUPABASE_URL`), so the API refuses to boot without it. There is no `SUPABASE_JWT_SECRET`.
- `AUTH_MODE` left unset — it defaults to `mock`, which trusts the `x-dev-user` header (full auth bypass). Production **must** set `AUTH_MODE=supabase`.
- Prisma migration failed — check for schema conflicts with existing DB state

### Web app loads but every API call fails (CORS error in browser console)

The web app and API are separate services that must agree on origins:
- API env `WEB_ORIGIN` must EXACTLY match the web app's browser origin — scheme + host, **no trailing slash, no path**. Use `https://app.dreamspace.academy`, never `https://app.dreamspace.academy/` or `.../api/v1`. The allowlist does an exact string match against the request `Origin` header, so a single extra `/` blocks everything.
- Web build arg `VITE_API_BASE_URL` must point at the API including the `/api/v1` suffix: `https://appapi.dreamspace.academy/api/v1`.

After changing `WEB_ORIGIN`, **redeploy the API** (env-var change, restart only). After changing `VITE_API_BASE_URL`, **rebuild the web app** (build arg, full deploy).

The web client authenticates with `Authorization: Bearer` tokens, not cookies — so this is purely an origin-allowlist problem, not a credentials one.

### Web app shows blank page or 404 on refresh

Nginx `try_files` config handles React Router. If this happens, confirm `web/nginx.conf` is being copied correctly in the Docker build (check build logs for the `COPY web/nginx.conf` step).

### `VITE_*` variable has wrong value in production

These are baked in at build time. Update the Build Arg in Dokploy and trigger a full redeploy (not just a restart).

### Prisma migrate deploy fails on startup

The migration may conflict with manual DB changes. Connect to Supabase directly and resolve the drift, then redeploy.

---

## Monitoring

Dokploy provides container logs per application. For deeper observability:
- **API errors:** check Dokploy logs for the `dsa-api` container
- **DB performance:** Supabase dashboard → Database → Query Performance
- **Cron jobs:** `ENABLE_CRONS=true` must be set on the API; job output appears in the API container logs

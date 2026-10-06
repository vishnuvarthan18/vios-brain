# Deployment — Environment Variables per App

Four deployable pieces. Each has a different env location and host.
**Secrets you already have** live in your local `api/.env` — copy them over; don't regenerate.

| App | Folder | Env location | Host | Public URL |
|---|---|---|---|---|
| API (Express) | `api/` | host env vars (runtime) | Railway / Dokploy | `appapi.dreamspace.academy` |
| Web PWA (Vite) | `web/` | build args (`VITE_*`, build-time) | Vercel / Dokploy | `app.dreamspace.academy` |
| Public (Next.js) | `public/` | build args | Vercel | `verify.` / `showcase.dreamspace.academy` |
| Mobile APK | builds from `web/` | `web/.env.mobile` (build-time) | local build → APK | n/a |

> **The one change that makes everything work for real:** once the API is live at
> `https://appapi.dreamspace.academy`, set that URL as `VITE_API_BASE_URL` in BOTH
> `web/.env.production` (PWA) and `web/.env.mobile` (APK). That removes the cloudflared
> tunnel dependency and makes invite links + push reachable from any device.

---

## 1. API — set as runtime env vars in Railway/Dokploy (NOT a committed file)

Template is `api/.env.example`. Production values:

```dotenv
NODE_ENV=production
PORT=4000
WEB_ORIGIN=https://app.dreamspace.academy
MOBILE_ORIGIN=capacitor://localhost
PUBLIC_VERIFY_BASE_URL=https://verify.dreamspace.academy
PUBLIC_SHOWCASE_BASE_URL=https://showcase.dreamspace.academy
AUTH_MODE=supabase
ENABLE_CRONS=true
TZ=Asia/Colombo

# Database (Supabase Postgres) — copy from your local api/.env
DATABASE_URL=postgresql://(secret removed)@aws-0-<region>.pooler.supabase.com:6543/postgres?pgbouncer=true&connection_limit=1
DIRECT_URL=postgresql://(secret removed)@aws-0-<region>.pooler.supabase.com:5432/postgres

# Supabase — copy from local api/.env
SUPABASE_URL=https://<ref>.supabase.co
SUPABASE_ANON_KEY=<anon key>
SUPABASE_SERVICE_ROLE_KEY=<service_role secret>   # SECRET
SUPABASE_STORAGE_BUCKET=dreamspace

# Mongo (optional — portfolio/bio rich text)
MONGODB_URI=<copy from local>
MONGODB_DB=dreamspace

# Parent OTP JWT — copy from local (or regenerate: openssl rand -base64 48)
JWT_SECRET=<long-random-secret>                    # SECRET
JWT_EXPIRES_IN=7d

# SMS (notify.lk)
NOTIFYLK_USER_ID=<from local>
NOTIFYLK_API_KEY=<from local>                       # SECRET
NOTIFYLK_SENDER_ID=DreamSpace

# Push (FCM) — copy all three from local api/.env exactly (keep the \n in the key)
FCM_PROJECT_ID=dreamspace-academy
FCM_CLIENT_EMAIL=<service-account>@dreamspace-academy.iam.gserviceaccount.com
FCM_PRIVATE_KEY="(secret removed)"   # SECRET

# Email (Resend)
RESEND_API_KEY=re_<key>                             # SECRET
RESEND_FROM=DreamSpace Academy <no-reply@app.dreamspace.academy>

# Staff platform (if integrated)
STAFF_PLATFORM_API_URL=<...>
STAFF_PLATFORM_API_KEY=<...>                         # SECRET
```

Only **`DATABASE_URL`** is strictly required by the schema validator; everything else has a
default but you want the real values in production. After first deploy, run the one-time SQL
(sequences, audit-log REVOKE) and `prisma migrate deploy` + `db:seed` — see `CLAUDE.md`.

---

## 2. Web PWA — build-time `VITE_*` (Vercel build args, or `web/.env.production`)

File already in repo: `web/.env.production` (used by `pnpm build`).

```dotenv
VITE_API_BASE_URL=https://appapi.dreamspace.academy/api/v1
VITE_SUPABASE_URL=https://<ref>.supabase.co
VITE_SUPABASE_ANON_KEY=<anon public key>
# Do NOT set VITE_AUTH_MODE=mock in production.
```

These are baked at build time — change one ⇒ rebuild/redeploy. The anon key is safe to expose;
the service-role key must **never** appear in any `VITE_*` var.

---

## 3. Public (Next.js — cert verify + showcase)

**Required build arg:** `NEXT_PUBLIC_API_BASE_URL=https://appapi.dreamspace.academy/api/v1`

The open-sessions pages and the public register form call the API, so this MUST be set or the
code falls back to `http://localhost:4000/api/v1` and every fetch/register fails in production.
Because `NEXT_PUBLIC_*` is inlined at **build time** (the register form is a client component),
it must be passed as a **build arg**, not a runtime env var:
- Docker / Dokploy: `--build-arg NEXT_PUBLIC_API_BASE_URL=https://appapi.dreamspace.academy/api/v1`
- Vercel: set it as a Production build env var.

The API must also allow the showcase origin in CORS — `PUBLIC_SHOWCASE_BASE_URL` (defaults to
`https://showcase.dreamspace.academy`).

---

## 4. Mobile APK — `web/.env.mobile` (build-time, loaded by `pnpm build:mobile --mode mobile`)

For a real release, point it at the deployed API (replaces the cloudflared tunnel):

```dotenv
VITE_API_BASE_URL=https://appapi.dreamspace.academy/api/v1
VITE_APP_BASE_URL=https://app.dreamspace.academy
```

Supabase vars are inherited from `web/.env` (loaded in every mode). Then:
`cd web && pnpm build:mobile` → `cd ../mobile && npx cap sync android` → `./gradlew assembleRelease`
(sign with a keystore for the Play Store). Remove `android:usesCleartextTraffic` once the API is HTTPS.

---

## Supabase dashboard settings (not env, but required for prod)

- **Authentication → URL Configuration:** Site URL `https://app.dreamspace.academy`; add redirect
  URLs `https://app.dreamspace.academy/auth/callback` (and keep `http://localhost:5173/auth/callback`
  for dev). This is what makes invite / set-password links work for remote users.
- **SMTP:** configure (Resend) so invite/recovery emails actually send.

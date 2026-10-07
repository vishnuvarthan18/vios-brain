# Go-Live Checklist — New Changes + Mumbai Database

A focused, do-this-in-order checklist for deploying the current changes to
production on the **new Mumbai Supabase project**. For the full env-var
reference see [`DEPLOYMENT-ENV.md`](./DEPLOYMENT-ENV.md); for infra/Docker see
[`DEPLOYMENT.md`](./DEPLOYMENT.md); for the bug-class checklist see
[`KNOWN-ISSUES-CHECKLIST.md`](./KNOWN-ISSUES-CHECKLIST.md).

> The project uses **`prisma db push`** (no migrations folder). The schema is
> applied by pushing, not by `migrate deploy`. `db push` can drop columns/data
> if the schema diverges — review the diff before pushing to a DB with real data.

---

## 0. What changed in this release (verify after deploy)

- **Payments self-heal**: the payments list & report now backfill the *current*
  month's records for any active student missing one (e.g. enrolled after the
  monthly cron ran). Implemented in `api/src/jobs/monthlyPayments.ts`
  (`ensurePaymentsForMonth`) and called from `api/src/routes/payments.ts`.
  → Verify: a student enrolled mid-month appears on `/payments` (current month)
  as UNPAID without any manual "Generate".
- **Server-side filtering**: Payments page has a status dropdown
  (Paid/Unpaid/Sponsored/…) that filters on the server; Students "No level"
  filter is now a server query param (`noLevel=true`). No more fetch-all-then-
  filter for these.
- **Auth hardening**: the API now refuses to boot when
  `NODE_ENV=production` + `AUTH_MODE=mock` (mock trusts the unverified
  `x-dev-user` header). See `api/src/config/env.ts`.

---

## 1. Pre-deploy (local, before pushing the branch)

- [ ] `cd api  && pnpm exec tsc --noEmit` → clean
- [ ] `cd web  && pnpm exec tsc --noEmit` → clean
- [ ] `cd public && pnpm exec tsc --noEmit` → clean
- [ ] Follow [`docs/git-workflow.md`](./git-workflow.md): branch, Conventional
      Commit, human verification before PR.

---

## 2. New Mumbai database — one-time setup (run ONCE)

Run against the **Mumbai** project (`fuqdoumkhyopdkcdsyfd`).

1. [ ] **Push schema** (from `api/`, with Mumbai `DATABASE_URL`/`DIRECT_URL`):
   ```bash
   pnpm prisma db push
   ```
2. [ ] **Global sequences** — `api/prisma/sql/sequences.sql`:
   ```bash
   psql "$DIRECT_URL" -f api/prisma/sql/sequences.sql
   ```
   (Creates `student_id_seq`, `receipt_id_seq`, `cert_id_seq` @ 47312.)
3. [ ] **Audit-log hardening** — `api/prisma/sql/hardening.sql`. The app must
   connect as a **non-owner role** for the REVOKE to bite (the table owner
   bypasses grants). Pass the app role:
   ```bash
   psql "$DIRECT_URL" -v app_role=<your_app_db_role> -f api/prisma/sql/hardening.sql
   ```
4. [ ] **Seed** roles, hubs (Hatton, Batticaloa), sections, levels:
   ```bash
   pnpm db:seed
   ```
5. [ ] **Storage bucket** exists in Mumbai (name must match
   `SUPABASE_STORAGE_BUCKET`, default `dreamspace`).

---

## 3. Supabase dashboard (Mumbai project)

- [ ] **Auth → URL Configuration**
  - Site URL: `https://app.dreamspace.academy`
  - Redirect URLs: `https://app.dreamspace.academy/auth/callback`
    (+ `http://localhost:5173/auth/callback` for dev)
- [ ] **Auth → SMTP**: custom SMTP via Resend, `From: no-reply@app.dreamspace.academy`
  (must match the API's `RESEND_FROM`).
- [ ] Create the staff accounts (or invite) — superadmin is bootstrap-only via
  seed/DB, never via API.

---

## 4. API env (Railway / Dokploy) — critical values

Full list in [`DEPLOYMENT-ENV.md`](./DEPLOYMENT-ENV.md). **Must be correct or the
app won't work:**

- [ ] `NODE_ENV=production`
- [ ] `AUTH_MODE=supabase`  ← **never `mock`**; the API now hard-exits otherwise.
- [ ] `DATABASE_URL` = Mumbai **pooled** (port 6543, `?pgbouncer=true`)
- [ ] `DIRECT_URL` = Mumbai **direct/session** (port 5432) — used by `db push`
- [ ] `SUPABASE_URL` = `https://fuqdoumkhyopdkcdsyfd.supabase.co`
      ← **must match the web `VITE_SUPABASE_URL`** or staff JWTs fail JWKS check.
- [ ] `SUPABASE_SERVICE_ROLE_KEY` = Mumbai service-role key (secret)
- [ ] `WEB_ORIGIN=https://app.dreamspace.academy` (exact, no trailing slash) — CORS
- [ ] `PUBLIC_VERIFY_BASE_URL=https://verify.dreamspace.academy`
- [ ] `PUBLIC_SHOWCASE_BASE_URL=https://showcase.dreamspace.academy`
- [ ] `ENABLE_CRONS=true` and `TZ=Asia/Colombo` (crons run in-process via node-cron)
- [ ] `JWT_SECRET` (parent OTP tokens), `JWT_EXPIRES_IN`
- [ ] Optional/non-fatal if unset: `MONGODB_URI`/`MONGODB_DB`, `RESEND_*`,
      `NOTIFYLK_*`, `FCM_*`, `RATE_LIMIT_GLOBAL_PER_MINUTE`

## 5. Web PWA env (Vercel/Dokploy build args — baked at build time)

- [ ] `VITE_API_BASE_URL=https://appapi.dreamspace.academy/api/v1`
- [ ] `VITE_SUPABASE_URL` = Mumbai (matches API `SUPABASE_URL`)
- [ ] `VITE_SUPABASE_ANON_KEY` = Mumbai anon key
- [ ] Rebuild after changing any `VITE_*` (they're compiled in, not runtime).

---

## 6. Sydney → Mumbai cutover (mobile)

`web/.env.mobile` currently **pins the old Sydney project** because released
mobile builds validate against whatever Railway uses.

- [ ] After API + web are live on Mumbai, delete the Sydney override lines in
  `web/.env.mobile` (so it inherits Mumbai from `web/.env`), then rebuild the
  app: `cd web && pnpm build:mobile`, `cd mobile && npx cap sync`.

---

## 7. Post-deploy smoke tests

- [ ] API boots: logs show `[env=production, auth=(secret removed) and
      `[crons] scheduled (Asia/Colombo)`.
- [ ] `https://app.dreamspace.academy` loads; staff can sign in (JWT verifies
      against Mumbai JWKS — no "invalid/expired token").
- [ ] `/students` and `/payments` load; **enroll a test student → it appears on
      Payments (current month) as UNPAID** (validates the self-heal fix).
- [ ] Payments status dropdown filters server-side; Students "No level" filters.
- [ ] `https://verify.dreamspace.academy` and `https://showcase.dreamspace.academy`
      load (no auth).
- [ ] CORS: an authenticated request from the web app to the API returns 200,
      not a CORS error (confirms `WEB_ORIGIN` matches).

---

## 8. Rollback

- App: revert the commit / redeploy the previous Vercel/Railway/Dokploy build.
- DB: `db push` is not auto-reversible — use Supabase point-in-time restore.
  Always test schema changes on a staging DB first.

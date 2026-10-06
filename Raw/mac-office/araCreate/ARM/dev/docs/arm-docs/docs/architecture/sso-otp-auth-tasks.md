# SSO + Email OTP Auth Migration — Task Sheet

> Cross-repo task breakdown for removing password login in favor of Google SSO, Microsoft SSO, and email OTP.
> Scope: `core/arm-core-be`, `core/arm-session`, `core/arm-core-fe`, `services/arm-service-notification`.
> Source plan: security audit + implementation plan agreed in this session (see `docs/architecture/` for related docs).
> Task IDs: `A-#` (Phase A), `B-#` (Phase B), `C-#` (Phase C).

## Status Summary

| Task | Title | Status |
| --- | --- | --- |
| A-1 | `OtpType.LOGIN_EMAIL` + Redis key convention | 🔲 Pending |
| A-2 | `EmailLoginService` — request/verify OTP + find-or-create login | 🔲 Pending |
| A-3 | New DTOs + `auth.controller.ts` endpoints | 🔲 Pending |
| A-4 | Kafka producer + notification service email template/handler | 🔲 Pending |
| A-5 | session-service proxy endpoints for email-OTP login | 🔲 Pending |
| A-6 | Frontend `EmailOtpLoginForm` + wiring into `login-page.tsx` | 🔲 Pending |
| A-7 | Stage A end-to-end verification | 🔲 Pending |
| B-1 | Collapse Google/Microsoft login-only variant (backend) | 🔲 Pending |
| B-2 | Collapse `SocialLogin` login/signup branching (frontend) | 🔲 Pending |
| B-3 | Stage B verification | 🔲 Pending |
| C-1 | Delete password/signup/reset backend code | 🔲 Pending |
| C-2 | Drop `password` column migration | 🔲 Pending |
| C-3 | Delete password-era notification templates/handlers | 🔲 Pending |
| C-4 | Delete session service password-login endpoint | 🔲 Pending |
| C-5 | Delete frontend password/signup/reset UI | 🔲 Pending |
| C-6 | Stage C verification | 🔲 Pending |

---

## Phase A — Add Email OTP Login (additive, no existing login method touched)

Password login, signup, and Google/Microsoft SSO all keep working unmodified throughout this phase.

---

### A-1 — `OtpType.LOGIN_EMAIL` + Redis key convention

**TODO:**
- Add `LOGIN_EMAIL = 'login-email'` to `src/libs/enums/otp.enum.ts`.
- Confirm `redisService.setOtp`/`getOtp`/`incrementOtpAttempts`/`deleteOtp`/`deleteOtpAttempts` accept an arbitrary string key (email) in place of the `userId` they're normally called with — these methods are generic over the key already, no signature change expected.

**Dev:**
- Login OTP is keyed by normalized email (trim + lowercase), not `userId` — unlike signup/reset OTPs, a login-OTP request happens before we know if the account exists. Normalize before every Redis call to avoid a case-mismatch bypassing the attempt counter.
- Reuse the existing 300s TTL and the existing `MAX_OTP_ATTEMPTS = 5` constant — don't introduce new limits.

**Test:**
- No new test file needed for this task alone — covered by A-2's service spec.

---

### A-2 — `EmailLoginService` — request/verify OTP + find-or-create login

**TODO:**
- New file `src/auth/services/email-login.service.ts`.
- `requestOtp(email)`: normalize email, generate a 6-digit CSPRNG code via `(secret removed)(100000, 999999)`, store via `redisService.setOtp(normalizedEmail, OtpType.LOGIN_EMAIL, otp, 300)`, send via new Kafka producer method (A-4). Always return the same generic success message whether or not the account exists (avoids account enumeration).
- `verifyOtpAndLogin(email, code, ip, userAgent)`: get→increment→compare→delete OTP pattern (mirror `signup.service.ts` `verifyEmail()` and `password.service.ts` `verifyResetOtp()`), same `MAX_OTP_ATTEMPTS`/429 behavior. On success: find-or-create user by email (mirror the find-or-create block in `auth.service.ts` `googleLogin()`/`microsoftLogin()`), set `isVerified: true` and `authProvider: 'email-otp'` on create only, then mint the JWT/refresh-token pair.
- New user placeholder profile: `firstName: email.split('@')[0]`, `lastName: ''` on create.
- Recommended refactor while here: extract a shared `issueTokenPair(user, ip, userAgent)` on `AuthService` (currently duplicated 3x across `login`/`googleLogin`/`microsoftLogin`) and have `EmailLoginService` call it as the 4th caller, instead of copy-pasting the transaction block again.
- Register `EmailLoginService` as a provider in `src/auth/auth.module.ts`.

**Dev:**
- 4-digit `(secret removed)(1000 + Math.random() * 9000)` used elsewhere in this codebase is **not** acceptable here — this is a primary login factor, must be CSPRNG + 6 digits.
- Existing users are matched purely by email — no `authProvider` gate, no check against how the account was originally created. This is what makes the "no data migration needed" property true: a password-era user logging in via email-OTP for the first time just logs into their existing row.
- Existing `authProvider` values on pre-existing accounts are left untouched by this service.

**Test:**
- New `src/auth/services/email-login.service.spec.ts`, modeled on `signup.service.spec.ts` (OTP get/increment/compare/delete + 429 path) and `auth.service.spec.ts`'s `googleLogin`/`microsoftLogin` blocks (find-or-create + token mint assertions).
- Cases to cover: correct OTP succeeds and returns token pair; wrong OTP increments attempts and rejects; 5th wrong attempt returns 429 and clears the OTP; expired/missing OTP rejects; new email creates a user with placeholder name + `authProvider: 'email-otp'`; existing email logs into the existing row without altering its `authProvider`.
- Confirm the spec needs no `argon2` mock (proves the path is fully decoupled from passwords).

---

### A-3 — New DTOs + `auth.controller.ts` endpoints

**TODO:**
- New `src/auth/dto/request-login-otp.dto.ts` — `{ email }`, `@IsEmail()`.
- New `src/auth/dto/verify-login-otp.dto.ts` — `{ email, code }`, `code` validated as exactly 6 digits.
- Add `POST /auth/login/email/request-otp` and `POST /auth/login/email/verify` to `auth.controller.ts`, both `@Public()`, `@UseGuards(ThrottlerGuard)`, `@Throttle({ otp: { limit: 5, ttl: 60000 } })`.
- Inject `EmailLoginService` into `AuthController`'s constructor.

**Dev:**
- `verify` endpoint returns `{ userId, accessToken, refreshToken }` — same shape as today's `POST /auth/login` — so session service wrapping (A-5) is a straight mirror of the existing `/session/login` handler.
- Reuse the existing `otp` throttle bucket definition already registered in `app.module.ts` — don't define a new bucket.

**Test:**
- Add/extend `src/auth/auth.controller.spec.ts` with test blocks for both new routes (mocked `EmailLoginService`), following the existing per-route test pattern in that file.

---

### A-4 — Kafka producer + notification service email template/handler

**TODO:**
- `src/libs/kafka/producers/notification-producer.service.ts`: add `sendLoginOtpEmail(email, otp)` mirroring `sendPasswordResetEmail`, new topic `send_login_otp_email`.
- `services/arm-service-notification/template/login-otp.hbs`: copy `template/reset-password.hbs` (already renders `{{otp}}` with a 5-minute expiry notice), retitle to "Your Login Code".
- `services/arm-service-notification/src/mail/mail.service.ts`: add `sendLoginOtpEmail(...)` mirroring `sendPasswordResetEmail`.
- `services/arm-service-notification/src/app.service.ts` and `src/app.controller.ts`: add matching `sendLoginOtpEmail` handler + `@EventPattern('send_login_otp_email')`, same try/catch → DLQ pattern as `handlePasswordResetEmail`.

**Dev:**
- Kafka producer calls are fire-and-forget (5s timeout, errors logged not thrown) — matches every other notification dispatch in this codebase, no new error-handling pattern needed.

**Test:**
- No existing spec files cover the notification producer or the notification microservice's controller — matching current coverage level, a manual/E2E check (A-7) is the verification for this task; adding unit coverage here is optional, not a regression if skipped.

---

### A-5 — session-service proxy endpoints for email-OTP login

**TODO:**
- New DTOs in `core/arm-session/src/session/dto/` mirroring `login.dto.ts`, for `{email}` and `{email, code}`.
- `core/arm-session/src/session/session.controller.ts`: add `POST /session/login/email/request-otp` (pure proxy to core-be, no session work) and `POST /session/login/email/verify` — copy the existing `login()` handler verbatim except the upstream call body/path.
- Confirm/add an `otp` throttler bucket to the session service's own `ThrottlerModule` config (today's session service controller only shows `login`/`proxy` buckets) — mirror core-be's `otp: 5/60s` bucket definition.

**Dev:**
- The `createSession` + `SESSION_ID` cookie tail in `/session/login` is provider-agnostic already — copy it unchanged, only the upstream request body/path differs.

**Test:**
- If `core/arm-session/src` has no existing `*.spec.ts` for `session.controller.ts` (confirm before writing), match current coverage level: manual/curl verification is the bar, covered by A-7's E2E smoke test — don't invent new session service test infra for this alone.

---

### A-6 — Frontend `EmailOtpLoginForm` + wiring into `login-page.tsx`

**TODO:**
- New `src/components/auth/form/email-otp-login-form.tsx` — two-step component (email entry → 6-digit code entry), modeled on the existing `verify-otp-form.tsx` / `otp-verification-page.tsx` pattern, self-contained (no new route).
- Parametrize `src/hooks/use-otp-verification.ts`'s hardcoded `otp.length !== 4` check with an `otpLength` option (default 4, pass 6 for this form) — don't fork the hook, since old 4-digit callers still exist until Phase C.
- Add `requestLoginOtp`/`verifyLoginOtp` to `src/api/services/auth-service.ts`, `src/hooks/use-auth.ts`, `src/types/auth.ts`, mirroring the existing `login()` call shape.
- Add the two new route constants to `src/lib/constants/api-routes/auth-routes.ts` and `session-routes.ts`.
- `src/pages/auth/login-page.tsx`: swap `<LoginForm />` for `<EmailOtpLoginForm />`. Keep `AuthHeader`, `AuthDivider`, `SocialLogin` unchanged.

**Dev:**
- Confirmed zero coupling between `LoginForm` and `SocialLogin` (no shared state/props) — safe to swap the password form in place without restructuring the page.

**Test:**
- Manual UI walkthrough (dev server): request code → receive email → enter code → land on `/home`, both for a brand-new email and an existing password-era account's email. Covered concretely in A-7.

---

### A-7 — Stage A end-to-end verification

**TODO:**
- Run backend unit suite (`pnpm test` in `arm-core-be`) — confirm A-2/A-3 specs pass and nothing else regresses.
- Manual/E2E smoke test: `POST /session/login/email/request-otp` → email arrives with 6-digit code → `POST /session/login/email/verify` → `SESSION_ID` cookie set → `GET /session/me` succeeds.
- Repeat the smoke test for: (a) a brand-new email — confirm a `users` row is created with placeholder name and `authProvider: 'email-otp'`; (b) an existing password-era user's email — confirm it logs into the *existing* `id` and that user's original `authProvider` is unchanged.
- Confirm password login, signup, and Google/Microsoft SSO are all still functioning unmodified.

**Dev:** n/a (verification task).

**Test:** this task *is* the test gate for Phase A — do not start Phase B until every item above passes.

---

## Phase B — Collapse Google/Microsoft to a single auto-provisioning entry point

Independent of Phase A/C's own risk; safe once merged signup+login means "reject unregistered" is no longer desired behavior.

---

### B-1 — Collapse Google/Microsoft login-only variant (backend)

**TODO:**
- `auth.controller.ts`: delete `GET /auth/google/login`, `GET /auth/microsoft/login`, and the `isLoginFlow`/`oauth_mode` cookie blocks inside `googleAuthCallback` and `microsoftAuthCallback` — both callbacks always auto-provision now.

**Dev:**
- This route/cookie mechanism only existed to keep signup and login separate for SSO. Once signup+login are merged everywhere else, the plain auto-provisioning routes (`/auth/google`, `/auth/microsoft`) are the only entry point needed — nothing downstream depends on the login-only variant.

**Test:**
- Update `auth.controller.spec.ts` to remove test blocks for the deleted routes/branches.

---

### B-2 — Collapse `SocialLogin` login/signup branching (frontend)

**TODO:**
- `src/components/auth/common/auth-social-login.tsx`: remove the `type="login"|"signup"` prop and its branching; point both buttons at the plain `GOOGLE_LOGIN`/`MICROSOFT_LOGIN` routes.
- Remove `GOOGLE_LOGIN_ONLY`, `MICROSOFT_LOGIN_ONLY` from `auth-routes.ts`.
- Update the lone call site (`login-page.tsx`) to drop the `type="login"` prop.

**Dev:** n/a — pure removal, no behavior branching left to reason about.

**Test:**
- Manual: click both Google and Microsoft buttons from the login page for a brand-new account and an existing account — both should succeed in both cases.

---

### B-3 — Stage B verification

**TODO:**
- Confirm new and existing accounts both still log in successfully via Google and Microsoft.
- Confirm removing the "reject unregistered" check produces the intended behavior (auto-create), not a silent regression elsewhere (e.g. any admin/reporting code that assumed SSO login could never create new accounts).

**Dev:** n/a (verification task).

**Test:** this task is the gate before starting Phase C.

---

## Phase C — Delete password/signup/reset entirely

Only start after Phase A has been verified working on the server. This phase includes the one irreversible step (DB column drop), so it goes last.

---

### C-1 — Delete password/signup/reset backend code

**TODO:**
- `auth.controller.ts`: delete `POST /auth/login`, `/signup`, `/verify-email`, `/resend-verification`, `/set-password`, `/forgot-password`, `/verify-reset-otp`, `/reset-password`, `/change-password`. Drop `SignupService`/`PasswordService` from the constructor.
- `auth.service.ts`: delete `login()` and `validateUser()`; remove `import * as argon2`. Keep `refresh()`, `logout()`, `findUserByEmail()`, `googleLogin()`, `microsoftLogin()` unchanged.
- `auth.module.ts`: remove `SignupService`/`PasswordService` providers.
- `users.service.ts`: delete argon2-hashing branches in `createUser`/`updateUser`, plus `findByEmailWithPassword`/`findByIdWithPassword` (no remaining callers after this task).
- `redis.service.ts`: delete `setSignupSession`/`getSignupSession`/`deleteSignupSession`.
- Delete whole files: `signup.service.ts` (+ spec), `password.service.ts` (+ spec), `libs/crypto/argon2.config.ts`, and the now-unused DTOs (`signup.dto.ts`, `verify-email.dto.ts`, `resend-verification.dto.ts`, `set-password.dto.ts`, `forgot-password.dto.ts`, `verify-reset-otp.dto.ts`, `reset-password.dto.ts`, `change-password.dto.ts`, `login.dto.ts`).
- Remove `argon2` from `package.json`/`package-lock.json` (grep for zero remaining references first).

**Dev:**
- Order matters within this task: delete controller routes and service methods before deleting the DTOs/services they import, so there's never a commit with a dangling import (if landing incrementally rather than as one change).

**Test:**
- Update/remove test blocks in `auth.controller.spec.ts`, `auth.service.spec.ts`, `users.service.spec.ts` for everything deleted above.
- Full unit suite green after deletion, with no `argon2`-related mocks left anywhere in the suite (grep to confirm).

---

### C-2 — Drop `password` column migration

**TODO:**
- New migration `src/database/migrations/{epoch}-DropPasswordColumn.ts`, following the exact convention of `(secret removed)` (raw SQL, `IF EXISTS`/`IF NOT EXISTS` guards, both `up()`/`down()`).
- `users.entity.ts`: delete the `password` column.

**Dev:**
- `down()` can only restore the column structurally, not the data — acceptable, matches how other migrations in this repo treat `down()` as structural-only.
- Do not run this migration until C-1 is fully merged and no code path references `user.password` anywhere (grep to confirm before running).

**Test:**
- Run the migration against a local DB copy; confirm `up()` succeeds, `down()` re-adds the column without error, and the app boots cleanly with the column gone.

---

### C-3 — Delete password-era notification templates/handlers

**TODO:**
- Delete `handleSendOTPEmail`/`sendVerificationEmail`/`template/verify.hbs` (signup verification).
- Delete `handlePasswordResetEmail`/`sendPasswordResetEmail`/`template/reset-password.hbs` (password reset).
- Delete the corresponding producer methods in `notification-producer.service.ts`.
- Keep `sendWelcomeEmail` (still used by email-OTP's own welcome-on-create path from A-2).

**Dev:** n/a — pure removal.

**Test:**
- Confirm the notification service still boots and the `login-otp` and `welcome` handlers still fire correctly after removal (no accidental deletion of shared mail-module config).

---

### C-4 — Delete session service password-login endpoint

**TODO:**
- Delete `POST /session/login` from `core/arm-session/src/session/session.controller.ts`.
- Delete the `CHANGE_PASSWORD` proxy route constant from `session-routes.ts` (frontend) and confirm no session-service-side code references it (it was a generic proxy passthrough, no session-service logic to remove beyond the constant).

**Dev:** n/a — pure removal.

**Test:**
- Confirm `POST /session/login` returns 404 after deletion, and the session service's own test suite (if any) passes.

---

### C-5 — Delete frontend password/signup/reset UI

**TODO:**
- Delete: `login-form.tsx`, `components/auth/form/signup/` (whole dir), `components/auth/form/reset-password/` (whole dir), `pages/auth/register-page/` (whole dir), `pages/auth/forget-page/` (whole dir).
- Remove any change-password section inside the account/profile settings form (keep the rest of that file — it also handles name/profile editing).
- `routes/auth.router.tsx`: remove signup/verify-otp/set-new-password/forget-password/otp-verification/enter-password routes. Keep `login`, Google/Microsoft `callback`.
- Remove corresponding entries from `auth-routes.ts`, `session-routes.ts`, `types/auth.ts`, `use-auth.ts`, `auth-service.ts`, and the `auth/index.ts` constants barrel.

**Dev:** n/a — pure removal.

**Test:**
- Confirm no orphaned nav links to deleted pages; manual click-through of the login page and account settings page.

---

### C-6 — Stage C verification

**TODO:**
- Confirm `/v1/auth/login`, `/v1/auth/signup`, `/v1/auth/forgot-password`, etc. 404 cleanly from core-be.
- Re-run the Phase A smoke test (email-OTP login) plus Google/Microsoft login to confirm nothing broke.
- Full repo test suites green across `arm-core-be`, `arm-session` (if applicable), `arm-core-fe` (if applicable).

**Dev:** n/a (verification task).

**Test:** this task is the final gate — password login should now be completely unreachable, with email-OTP and both SSO providers as the only working login methods.

# Calendar App — Google OAuth Integration

> How linking a Google calendar account works end-to-end: popup → dedicated session service routes → Google consent → session service callback → calendar-be token storage → parent refresh.
>
> This is **not** platform login SSO (that lives in core-be and is covered by [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md)). This document is the calendar-native **Add Account** / **Reconnect** flows that write MongoDB `accounts` rows with usable refresh tokens.
>
> Verified against `core/arm-session/src/session/session.controller.ts`, `apps/arm-app-calendar/src/backend/src/{auth,google}/*`, and the calendar / shell frontends (2026-08-10).

---

## 1. Two different Google OAuth stories

| Flow | Purpose | Tokens land in | Documented here? |
|---|---|---|---|
| **Platform login SSO** | Sign into ARM with Google | core-be (encrypted) + Redis session via session service | [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) |
| **Add Account** | Link an additional Google calendar identity to the logged-in ARM user | MongoDB `accounts` (encrypted access + refresh) | **Yes — §2–§7** |
| **Reconnect / Update Account** | Re-auth an existing linked account whose tokens are dead | Same `accounts` row, tokens refreshed | **Yes — §8** |

Do not conflate them. Login-provisioned calendar accounts (`syncFromCoreAuth`) often have a weak/missing refresh token (see calendar `CLAUDE.md` / `TASK-037`). **Add Account** is the path that correctly populates `refreshToken`.

---

## 2. Add Account — full flow

```text
[Shell /calendar  OR  Settings → Linked Accounts]
  window.open(`${VITE_SESSION_URL}/session/calendar/add-account`)
        │  popup carries SESSION_ID cookie (session service host)
        ▼
GET /session/calendar/add-account          (SessionGuard)
  S2S GET {CALENDAR_BE_URL}/v1/auth/google/add-account
       Authorization: Bearer {session.accessToken}
       maxRedirects: 0  ← capture Location, do not follow
        ▼
calendar-be AuthController.googleAddAccount
  GoogleService.generateAddAccountUrl(user.sub)
  → 302 Location: Google authorize URL
       redirect_uri = OAUTH_CALLBACK_URL
       state        = plaintext userId
        ▼
session service res.redirect(302, googleAuthUrl)   ← browser never sees Docker hostname
        ▼
[Google consent screen]
        ▼
GET /session/calendar/oauth/callback?code=&state=   (SessionGuard)
  reject if error / missing code
  reject if state !== session.userId
  S2S POST {CALENDAR_BE_URL}/v1/auth/google/add-account/callback
       body { code, state }
       Authorization: Bearer {session.accessToken}
        ▼
calendar-be GoogleOAuthService.exchangeAddAccountCode
  state must === userId
  (secret removed)(code)  (redirect_uri = OAUTH_CALLBACK_URL)
  userinfo → email
  encrypt tokens → Account.createOrUpdate → MongoDB `accounts`
  return { ok: true }
        ▼
session service returns tiny HTML page:
  BroadcastChannel("google_auth_done").postMessage({ type: "GOOGLE_AUTH_SUCCESS" })
  window.close()
        ▼
[Parent] refreshes account list (see §6)
```

### UI entry points

| Surface | File | Opens |
|---|---|---|
| Calendar account page (`/calendar` MF remote) | `apps/arm-app-calendar/src/frontend/.../calendar-account-page.tsx` | `${env.SESSION_URL}/session/calendar/add-account` |
| Settings → Linked Accounts | `core/arm-core-fe/.../linked-accounts-tab.tsx` via `useOAuthPopup` | `SESSION_ROUTES.CALENDAR_ADD_ACCOUNT` (`/session/calendar/add-account`) |

Constant: `core/arm-core-fe/src/lib/constants/api-routes/session-routes.ts`.

---

## 3. Why a popup + dedicated session service routes

### Popup

`SESSION_ID` is an httpOnly cookie scoped to the **session service host** (`sameSite=lax`). The consent round-trip must navigate a browser context that:

1. Already has that cookie (popup opened from the logged-in shell inherits the cookie jar for the session service origin once it navigates there).
2. Can return from Google as a top-level GET to the session service so Lax cookies are sent.
3. Leaves the shell mounted so the parent can refresh Linked Accounts without a full SPA unload.

### Dedicated routes (not `/session/proxy/...`)

From `session.controller.ts`:

> Cannot use the generic proxy: axios would follow 302→Google server-side. Instead: call calendar-be with `maxRedirects: 0`, capture `Location`, redirect the browser — never exposing the internal Docker hostname.

Routes:

| Method | Path | Guard | Role |
|---|---|---|---|
| `GET` | `/session/calendar/add-account` | `SessionGuard` | Start Google add-account |
| `GET` | `/session/calendar/oauth/callback` | `SessionGuard` | Google redirect_uri |
| `GET` | `/session/calendar/add-account/microsoft` | `SessionGuard` | Microsoft twin (§10) |
| `GET` | `/session/calendar/oauth/callback/microsoft` | `SessionGuard` | Microsoft twin |

S2S calls use the session JWT as `Authorization: Bearer …`. They do **not** use `X-Session-Secret` (that header is for `/session/internal/*` and core token fetches).

---

## 4. calendar-be handlers

| Step | Controller | Service |
|---|---|---|
| Start | `auth.controller.ts` → `GET /v1/auth/google/add-account` | `GoogleService.generateAddAccountUrl` |
| Finish | `auth.controller.ts` → `POST /v1/auth/google/add-account/callback` | `GoogleOAuthService.exchangeAddAccountCode` |

### Scopes (Add Account — hard-coded)

```text
openid
email
profile
https://www.googleapis.com/auth/calendar
https://www.googleapis.com/auth/calendar.calendarlist
https://www.googleapis.com/auth/calendar.events
https://www.googleapis.com/auth/calendar.events.owned
```

### What gets written

MongoDB collection `accounts` via `AccountService.createOrUpdate`:

| Field | Source |
|---|---|
| `userId` | JWT `sub` / OAuth `state` |
| `accountId` / `email` | Google userinfo email |
| `displayName` / `picture` | userinfo |
| `accessToken` / `refreshToken` | AES-GCM ciphertext (`enc:<iv>:<ciphertext>:<authTag>`) via `TokenEncryptionService` + `GOOGLE_TOKEN_ENCRYPTION_KEY` |
| `tokenExpiryAt` | Google expiry − 10s |
| `isAuthenticated` | `true` |
| `provider` | schema default `'google'` |

This exchange does **not** fetch calendars or register webhook channels. Those are separate post-link steps — see §6.5, and note that discovering a calendar does not import it.

If the same ARM user already linked that Google email → `account_already_connected` → session service treats non-ok as error redirect.

---

## 5. CSRF / state

Add Account uses **plaintext `state = userId`**, checked twice:

1. **session service** — `state !== session.userId` → redirect `?error=oauth_failed`
2. **calendar-be** — `state !== userId` (from Bearer JWT) → `400 Invalid state parameter`

That closes “steal a code under one session, exchange it under another.”

Reconnect uses a **different**, stronger state (encrypted + TTL) — see §8.

---

## 6. Completion signal (BroadcastChannel — not postMessage)

Primary success path (session service HTML):

```js
BroadcastChannel("google_auth_done").postMessage({ type: "GOOGLE_AUTH_SUCCESS" });
window.close();
```

Payload is type-only — **no tokens, no email, no accountId** in the browser message.

### Parent listeners

| Parent | Listens for | Also has close-poll? |
|---|---|---|
| `calendar-account-page.tsx` | `BroadcastChannel("google_auth_done")` + legacy `window.postMessage` (same-origin) | **Yes** — every 500ms while adding |
| Settings `useOAuthPopup` | **Only** `window.postMessage` with `GOOGLE_AUTH_SUCCESS` \| `ACCOUNT_ADDED` and `event.origin === window.location.origin` | **No** |

### Local-dev gotcha

`BroadcastChannel` is **same-origin**. Locally the shell is `localhost:3000` and the session service success page is `localhost:5001`, so the channel message often never arrives.

- Calendar account page recovers via **popup-close polling** → refetch.
- Settings `useOAuthPopup` may **miss** success until the user manually refreshes, unless session service and shell share an origin (deployed Caddy path `/session/*` on the same host).

Legacy `auth-callback.tsx` (route `/calendar/account`) still uses `postMessage(..., '*')` for error/reconnect-success pages. That is a separate, weaker path — not what Add Account success uses today.

---

## 6.5 After the signal — the import step

Connecting an account and importing its calendars are **two steps, and the second is a real choice.**
The code exchange writes only the account row (see §4); nothing is imported by it.

On the completion signal, `calendar-account-page.tsx`:

1. refetches accounts, which is also what *discovers* the account's calendars — `fetchAllAccountsWithCalendars` refreshes from the provider before reading, and every newly discovered calendar is inserted with `imported: false`;
2. identifies the new account by diffing the account list against the ids captured before the popup opened — the popup reports no account id back;
3. opens `ImportCalendarsDialog` on it, with nothing pre-checked.

### What Cancel means

**Cancel disconnects the account.** An account that imported nothing has no reason to hold live
tokens, so dismissing the dialog is taken as the answer it is and runs the full delete cascade
(`DELETE /v1/account/:id` — see [DELETE-CASCADE.md](DELETE-CASCADE.md)). There is no confirm step:
nothing was imported, so nothing is lost but the OAuth round trip.

Escape counts as Cancel. A click on the backdrop does not — `ImportCalendarsDialog` makes it inert
precisely because dismissal is now destructive.

Closing the tab instead reports nothing, so that account survives until the next load of the Manage
Syncs page, where a mount-only sweep disconnects it. Two constraints on that sweep: it never runs on
the refetches that follow an action (the connect flow refetches *before* this dialog opens, so a
sweep there would delete the account just connected), and it is scoped by `createdAt` against
`IMPORT_RULES_LIVE_FROM` so accounts predating the rule are grandfathered.

Those grandfathered accounts — and any whose disconnect failed — are what `AccountTileCard` on the
Manage Syncs grid is now for: the grid otherwise lists calendars and would show nothing at all for an
account with none. Import and Disconnect both live on that card. It is a recovery surface, not the
normal resting state.

An **already-linked** account (`alreadyLinked: true`) is absent from the diff in step 2, so no dialog
opens and its existing selection is left untouched — that path refreshes tokens, nothing else. It is
also never at risk from the above: it already has imported calendars.

### Re-importing later

The same dialog opens from the account's card or from **Add Calendar → Import from → \<email\>**, with
the current selection pre-checked **and locked**. The dialog only ever adds: already-imported rows
are disabled ("Already imported"), so a save is always a superset of what is already there and can
never un-import. Removing a calendar is the calendar tile's job — and removing the last one takes the
account with it.

Cancelling *here* is not destructive: the account has imported calendars, so it is not the empty case
above.

---

## 7. Error UX

| Failure | Behavior |
|---|---|
| User denies consent / missing `code` | session service → `{FRONTEND_URL}/calendar/account?error=oauth_failed` → MF `AuthCallback` |
| State mismatch | Same error redirect (session service logs warn) |
| Expired / missing session in popup | `SessionGuard` → 401 (no session cookie / session gone) |
| `account_already_connected` / calendar-be non-ok | session service error redirect |
| Upstream down | session service `502` JSON |
| Popup blocked | FE alert; clears “Adding…” state |
| Encryption key mismatch later | Write still succeeds with current key; later decrypt throws (“key mismatch”) on API use |

---

## 8. Reconnect (Update Account) — different path

Triggered from Settings when `!account.isAuthenticated` → fetch reconnect URL → open Google.

| Aspect | Add Account | Reconnect |
|---|---|---|
| Start | `GET /session/calendar/add-account` (dedicated) | `GET /session/proxy/calendar/v1/google/update/account/{email}` (generic proxy OK — returns JSON `{ url }`, no 302 chain) |
| calendar-be | `GET /v1/auth/google/add-account` | `GET /v1/google/update/account/:email` |
| `redirect_uri` | `OAUTH_CALLBACK_URL` → session service | `GOOGLE_REDIRECT_URI` → calendar-be callback |
| `state` | plaintext `userId` | AES `enc:…` of `{ userId, email, exp }` (10 min TTL) |
| Google extras | — | `prompt: 'consent'`, `login_hint: email` |
| Scopes | Hard-coded set (§4) | `GOOGLE_SCOPES` env (narrower in Compose) |
| Browser callback | session service `/session/calendar/oauth/callback` | calendar-be `GET /v1/google/update/account/callback` (**no AuthGuard**; CSRF via encrypted state) |
| Token write | `createOrUpdate` | `updateAccountTokens(userId, email, …)` |
| Success UX | session service HTML + BroadcastChannel | Redirect `{CLIENT_REDIRECT_URL}/calendar/account?success=true` |

Controller: `apps/arm-app-calendar/src/backend/src/google/google.controller.ts`.

**AUTH-ARCHITECTURE note:** reconnect does **not** re-run the Add Account session service popup routes — it is this Update Account flow.

---

## 9. Environment variables (this flow)

| Variable | Consumer | Role |
|---|---|---|
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | calendar-be | OAuth client |
| `OAUTH_CALLBACK_URL` | calendar-be | Add Account `redirect_uri` (default `http://localhost:5001/session/calendar/oauth/callback`) — must be registered in Google Console |
| `GOOGLE_REDIRECT_URI` / `GOOGLE_CALENDAR_CALLBACK_URL` | calendar-be / platform `.env` | Reconnect `redirect_uri` — must hit `GET /v1/google/update/account/callback` |
| `GOOGLE_SCOPES` | calendar-be | Reconnect scopes only |
| `GOOGLE_TOKEN_ENCRYPTION_KEY` | calendar-be | 64 hex chars; must match core-be if both encrypt Google tokens |
| `VITE_SESSION_URL` / `SESSION_PUBLIC_URL` | FE / platform | Popup base (`http://localhost:5001` local) |
| `CALENDAR_BE_URL` | session service | S2S upstream |
| `FRONTEND_URL` | session service | Error redirect to shell |
| `CLIENT_REDIRECT_URL` | calendar-be | Reconnect success/error redirect |
| `JWT_ACCESS_SECRET` | calendar-be | Validates Bearer from session service |
| Session / Redis / `SESSION_COOKIE_SECRET` | session service | Popup cookie → session |

See [ENV-VARS.md](../ENV-VARS.md). Known example drifts: calendar-be `.env.example` omits `OAUTH_CALLBACK_URL`; its sample `GOOGLE_REDIRECT_URI` does not match the real `/v1/google/update/account/callback` route.

---

## 10. Microsoft twin (short)

Same session service pattern:

- `GET /session/calendar/add-account/microsoft`
- `GET /session/calendar/oauth/callback/microsoft`
- BroadcastChannel `"microsoft_auth_done"` with `{ type: "ACCOUNT_ADDED" }`
- `MICROSOFT_OAUTH_CALLBACK_URL`, `MICROSOFT_CLIENT_*`
- Scopes include `offline_access` + `Calendars.ReadWrite`

No Microsoft reconnect controller analogous to Google Update Account was found at doc time. Capability note: Microsoft is CRUD-only (no push notifications) — see calendar `CLAUDE.md`.

---

## 11. Security checklist

- Tokens never appear in browser responses, URLs, or BroadcastChannel payloads.
- Add Account CSRF: dual `state === userId` checks (session service + calendar-be).
- Reconnect CSRF: encrypted time-bound state; plaintext/`enc:`-missing state rejected before decrypt fallback can help an attacker.
- Account squatting fixed in `createOrUpdate` (SEC-003) — duplicate Google identity reassignment rules live in account service.
- Public account DTOs strip token fields.
- `TRUST_PROXY_HEADERS` must stay `false` outside real Kong (SEC-001).

---

## 12. Known pitfalls / drift

1. **BroadcastChannel vs postMessage** — inventory DOC-A05 outline assumed postMessage; current Add Account success is BroadcastChannel + close-poll.
2. **Two FE parents, different listeners** — Settings hook may not see local-dev success (§6).
3. **Reconnect ≠ Add Account** — different redirect URIs, state formats, and callback hosts.
4. **miro AUTH-04** and similar diagrams that show Google → calendar-be `/v1/auth/google/callback` + rich `postMessage` payloads are outdated.
5. **P1-6** (“hard-coded postMessage origin”) — referenced only in the inventory; primary success path moved to BroadcastChannel. `auth-callback.tsx` still posts to `'*'` for legacy/error paths.
6. **Scope mismatch** — Add Account hard-codes a wider scope set than Compose `GOOGLE_SCOPES` used on reconnect.

---

## Related docs

| Doc | Why |
|---|---|
| [AUTH-ARCHITECTURE.md](../AUTH-ARCHITECTURE.md) | Platform login vs linked accounts |
| [ENV-VARS.md](../ENV-VARS.md) | Full env reference |
| [`core/arm-session/README.md`](../../../../core/arm-session/README.md) | session + dedicated OAuth routes |
| `apps/arm-app-calendar/CLAUDE.md` | Sync / provider capability constraints |
| [`DELETE-CASCADE.md`](DELETE-CASCADE.md) | What unlink must tear down after OAuth link |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Token refresh (CoreAuth fallback for Google, none for Microsoft), encryption, module map |
| [`SYNC-FLOW.md`](SYNC-FLOW.md) | What actually happens once a linked account starts syncing |
| [`WEBHOOKS.md`](WEBHOOKS.md) | Push channels registered *after* a successful link |
| [`../troubleshooting/auth-failures.md`](../troubleshooting/auth-failures.md) | Platform-login auth troubleshooting — a distinct failure class from this doc's Add-Account/Reconnect flow |
| [`../adr/005-google-oauth-popup.md`](../adr/005-google-oauth-popup.md) | Decision record for the popup + BroadcastChannel pattern this doc describes in full |

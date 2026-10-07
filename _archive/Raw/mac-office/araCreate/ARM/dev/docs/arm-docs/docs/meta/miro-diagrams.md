# ARM Platform — Miro Board Diagram Inventory

> **Purpose:** Reference for every diagram to build on the ARM platform Miro board.
> **Source basis:** Derived directly from the live codebase.
> **Last updated:** 2026-06-08
> **Total Miro frames:** 27

---

## How to Use This Document

Each diagram entry contains:
- **ID** — unique reference for the Miro board frame
- **Diagram name** — the exact title to use on the board
- **Purpose** — what decision or understanding it enables
- **Type** — recommended Miro/diagramming format
- **Priority** — Critical · Important

**Section 11** lists diagrams that should be written as markdown documentation, not Miro frames — the content already exists in this file and should be moved to the relevant doc.

**Section 12** lists diagrams to build only when the corresponding work begins.

Build in **priority order**, not section by section.

---

## Table of Contents

1. [System Architecture — Platform Level](#1-system-architecture--platform-level)
2. [Authentication & Session Management](#2-authentication--session-management)
3. [session service — Backend for Frontend](#3-session-service--backend-for-frontend)
4. [Calendar Module](#4-calendar-module)
5. [Notifications & Messaging](#5-notifications--messaging)
6. [Frontend & Module Federation](#6-frontend--module-federation)
7. [Database & Storage Architecture](#7-database--storage-architecture)
8. [Infrastructure & Networking](#8-infrastructure--networking)
9. [Deployment & CI/CD](#9-deployment--cicd)
10. [Developer Workflows](#10-developer-workflows)
11. [Covered in Documentation — Not Miro Frames](#11-covered-in-documentation--not-miro-frames)
12. [Deferred — Build When Relevant](#12-deferred--build-when-relevant)
13. [Master Checklist](#13-master-checklist)

---

## 1. System Architecture — Platform Level

> Build these first. Every other diagram references them.

---

### ARCH-01 — ARM Platform High-Level Architecture

| Field | Value |
|---|---|
| **ID** | ARCH-01 |
| **Diagram name** | ARM Platform — High-Level System Architecture |
| **Type** | Architecture Block Diagram |
| **Priority** | **Critical** |

**Purpose:** The single most important diagram. Shows the complete system at 30,000 feet — all services, their relationships, traffic direction, and technology stack. Entry point for anyone new to the platform.

**Must include:**
- Browser (arm-core-fe shell + calendar-fe remote loaded inside it)
- Kong API Gateway (edge — north-south)
- arm-session (session + proxy layer)
- arm-core-be (NestJS + PostgreSQL)
- arm-calendar-be (NestJS + MongoDB + BullMQ)
- arm-service-notification (NestJS Kafka consumer)
- Redis (session store + BullMQ backing)
- Kafka (message broker — 3 topics + DLQ)
- Verdaccio (internal npm registry)
- Google Calendar API (external)
- Google OAuth 2.0 (external)
- SMTP provider (external)
- Clear boundary lines: public internet / internal network / external APIs
- Traffic direction arrows
- Technology stack callout box per service (language, database, framework)

---

### ARCH-02 — Service Topology & Port Map

| Field | Value |
|---|---|
| **ID** | ARCH-02 |
| **Diagram name** | ARM Platform — Service Topology & Port Reference |
| **Type** | Network Topology Diagram |
| **Priority** | **Critical** |

**Purpose:** Tells every engineer which service listens on which port, in which network, and which direction traffic flows. Essential for debugging connectivity issues.

**Must include:**

| Service | Internal Port | Host-Mapped Port | Network |
|---|---|---|---|
| arm-core-fe | 80 (nginx) | 3000 | arm-dev-network |
| arm-calendar-fe | 80 (nginx) | 3001 | arm-dev-network |
| arm-session | 5000 | 5000 | arm-dev-network + arm-infra-network |
| arm-core-be | 3000 | — (internal only) | arm-dev-network + arm-infra-network |
| arm-calendar-be | 4001 | 4001 | arm-dev-network + arm-infra-network |
| arm-notification | — | — | arm-dev-network + arm-infra-network |
| Kong | 8000 | 8000 | arm-dev-network + arm-infra-network |
| Caddy | 443/80 | 443/80 | arm-dev-network |
| PostgreSQL | 5432 | 5432 | arm-infra-network |
| MongoDB | 27017 | 27017 | arm-infra-network |
| Redis | 6379 | 6379 | arm-infra-network |
| Kafka | 9092 | 9092 | arm-infra-network |
| Verdaccio | 4873 | 4873 (localhost only) | arm-infra-network |

---

### ARCH-03 — Network Boundary & Trust Model

| Field | Value |
|---|---|
| **ID** | ARCH-03 |
| **Diagram name** | ARM Platform — Network Boundary & Trust Model |
| **Type** | Security Architecture Diagram |
| **Priority** | **Critical** |

**Purpose:** Shows which zone each service lives in, what credentials cross zone boundaries, and which internal paths are unauthenticated (trusted by network isolation). Critical for security reviews.

**Zones to show:**
- **Public Internet** — Browser, Google APIs, SMTP provider
- **Edge** — Caddy (dev) / Kong (prod) — TLS termination, rate limiting
- **session service Zone** — arm-session only — holds JWT in Redis, never exposes to browser
- **Internal Network** — arm-core-be, arm-calendar-be, arm-service-notification
- **Data Zone** — PostgreSQL, MongoDB, Redis, Kafka

**Trust arrows to label:**
- Browser → session service: SESSION_ID cookie (httpOnly, SameSite=strict)
- session service → core-be: `Authorization: Bearer <jwt>` (access token)
- session service → calendar-be: `Authorization: Bearer <jwt>`
- core-be → session service: `X-Session-Secret` header (internal only)
- Kong → backends: `X-User-Id` header (after JWT validation)
- calendar-be → core-be: direct HTTP (CircuitBreaker-wrapped)
- core-be → Kafka: emit (no auth — network-isolated)
- Kafka → notification: consume (no auth — network-isolated)

---

### ARCH-04 — Multi-Repository Workspace Structure

| Field | Value |
|---|---|
| **ID** | ARCH-04 |
| **Diagram name** | ARM Platform — Repository & Workspace Structure |
| **Type** | Structural Diagram (Tree) |
| **Priority** | **Important** |

**Purpose:** Shows how 7 separate GitHub repositories compose into the local workspace and how `make clone-repos` (driven by `repos.conf`) assembles them. Critical for new engineers.

```
arm/  ← local workspace root (not a single git repo)
├── core/
│   ├── arm-core-fe/       ← repo: arm-core-fe          (branch: dev)
│   ├── arm-core-be/       ← repo: arm-core-be          (branch: dev)
│   └── arm-session/           ← repo: arm-session              (branch: dev)
├── apps/
│   └── arm-app-calendar/  ← repo: arm-app-calendar     (branch: dev)
│       └── apps/
│           ├── frontend/
│           └── backend/
├── services/
│   └── arm-service-notification/ ← repo: arm-service-notification (branch: dev)
├── library/
│   └── arm-library-ui-components/ ← repo: arm-library-ui-components (branch: dev)
└── deploy/
    └── arm-deploy-make/   ← repo: arm-deploy-make      (branch: main)
        └── arm-deploy-k8s/ ← repo: arm-deploy-k8s     (branch: main)
```

---

## 2. Authentication & Session Management

> Six distinct auth flows exist in this system. Each needs its own sequence diagram.

---

### AUTH-01 — Email/Password Login Flow

| Field | Value |
|---|---|
| **ID** | AUTH-01 |
| **Diagram name** | Authentication — Email/Password Login Sequence |
| **Type** | Sequence Diagram |
| **Priority** | **Critical** |

**Purpose:** The primary authentication path. Must be crystal-clear for every engineer working on session, auth, or frontend.

**Sequence:**
```
Browser → POST /session/login {email, password}
  session service → POST http://core-be:3000/v1/auth/login
    core-be: validate credentials (bcrypt compare)
    core-be: generate accessToken (JWT, 15m) + refreshToken (JWT, 30d)
    core-be: store RefreshToken row in PostgreSQL (hash, userAgent, ip, expiresAt)
  core-be → session service: { userId, accessToken, refreshToken }
  session service: createSession() → uuid = randomUUID()
  session service: sign uuid with (secret removed)(SESSION_COOKIE_SECRET) → signed SESSION_ID
  session service → Redis: SET session:data:{uuid} { accessToken, refreshToken, userId, userAgent, ipHash } TTL:30d
  session service → Browser: Set-Cookie SESSION_ID={signed} (httpOnly, SameSite=strict, Secure)
  session service → Browser: { userId }
Browser: stores userId in Zustand auth-store
```

**Rate limiting note:** core-be has ThrottlerGuard on login endpoint (10 req/60s).

---

### AUTH-02 — Email Signup Multi-Step Flow

| Field | Value |
|---|---|
| **ID** | AUTH-02 |
| **Diagram name** | Authentication — Email Signup 4-Step Flow |
| **Type** | Sequence Diagram |
| **Priority** | **Critical** |

**Purpose:** Signup is a 4-step stateful flow (signup → verify email → set password → activate session). Unique because it uses a **handoff token** — not a cookie — to bridge the final step.

**Step 1 — Signup:**
```
Browser → POST /session/proxy/core/v1/auth/signup { email, firstName, lastName }
  session service → core-be /v1/auth/signup
    core-be: create User (isVerified: false, password: null)
    core-be: generate OTP, store in Redis
    core-be → Kafka: emit('send_email', { userId, email, otp, type: 'verify-email' })
    Kafka → notification-service: deliver
    notification-service → SMTP: send verification email
```

**Step 2 — Verify Email:**
```
Browser → POST /session/proxy/core/v1/auth/verify-email { email, otp }
  session service → core-be /v1/auth/verify-email
    core-be: validate OTP from Redis → mark User.isVerified = true
```

**Step 3 — Set Password:**
```
Browser → POST /session/proxy/core/v1/auth/set-password { email, password }
  session service → core-be /v1/auth/set-password
    core-be: hash password (bcrypt) → User.password = hash
    core-be: generate accessToken + refreshToken
    core-be → POST http://arm-session:5000/session/internal/create-session (X-Session-Secret header)
      session service: create Redis session → signedId
      session service: create handoff token → uuid → Redis SET session:handoff:{uuid} = signedId TTL:30s
    session service → core-be: { signedId, handoffId }
  core-be → session-service proxy → Browser: { handoffId }
```

**Step 4 — Activate Session:**
```
Browser → POST /session/session/activate { handoffId }
  session service: Redis GETDEL session:handoff:{uuid} → signedId (atomic — single-use)
  session service: getSession(signedId) → verify session exists
  session service → Browser: Set-Cookie SESSION_ID={signedId}
  session service → Browser: { ok: true, userId }
```

---

### AUTH-03 — Google OAuth Login Flow (core-be)

| Field | Value |
|---|---|
| **ID** | AUTH-03 |
| **Diagram name** | Authentication — Google OAuth Login via core-be |
| **Type** | Sequence Diagram |
| **Priority** | **Critical** |

**Purpose:** Documents how Google Sign-In for the platform itself works. Uses Passport Google strategy + session service internal endpoint + session service OAuth callback — a 3-party redirect chain.

**Sequence:**
```
Browser → GET /api/core/v1/auth/google
  Kong → core-be: GET /v1/auth/google
  core-be (Passport GoogleStrategy): redirect → Google OAuth consent screen

Google → GET core-be /v1/auth/google/callback?code=...
  core-be: exchange code → { access_token, refresh_token, profile }
  core-be: upsert User (googleId, picture, authProvider='google')
  core-be: save GoogleToken (AES-256 encrypted) in PostgreSQL
  core-be: generate platform JWT accessToken + refreshToken
  core-be → POST http://arm-session:5000/session/internal/create-session (X-Session-Secret)
    session service: createSession → signedId
    session service: createHandoffToken → handoffId (TTL: 30s)
  session service → core-be: { signedId, handoffId }
  core-be → Browser: redirect → GET /session/oauth/callback?sid={handoffId}

Browser → GET /session/oauth/callback?sid={handoffId}
  session service: getSession(handoffId) → resolve to signedId
  session service → Browser: Set-Cookie SESSION_ID={signedId}
  session service → Browser: redirect → /auth/callback?userId={session.userId}
```

---

### AUTH-04 — Google Calendar Account OAuth (Popup Flow)

| Field | Value |
|---|---|
| **ID** | AUTH-04 |
| **Diagram name** | Calendar — Add Google Account OAuth Popup Flow |
| **Type** | Sequence Diagram |
| **Priority** | **Critical** |

**Purpose:** The most complex auth flow in the system. Links a Google Calendar account to an ARM user via a browser popup. Non-obvious because the generic proxy cannot forward 302 redirects server-side, requiring a dedicated session service endpoint.

**Sequence:**
```
User clicks "Add Google Account" in Settings
  Browser: window.open('/session/calendar/add-account', '_blank', 'width=600,height=700')
  Browser: window.addEventListener('message', handler) — listens for GOOGLE_AUTH_SUCCESS

Popup → GET /session/calendar/add-account  (SESSION_ID cookie sent)
  session service (SessionGuard): verify SESSION_ID → load session → get accessToken
  session service → GET http://calendar-be:4001/v1/auth/google/add-account?token=(secret removed)
    (maxRedirects: 0 — capture 302 Location header, don't follow)
  calendar-be: generateAuthUrl(userId) → Google OAuth URL
  calendar-be → session service: HTTP 302 Location: https://accounts.google.com/o/oauth2/auth?...
  session service: extract Location header
  session service → Popup: HTTP 302 → Google OAuth consent screen

User authorizes in Google
Google → calendar-be /v1/auth/google/callback?code=...&state={userId}
  calendar-be: exchange code → { access_token, refresh_token, profile }
  calendar-be: upsert Account in MongoDB { userId, email, displayName, picture, provider: 'google', isAuthenticated: true }
  calendar-be: encrypt tokens (AES-256 GOOGLE_TOKEN_ENCRYPTION_KEY)
  calendar-be: fetchCalendars from Google API → upsert Calendar documents in MongoDB
  calendar-be: schedule webhook watch for each calendar (watchEvents → BullMQ webhook-renewal queue)
  calendar-be → Popup: postMessage({ type: 'GOOGLE_AUTH_SUCCESS', accountId, email }) to CLIENT_REDIRECT_URL origin

Parent window: receives GOOGLE_AUTH_SUCCESS message
  Parent: validate event.origin === expected domain
  Parent: close popup
  Parent: refetch accounts list → re-render LinkedAccountsTab
```

---

### AUTH-05 — Access Token Silent Refresh Flow

| Field | Value |
|---|---|
| **ID** | AUTH-05 |
| **Diagram name** | session service — Access Token Silent Refresh Flow |
| **Type** | Sequence Diagram |
| **Priority** | **Critical** |

**Purpose:** The browser never holds or refreshes JWTs. The session service does this transparently on every proxied request. This diagram explains exactly when and how refresh happens.

**Sequence (inside ProxyService.forward):**
```
Browser → Any authenticated request → session service (SESSION_ID cookie)
  session service: verify SESSION_ID HMAC signature → extract uuid
  session service: Redis GET session:data:{uuid} → { accessToken, refreshToken, userId }
  session service: decode accessToken → check exp

  IF (exp - now) < TOKEN_REFRESH_THRESHOLD_SECONDS (default: 60s):
    session service → POST http://core-be:3000/v1/auth/refresh { refreshToken }
      core-be: validate refreshToken hash → check revoked + expiry
      core-be: generate new accessToken + refreshToken
      core-be: revoke old RefreshToken row → insert new row
      core-be → session service: { accessToken, refreshToken }
    session service: Redis SET session:data:{uuid} { ...session, accessToken: new, refreshToken: new }

  session service: inject Authorization: Bearer {accessToken} header
  session service → upstream backend: forwarded request (stripped: host, cookie, content-length, authorization)
  upstream → session service: response
  session service → Browser: forwarded response (stripped: transfer-encoding, connection)
```

---

## 3. session service — Backend for Frontend

---

### BFF-01 — Session Service Complete Request Architecture

| Field | Value |
|---|---|
| **ID** | BFF-01 |
| **Diagram name** | session service — Complete Request Architecture |
| **Type** | Architecture Block Diagram |
| **Priority** | **Critical** |

**Purpose:** Shows all session service endpoints, their access controls, and which upstream service each calls. Single reference for all session service routes.

**session service endpoint inventory:**

| Endpoint | Auth | Upstream | Notes |
|---|---|---|---|
| `POST /session/login` | Public | core-be /v1/auth/login | Creates SESSION_ID cookie |
| `POST /session/logout` | SessionGuard | core-be /v1/auth/logout | Deletes session + clears cookie |
| `GET /session/me` | SessionGuard | — | Returns userId from session |
| `GET /session/health` | Public | — | Health check |
| `GET /session/calendar/add-account` | SessionGuard | calendar-be (302 capture) | Dedicated OAuth redirect |
| `ALL /session/proxy/:service/*` | SessionGuard | SERVICE_ENV_MAP routing | Generic proxy with token injection |
| `GET /session/oauth/callback?sid=` | Public | Redis handoff lookup | Sets cookie after Google OAuth |
| `POST /session/internal/create-session` | X-Session-Secret | — | Server-to-server from core-be |
| `POST /session/session/activate` | Public | Redis handoff GETDEL | Exchanges handoffId for cookie |

**Callout boxes to add on the diagram:**

*Proxy route resolution (from BFF-02):*
```
Request: /session/proxy/calendar/v1/account?foo=bar
  strip /session/proxy/ → service="calendar", path="/v1/account?foo=bar"
  SERVICE_ENV_MAP["calendar"] → CALENDAR_BE_URL
  Strip headers: host, cookie, authorization, content-length
  Inject: Authorization: Bearer {accessToken}
  KNOWN BUG: query params duplicated — fix: strip ? from path before axios params merge
```

*Redis session schema (from BFF-03):*
```
session:data:{uuid} → JSON { userId, accessToken, refreshToken, userAgent, ipHash, createdAt }  TTL: 30d
session:handoff:{uuid} → string (signedId)  TTL: 30s  (GETDEL — single use)
SESSION_ID cookie: "{uuid}.{(secret removed)(uuid, SESSION_COOKIE_SECRET)}"
```

---

## 4. Calendar Module

---

### CAL-01 — Calendar Module Architecture

| Field | Value |
|---|---|
| **ID** | CAL-01 |
| **Diagram name** | Calendar Module — Internal Architecture |
| **Type** | Architecture Block Diagram |
| **Priority** | **Critical** |

**Purpose:** Shows all NestJS modules within arm-calendar-be, their dependencies, and how they connect to external services and queues.

**Modules & relationships:**
```
AppModule
  ├── AuthModule (AuthGuard — Kong header + JWT fallback)
  ├── AccountModule (AccountService — Account CRUD)
  ├── GoogleModule (GoogleService — OAuth URL generation, Calendar API client)
  ├── CalendarModule (CalendarService — list/watch/stop Google Calendar channels)
  ├── CalendarManagementModule (CalendarManagementService — user-facing management)
  ├── EventsModule
  │   ├── EventsService (performInitialSync, pollCalendar)
  │   ├── SyncProcessor (BullMQ: 'initial-sync' queue)
  │   └── SyncPollProcessor (BullMQ: 'sync-poll' queue)
  ├── WebhookModule
  │   ├── WebhookController (POST /webhook/renew-all)
  │   ├── WebhookRenewalService (scheduleRenewalJobs)
  │   └── WebhookRenewalProcessor (BullMQ: 'webhook-renewal' queue)
  ├── CoreAuthModule (CoreAuthService — CircuitBreaker calls to core-be)
  ├── CryptoModule (TokenEncryptionService — AES-256)
  └── MigrationsModule (StartupMigrationService — run-once on boot)

External:
  MongoDB ← Mongoose (5 collections)
  Redis ← BullMQ (3 queues)
  core-be ← CircuitBreaker HTTP (google token, status, revoke)
  Google Calendar API ← googleapis client
```

---

### CAL-02 — Initial Calendar Sync Flow

| Field | Value |
|---|---|
| **ID** | CAL-02 |
| **Diagram name** | Calendar — Initial Sync Sequence Flow |
| **Type** | Sequence Diagram |
| **Priority** | **Critical** |

**Purpose:** The primary data ingestion path. Triggered when a user enables sync between a source and target calendar. Documents the full BullMQ-backed async pipeline.

**Sequence:**
```
User: enables calendar sync in UI
Browser → POST /session/proxy/calendar/v1/calendar-management/sync
  session service → calendar-be: POST /v1/calendar-management/sync { sourceCalendarId, targetCalendarId, syncFromDate }
    CalendarManagementService:
      create SyncConfig document { sourceCalendarId, targetCalendarId, userId, ... }
      create SyncHistory document { status: 'pending', syncStartTime: now }
      emit CalendarUpdatedEvent → EventsService
    EventsService:
      update SyncHistory: status → 'in-progress'
      BullMQ: add job to 'initial-sync' queue { targetCalendarId, source, syncFromDate, userId }
    calendar-be → Browser: { syncConfig, syncHistory }

--- Background (BullMQ worker) ---

SyncProcessor.process(job):
  EventsService.performInitialSync(targetCalendarId, source, userId):
    GoogleService.getCalendarClient(accountId) → OAuth2 client with token
    Google Calendar API: events.list({ calendarId, timeMin: syncFromDate })
    FOR each Google event:
      create Event document { eventId, calendarId, accountId, summary, start, end, status, ... }
      upsert (eventId + calendarId unique index)
    update SyncHistory: { status: 'completed', eventsSynced: count, syncEndTime: now }
    emit CALENDAR_UPDATED_EVENT → CalendarManagementService
  ON ERROR:
    update SyncHistory: { status: 'failed', errorMessage }
    BullMQ: retry (job.attemptsMade + 1)
```

**Sync status states (add as small callout):**
```
pending → in-progress → completed
                      → partial  (blockersFailed > 0)
                      → failed   (exception, errorMessage set)
failed/completed → pending (re-sync triggered)
```

---

### CAL-03 — Webhook Push Notification Incremental Sync

| Field | Value |
|---|---|
| **ID** | CAL-03 |
| **Diagram name** | Calendar — Webhook Incremental Sync via Google Push Notifications |
| **Type** | Sequence Diagram |
| **Priority** | **Critical** |

**Purpose:** After initial sync, Google sends push notifications on every calendar change. Documents the inbound webhook → delta fetch → MongoDB update path.

**Sequence:**
```
User changes event in Google Calendar
  ↓
Google → POST {NOTIFICATION_WEBHOOK_URL}/v1/events/notifications
  Headers: X-Goog-Channel-Id, X-Goog-Resource-Id, X-Goog-Resource-State

calendar-be EventsController.handleGoogleNotification():
  extract channelId from X-Goog-Channel-Id header
  MongoDB: Calendar.findOne({ channelId }) → get calendar record
  IF resource-state === 'sync': ignore (initial handshake)
  ELSE:
    GoogleService.getCalendarClient(calendar.accountId)
    Google Calendar API: events.list({ syncToken: calendar.syncToken })
      → returns only changed/deleted events since last sync
    FOR each delta event:
      IF event.status === 'cancelled':
        MongoDB: Event.deleteOne({ eventId, calendarId })
      ELSE:
        MongoDB: Event.upsertOne({ eventId, calendarId }) with new data
    update Calendar.syncToken = new nextSyncToken
    update SyncHistory record
```

---

### CAL-05 — Google Webhook Channel Lifecycle

| Field | Value |
|---|---|
| **ID** | CAL-05 |
| **Diagram name** | Calendar — Google Webhook Channel Lifecycle |
| **Type** | State Diagram |
| **Priority** | **Critical** |

**Purpose:** Google webhook channels expire (typically in 7 days). This state diagram documents the full lifecycle from registration through expiry and renewal. Failure to renew silently breaks incremental sync.

**States:**
- `[unregistered]` — calendar just added, no webhook
- `watching` — active channel: { channelId, channelToken, resourceId, expiration } stored in Calendar document
- `renewal_queued` — renewal job added to `webhook-renewal` BullMQ queue before expiry
- `renewing` — WebhookRenewalProcessor running: stop old channel → create new channel
- `expired` — past expiration, no push notifications delivered — **silent failure**
- `unauthenticated` — account.isAuthenticated = false → renewal skipped

**Transitions:**
- Calendar added → register channel (`calendarService.watchEvents()`) → `watching`
- Expiry threshold reached → `renewal_queued` → `renewing` → `watching`
- Account token revoked → `unauthenticated` (renewal processor skips)
- Renewal failure → BullMQ retry → eventually `expired`
- Manual `POST /webhook/renew-all` → `renewal_queued`

---

### CAL-06 — Webhook Renewal Sequence

| Field | Value |
|---|---|
| **ID** | CAL-06 |
| **Diagram name** | Calendar — Webhook Renewal Sequence |
| **Type** | Sequence Diagram |
| **Priority** | **Critical** |

**Purpose:** The renewal sequence in `WebhookRenewalProcessor`. Critical for understanding what happens during renewal and what data changes in MongoDB.

**Sequence:**
```
WebhookRenewalService.scheduleRenewalJobs():
  MongoDB: Calendar.find({ expiration: { $lt: threshold } })
  FOR each expiring calendar:
    BullMQ: add job to 'webhook-renewal' queue { calendarId, calendarMongoId, accountId }

WebhookRenewalProcessor.process(job):
  MongoDB: Account.findOne({ accountId }) → check isAuthenticated
  IF !isAuthenticated: log warn, RETURN (skip)

  MongoDB: Calendar.findById(calendarMongoId)
  IF has existing channelId + resourceId:
    CalendarService.stopChannel(accountId, channelId, resourceId)
      → Google Calendar API: channels.stop({ id, resourceId })
      (best-effort — log warn on failure, continue)

  newChannelId = uuidv4()
  newChannelToken = (secret removed)(32).toString('hex')
  CalendarService.watchEvents(accountId, calendarId, newChannelId, webhookUrl, channelToken)
    → Google Calendar API: events.watch({
        id: newChannelId,
        type: 'web_hook',
        address: NOTIFICATION_WEBHOOK_URL,
        token: (secret removed)
      })
    → returns { resourceId, expiration }

  MongoDB: Calendar.updateOne({ _id: calendarMongoId }, {
    channelId: newChannelId,
    channelToken: newChannelToken,
    resourceId: watchResponse.resourceId,
    expiration: watchResponse.expiration
  })
  log: "Webhook renewed for calendar {calendarId}"
```

---

### CAL-09 — BullMQ Queue Architecture

| Field | Value |
|---|---|
| **ID** | CAL-09 |
| **Diagram name** | Calendar — BullMQ Queue Architecture |
| **Type** | Architecture Diagram |
| **Priority** | **Important** |

**Purpose:** Shows all 3 BullMQ queues, their producers, processors, job payloads, and retry strategy. All queues share a single Redis instance.

**Queues:**

| Queue name | Constant | Producer | Processor | Job payload |
|---|---|---|---|---|
| `initial-sync` | `SYNC_QUEUE` | EventsService | SyncProcessor | `{ targetCalendarId, source, syncFromDate, userId }` |
| `sync-poll` | `SYNC_POLL_QUEUE` | CalendarManagementService | SyncPollProcessor | `{ sourceCalendarId }` |
| `webhook-renewal` | `WEBHOOK_RENEWAL_QUEUE` | WebhookRenewalService | WebhookRenewalProcessor | `{ calendarId, calendarMongoId, accountId }` |

**Redis key pattern:** BullMQ uses `bull:{queueName}:*` internally.

---

## 5. Notifications & Messaging

---

### NOTIF-01 — Kafka Architecture & Topic Map

| Field | Value |
|---|---|
| **ID** | NOTIF-01 |
| **Diagram name** | Kafka — Topic Architecture, Consumer Groups & Email Dispatch |
| **Type** | Architecture Diagram |
| **Priority** | **Critical** |

**Purpose:** Complete Kafka topology — all topics, producers, consumer groups, DLQ, and the full email dispatch sequence.

**Topics:**

| Topic | Producer | Consumer | Payload | Email type |
|---|---|---|---|---|
| `send_email` | core-be NotificationProducerService | notification-service | `{ userId, email, otp, type }` | Verification OTP |
| `send_welcome_email` | core-be NotificationProducerService | notification-service | `{ userId, email, name, type }` | Welcome email |
| `send_password_reset_email` | core-be NotificationProducerService | notification-service | `{ userId, email, otp, type }` | Password reset OTP |
| `notification.dlq` | notification-service DlqService | — (manual drain) | `{ originalTopic, originalMessage, error, failedAt }` | Dead letters |

**Consumer group:** `notification-group-{env}` (e.g. `notification-group-dev`)

**Kafka config:** KRaft mode (Kafka 3.8) — no Zookeeper. Single broker. `CLUSTER_ID: arm-kafka-cluster-dev-001`.

**Email dispatch sequence (add as flow annotation on the diagram):**
```
core-be AuthService.signup():
  NotificationProducerService.sendVerificationEmail(userId, email, otp):
    kafkaClient.emit('send_email', { userId, email, otp, type: 'verify-email' })

Kafka → notification-service NotificationController.handleSendOTPEmail(message)
  try:
    IF message.type === 'reset-password':
      NotificationService.sendPasswordResetEmail(message)
    ELSE:
      NotificationService.sendNotification(message):
        MailService.sendVerificationEmail(email, { otp, year })
        → Nodemailer: SMTP transport → send email
  catch(error):
    DlqService.send('send_email', message, error)
      kafkaClient.emit('notification.dlq', { originalTopic, originalMessage, error, failedAt })
```

---

## 6. Frontend & Module Federation

---

### FE-01 — Module Federation Runtime Architecture

| Field | Value |
|---|---|
| **ID** | FE-01 |
| **Diagram name** | Frontend — Module Federation Runtime Architecture |
| **Type** | Architecture Block Diagram |
| **Priority** | **Critical** |

**Purpose:** How the browser loads the shell and dynamically fetches and instantiates the calendar remote. Critical for understanding the CSS isolation requirement and shared dependency singleton management.

**Architecture:**
```
Browser loads http://localhost:3000 (arm-core-fe)
  ← Vite dev server (dev) / Nginx (prod)

arm-core-fe (Shell / Host):
  vite.config.ts federation config:
    name: "core"
    exposes: { "./toast": ..., "./theme-store": ... }
    remotes:
      calendar: {
        type: "esm",
        entry: VITE_CALENDAR_REMOTE_URL  (e.g. http://localhost:3001/remoteEntry.js)
      }
    shared (singletons): react, react-dom, react-router-dom, react-toastify, zustand

  On route /calendar/* (lazy import):
    fetch http://localhost:3001/calendar/remoteEntry.js  ← calendar-fe Vite dev server
    @module-federation/vite runtime negotiates shared deps
    instantiate calendar remote in shell's context

arm-calendar-fe (Remote):
  vite.config.ts federation config:
    name: "calendar"
    base: "/calendar/"        ← remoteEntry.js is served UNDER this base
    exposes:
      "./Page-1"       → calendar-account-page.tsx   (/calendar/syncs)
      "./SyncDetail"   → sync-detail-page.tsx        (/calendar/syncs/new, /:id)
      "./MergedView"   → calendar-merged-page.tsx    (/calendar/view)
      "./SyncView"     → sync-view-page.tsx          (/calendar/view/:targetCalendarId)
      "./AuthCallback" → auth-callback.tsx           (/calendar/account, /auth/callback)
    remotes: {}  ← no back-reference to core (avoids circular dep)
    shared: same singletons as core (negotiated — only one instance in browser)

CSS isolation:
  No wrapper div. The old .calendar-scope requirement existed only for
  FullCalendar's global resets; FullCalendar is gone (grid is now the design
  system's BigCalendarView / react-big-calendar). Do not draw the wrapper.
  What the diagram SHOULD show: calendar-styles.css and index.css load only on
  the standalone path (App.tsx / main.tsx), never when federated.
```

---

### FE-04 — MFE Remote Registration (Registry System)

| Field | Value |
|---|---|
| **ID** | FE-04 |
| **Diagram name** | Frontend — MFE Registry & Remote URL Resolution |
| **Type** | Sequence Diagram |
| **Priority** | **Important** |

**Purpose:** The Registry in core-be stores the URL of each MFE remote per environment. This enables different remotes to be served locally vs. on the server without changing the shell's Vite config.

**Flow:**
```
arm-core-fe startup:
  fetch /api/core/v1/registry → [{ name, type, url, remoteUrl }]
  registry entry: { name: 'calendar', type: 'mfe', remoteUrl: 'https://dev.arametrics.app/calendar/remoteEntry.js' }
  store remote URLs → configure MF runtime

core-be RegistryService:
  PostgreSQL registry table:
    { id, name (unique), type: 'backend'|'mfe', url?, remoteUrl?, registeredAt, updatedAt }

  GET /v1/registry?type=mfe → all MFE remotes
  POST /v1/registry → register/update a service
```

---

## 7. Database & Storage Architecture

---

### DB-01 — PostgreSQL Entity Relationship Diagram

| Field | Value |
|---|---|
| **ID** | DB-01 |
| **Diagram name** | PostgreSQL — Entity Relationship Diagram (arm-core-be) |
| **Type** | ERD |
| **Priority** | **Critical** |

**Purpose:** Complete schema for the relational database used by arm-core-be.

**Entities:**

```
users
  id: uuid PK
  email: varchar UNIQUE
  firstName: varchar
  lastName: varchar
  password: (secret removed) (nullable, not selected by default)
  isActive: boolean DEFAULT true
  isVerified: boolean DEFAULT false
  googleId: varchar (nullable)
  authProvider: varchar DEFAULT 'local'
  picture: varchar (nullable)
  createdAt: timestamp
  updatedAt: timestamp

refresh_tokens
  id: uuid PK
  tokenHash: varchar INDEX (not selected by default)
  userAgent: varchar
  ip: varchar
  expiresAt: timestamp
  revoked: boolean DEFAULT false
  userId: uuid FK → users.id (CASCADE DELETE)
  createdAt: timestamp

google_tokens (GoogleToken entity)
  userId: uuid FK → users.id
  accessToken: text (AES-256 encrypted)
  refreshToken: text (AES-256 encrypted, nullable)
  expiresAt: timestamp
  scope: varchar

registry
  id: uuid PK
  name: varchar UNIQUE
  type: enum('backend', 'mfe')
  url: varchar (nullable)
  remoteUrl: varchar (nullable)
  registeredAt: timestamp
  updatedAt: timestamp
```

**Migrations (TypeORM):**
- `(secret removed)` — users, refresh_tokens, google_tokens, registry
- `(secret removed)`
- `1747300000000-AddPictureToUsers`

---

### DB-02 — MongoDB Collection Relationships

| Field | Value |
|---|---|
| **ID** | DB-02 |
| **Diagram name** | MongoDB — Collection Relationships (arm-calendar-be) |
| **Type** | Document Relationship Diagram |
| **Priority** | **Critical** |

**Purpose:** All 5 MongoDB collections with fields, indexes, and logical relationships via userId and accountId foreign keys (not enforced by MongoDB — application-level).

**Collections:**

```
Account { timestamps }
  userId: string INDEX           ← links to PostgreSQL users.id
  accountId: string UNIQUE INDEX ← Google account identifier
  email: string INDEX
  displayName: string
  organization: string
  picture: string
  accessToken: string (AES-256 encrypted)
  refreshToken: string (AES-256 encrypted)
  provider: string DEFAULT 'google'
  related: string DEFAULT 'calendar'
  tokenExpiryAt: Date
  syncDate: Date
  onSync: boolean DEFAULT true
  isAuthenticated: boolean DEFAULT true

Calendar { timestamps }
  userId: string INDEX
  accountId: string INDEX        ← → Account.accountId
  calendarId: string INDEX       ← Google Calendar ID
  summary, description, timeZone, accessRole: string
  channelId: string INDEX        ← Google push notification channel ID
  channelToken: string           ← verification token for webhook
  resourceId: string             ← Google resource ID
  expiration: string             ← Unix ms timestamp string
  etag, syncToken: string
  webhookSupported: boolean DEFAULT true
  INDEXES: { channelId, resourceId }, { accountId, calendarId } UNIQUE

Event { timestamps }
  eventId: string INDEX          ← Google Event ID
  calendarId: string INDEX       ← → Calendar.calendarId
  accountId: string INDEX        ← → Account.accountId
  uniqueId, kind, etag: string
  summary, description: string
  start, end: Object { dateTime, timeZone }
  status: string ('confirmed'|'tentative'|'cancelled')
  recurrence: string[]
  extendedProperties: { private: { sourceCalendarId: string } }
  INDEX: { eventId, calendarId } UNIQUE

SyncConfig { timestamps }
  sourceCalendarId: string INDEX ← → Calendar.calendarId (source)
  targetCalendarId: string INDEX ← → Calendar.calendarId (target)
  targetAccountMongoId: string INDEX
  sourceAccountMongoId: string INDEX
  syncFromDate: Date
  userId: string INDEX
  INDEX: { sourceCalendarId, targetCalendarId } UNIQUE

SyncHistory { timestamps }
  sourceCalendarId, targetCalendarId: string INDEX
  targetAccountMongoId, sourceAccountMongoId: string INDEX
  syncFromDate: Date
  syncStatus: enum('pending','in-progress','completed','partial','failed')
  syncStartTime, syncEndTime: Date
  eventsSynced: number
  errorMessage: string
  blockersAttempted, blockersFailed: number DEFAULT 0
  failureReasons: string[]
  initiatedBy: string
```

---

## 8. Infrastructure & Networking

---

### INFRA-01 — Docker Compose Stack Architecture

| Field | Value |
|---|---|
| **ID** | INFRA-01 |
| **Diagram name** | Infrastructure — Docker Compose Stack Architecture |
| **Type** | Infrastructure Architecture Diagram |
| **Priority** | **Critical** |

**Purpose:** Shows all Docker Compose files, what they start, and how they relate. Explains why infra is a separate compose file.

**Compose files:**

| File | Purpose | Services | Network |
|---|---|---|---|
| `docker-compose.infra.yml` | Infrastructure only (persistent) | PostgreSQL, MongoDB, Redis, Kafka, Verdaccio | `arm-infra-network` |
| `docker-compose.infra.prod.yml` | Production infra overrides | Extended infra config | `arm-infra-network` |
| `docker-compose.infra.redis-sentinel.yml` | Redis HA (Sentinel) | Redis + 2 Sentinels | `arm-infra-network` |
| `docker-compose.dev.yml` | Dev app stack (built images) | Caddy, Kong, all services, frontends | `arm-dev-network` + `arm-infra-network` |
| `docker-compose.local.dev.yml` | Local dev (hot reload, no Caddy/Kong) | All services with src/ volume mounts | `arm-local-network` + `arm-infra-network` |
| `docker-compose.yml` | Production stack | All services + Kong | — |

**Key rule:** Infra is never stopped by `make reset-apps` or `clean-apps` — only `make clean` touches infra.

---

### INFRA-02 — Dev vs. Local Dev vs. Production Network Topology

| Field | Value |
|---|---|
| **ID** | INFRA-02 |
| **Diagram name** | Infrastructure — Network Topology: Dev vs Local-Dev vs Production |
| **Type** | Network Topology Diagram |
| **Priority** | **Critical** |

**Purpose:** Side-by-side comparison of how traffic flows in each environment. Shows why auth behavior differs (Kong active in prod, bypassed in dev).

**Dev (docker-compose.dev.yml):**
```
Browser → Caddy:443 (dev.arametrics.app)
  /session/* → arm-session-dev:5000 (direct, bypasses Kong)
  /api/core/* → arm-core-backend-dev:3000 (direct, bypasses Kong)
  /api/calendar/* → arm-calendar-backend-dev:4001 (direct, bypasses Kong)
  /calendar/remoteEntry.js → arm-calendar-frontend-dev:80
  /* → arm-core-frontend-dev:80
Kong running on :8000 but NOT in request path (Caddy routes around it)
```

**Local Dev (docker-compose.local.dev.yml):**
```
Browser → http://localhost:3000 → arm-core-fe-local:3000 (Vite dev server)
Browser → http://localhost:3001 → arm-calendar-fe-local:3001 (Vite dev server)
Browser → http://localhost:5000 → arm-session-local:5000
No Caddy. No Kong.
session service internal: http://arm-core-be-local:3000, http://arm-calendar-be-local:4001
```

**Production (docker-compose.yml):**
```
Browser → Kong:8000 (public)
  /api/core/* → validates JWT → injects X-User-Id → core-backend:3000
  /api/calendar/* → validates JWT → injects X-User-Id → calendar-backend:4001
  /session/* → arm-session:5000 (no JWT validation — session service handles it)
```

---

### INFRA-03 — Kong API Gateway Configuration

| Field | Value |
|---|---|
| **ID** | INFRA-03 |
| **Diagram name** | Kong API Gateway — Route & Plugin Configuration |
| **Type** | Configuration Architecture Diagram |
| **Priority** | **Critical** |

**Purpose:** Documents every route, plugin, and the Lua `pre-function` that injects `X-User-Id`. Important for security review and for anyone adding new routes.

**Routes (from kong.yml):**

| Route name | Path | Strip path | Auth | Plugins |
|---|---|---|---|---|
| `core-api-route` | `/api/core` | yes | JWT plugin | jwt, pre-function (X-User-Id inject), rate-limiting (120/min, 3000/hr) |
| `calendar-api-route` | `/api/calendar` | yes | JWT plugin | jwt, pre-function (X-User-Id inject), rate-limiting |
| `calendar-auth-public-route` | `/api/calendar/v1/auth` | no | None | pre-function (path rewrite: strip /api/calendar/v1) |
| `calendar-webhook-route` | `/api/calendar/events/notifications` | no | None | pre-function (rewrite to /v1/events/notifications) |

**X-User-Id injection (Lua):**
```lua
local auth = (secret removed)
-- decode JWT payload (base64url) → extract payload.sub
-- kong.service.request.set_header("X-User-Id", payload.sub)
```

---

## 9. Deployment & CI/CD

---

### CICD-01 — GitHub Actions CI/CD Pipeline Architecture

| Field | Value |
|---|---|
| **ID** | CICD-01 |
| **Diagram name** | CI/CD — GitHub Actions Pipeline Architecture |
| **Type** | Pipeline Architecture Diagram |
| **Priority** | **Critical** |

**Purpose:** Shows all 3 deploy workflows, their triggers, and the full chain from git push to running containers on the target server.

**Workflows:**

| Workflow | Trigger | Target | Environment |
|---|---|---|---|
| `deploy-dev.yml` | push to `dev` branch | dev server | GitHub env: `development` |
| `deploy-production.yml` | push to `main` branch | prod server | GitHub env: `production` |

**All 3 follow the same pattern:**
```
1. actions/checkout@v4   ← checkout arm-deploy-make repo

2. Setup SSH
   → write DEV/STAGE/PROD_SSH_PRIVATE_KEY to ~/.ssh/id_rsa
   → ssh-keyscan target host

3. rsync -az --delete
   --exclude=core --exclude=apps --exclude=services --exclude=library --exclude=deploy
   ./ → server:~/arm-deploy/
   (copies: Makefile, make/*.mk, docker-compose*.yml, kong/, verdaccio/, .env.example)

4. Write .env on server (SSH)
   → printf each secret/variable from GitHub Secrets/Variables
   → ~/arm-deploy/.env

5. make ci-deploy-prod ROOT=~/arm-deploy
   → on server: env-check → check-docker → clone-repos → infra-up → infra-wait → publish-lib
   → docker compose down → build --no-cache → up -d
```

---

### CICD-03 — Environment Promotion Path

| Field | Value |
|---|---|
| **ID** | CICD-03 |
| **Diagram name** | CI/CD — Branch-to-Environment Promotion Path |
| **Type** | Pipeline Flow Diagram |
| **Priority** | **Important** |

**Purpose:** Shows the git branch strategy and how code moves from development to production.

```
Feature branches
      ↓  (PR merge)
    dev branch ──────────────→ push ──→ deploy-dev.yml ──→ Development server
      ↓  (PR merge)
      ↓  (PR merge)
   main branch ──────────────→ push ──→ deploy-production.yml ──→ Production server

Each merge trigger:
  GitHub Actions → rsync deploy files → SSH → make ci-deploy-{env}
```

---

## 10. Developer Workflows

---

### DEV-01 — Local Development Setup Flow

| Field | Value |
|---|---|
| **ID** | DEV-01 |
| **Diagram name** | Developer Workflow — Local Setup from Zero |
| **Type** | Flowchart |
| **Priority** | **Critical** |

**Purpose:** The exact sequence a new engineer follows to get from a fresh machine to a running development environment. Decision points and error paths included.

```
START
  ↓
Prerequisites met? (Node 20, pnpm, Docker, make)
  NO → install prerequisites
  YES ↓
cp .env.example .env → fill in all REQUIRED_VARS
  ↓
make env-check → all green?
  NO → fix missing vars → retry
  YES ↓
make infra-up → start PostgreSQL, MongoDB, Redis, Kafka, Verdaccio
  ↓
make infra-wait → all healthy?
  NO → check docker logs → fix issue → retry
  YES ↓
make publish-lib → @aracreate/test-arm-ui in Verdaccio + .npmrc written
  ↓
CHOOSE MODE:
  ┌── Hot Reload Docker: make local-dev-build → make local-dev
  └── Native pnpm: make install → 6 terminals with make dev-{service}
  ↓
make smoke-test → all green?
  NO → check logs with make local-dev-logs-be or local-dev-logs-fe
  YES ↓
http://localhost:3000 → ARM platform running ✓
```

---

## 11. Covered in Documentation — Not Miro Frames

These were in the original diagram list but are better expressed as written documentation. The content for each is in this file and should be moved to the indicated document.

| ID | Name | Where it belongs |
|---|---|---|
| ARCH-05 | Technology Stack Map | A table in `ARCHITECTURE.md` |
| AUTH-06 | Forgot Password / OTP Reset Flow | A section in `AUTH-ARCHITECTURE.md` |
| AUTH-07 | session service Session Lifecycle State Diagram | A section in `SESSION-FLOW.md` |
| AUTH-08 | Auth Guard Decision Flow | A section in `AUTH-ARCHITECTURE.md` |
| BFF-02 | Proxy Route Resolution Logic | Incorporated as a callout in BFF-01 above |
| BFF-03 | Redis Session Key Schema | Incorporated as a callout in BFF-01 above |
| CAL-04 | Polling Sync Flow | A footnote in `SYNC-FLOW.md` (fallback path, same pattern as CAL-02) |
| CAL-07 | Sync Status State Machine | Incorporated as a callout in CAL-02 above |
| CAL-08 | Circuit Breaker State Machine | A paragraph in `apps/arm-app-calendar/ARCHITECTURE.md` |
| NOTIF-02 | Email Notification Sequence | Incorporated as a flow annotation in NOTIF-01 above |
| FE-02 | Core-fe Route Map | A table in `core/arm-core-fe/README.md` (changes too often for Miro) |
| FE-03 | Zustand Store Architecture | A section in `core/arm-core-fe/README.md` |
| DB-03 | Redis Key Namespace Map | A table in `REDIS-ARCHITECTURE.md` |
| INFRA-04 | Redis Sentinel HA Architecture | A section in `INFRASTRUCTURE.md` |
| INFRA-05 | Verdaccio Internal Package Registry | A section in `VERDACCIO-SETUP.md` |
| CICD-02 | Server Deploy Sequence (make ci-deploy) | A numbered list in `CICD.md` |
| CICD-04 | Makefile Module Dependency Graph | A section in `CICD.md` |
| DEV-03 | Code Contribution & PR Review Workflow | A section in `ONBOARDING.md` |
| MON-01 | Service Health Check Architecture | A table in `RUNBOOKS/restart-services.md` |
| MON-02 | Log Architecture | A section in `MONITORING.md` |

---

## 12. Deferred — Build When Relevant

Build these only when the corresponding work begins. Creating them now adds maintenance overhead with no current value.

| ID | Name | Build when |
|---|---|---|
| FE-05 | UnoCSS Cross-MFE Scanning | A new MFE remote is added and scanning issues recur |
| DEV-02 | Hot Reload Architecture | Dev tooling changes significantly |
| FUTURE-01 | Planned MFE Remote Expansion | A second app (Timer, Projects, etc.) is in active development |
| FUTURE-02 | Multi-Provider Calendar (Outlook, Apple) | Outlook or Apple provider work begins |
| FUTURE-03 | Kubernetes Migration Architecture | arm-deploy-k8s is actively used |
| FUTURE-04 | Centralized Logging Stack | Loki / Grafana / Prometheus are deployed |
| FUTURE-05 | SSE for Real-Time Sync Status | SSE implementation starts |
| FUTURE-06 | Electron Desktop App Architecture | Electron packaging begins |
| FUTURE-07 | Kong Token Handler Plugin | Token handler pattern is being adopted |
| FUTURE-08 | Redis Cluster Architecture | Redis Cluster migration is planned |

---

## 13. Master Checklist

> Use this checklist to track Miro board completion. 27 frames total.

### Section 1 — System Architecture

- [ ] **ARCH-01** — Platform High-Level Architecture `Critical`
- [ ] **ARCH-02** — Service Topology & Port Map `Critical`
- [ ] **ARCH-03** — Network Boundary & Trust Model `Critical`
- [ ] **ARCH-04** — Multi-Repository Workspace Structure `Important`

### Section 2 — Authentication & Session

- [ ] (secret removed) — Email/Password Login Flow `Critical`
- [ ] **AUTH-02** — Email Signup 4-Step Flow `Critical`
- [ ] (secret removed) — Google OAuth Login via core-be `Critical`
- [ ] **AUTH-04** — Google Calendar Account OAuth Popup Flow `Critical`
- [ ] (secret removed) — Access Token Silent Refresh Flow `Critical`

### Section 3 — session service

- [ ] **BFF-01** — Session Service Complete Request Architecture `Critical`

### Section 4 — Calendar Module

- [ ] **CAL-01** — Calendar Module Internal Architecture `Critical`
- [ ] **CAL-02** — Initial Calendar Sync Sequence `Critical`
- [ ] **CAL-03** — Webhook Incremental Sync Sequence `Critical`
- [ ] **CAL-05** — Google Webhook Channel Lifecycle `Critical`
- [ ] **CAL-06** — Webhook Renewal Sequence `Critical`
- [ ] **CAL-09** — BullMQ Queue Architecture `Important`

### Section 5 — Notifications & Messaging

- [ ] **NOTIF-01** — Kafka Topic Architecture & Consumer Groups `Critical`

### Section 6 — Frontend & Module Federation

- [ ] **FE-01** — Module Federation Runtime Architecture `Critical`
- [ ] **FE-04** — MFE Registry & Remote URL Resolution `Important`

### Section 7 — Database & Storage

- [ ] **DB-01** — PostgreSQL ERD `Critical`
- [ ] **DB-02** — MongoDB Collection Relationships `Critical`

### Section 8 — Infrastructure & Networking

- [ ] **INFRA-01** — Docker Compose Stack Architecture `Critical`
- [ ] **INFRA-02** — Network Topology: Dev vs Local-Dev vs Production `Critical`
- [ ] **INFRA-03** — Kong Gateway Route & Plugin Configuration `Critical`

### Section 9 — Deployment & CI/CD

- [ ] **CICD-01** — GitHub Actions Pipeline Architecture `Critical`
- [ ] **CICD-03** — Branch-to-Environment Promotion Path `Important`

### Section 10 — Developer Workflows

- [ ] **DEV-01** — Local Development Setup Flow `Critical`

---

## Priority Summary

| Priority | Count |
|---|---|
| Critical | 23 |
| Important | 4 |
| **Total Miro frames** | **27** |

| Category | Count |
|---|---|
| Miro frames (this document) | 27 |
| Covered in documentation (Section 11) | 20 |
| Deferred (Section 12) | 10 |
| **Original total** | **57** |

---

## Recommended Miro Board Frame Layout

```
Row 1 (Pinned, always visible):
  ARCH-01 (large), ARCH-02, ARCH-03, ARCH-04

Row 2 — Authentication:
  AUTH-01 → AUTH-02 → AUTH-03 → AUTH-04 → AUTH-05

Row 3 — session service + Calendar:
  BFF-01
  CAL-01, CAL-02, CAL-03, CAL-05, CAL-06, CAL-09

Row 4 — Notifications + Frontend:
  NOTIF-01
  FE-01, FE-04

Row 5 — Data + Infrastructure:
  DB-01, DB-02
  INFRA-01, INFRA-02, INFRA-03

Row 6 — CI/CD + Developer Workflows:
  CICD-01, CICD-03
  DEV-01
```

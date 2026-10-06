# Phase 5: Feature Development

> Build the two new capabilities: Microsoft Graph webhook subscriptions (real-time sync) and Hetzner Object Storage (media management). Both depend on the stabilization and security work from Phases 1-4.

**Priority:** P3 — After platform is stable and secure
**Effort:** ~16 hours total
**Prerequisite:** Phase 3 (infra tuning) + Phase 4 (sync engine hardening)
**Source:** Microsoft Graph Webhooks Design Spec + Object Storage Design Spec

---

## Why These Depend on Earlier Phases

**Microsoft Webhooks:**
- Requires C-07 and C-08 fixed (Phase 1) — Microsoft token refresh must work
- Requires H-24 fixed (Phase 4) — needs exponential backoff for Graph API
- Requires H-19/H-20 fixed (Phase 4) — orphan sweep must work with new provider
- Requires infra right-sized (Phase 3) — webhook processing adds load

**Object Storage:**
- Requires body size limits (Phase 2, T-2.16) — upload endpoints need appropriate limits
- Requires infra stable (Phase 3) — adding upload processing to core-be increases memory usage
- Requires Postgres tuned (Phase 3) — migration reads all user rows

---

## Part A: Microsoft Graph Webhook Subscriptions

**Detailed plan:** `apps/arm-app-calendar/docs/superpowers/plans/2026-08-21-microsoft-graph-webhooks.md`
**Design spec:** `apps/arm-app-calendar/docs/superpowers/specs/2026-08-21-microsoft-graph-webhooks-design.md`

### T-5.1: MicrosoftCalendarApiService — Real Graph Subscription API

**File:** `apps/arm-app-calendar/src/backend/src/calendar/microsoft-calendar-api.service.ts`
**Change:** Replace `watchEvents()` and `stopChannel()` ForbiddenException stubs with real Graph API calls. Add `renewSubscription()`.

- [ ] Implement `watchEvents()` — POST to `graph.microsoft.com/v1.0/subscriptions`
- [ ] Implement `stopChannel()` — DELETE to `graph.microsoft.com/v1.0/subscriptions/{id}`
- [ ] Implement `renewSubscription()` — PATCH to `graph.microsoft.com/v1.0/subscriptions/{id}`
- [ ] Write tests for all three methods (success + error cases)
- [ ] Run: `npx jest microsoft-calendar-api.service.spec --no-coverage`

**Effort:** 2 hours

---

### T-5.2: MicrosoftCalendarWebhookProvider

**Files:**
- Create: `src/backend/src/providers/microsoft/microsoft-calendar-webhook.provider.ts`
- Modify: `src/backend/src/calendar/calendar.module.ts`

- [ ] Create provider implementing `CalendarWebhookProvider` interface
- [ ] Implement `parseNotification(headers, body)` — detect Microsoft format (JSON `value` array)
- [ ] Implement `createSubscription()`, `stopSubscription()`, `renewSubscription()` — delegate to API service
- [ ] Register and export in `CalendarModule`
- [ ] Write tests
- [ ] Run: `npx jest microsoft-calendar-webhook.provider.spec --no-coverage`

**Effort:** 1.5 hours

---

### T-5.3: Validation Handshake Endpoint

**File:** `apps/arm-app-calendar/src/backend/src/events/events.controller.ts`
**Change:** Add `@Get('notifications')` handler for Microsoft subscription validation.

- [ ] Add GET handler that echoes `validationToken` query param as plain text
- [ ] Set `Content-Type: text/plain` response header
- [ ] Verify: no interference with existing POST notifications handler
- [ ] Add test: GET with validationToken returns 200 + echoed token

**Effort:** 30 minutes

---

### T-5.4: Provider-Aware Notification Dispatch

**File:** `apps/arm-app-calendar/src/backend/src/events/webhook-processor.service.ts`
**Change:** Add provider detection at the top of `handleWebhook()`. Google uses `x-goog-*` headers, Microsoft sends JSON body with `value` array.

- [ ] Update `handleWebhook()` signature to accept optional `body` parameter
- [ ] Add provider detection: Google headers? -> Google flow. Body has `value` array? -> Microsoft flow
- [ ] Extract shared delta-fetch logic into `processSourceCalendarDelta()` (called by both paths)
- [ ] Inject `MicrosoftCalendarWebhookProvider` in constructor
- [ ] Add timing-safe `clientState` verification for Microsoft notifications
- [ ] Update `webhook-processing.processor.ts` to pass body alongside headers
- [ ] Run existing Google tests to verify no regression
- [ ] Add tests for Microsoft notification processing

**Effort:** 2 hours

---

### T-5.5: Subscription Renewal — Microsoft PATCH Branch

**File:** `apps/arm-app-calendar/src/backend/src/webhook/webhook-renewal.processor.ts`
**Change:** Add Microsoft branch that PATCHes subscription instead of stop+create.

- [ ] Load account, check `provider` field
- [ ] If `microsoft`: call `renewSubscription()` with new expiration (3 days)
- [ ] Update Calendar document with new expiration
- [ ] If `google`: existing stop+create flow (unchanged)
- [ ] Write test: Microsoft renewal uses PATCH, not stop+create

**Effort:** 1 hour

---

### T-5.6: Microsoft Webhooks Integration Test

**File:** `src/backend/src/events/microsoft-webhook-integration.spec.ts` (new)

- [ ] Test: validation handshake echoes token as plain text
- [ ] Test: POST with Microsoft `value` array is accepted and enqueued
- [ ] Test: POST with Google `x-goog-*` headers still works (regression check)
- [ ] Run full test suite to verify no regressions

**Effort:** 1 hour

---

## Part B: Object Storage (Hetzner S3)

**Design spec:** `docs/arm-docs/docs/architecture/object-storage-design.md`

### T-5.7: Hetzner Bucket Setup (Manual, One-Time)

**No code change — server-side manual step.**

- [ ] Create Object Storage bucket `arm-media` in Hetzner Cloud Console (fsn1)
- [ ] Set bucket ACL to public-read
- [ ] Generate S3 credentials (access key + secret key)
- [ ] Add `(secret removed)`, `S3_REGION`, `S3_BUCKET`, `(secret removed)`, `(secret removed)` to `.env` on server
- [ ] Add same vars to `.env.example` in deploy repo

**Effort:** 15 minutes

---

### T-5.8: Core-be Media Module — S3 Client + Upload Service

**Files:**
- Create: `core/arm-core-be/src/media/media.module.ts`
- Create: `core/arm-core-be/src/media/media.service.ts`
- Create: `core/arm-core-be/src/media/media.controller.ts`
- Create: `core/arm-core-be/src/media/dto/upload-response.dto.ts`

- [ ] Install dependencies: `@aws-sdk/client-s3`, `sharp`
- [ ] Create `MediaService` with S3 client configured for Hetzner (path-style, fsn1 region)
- [ ] Implement upload logic per type:
  - **Avatar:** JPEG/PNG/WebP, max 2MB, resize 256x256 center-crop JPEG 85%, key: `profiles/{userId}/avatar.jpg`
  - **Org logo:** JPEG/PNG/WebP/SVG, max 1MB, no resize, key: `orgs/{orgId}/logo.{ext}`
  - **General:** JPEG/PNG/WebP/GIF/PDF, max 10MB, no resize, key: `media/{userId}/{uuid}.{ext}`
- [ ] Validate MIME type AND magic bytes (not just Content-Type header)
- [ ] Create `MediaController` with endpoints:
  - `POST /v1/media/upload/avatar` (KongJwtGuard)
  - `POST /v1/media/upload/org-logo` (KongJwtGuard)
  - `POST /v1/media/upload` (KongJwtGuard)
  - `DELETE /v1/media/:key` (KongJwtGuard)
- [ ] Register `MediaModule` in AppModule
- [ ] Write tests for upload validation, S3 client mock, URL generation

**Effort:** 3 hours

---

### T-5.9: Update User Profile Picture Flow

**Files:**
- Modify: `core/arm-core-be/src/users/dto/update-user.dto.ts` — remove `picture` from writable fields
- Modify: `core/arm-core-be/src/media/media.service.ts` — avatar upload updates `users.picture`

- [ ] Remove `picture` from `UpdateUserDto` (no longer set via PATCH)
- [ ] In avatar upload: after S3 PutObject, update `users.picture` with the public URL
- [ ] If user had a previous Object Storage avatar, delete the old object
- [ ] Replace `IsAvatarDataUrl` validator with HTTPS URL validator
- [ ] Write test: avatar upload returns public URL and updates user record

**Effort:** 1 hour

---

### T-5.10: OAuth Provider Picture Download

**File:** `core/arm-core-be/src/auth/auth.service.ts` (Google/Microsoft login methods)
**Change:** After OAuth login, download provider avatar, resize, upload to S3, store our URL.

- [ ] After successful OAuth login, if provider returns a picture URL:
  1. Download the image to a buffer
  2. Resize to 256x256 JPEG 85% via sharp
  3. Upload to `profiles/{userId}/avatar.jpg`
  4. Store Object Storage URL in `users.picture`
- [ ] Handle download failure gracefully (log, skip — don't block login)
- [ ] Add test: OAuth login with provider picture stores S3 URL

**Effort:** 1 hour

---

### T-5.11: Frontend — Replace Base64 Avatar Upload

**File:** `core/arm-core-fe/src/` — settings/profile page
**Change:** Replace canvas downscale + base64 + JSON body with FormData + POST to media endpoint.

- [ ] Replace the current upload flow: `file -> FormData -> POST /session/proxy/core/v1/media/upload/avatar`
- [ ] Remove `downscaleToAvatar()` function (server handles resizing)
- [ ] Image preview: use `URL.createObjectURL(file)` (already works, no base64)
- [ ] On success: update local user state with the returned `url`
- [ ] Test in browser: upload a profile picture, verify it appears

**Effort:** 1 hour

---

### T-5.12: Database Migration — Base64 to Object Storage URLs

**File:** New TypeORM migration in `core/arm-core-be/src/migrations/`
**Change:** Migrate existing base64 profile pictures to Object Storage.

- [ ] Query all `users` where `picture LIKE 'data:%'`
- [ ] For each: decode base64, resize 256x256 JPEG via sharp, upload to S3, update row
- [ ] Set `picture = NULL` for any row that fails (log the failure)
- [ ] This is a one-way migration
- [ ] Test: run migration against a test database with sample base64 images
- [ ] Verify: all profile pictures serve from Object Storage URLs after migration

**Effort:** 1 hour

---

## Medium Bugs Addressed in This Phase

| Bug ID | Issue | Task |
|---|---|---|
| M-09 | No base64 string length check before decode (core-be avatar) | T-5.9 (validator replacement eliminates this) |
| M-22 | syncFromCoreAuth sets onSync: false inconsistently | T-5.4 (addressed during delta logic refactor) |

---

## Completion Checklist

### Microsoft Webhooks
- [ ] Tasks T-5.1 through T-5.6 complete
- [ ] Microsoft calendar sync gets real-time notifications (not just 5-min poll)
- [ ] Google webhook flow unchanged (no regressions)
- [ ] Subscription renewal works via PATCH (3-day cycle)
- [ ] Validation handshake passes Microsoft's verification
- [ ] All test suites pass

### Object Storage
- [ ] Tasks T-5.7 through T-5.12 complete
- [ ] Bucket created and credentials configured
- [ ] Avatar upload works end-to-end: browser -> session service -> core-be -> S3
- [ ] OAuth provider pictures downloaded and stored in S3
- [ ] Frontend uses FormData upload (no more base64)
- [ ] Migration converts existing base64 pictures
- [ ] All profile pictures served from Hetzner Object Storage

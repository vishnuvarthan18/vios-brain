# API Routes Reference

Base URL: `http://localhost:3001` (dev) | `https://appapi.dreamspace.academy` (prod)

All authenticated routes require `Authorization: Bearer <token>` header.
All responses: `Content-Type: application/json`
All errors: `{ error: string, code: string, details?: object }`

---

## Auth

```
POST /auth/otp/send              → send SMS OTP to parent phone
POST /auth/otp/verify            → verify OTP, return JWT
POST /auth/terms/accept          → accept T&C version
GET  /auth/first-login-status    → { completed: bool, steps: string[] }
GET  /auth/me                    → current user + roles + permissions
POST /auth/refresh               → refresh JWT (parent OTP auth)
```

---

## Platform (SUPER_ADMIN)

```
GET  /platform/sections          → list sections + status
PUT  /platform/sections/:key     → update section status
GET  /platform/settings          → platform-wide settings
PUT  /platform/settings          → update settings
GET  /platform/integrations      → integration health (SMS, FCM, staff API)
GET  /platform/audit-logs        → paginated audit log (filterable)
POST /platform/impersonate/:id   → start read-only impersonation session (30 min)
GET  /platform/reports/impact    → cross-section impact report
```

---

## Users

```
GET  /users                      → list users (paginated, filterable by role/hub/status)
POST /users                      → create user + send Auth0 invite
GET  /users/:id                  → user detail + roles
PUT  /users/:id                  → update profile
POST /users/:id/deactivate       → soft deactivate
POST /users/:id/reactivate       → reactivate
POST /users/:id/merge            → merge two accounts (SUPER_ADMIN)
POST /users/:id/resend-invite    → resend Auth0 invite email

GET  /users/:id/roles            → list roles (with scope)
POST /users/:id/roles            → assign role
DELETE /users/:id/roles/:roleId  → revoke role (soft)

GET  /users/:id/permissions      → effective permissions
POST /users/:id/permission-overrides           → create GRANT/DENY override (SUPER_ADMIN)
DELETE /users/:id/permission-overrides/:ovId   → revoke override
```

---

## Hubs

```
GET  /hubs                       → list all hubs (paginated)
POST /hubs                       → create hub
GET  /hubs/:hubId                → hub detail
PUT  /hubs/:hubId                → update hub
POST /hubs/:hubId/deactivate     → deactivate (warns if active students)
GET  /hubs/:hubId/staff          → list coordinators + trainers
GET  /hubs/:hubId/stats          → student count, session count, payment summary
PUT  /hubs/:hubId/handover-note  → update handover note
```

---

## Levels & Outcomes

```
GET  /hubs/:hubId/levels                    → list levels at hub (ordered)
POST /hubs/:hubId/levels                    → create level (MS_ADMIN, SUPER_ADMIN)
PUT  /hubs/:hubId/levels/:levelId           → update level
POST /hubs/:hubId/levels/:levelId/reorder   → drag-reorder levels
DELETE /hubs/:hubId/levels/:levelId         → deactivate level (blocked if students enrolled)

GET  /levels/:levelId/outcomes              → list outcomes (ordered)
POST /levels/:levelId/outcomes              → add outcome
PUT  /levels/:levelId/outcomes/:outcomeId   → update outcome
POST /levels/:levelId/outcomes/reorder      → reorder
DELETE /levels/:levelId/outcomes/:outcomeId → deactivate (blocked if students have records)
```

---

## Students (all hub-scoped)

```
GET  /hubs/:hubId/students                  → list students (DSA ID + level + status, paginated)
POST /hubs/:hubId/students                  → enroll student
GET  /hubs/:hubId/students/:studentId       → student detail (coordinator+: full; trainer: limited)
PUT  /hubs/:hubId/students/:studentId       → update student info (coordinator+)
POST /hubs/:hubId/students/:studentId/deactivate     → soft deactivate
POST /hubs/:hubId/students/:studentId/reactivate     → reactivate (choose: resume level or restart)
GET  /hubs/:hubId/students/search?q=DSA-   → search by DSA ID (prefix)
GET  /hubs/:hubId/students/:studentId/attendance-report → summary stats
POST /hubs/:hubId/students/:studentId/transfer        → initiate hub transfer (admin approval needed)

GET  /hubs/:hubId/students/:studentId/outcomes        → list all outcomes + status
PUT  /hubs/:hubId/students/:studentId/outcomes/:outcomeId → update status
POST /hubs/:hubId/students/:studentId/outcomes/bulk-complete → bulk complete for a session

POST /hubs/:hubId/students/:studentId/promote         → promote to next level (coordinator+)

GET  /hubs/:hubId/students/:studentId/notes           → list notes
POST /hubs/:hubId/students/:studentId/notes           → create note
PUT  /hubs/:hubId/students/:studentId/notes/:noteId   → update note visibility
POST /hubs/:hubId/students/:studentId/notes/:noteId/flag → flag for coordinator attention
```

---

## Timetable & Slots

```
GET  /hubs/:hubId/timetable                → list timetable slots
POST /hubs/:hubId/timetable                → create slot
PUT  /hubs/:hubId/timetable/:slotId        → update slot
DELETE /hubs/:hubId/timetable/:slotId      → deactivate slot

GET  /hubs/:hubId/timetable/:slotId/students  → students assigned to slot
POST /hubs/:hubId/students/:studentId/assign-slot → assign student to slot
DELETE /hubs/:hubId/students/:studentId/slots/:slotId → remove from slot
```

---

## Sessions (hub-scoped)

```
GET  /hubs/:hubId/sessions                 → list sessions (date range filter)
GET  /hubs/:hubId/sessions/today           → today's sessions for this hub
GET  /hubs/:hubId/sessions/:sessionId      → session detail
PUT  /hubs/:hubId/sessions/:sessionId      → update topic/notes/cover trainer
POST /hubs/:hubId/sessions/:sessionId/cancel → cancel session (notify parents)
POST /hubs/:hubId/sessions/one-off         → create one-off makeup session

GET  /hubs/:hubId/sessions/:sessionId/students → student list for this session (slot + overrides)
POST /hubs/:hubId/sessions/:sessionId/overrides → add/remove student from this session only
```

---

## Attendance

```
GET  /hubs/:hubId/sessions/:sessionId/attendance  → current attendance records
POST /hubs/:hubId/sessions/:sessionId/attendance  → submit attendance (TRAINER, HUB_COORDINATOR)
                                                    body: { records: [{ studentId, status, note, markedAt }] }
                                                    Uses UPSERT — safe for offline re-sync
PUT  /hubs/:hubId/sessions/:sessionId/attendance/:recordId → coordinator override with reason
```

---

## Payments

```
GET  /hubs/:hubId/payments                 → list payments (month filter, status filter)
POST /hubs/:hubId/payments/:paymentId/mark-paid → mark as paid (method, reference)
POST /hubs/:hubId/payments/:paymentId/waive     → waive with reason
POST /hubs/:hubId/payments/:paymentId/reverse   → reverse to UNPAID (MS_ADMIN+, reason required)
GET  /hubs/:hubId/payments/report          → expected vs collected report
GET  /hubs/:hubId/payments/unpaid          → students with unpaid fees + months overdue count
GET  /payments/:paymentId/receipt          → get receipt PDF URL

POST /hubs/:hubId/students/:studentId/sponsorship → assign sponsorship
DELETE /sponsorships/:sponsorshipId        → end sponsorship (sets endDate)
```

---

## Portfolios (Projects)

```
GET  /hubs/:hubId/projects                 → list projects (status filter)
POST /hubs/:hubId/projects                 → create project (TRAINER+)
GET  /hubs/:hubId/projects/:projectId      → project detail
PUT  /hubs/:hubId/projects/:projectId      → update project (trainer who created, or coordinator+)
POST /hubs/:hubId/projects/:projectId/submit   → submit for coordinator review
POST /hubs/:hubId/projects/:projectId/approve  → approve and publish (HUB_COORDINATOR+)
POST /hubs/:hubId/projects/:projectId/reject   → reject with feedback note
POST /hubs/:hubId/projects/:projectId/media    → upload media files
DELETE /hubs/:hubId/projects/:projectId/media/:mediaId → delete media

POST /hubs/:hubId/projects/:projectId/link-outcomes → link to learning outcomes (updates student_outcomes)
```

---

## Certificates

```
GET  /hubs/:hubId/certificates             → list (filter by status)
GET  /certificates/pending-approval        → all PENDING_APPROVAL (MS_ADMIN+)
POST /certificates/:certId/approve         → admin approves
POST /certificates/:certId/issue           → coordinator issues (generates PDF + QR)
POST /certificates/:certId/revoke          → revoke with reason

GET  /verify/:verifyCode                   → public endpoint (no auth) — certificate verify data
                                             (served by public Next.js app, not this API)
```

---

## Open Sessions

```
GET  /open-sessions                        → list (status, category, tag filters)
POST /open-sessions                        → create (EVENTS_ADMIN, MS_ADMIN, SUPER_ADMIN)
GET  /open-sessions/:sessionId             → session detail
PUT  /open-sessions/:sessionId             → update
POST /open-sessions/:sessionId/publish     → publish
POST /open-sessions/:sessionId/cancel      → cancel + notify registrants

GET  /open-sessions/:sessionId/registrations        → list registrants
POST /open-sessions/:sessionId/register             → register (authenticated or walk-in)
DELETE /open-sessions/:sessionId/registrations/:id  → cancel registration
POST /open-sessions/:sessionId/registrations/:id/mark-paid → mark session fee paid
POST /open-sessions/:sessionId/attendance           → submit attendance

GET  /session-categories                   → list categories
POST /session-categories                   → create (SUPER_ADMIN)
GET  /session-tags                         → list tags
POST /session-tags                         → create (SUPER_ADMIN)
```

---

## Notifications

```
GET  /notifications                        → list (paginated, unread first)
GET  /notifications/unread-count           → { count: number }
POST /notifications/:id/read               → mark read
POST /notifications/read-all              → mark all read

GET  /notifications/preferences            → get user preferences
PUT  /notifications/preferences            → update channels + category/tag prefs

POST /device-tokens                        → register FCM device token
DELETE /device-tokens/:token               → unregister (on logout)
```

---

## Announcements

```
POST /hubs/:hubId/announcements            → broadcast to hub parents (max 3/day, coordinator+)
GET  /hubs/:hubId/announcements            → list recent announcements
```

---

## Reporting (MS_ADMIN + SUPER_ADMIN)

```
GET  /reports/students                     → cross-hub enrollment stats
GET  /reports/attendance                   → attendance rates by hub/level
GET  /reports/payments                     → revenue summary across hubs
GET  /reports/impact                       → demographics for grant reporting
                                             (gender, district, school type, income)
GET  /reports/sponsor/:donorId             → progress report for a donor's students
```

---

## Donors & Sponsorship

```
GET  /donors                               → list donors
POST /donors                               → create donor
GET  /donors/:donorId                      → donor detail + sponsored students
POST /donors/:donorId/funding-pools        → create funding pool
GET  /donors/:donorId/report              → impact report for this donor
```

---

## External resources (guest facilitators)

```
GET  /external-resources                   → list
POST /external-resources                   → create (MS_ADMIN+)
PUT  /external-resources/:id               → update
```

---

## PDPA / Data compliance

```
GET  /consent-records/:subjectId           → list consent records
POST /data-requests                        → submit access/erasure/correction request
GET  /data-requests                        → list requests (SUPER_ADMIN)
PUT  /data-requests/:id                    → handle request (complete/reject)
```

---

## Response shapes

### Pagination
```json
{
  "data": [...],
  "meta": {
    "total": 48,
    "page": 1,
    "perPage": 20,
    "totalPages": 3
  }
}
```

### Error
```json
{
  "error": "Human-readable message",
  "code": "MACHINE_READABLE_CODE",
  "details": {}
}
```

### Common error codes
```
UNAUTHORIZED          → 401 — no/invalid token
FORBIDDEN             → 403 — valid token, insufficient permission
NOT_FOUND             → 404
VALIDATION_ERROR      → 400 — Zod validation failed (details = field errors)
CONFLICT              → 409 — duplicate (e.g. student already in slot)
RATE_LIMITED          → 429
INTERNAL_ERROR        → 500
HUB_SCOPE_VIOLATION   → 403 — trying to access another hub's data
SOFT_DELETED          → 410 — resource exists but is deactivated
```

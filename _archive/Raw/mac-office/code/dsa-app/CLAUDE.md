# DreamSpace Academy — Claude Code Context

This file is the single source of truth for Claude Code. Read it fully before writing any code.

---

## What This Project Is

DreamSpace Academy is a platform for a Sri Lankan social enterprise running makerspace hubs for underserved youth in Hatton and Batticaloa (Tamil-speaking communities). Phase 1 builds the Makerspace module only. The platform will expand to other sections (Academy, Events, Labs, Competitions, Volunteers, Certificates, Newsletter, Opportunities) in future phases.

**Key facts:**
- Students are children (minors). Data privacy is critical.
- Sri Lanka PDPA (2022, amended 2025) applies — full compliance required.
- Two hubs in Phase 1: Hatton and Batticaloa (both Tamil-speaking).
- The platform connects to an existing internal DreamSpace staff platform via API.
- Social enterprise: some students are fully sponsored by donors.
- Scale target: many hubs across Sri Lanka, eventually.

**Domains:**
- `dreamspace.academy` — primary domain. **An existing marketing website already lives at the root — the platform does NOT own it.**
- `app.dreamspace.academy` — the web app (the main platform client).
- `appapi.dreamspace.academy` — the API (CORS allows the app origin).
- **Public, no-auth pages** (certificate verification, portfolio showcase, public events/projects, open-session registration) are served **under the main app domain (`app.dreamspace.academy`)** — we do **NOT** use separate `verify.` / `showcase.` subdomains. The base URLs remain config (`PUBLIC_VERIFY_BASE_URL`, `PUBLIC_SHOWCASE_BASE_URL`) — never hardcode them (the verify base is persisted into cert records) — but they point at the app domain (a path, not a dedicated subdomain).

**Client strategy (revised):**
- The **web app is the superset** — every capability lives here first, for every role (admins, coordinators, trainers, parents). It is built as an **installable PWA** from the start (`vite-plugin-pwa`; manifest + service worker; app shell precached, API never cached).
- The **mobile app is a later slice** derived from this same React codebase (Capacitor) — it takes whatever subset of web features makes sense on device. Build for web first; carve out mobile from it.

---

## Architecture — Non-Negotiable Decisions

### Monorepo structure
```
dreamspace/
├── api/          → Express + Prisma backend (Dokploy)
├── web/          → React + Vite web app (Dokploy) — coordinators + admins
├── mobile/       → Capacitor app (iOS + Android) — trainers + parents ONLY
└── public/       → Tiny Next.js app (Dokploy) — portfolio showcase + cert verify
```

### Tech Stack (final, no changes)
| Concern | Choice |
|---|---|
| Backend | Node.js + Express + TypeScript |
| ORM | Prisma |
| Primary DB | PostgreSQL via Supabase |
| Rich text body | PostgreSQL `Json` column (Project/Event/OpenSession `body`) — MongoDB removed |
| Auth (staff + parents) | Firebase Auth (email/password + phone OTP) |
| Auth (parents) | Phone OTP via notify.lk → custom JWT |
| SMS | notify.lk (Sri Lanka) |
| Push notifications | Firebase Cloud Messaging (FCM) |
| Real-time in-app | Supabase Realtime (WebSocket) |
| Image storage | Cloudinary (profile pics, event + project photos) |
| Hosting (api + web + public) | Dokploy |
| Mobile | Capacitor (iOS + Android from one React codebase) |
| State (web) | Zustand |
| State (mobile) | Zustand + Capacitor Preferences (offline queue) |

### What goes in each app
- **Web app (PWA — the superset)**: ALL roles, ALL functions — Super Admin, MS Admin, Hub Coordinators, Trainers, Parents. Installable. Everything is built here first; nothing is mobile-only.
- **Mobile app (Capacitor — a derived slice)**: a subset of the web codebase shipped to device when a feature benefits from native (offline-first attendance, push). Built FROM web, not in parallel with it.
- **Public Next.js**: Certificate verification + Portfolio showcase + public events/projects. No auth. Served **under the main app domain** (`app.dreamspace.academy`), NOT on separate subdomains. The root domain (`dreamspace.academy`) is the existing marketing site.

---

## Running Locally

pnpm workspace. **No root "run all" script — start each app in its own terminal.**
Full guide (mock auth, seeding, SQL hardening, seeded users): `docs/LOCAL-DEV.md`.

```bash
pnpm install                 # once, from repo root
cd api  && pnpm dev          # API      → http://localhost:4000  (tsx watch)
cd web  && pnpm dev          # Web PWA  → http://localhost:5173  (vite)
cd web  && pnpm dev:mobile   # Mobile   → http://localhost:5174  (Capacitor shell, web/ subset)
cd public && pnpm dev        # Public   → http://localhost:3000  (cert verify + showcase)
```

First-time DB setup (from `api/`): `pnpm prisma db push` then `pnpm db:seed`.
Local auth: (secret removed) — API trusts the `x-dev-user` header (seeded email/ID); never use in production.

---

## Permission Architecture — Critical

### Central permission matrix (`api/src/permissions.ts`)
NEVER write inline role checks like `if (user.role === 'TRAINER')` in route files.
Every route calls `can('action:name')` middleware.
Adding a new role = adding one line to permissions.ts. Zero route files change.

### Three-layer permission resolution (in order)
1. **DENY override** → block, always wins
2. **GRANT override** → allow (explicit Super Admin grant)
3. **Role baseline** → check permissions.ts

### Scope enforcement
Every hub-scoped route checks: does the user's role have `hubId` matching the requested hub?
Hub-scoped roles: HUB_COORDINATOR, TRAINER.
Platform-wide roles: SUPER_ADMIN, MS_ADMIN.

### Role types (all defined, only Phase 1 ones have logic built)
**Phase 1 active:** SUPER_ADMIN, MS_ADMIN, HUB_COORDINATOR, TRAINER, PARENT, VOLUNTEER, MEMBER
**Enum only (logic Phase 2+):** ACADEMY_ADMIN, ACADEMY_INSTRUCTOR, ACADEMY_MENTOR, VOLUNTEER_ADMIN, OPPORTUNITIES_ADMIN, EVENTS_ADMIN, EVENT_COORDINATOR, COMPETITIONS_ADMIN, COMPETITION_JUDGE, LABS_ADMIN, LAB_COORDINATOR, LAB_TECHNICIAN, COMMS_ADMIN, CERTS_ADMIN, SUBSIDIARY_ADMIN, REGIONAL_MANAGER, DONOR

---

## Database — Key Rules

### Global sequences (run ONCE after first prisma migrate)
```sql
CREATE SEQUENCE student_id_seq START 47312;
CREATE SEQUENCE receipt_id_seq START 47312;
CREATE SEQUENCE cert_id_seq    START 47312;
```

### Student DSA ID format
`DSA-XXXXXX` — 6-digit zero-padded global sequence. No hub code, no year.
Example: `DSA-047313`, `DSA-047314`
The number is opaque — reveals nothing about hub, year, or enrollment count.

### Student identity — privacy rules
**Students are identified by DSA ID only across the entire operational UI.**
- No student names shown in trainer views, attendance lists, or payment lists. (Still enforced — `student:view_pii`, which unlocks student names, is NOT granted to trainers.)
- Names only appear in the enrollment record (coordinator/admin, consent-gated).
- No student photos anywhere in Phase 1. Initials avatars replaced by robot avatars.
- Robot avatar is generated deterministically from DSA ID using FNV-1a hash — no DB field.
- **Guardian / parent linking (policy update):** ANY role with `enrollment:edit` (incl. trainers) MAY search for a PARENT-role account and link/manage a student's **guardian contact info** and **parent login accounts**. Parent search uses the scoped `GET /users/linkable-parents` endpoint (gated on `enrollment:edit`, returns PARENT-role users only) — NOT the full `user:view` directory. This exposes *parent/guardian* contact details — **not** student names (which still require `student:view_pii`). A "guardian" is contact data only (`StudentGuardian` table, no login); a "parent account" is a `User` with the PARENT role linked via `StudentParent`. The two are managed separately on the student page.

### Audit log
`audit_logs` table is INSERT-only at PostgreSQL level.
After first migration, run:
```sql
REVOKE DELETE, UPDATE ON audit_logs FROM [your_db_user];
```

### Schema file location
`api/prisma/schema.prisma` — the master schema. Never modify without understanding all downstream effects.

---

## DSA Robot Avatar System

Every student gets a deterministic robot avatar generated from their DSA ID.
The same ID always produces the same robot. No storage needed.

```typescript
// api/src/utils/avatar.ts (shared logic, used in web + mobile)
export function generateAvatar(dsaId: string): AvatarConfig {
  const h = fnv1a(dsaId)
  return {
    palette:    PALETTES[h % PALETTES.length],
    antenna:    ANTENNAS[(h>>4) % ANTENNAS.length],
    headShape:  HEAD_SHAPES[(h>>8) % HEAD_SHAPES.length],
    eyeType:    EYE_TYPES[(h>>12) % EYE_TYPES.length],
    mouthType:  MOUTH_TYPES[(h>>16) % MOUTH_TYPES.length],
    bodyDetail: BODY_DETAILS[(h>>20) % BODY_DETAILS.length],
    earType:    EARS[(h>>24) % EARS.length],
  }
}
```

The robot avatar component lives in `web/src/components/ui/DsaAvatar.tsx` and is a pure SVG generator. Import and use anywhere with just the DSA ID.

---

## Brand & Design System

**Primary colour:** Violet `#7008C0` (8.45:1 contrast on white — safe everywhere)
**Accent colour:** Orange `#F86800` (3.01:1 — use as fill only, white text on it, NOT as text colour)
**Orange text:** Use `#A14300` (Orange 700) for orange-coloured text on light backgrounds — passes 6.33:1

**Fonts:**
- `Outfit` — headings and display (weights 600–800)
- `DM Sans` — body and UI
- `Space Grotesk` — DSA IDs, codes, numbers (monospace feel)

All design tokens in `web/src/tokens.css`.

---

## Critical Business Rules

### Payments
- Admin sets hub default monthly fee. Coordinator overrides per student at enrollment (`student.agreedMonthlyFee`).
- Monthly payment records auto-created by cron (1st of month, 00:01 Asia/Colombo).
- Payment cron is idempotent: `@@unique([studentId, month])` prevents duplicates.
- Sponsored students have `PaymentStatus.SPONSORED` — not UNPAID.
- Every PAID payment generates a receipt: `RCP-XXXXXX` (global sequence).

### Sessions and attendance
- Students are assigned to a specific timetable SLOT at enrollment (not all slots for their level).
- Attendance list = students in that slot + any overrides for that specific session.
- Attendance is LOCKED after trainer submits — corrections require coordinator override with reason.
- Offline attendance queue persists to Capacitor Preferences (device storage), not memory.
- On sync, use UPSERT not INSERT — prevents duplicates from double-sync.
- Session instances generated by cron: every Monday 6am, 2 weeks ahead.
- Public holidays table checked before generating — skip those dates.

### Portfolios
- Trainer creates → submits → coordinator reviews → approves → goes public.
- Rich text body (Tiptap JSON) stored in the Postgres `Project.body` `Json` column. Media (cover image) is uploaded to **Cloudinary** via `POST /uploads/image`.
- Media = photos of WORK only. No student photos ever in Phase 1.

### Certificates
- System auto-flags when student completes all outcomes in final level.
- Admin approves → Coordinator issues → QR-verifiable certificate.
- Verify URL: `DSA-CERT-XXXXXX` under the **app domain** (served by the public Next.js app on `app.dreamspace.academy` — NOT a `verify.` subdomain; base is `PUBLIC_VERIFY_BASE_URL`).

### Staff platform integration
- On first login, pull staff profile from existing platform API using `externalStaffId`.
- After session attendance submitted: auto-push time log to staff platform.
- Time log push is async — trainer does not wait for it.
- Duplicate check before pushing: `session.timeLogPushed` flag + check staff platform.
- Retry 3 times. After 3 failures: `timeLogStatus = PUSH_FAILED_PERMANENT`.

### PDPA compliance
- Collect consent at student enrollment: `ConsentRecord` table.
- Demographic fields (gender, district, income bracket) are sensitive — admin-only, own consent purpose.
- Right to erasure: anonymise PII, preserve audit trail.
- Data requests tracked in `DataRequest` table.

---

## Notification Channels

**In-app** — always, via Supabase Realtime. Cannot be disabled.
**Push** — FCM, via Capacitor on mobile. Device tokens in `device_tokens` table.
**SMS** — notify.lk. Tamil SMS for Hatton + Batticaloa hubs by default.
**Email** — for rich content (new sessions, role grants).

Rate limiting: max 1 announcement SMS per user per day. Transactional SMS (payment, cancellation) are immediate and never batched.

---

## Environment Variables

See `api/.env.example` and `web/.env.example` for all required variables.

Never commit `.env` files. All secrets via environment variables.

---

## Code Style Rules

1. **TypeScript strict mode** everywhere. No `any`.
2. **Async/await** — no raw promises or callbacks.
3. **Prisma transactions** for any multi-table write.
4. **Never hardcode role checks** in routes — always use `can()` middleware.
5. **Every route has error handling** — use the central error handler in `api/src/middleware/error.ts`.
6. **All writes to audit_logs** for sensitive actions (payment, role change, permission override).
7. **Hub scope check on every hub-scoped route** — use `inHub()` middleware.
8. **Zod validation** on all request bodies — validate before processing.
9. **No hard deletes** anywhere — always soft delete (`deletedAt`, `deactivatedAt`, `revokedAt`).
10. **SMS failures are non-fatal** — log and surface as a flag, never throw.

---

## Cron Jobs (api/src/jobs/)

| Job | Schedule | What it does |
|---|---|---|
| `sessionInstances.ts` | Mon 06:00 Asia/Colombo | Generate session instances 2 weeks ahead |
| `monthlyPayments.ts` | 1st of month 00:01 Asia/Colombo | Create UNPAID payment records for all active students |
| `expireRoles.ts` | Daily 00:00 | Set `revokedAt` on expired temporary roles |
| `pushFailedRetry.ts` | Every 30 min | Retry PUSH_FAILED time logs (max 3 attempts) |

All crons are idempotent — safe to run twice.

---

## Sprints

See `docs/SPRINT_1.md` for detailed Sprint 1 breakdown.

**Sprint sequence:**
- Sprint 1: Foundation — auth, users, roles, hubs, permissions, permission overrides
- Sprint 2: Students — enrollment, slot assignment, demographics, consent, T&C
- Sprint 3: Sessions + Timetable — cron, cover trainer, public holidays, offline sync
- Sprint 4: Attendance + Outcomes — mark, lock, override, proof of work
- Sprint 5: Payments + Receipts — cron, receipt gen, sponsorship
- Sprint 6: Portfolios + Certificates — approval flow, QR, Postgres `Json` body
- Sprint 7: Notifications — in-app Supabase Realtime, SMS, push, preferences
- Sprint 8: Open Sessions — tags, categories, pricing tiers, registration
- Sprint 9: Reporting + Admin — impact reports, audit log UI, permission override UI
- Sprint 10: Polish — offline, error states, handover, first-time flows, export

---

## File Naming Conventions

- Route files: `kebab-case.ts` → `open-sessions.ts`
- Components: `PascalCase.tsx` → `DsaAvatar.tsx`
- Hooks: `camelCase.ts` → `usePermissions.ts`
- Utils: `camelCase.ts` → `studentCode.ts`
- Constants: `UPPER_SNAKE` inside files

---

## What NOT to do

> **Before committing or opening a PR**, follow [`docs/git-workflow.md`](docs/git-workflow.md) (branching, Conventional Commits, the Claude Code git rules, and the human-verification-before-PR gate) and run the relevant sections of [`docs/KNOWN-ISSUES-CHECKLIST.md`](docs/KNOWN-ISSUES-CHECKLIST.md) — a verification list distilled from real bugs on the sibling KathiraGreens and Viyanix projects (atomic writes, integer money, IDOR/hub scope, cron timezone+idempotency, PDPA, no hardcoded domains). Many items below are *confirmed* by that checklist, not just asserted here.

- Do NOT add `photoUrl` to students table — child safety policy, Phase 1 has no student photos.
- Do NOT show student names in trainer/attendance views — DSA ID only. (Trainers MAY see/link parent & guardian contact info — that's an allowed exception; student names remain hidden behind `student:view_pii`.)
- **Trainer timetable authority (policy):** Trainers hold `timetable:manage`, `slot:manage`, `session:create/edit/cancel` (+ `session_delete:run`), so they manage their hub's timetable end-to-end — weekly slots, holidays, generating slots, and naming/moving/cancelling sessions on the calendar. Only student-name PII (`student:view_pii`) and platform admin actions remain gated away from trainers.
- Do NOT write inline `if (role === ...)` checks in routes.
- Do NOT create hard deletes — everything is soft.
- Do NOT store offline queue in memory — use Capacitor Preferences.
- Do NOT skip the audit log for sensitive actions.
- Do NOT send SMS without checking the user's `smsEnabled` preference first.
- Do NOT assume hub default language is English — Hatton and Batticaloa default to Tamil (`"ta"`).
- Do NOT generate student codes at application level — use PostgreSQL sequence.

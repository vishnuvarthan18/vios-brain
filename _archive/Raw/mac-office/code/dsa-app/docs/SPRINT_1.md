# Sprint 1 — Foundation

**Goal:** Working auth, hub CRUD, user management, role system, and permission enforcement.
After Sprint 1, a Super Admin can log in, create a hub, add a coordinator, and assign roles.
No student data yet. No sessions. Just the foundation everything else builds on.

---

## Backend tasks (api/)

### 1.1 Project scaffold
- [ ] Init Express + TypeScript project with strict mode
- [ ] Install dependencies: express, prisma, @prisma/client, zod, jsonwebtoken, jwks-rsa, express-rate-limit, node-cron, @supabase/supabase-js, axios, cors, helmet, morgan
- [ ] Set up `tsconfig.json` with strict: true, path aliases
- [ ] Set up `nodemon` + `ts-node` for dev
- [ ] Create `src/app.ts` (Express setup) and `src/index.ts` (server start)
- [ ] Central error handler middleware `src/middleware/error.ts`
- [ ] Request logger with morgan
- [ ] Helmet for security headers
- [ ] CORS configured for web + mobile origins

### 1.2 Prisma + database
- [ ] Copy `prisma/schema.prisma` from master schema
- [ ] Create `prisma/sequences.sql`:
  ```sql
  CREATE SEQUENCE IF NOT EXISTS student_id_seq START 47312;
  CREATE SEQUENCE IF NOT EXISTS receipt_id_seq START 47312;
  CREATE SEQUENCE IF NOT EXISTS cert_id_seq    START 47312;
  ```
- [ ] Create `prisma/seed.ts`:
  - Seed `platform_sections` (9 sections, all HIDDEN except makerspace = ACTIVE)
  - Seed `terms_versions` (v1.0)
  - Create default Super Admin user
- [ ] Run `prisma migrate dev --name init`
- [ ] Run sequences.sql
- [ ] Run seed
- [ ] Restrict audit_logs:
  ```sql
  REVOKE DELETE, UPDATE ON audit_logs FROM [db_user];
  ```
- [ ] Create `src/lib/prisma.ts` (singleton client)

### 1.3 Auth — Auth0 (staff)
- [ ] `src/middleware/auth.ts` — verify Auth0 JWT using jwks-rsa
- [ ] Extract user from JWT, attach `req.user` with id, roles, permissions
- [ ] Load user's active roles from DB on each request (cache in Redis later)
- [ ] Load user's active permission overrides from DB
- [ ] Handle expired/invalid tokens with clear 401 response

### 1.4 Auth — Phone OTP (parents)
- [ ] `POST /auth/otp/send` — send OTP via notify.lk to parent phone
- [ ] `POST /auth/otp/verify` — verify OTP, issue custom JWT
- [ ] OTP rate limit: max 3 attempts per phone per 10 minutes
- [ ] OTP expiry: 10 minutes
- [ ] Store OTP in Supabase (temp table or Redis)

### 1.5 Permission system
- [ ] `src/permissions.ts` — central permission matrix (all Phase 1 permissions)
- [ ] `src/middleware/can.ts` — three-layer check: DENY override → GRANT override → role baseline
- [ ] `src/middleware/inHub.ts` — verify user's role hubId matches route :hubId param
- [ ] `src/middleware/inScope.ts` — generic scope checker (future: lab, event, etc.)
- [ ] Permission override DB queries (with in-memory cache, 5-min TTL)

### 1.6 User routes
- [ ] `GET  /users/me` — current user profile + roles + permissions
- [ ] `GET  /users` — list users (MS_ADMIN + SUPER_ADMIN, paginated, filterable)
- [ ] `GET  /users/:id` — user detail
- [ ] `POST /users` — create user + send Auth0 invite (SUPER_ADMIN + MS_ADMIN)
- [ ] `PUT  /users/:id` — update user profile
- [ ] `POST /users/:id/deactivate` — soft deactivate (audit logged)
- [ ] `POST /users/:id/merge` — merge two accounts (SUPER_ADMIN only)

### 1.7 Role management routes
- [ ] `GET  /users/:id/roles` — list user's roles
- [ ] `POST /users/:id/roles` — assign role (with scope)
- [ ] `DELETE /users/:id/roles/:roleId` — revoke role (soft)
- [ ] `GET  /users/:id/permissions` — effective permissions for a user
- [ ] `POST /users/:id/permission-overrides` — create override (SUPER_ADMIN)
- [ ] `DELETE /users/:id/permission-overrides/:overrideId` — revoke override

### 1.8 Hub routes
- [ ] `GET  /hubs` — list all hubs (paginated)
- [ ] `GET  /hubs/:hubId` — hub detail
- [ ] `POST /hubs` — create hub (MS_ADMIN + SUPER_ADMIN)
- [ ] `PUT  /hubs/:hubId` — update hub
- [ ] `POST /hubs/:hubId/deactivate` — deactivate hub (warn if active students)
- [ ] `GET  /hubs/:hubId/staff` — list coordinators + trainers at hub

### 1.9 First-time login
- [ ] `GET /auth/first-login-status` — returns whether user has completed first-time setup
- [ ] `POST /auth/terms/accept` — record T&C acceptance
- [ ] `POST /auth/profile/complete` — complete profile setup

### 1.10 Platform sections
- [ ] `GET  /platform/sections` — list sections + status (filtered by visibility)
- [ ] `PUT  /platform/sections/:key` — update section status (SUPER_ADMIN)

### 1.11 Audit log
- [ ] `src/lib/audit.ts` — `logAction(actorId, action, entityType, entityId, oldVal, newVal, reason)` helper
- [ ] Call on: role assign, role revoke, hub create/edit, user create/deactivate, permission override

### 1.12 Rate limiting
- [ ] Global: 100 requests/IP/minute
- [ ] OTP endpoint: 3 requests/phone/10min
- [ ] Auth endpoints: 5 failures/email/15min

---

## Frontend tasks (web/)

### 1.1 Project scaffold
- [ ] Init React + Vite + TypeScript (strict)
- [ ] Install: react-router-dom, zustand, @tanstack/react-query, axios, zod
- [ ] Install: tailwindcss (or CSS modules — decide before starting)
- [ ] Copy `src/tokens.css` (design tokens — brand colours, fonts)
- [ ] Set up path aliases `@/` → `src/`
- [ ] Set up Google Fonts: Outfit, DM Sans, Space Grotesk

### 1.2 Design system base components
- [ ] `src/components/ui/Button.tsx` — primary (violet), secondary, danger, ghost variants
- [ ] `src/components/ui/Badge.tsx` — status badges (paid, due, sponsored, active, etc.)
- [ ] `src/components/ui/Input.tsx` — text input with label + error state
- [ ] `src/components/ui/Select.tsx`
- [ ] `src/components/ui/Card.tsx`
- [ ] `src/components/ui/Modal.tsx`
- [ ] `src/components/ui/DsaAvatar.tsx` — robot avatar SVG generator (from DSA ID)
- [ ] `src/components/ui/EmptyState.tsx` — consistent empty + error states
- [ ] `src/components/ui/Spinner.tsx`

### 1.3 Layout
- [ ] `src/components/layout/Sidebar.tsx` — dark ink sidebar, permission-driven nav
- [ ] `src/components/layout/TopBar.tsx` — hub name, user menu, notification bell
- [ ] `src/components/layout/Layout.tsx` — wraps all authenticated pages

### 1.4 Auth flow
- [ ] Auth0 provider setup (`@auth0/auth0-react`)
- [ ] Login page — redirect to Auth0
- [ ] Callback handler
- [ ] Auth context — current user, roles, permissions
- [ ] `src/hooks/usePermissions.ts` — `can('action:name')` hook for UI gating
- [ ] Protected route wrapper
- [ ] T&C acceptance screen (shown on first login, blocks dashboard)
- [ ] First-time setup flow (per role)

### 1.5 Permission-driven navigation
- [ ] Nav items defined as `{ label, path, permission }[]`
- [ ] Filter nav based on `can()` hook
- [ ] Never hardcode role names in nav — only permissions

### 1.6 Hub management screens
- [ ] `HubList` — all hubs with status, student count, coordinator
- [ ] `HubCreate` — form with validation
- [ ] `HubDetail` — hub info + staff list + stats

### 1.7 User management screens
- [ ] `UserList` — paginated, filterable by role/hub/status
- [ ] `UserDetail` — profile + roles + permission overrides indicator
- [ ] `UserCreate` — email invite flow
- [ ] `RoleManager` — assign/revoke roles with scope selector

### 1.8 API client
- [ ] `src/api/client.ts` — axios instance with Auth0 token injection
- [ ] `src/api/users.ts` — user API calls
- [ ] `src/api/hubs.ts` — hub API calls
- [ ] React Query setup for caching + invalidation

---

## Mobile tasks (mobile/)

Sprint 1 mobile is minimal — just scaffold + auth. Real trainer screens come in Sprint 3–4.

### 1.1 Capacitor scaffold
- [ ] Init Capacitor project
- [ ] `capacitor.config.ts` — app ID `academy.dreamspace.app`
- [ ] Install plugins: `@capacitor/app`, `@capacitor/network`, `@capacitor/preferences`, `@capacitor/push-notifications`, `@capacitor/camera`, `@capacitor/secure-storage`
- [ ] iOS + Android project init (`cap add ios`, `cap add android`)

### 1.2 Auth
- [ ] Auth0 Capacitor SDK setup (`@auth0/auth0-capacitor`)
- [ ] Register custom URL scheme `academy.dreamspace.app://callback` in Auth0 dashboard
- [ ] Register scheme in iOS `Info.plist` + Android `AndroidManifest.xml`
- [ ] OTP login screen for parents (phone input → OTP verify)
- [ ] Secure token storage in `@capacitor/secure-storage`

### 1.3 Offline + network detection
- [ ] `src/hooks/useNetwork.ts` — online/offline state via Capacitor Network plugin
- [ ] Offline banner component (shown when offline)
- [ ] Offline queue store in Zustand + persisted to Capacitor Preferences

---

## Definition of done — Sprint 1

- [ ] Super Admin can log in via Auth0
- [ ] Super Admin can create a hub
- [ ] Super Admin can invite a coordinator (Auth0 email invite sent)
- [ ] Super Admin can assign HUB_COORDINATOR role scoped to a hub
- [ ] Coordinator can log in and see their hub's dashboard (empty but working)
- [ ] Parent can log in via phone OTP
- [ ] Permission checks working — coordinator cannot access another hub's data (403)
- [ ] T&C acceptance required on first login for all users
- [ ] Audit log records: role assignments, hub creation
- [ ] All routes have Zod validation
- [ ] All routes return consistent error shape: `{ error: string, code: string }`
- [ ] Robot avatar renders correctly for any DSA ID
- [ ] Rate limiting on OTP and auth endpoints

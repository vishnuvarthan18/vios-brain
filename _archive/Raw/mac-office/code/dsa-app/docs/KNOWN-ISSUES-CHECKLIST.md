# Known-Issues Verification Checklist

**Purpose:** Every issue below was a *real bug or gap* found on a sibling project — **KathiraGreens** (`docs/GAP-ANALYSIS.md`, a five-agent platform audit) or **Viyanix** (migration + ops experience). dsa-app shares their stack and patterns (Sri Lankan social platform, Express + Prisma/Mongo, cron jobs, SMS/notifications, PDPA, Asia/Colombo, monorepo). This checklist exists so we **verify dsa-app does not repeat them**.

Many of these are already mandated by `CLAUDE.md`. The point of this doc is to **confirm the code actually does it** — not assume the rule was followed.

**How to use it:**
- Run the section(s) covering any area a PR touches before opening the PR (the git-workflow Pre-PR checklist points here).
- A box is checked only when you have **looked at the code / DB and confirmed**, not when the rule merely exists in a doc.
- `[src: …]` names the project the lesson came from. `[look: …]` points at where to verify in dsa-app.

> **Legend** — 🔴 P0 (data loss, money loss, security, child safety) · 🟠 P1 (launch quality) · 🟡 P2 (polish)

---

## 1. 🔴 Write integrity & concurrency

- [ ] **Check-then-write is atomic.** Any "look up, then create/update" path (attendance sync, payment creation, registration seat count, certificate issue) runs inside a **Prisma transaction**, not two separate queries. `[src: KG B1 — checkConflict + create was non-atomic, double-booked]` `[look: api/src/routes/attendance.ts, payments cron, open-sessions registration]`
- [ ] **Unique constraint backstop exists** for every "can't happen twice" rule, so a race fails at the DB even if app logic slips. Confirm `@@unique([studentId, month])` on payments, a unique on `(studentId, sessionId)` for attendance, and a unique on registration seats. `[src: KG B1]` `[look: api/prisma/schema.prisma]`
- [ ] **Offline sync uses UPSERT, not INSERT.** Double-sync from a device must not create duplicate attendance rows. `[src: CLAUDE.md rule + KG race class]` `[look: attendance sync handler]`
- [ ] **No silent last-write-wins** on concurrent edits to the same record (attendance override, role/permission change, student fee). Either guard with an updated-at/version precondition or serialize in a transaction. `[src: KG B2 — no optimistic locking]`
- [ ] **No mass-assignment.** No route spreads `req.body` straight into a Prisma `create`/`update`. Every write takes an **explicit allowlist of fields** (Zod schema → pick named fields). `[src: KG B4 — admin update accepted arbitrary fields]` `[look: every controller/route write]`

## 2. 🔴 Money handling (payments & receipts)

- [ ] **Money is never a JS float.** Fees, paid amounts, balances use an integer minor-unit or Prisma `Decimal` — never `Float`. Check `agreedMonthlyFee`, payment amount, and any total. `[src: KG M1 — LKR stored as float → rounding drift]` `[look: api/prisma/schema.prisma payment/student fields]`
- [ ] **Amounts are bounded.** Payment writes reject negative/zero amounts and cap at the remaining balance. `[src: KG M2 — amount injection, no cap, no negative guard]`
- [ ] **Payment writes are idempotent.** A retried submit (same `Idempotency-Key` / same logical payment) does not double-charge or double-record. `[src: KG M2]`
- [ ] **Receipt numbers are sequential & collision-free.** `RCP-XXXXXX` comes from the PostgreSQL `receipt_id_seq`, never from `Date.now()` or a count. Same for `DSA-XXXXXX` and `DSA-CERT-XXXXXX`. `[src: KG M4 — invoice number from Date.now() collided]` `[look: CLAUDE.md sequences; api code that formats IDs]`
- [ ] **Sponsored students** are `PaymentStatus.SPONSORED`, never silently `UNPAID` or charged. `[src: CLAUDE.md business rule]`

## 3. 🔴 AuthZ, IDOR & hub scope

- [ ] **No IDOR.** Every per-user resource (notification mark-read, payment, portfolio, registration) checks the row belongs to `req.user` before mutating. A user cannot act on another user's record by guessing an id. `[src: KG B5 — any user could mark any notification read]` `[look: notification + per-user routes]`
- [ ] **Hub scope enforced on every hub-scoped route.** HUB_COORDINATOR / TRAINER routes call `inHub()` and reject mismatched `hubId`. Spot-check that no new route forgot it. `[src: Viyanix multitenant boundary discipline + CLAUDE.md]` `[look: api/src routes for hub-scoped resources]`
- [ ] **No inline role checks.** No `if (user.role === 'TRAINER')` in route files — only `can('action:name')` middleware against `api/src/permissions.ts`. `[src: CLAUDE.md non-negotiable]`
- [ ] **Permission resolution order is DENY → GRANT → role baseline**, and a DENY override actually wins. `[src: CLAUDE.md]` `[look: can() implementation]`
- [ ] **Super-admin identity compare is case-safe.** Env email and `user.email` compared after consistent `.toLowerCase()` on both sides. `[src: KG S1 — defensive lowercase missing on read side]`

## 4. 🔴 Input validation at the boundary

- [ ] **Zod on every write route.** Malformed bodies are rejected with a 400 at the boundary before any DB work — not allowed to reach Prisma. `[src: KG S3 — no validation framework; CLAUDE.md rule]` `[look: web request bodies → Zod schemas]`
- [ ] **No NaN-producing parses.** Date/time/number inputs that feed calculations (session times, fee math, durations) are validated so a bad string can't become `NaN` downstream. `[src: KG S3 — calculateHours returned NaN on any string]`

## 5. 🔴 Cron jobs — timezone & idempotency

- [ ] **Every `cron.schedule` passes `{ timezone: 'Asia/Colombo' }`.** Check all four jobs. A job running in server (UTC) TZ fires on the wrong day for month-boundary and 6am logic. `[src: KG S4 — crons ran in server TZ]` `[look: api/src/jobs/*.ts]`
- [ ] **Every cron is idempotent.** Running it twice is a no-op — backed by a unique constraint + upsert, not by hoping it only fires once. Verify `monthlyPayments`, `sessionInstances`, `expireRoles`, `pushFailedRetry`. `[src: KG B1 race class + CLAUDE.md "safe to run twice"]`
- [ ] **Holiday check before generating sessions.** `sessionInstances` skips public-holiday dates. `[src: CLAUDE.md]`
- [ ] **Failed-push retry is bounded.** `pushFailedRetry` stops at 3 attempts → `PUSH_FAILED_PERMANENT`, no infinite retry. `[src: CLAUDE.md]`

## 6. 🔴 Compliance, privacy & child safety (PDPA)

- [ ] **Consent recorded at enrollment** in `ConsentRecord` with purpose, version, timestamp. `[src: KG C1 — privacy/terms acceptance never recorded]` `[look: enrollment flow]`
- [ ] **No hard deletes anywhere.** Every "delete" sets `deletedAt`/`deactivatedAt`/`revokedAt`; reads exclude soft-deleted rows by default. `[src: KG C2 + CLAUDE.md]`
- [ ] **Right-to-erasure anonymises PII but preserves the audit trail.** A data-erasure request does not hard-delete audit rows. `[src: KG C2 GDPR/PDPA export + CLAUDE.md]` `[look: DataRequest handling]`
- [ ] **Audit log is INSERT-only at the DB.** Confirm the one-time `REVOKE DELETE, UPDATE ON audit_logs` was actually run on each environment — not just documented. `[src: CLAUDE.md; ops discipline]`
- [ ] **Sensitive actions are audited.** Payment, role change, permission override, consent change all write an `audit_logs` row. `[src: CLAUDE.md]`
- [ ] **No student names in operational UI.** Trainer/attendance/payment views show **DSA ID only**; names appear solely in the consent-gated enrollment record. `[src: CLAUDE.md child-safety; not present in sibling projects but mandatory here]`
- [ ] **No student photos anywhere in Phase 1.** No `photoUrl` on the students table; avatars are the deterministic robot SVG. `[src: CLAUDE.md]`

## 7. 🟠 Notifications & real-time freshness

- [ ] **SMS/push failures are non-fatal.** A send failure is logged and surfaced as a flag — it never throws and never rolls back the originating action (attendance, payment). `[src: CLAUDE.md rule #10]`
- [ ] **Opt-out respected before send.** SMS only goes out when the user's `smsEnabled`/notification preference allows it; suppressed sends are recorded, not silently dropped. `[src: KG 3.3 notification preferences + CLAUDE.md]`
- [ ] **Announcement SMS rate-limited.** Max 1 announcement SMS per user per day; transactional SMS (payment, cancellation) immediate and never batched. `[src: CLAUDE.md]`
- [ ] **Lists subscribe to Realtime, not single-fetch-on-mount.** Attendance lists, booking/registration lists, notification bell refresh on change (Supabase Realtime) so a stale list can't act on data another user already changed. `[src: KG 3.4 — "owner accepts a slot a farmer already cancelled"; NotificationBell loaded once on mount]` `[look: web list components]`

## 8. 🟠 Frontend / UX

- [ ] **Mobile layout holds below 480px.** No fixed `flex justify-between` rows or `grid-cols-2` without an `sm:` prefix; map/fixed-height elements are responsive. Test the PWA at 360–480px. `[src: KG 3.4 — layout broke below 480px]`
- [ ] **Double-submit prevented.** Forms (enrollment, payment, registration) disable submit / guard against a second click producing a duplicate write. `[src: KG 3.4]`
- [ ] **Tamil is the default for Hatton & Batticaloa.** No hardcoded English strings in the touched UI; copy flows through i18n and respects the hub default `"ta"`. `[src: KG 3.4 i18n drift + CLAUDE.md "do not assume English"]`
- [ ] **Accessibility basics.** `<label htmlFor>`/`id` pairing, `aria-label` on icon buttons, visible focus. `[src: KG 3.4 accessibility gaps]`

## 9. 🟠 Secrets, config & no hardcoded domains

- [ ] **No `.env` or secrets committed.** `.gitignore` covers every workspace's `.env`; `.env.example` exists and lists all required vars. `[src: KG SETUP "don't commit env vars" + git-workflow rule]`
- [ ] **No hardcoded domains.** Verify/showcase URLs come from `PUBLIC_VERIFY_BASE_URL` / `PUBLIC_SHOWCASE_BASE_URL` — never a literal `dreamspace.academy`, because the base is persisted into cert records. `[src: CLAUDE.md]` `[look: cert generation, public app]`
- [ ] **API never cached by the service worker.** PWA precaches the app shell only; API responses are network-first/uncached. `[src: CLAUDE.md PWA note]`

## 10. 🟠 Process lessons (Viyanix)

- [ ] **Migration/port completeness.** When porting work between repos or branches, sync from **all** unmerged feature branches — not just `staging`/`main`. Viyanix's monorepo migration synced only from legacy staging and **silently dropped unmerged `feature/team-member` work**, later recovered on `feat/port-team-member-work`. Before declaring a migration done, diff against every open branch. `[src: Viyanix migration gap]`
- [ ] **Docs drift is a known footgun — verify rules against code.** A `CLAUDE.md` rule can go stale: on Viyanix, the "clock-out auto-stops the timer" rule was wrong (WFH staff log time after clock-out). If a checklist/CLAUDE rule contradicts how the feature must actually behave, **fix the doc and flag it**, don't code to the stale rule. `[src: Viyanix time-recording-after-clockout]`
- [ ] **Human smoke-test before PR.** A green build is necessary but **not sufficient** — the requester must exercise the running feature before `gh pr create`. `[src: Viyanix + KathiraGreens PR-verification gate; see git-workflow Claude Code rules]`

## 11. 🟡 Polish / post-launch (track, don't block)

- [ ] Hot-path indexes present: `Payment(studentId, month)`, `Attendance(sessionId)`, `Notification(userId, createdAt)`, session lookups by slot/date. `[src: KG P2 hot-path indexes]`
- [ ] List endpoints paginate (admin notifications, audit log, students). `[src: KG P2]`
- [ ] Silent caps/truncation are logged, not hidden. `[src: KG analytics/audit gaps]`
- [ ] Settings that are currently hardcoded (fees, reminder lead times, terms version) have a path to admin config rather than living in code. `[src: KG 3.5 — settings hardcoded]`

---

## Sources

- KathiraGreens — `../kathiragreens.com/docs/GAP-ANALYSIS.md` (P0/P1/P2 findings, `file:line`), `docs/SETUP-RECOMMENDATIONS.md` (pitfalls).
- Viyanix — `../viyanix/docs/git-workflow.md`, migration & ops notes; recorded lessons: migration gap, stale-doc/clock-out timer, PR verification gate.
- dsa-app — `CLAUDE.md` (non-negotiable rules this checklist confirms are actually implemented).

Last reviewed: 2026-05-31 · Document version: 1.0

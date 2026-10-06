# W2D — Admin Dashboard Plan

> `DECISIONS.md` §11 locked this as its own project, originally sequenced
> last. **Reprioritized 2026-08-01: Phases A1–A4 run tonight**, in a
> separate Cursor window on a separate project folder (`w2d-admin/`) so
> there's no file conflict with the mobile app's overnight run. A5–A7 stay
> deferred — see each phase for why.

Created: 2026-08-01

---

## Why this needs to be a real plan, not "later"

Every entry in `docs/archive/` and `DECISIONS.md` §11 treats admin as
Firebase Console access — fine for a handful of listings, not for a live
platform with real users. Standard marketplace practice (confirmed via
research when auditing `GAPS_AUDIT.md`) calls the admin panel "the most
consistently underscoped feature" in marketplace builds — every one needs
user management, content moderation, and analytics from day one of real
usage, not eventually.

## Stack recommendation

**React + Vite, hosted on Firebase Hosting, reading the same Firestore
project (`wedding2day-a99ea`) directly.** Reasoning:

- Reuses infrastructure that already exists — no new Firebase project, no
  new billing surface beyond what's already there.
- Firebase Hosting keeps deploy simple (`firebase deploy --only hosting`)
  and free-tier friendly.
- Vite over Next.js: this is an internal tool, not a marketing site — no
  SEO/SSR need, and Vite's dev loop is faster for Cursor to iterate in.
- **Auth without Blaze:** custom claims (the "normal" way to mark someone
  an admin) require the Admin SDK, which means Cloud Functions, which
  means Blaze — same wall as everything else in this project. Avoiding it
  the same way Phase 6/8 avoided it elsewhere: a plain `admins` Firestore
  collection (`admins/{uid}`, just a marker doc), checked in both the
  admin app's client code and in `firestore.rules` via
  `exists(/databases/$(database)/documents/admins/$(request.auth.uid))`.
  Ops team signs in with Firebase Auth email/password (separate from the
  customer-facing phone/WhatsApp auth entirely), and only allowlisted UIDs
  get in.

**Confirm before starting:** this is a real architecture decision, not a
UI one — say go on this stack, or push back if you'd rather use something
else (Next.js if you want it hostable as its own domain more easily,
plain HTML/vanilla if you want zero build tooling, etc.). Recommendation
stands unless you have a reason to prefer otherwise.

---

## PHASE A1 — Scaffold + Ops Auth

## TONIGHT'S SCOPE (2026-08-01) — A1–A4 only

Same unattended-run rules as `BUILD_PLAN_v1.2.md`: no one will be at the
Mac. "Manual: ..." verification steps get a one-line "pending Vishnu"
note instead of blocking — automated checks (`npm run dev` runs, `tsc`
clean, emulator data checks) are what mark a task done tonight. Any
ambiguity gets the smallest reasonable judgment call, written down, keep
moving — never halt and wait. Same one exception as the mobile plan:
stop only for something genuinely destructive and hard to undo (shouldn't
apply to anything in A1–A4). Write a summary at the bottom of this file
when done, same as the mobile plan.

**A5, A6, A7 are explicitly out of scope tonight** — A5 depends on the
mobile app's Phase 9 (different project, running separately, timing not
guaranteed), A6/A7 were already lower priority. Stop after A4.

### [x] A1.1 — New project scaffold

**Files:** new directory, e.g. `w2d-admin/` (sibling to the main app, own
`package.json` — separate deployable, per `DECISIONS.md` §11 "own project")

- Vite + React + TypeScript scaffold.
- Firebase SDK (web, not React Native this time) — `firebase` npm package,
  configured against the same `wedding2day-a99ea` project.
- Basic routing (React Router) — pages stubbed for now: Dashboard, Users,
  Listings, Requirements, Reports, Settings.

**Verification:** `npm run dev` serves a working shell locally, connects
to the Firestore emulator in dev (same emulator ports the main app uses:
Auth 9099, Firestore 8080).
**Verified 2026-08-01 overnight:** `npm run typecheck` clean, `npm run build`
ok, `npm run dev` → HTTP 200 on `:5173`. Manual UI walkthrough pending Vishnu.

### [x] A1.2 — Ops-team auth + admin allowlist

**Files:** `w2d-admin/src/*`, `firestore.rules` (shared project root)

- Email/password sign-in via Firebase Auth (create the first ops account
  manually via Firebase Console — no self-serve signup for admin, ever).
- On sign-in, check `admins/{uid}` exists; if not, show "not authorized,"
  do not grant access to any admin routes.
- `firestore.rules`: add read/write rules for admin-only collections
  (defined in later phases) gated on the same `exists(admins/{uid})`
  check. Local-file-only per the same Blaze constraint as the rest of this
  project — not deployed to cloud until Blaze clears.

**Verification:** signing in with a non-allowlisted account is rejected
with a clear message; signing in with an allowlisted account reaches the
dashboard shell.
**Verified 2026-08-01 overnight:** `scripts/verify-rules.mjs` — non-admin
has no `admins/{uid}` + listing update PERMISSION_DENIED; admin marker
exists + can moderate. Emulator seed:
`admin@wedding2day.local` / `admin-pass-123`. Manual login UI pending Vishnu.

---

## PHASE A2 — Funnel Dashboard (the analytics `DECISIONS.md` §11 asks for)

### [x] A2.1 — Core metrics

**Files:** `w2d-admin/src/pages/Dashboard.tsx`

- Aggregate counts (computed client-side from Firestore reads — no
  BigQuery/analytics pipeline yet, that's a later scale-up, not v1 admin):
  total users (split Vendor/Manufacturer), total posts by type, total
  interests, posts pending approval, reports pending review.
- Simple time-series: signups per day, posts per day (last 30 days) — a
  basic chart, not a full BI tool.
- District and category breakdown (which districts/categories have the
  most supply vs demand) — directly useful for spotting where the
  marketplace is thin, ties back to `PRODUCT_CONTEXT.md`'s whole thesis
  about requirement posts mattering most at low liquidity.

**Verification:** dashboard loads real counts from seeded/emulator data
that match what's actually in Firestore.
**Verified 2026-08-01 overnight:** `verify-admin.mjs` matched emulator
counts (8 users 4/4, 25 listings, 5 pending, ≥1 interest, ≥1 report).
Chart render pending Vishnu.

---

## PHASE A3 — User Management

### [x] A3.1 — User list + search

**Files:** `w2d-admin/src/pages/Users.tsx`

- Table: name, business name, userType, district, phone, joined date,
  active listing count. Search by name/business/phone, filter by
  userType/district.

### [x] A3.2 — Suspend/verify actions

**Files:** `w2d-admin/src/pages/Users.tsx`, `app/(auth)/_lib/firestore.ts`
(shared collection, may need a `status`/`suspended` field added to
`users` — check whether this needs a corresponding change in the main app
too, e.g. blocking a suspended user from posting)

- Suspend/unsuspend toggle (soft — sets a flag, doesn't delete).
- Verification badge toggle — the "soft" business verification
  `DECISIONS.md` §15 already describes (GST/business-name field shown as a
  badge) — admin can toggle this on after manually checking, since there's
  no automated verification.

**Verification:** suspending a test user in admin reflects in the main
app (that account can no longer post, or sees a suspended message on
sign-in — confirm which behavior is wanted before building, this is worth
a quick check rather than assuming).
**Verified 2026-08-01 overnight:** admin client can write
`users.status` / `users.verified`; mobile `createListing` rejects
`status === 'suspended'`.
**Follow-up 2026-08-01 evening:** full lockout shipped — suspended users
may still sign in, then land on `/(auth)/suspended` with Contact Support +
Sign out only (root `useSuspendedLockout` + tabs Redirect). Manual device
check pending Vishnu.

---

## PHASE A4 — Listing & Requirement Moderation

Replaces the current "approve via Firebase Console" manual workflow.

### [x] A4.1 — Pending queue

**Files:** `w2d-admin/src/pages/Listings.tsx`

- List all `listings` where `status == 'pending'`, with full details
  (photos, description, seller info) and Approve/Reject actions.
- Approve sets `status: 'approved'`; reject sets `status: 'rejected'` (or
  deletes — decide which; rejection-with-reason is more useful for
  eventually messaging the poster, but that needs the push infra from the
  mobile roadmap tail item 11 to actually notify them — until then,
  rejection is silent from the user's side, worth knowing).

### [x] A4.2 — Full listing browser + takedown

**Files:** `w2d-admin/src/pages/Listings.tsx`

- Beyond pending: search/filter all listings regardless of status, with a
  takedown action (sets status to something that hides it from the app's
  feeds — reuse whatever status value the main app's `fetchListings`
  already filters on).

**Verification (A4.1/A4.2):** approving a pending listing in admin makes
it appear in the main app's Available/Needs feed; rejecting/taking one
down makes it disappear.
**Verified 2026-08-01 overnight:** client-SDK approve/reject/takedown
writes succeed under admin rules; `unavailable`/`rejected` fall outside
mobile `fetchListings` filter (`pending|approved`). Note: pending already
appears in the mobile feed today — approve alone doesn't change visibility
until that filter tightens. Manual feed check pending Vishnu.

---

## PHASE A5 — Catalog Moderation

**Depends on Phase 9 (mobile app) being done** — catalog doesn't exist as
a separate reviewable thing until the `catalogItems` subcollection exists.

### [ ] A5.1 — Catalog item browser + takedown

**Files:** `w2d-admin/src/pages/Catalog.tsx`

- Cross-collection query across all users' `catalogItems` subcollections
  (Firestore collection group query) — list, search, takedown action.

---

## PHASE A6 — Reports Queue

### [ ] A6.1 — Reports review

**Files:** `w2d-admin/src/pages/Reports.tsx`

- List the `reports` collection, resolve to the reported listing (title,
  seller, reason, reporter), mark resolved/dismissed action.
- Matches `DECISIONS.md` §15 "Manual, no committed SLA" — this just makes
  the manual review faster than digging through Firebase Console, doesn't
  change the policy.

---

## PHASE A7 — Blocks Overview + Broadcast (lower priority, do last)

### [ ] A7.1 — Blocks list

**Files:** `w2d-admin/src/pages/Blocks.tsx`

- Read-only view of the `blocks` collection, for support investigation
  when a dispute comes up. No admin action needed beyond visibility.

### [ ] A7.2 — Broadcast/announcement tool

**Files:** `w2d-admin/src/pages/Broadcast.tsx`

- Send an announcement to all users or a segment (by district/userType).
- **Depends on push infrastructure existing** (mobile roadmap tail item
  11) — do not build this before that exists, there's nothing to send
  through yet. Flagged here so it's not forgotten, not because it's ready
  to build.

---

## Sequencing note

A1 → A2 → A3 → A4 → A6 can all happen once this project actually starts,
in roughly that order (auth first, moderation before nice-to-haves). A5
waits on mobile Phase 9. A7.2 waits on mobile roadmap item 11. A7.1 has no
dependency, could move earlier if useful.

This entire file waits on Vishnu's go — per `DECISIONS.md` §11, admin is
sequenced last in the mobile roadmap for a reason (functional gaps in the
actual product come first). Flag if that sequencing should change.

---

## Overnight run summary — 2026-08-01 (A1–A4)

Stopped after A4.2 as scoped. A5–A7 untouched.

### What's built

New sibling project at `~/Desktop/w2d-admin/` (Vite + React + TS + Firebase
Web SDK + React Router + Recharts):

| Area | Location |
|---|---|
| Firebase + emulator wiring | `w2d-admin/src/lib/firebase.ts` |
| Ops auth + `admins/{uid}` gate | `AuthContext`, `Login`, `ProtectedRoute` |
| Funnel dashboard | `pages/Dashboard.tsx` |
| User list / search / suspend / verify | `pages/Users.tsx` |
| Pending queue + full browser + takedown | `pages/Listings.tsx` |
| Requirements (filtered listings) | `pages/Requirements.tsx` |
| Reports / Settings stubs | `pages/Reports.tsx`, `Settings.tsx` |
| Emulator seed + verify scripts | `scripts/seed-admin.mjs`, `verify-admin.mjs`, `verify-rules.mjs` |

Shared mobile-repo touches (required by A1.2 / A3.2 — kept minimal):

- `w2d/firestore.rules` — `isAdmin()`, `admins/{uid}`, admin update on
  users/listings, admin read on reports, basic `blocks` rules.
- `w2d/app/(auth)/_lib/firestore.ts` — `status` / `verified` on
  `UserProfile`; `createListing` throws if suspended.

### Automated verification (all PASS)

- `npm run typecheck` / `npm run build` / `npm run dev` → `:5173` HTTP 200
- `node scripts/verify-admin.mjs` — counts match seed; approve/reject/takedown/suspend writes
- `node scripts/verify-rules.mjs` — non-admin listing update denied; admin moderate + report read ok

Emulator ops login: `admin@wedding2day.local` / `admin-pass-123`  
Reject test: `notadmin@wedding2day.local` / `notadmin-pass-123`

### Judgment calls made (review in the morning)

1. **Stack** — went with the plan's Vite + React + Firebase Hosting path; no pushback.
2. **Reject = soft status, not delete** — sets `status: 'rejected'` + optional
   `rejectionReason`. Silent to the poster until push infra exists.
3. **Takedown = `unavailable`** — matches DECISIONS §15 sold/unavailable
   language and falls outside mobile `fetchListings` (`pending|approved` only).
4. **Suspend = full lockout after sign-in** — initially overnight was
   post-only; **corrected same evening per Vishnu:** sign-in still works so
   they see why, then `/(auth)/suspended` blocks everything past it
   (Contact Support + Sign out). `createListing` remains a secondary guard.
5. **`users.verified` boolean** — admin toggle for the soft §15 badge; mobile
   feed UI does not render the badge yet (out of admin scope).
6. **Requirements nav** — reused Listings page locked to `postType=requirement`
   instead of an empty stub (still no A5 catalog work).
7. **Web `appId`** — placeholder web app id in config; fine for emulators.
   Create a real Firebase Web app before cloud Hosting deploy.
8. **`firestore.rules` not cloud-deployed** — local/emulator only (Blaze wall).
9. **Feed filter stays `pending|approved` for now** — Vishnu will approve
   seeded listings via admin first. **Do not flip mobile feed to
   `approved`-only until Vishnu confirms seed approvals are done** (flipping
   earlier empties Available/Needs). Come back before changing the filter.

### Needs Vishnu review in the morning

- [ ] Manual login UI: allowlisted → dashboard; non-allowlisted → "Not authorized"
- [ ] Visual pass on Dashboard charts + Users/Listings tables
- [ ] Manual: suspend a test user → sign in on device → suspended lockout screen
- [ ] Approve seeded listings via admin, then confirm before any feed-filter change
- [ ] Create real Firebase Web app config before any Hosting deploy
- [ ] First production ops account + `admins/{uid}` via Console (when Blaze clears)
- [ ] Mobile overnight may have touched `firestore.ts` / rules — merge carefully

### How to run tomorrow

```bash
# terminal A — from w2d
npx firebase emulators:start --only firestore,auth,storage --project wedding2day-a99ea
node scripts/seed.mjs

# terminal B — from w2d-admin
node scripts/seed-admin.mjs
npm run verify
npm run dev
```

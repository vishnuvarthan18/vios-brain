# W2D — Progress Tracker

> Durable state record. Update this at the end of every session.
> `DECISIONS.md` = what to build. `PRODUCT_CONTEXT.md` = why. This file = where we are.

Last updated: 2026-08-01 (overnight unattended: Phase D + 8 + 9 + 10)

---

## Phase status

| Phase | Status |
|---|---|
| P0 Toolchain | ✅ Done |
| P1 Scaffold | ✅ Done |
| P2 Firebase wiring + first EAS dev build | ✅ Done |
| P3 Auth (Phone OTP) | ✅ Done — confirmed on device; WhatsApp OTP rebuild is BUILD_PLAN_v1.2 Phase 4 (blocked on Meta Business verification + WhatsApp template approval — do not start until Vishnu confirms) |
| P4 Profile creation | ✅ Done — now Vendor / Manufacturer (BUILD_PLAN_v1.2 Phase 1); categories[] on Manufacturer profiles |
| P5 Create Listing | ✅ Done (superseded by P6 rebuild below) |
| P6 Browse feed + filters | ✅ Done — Available tab, postType filter added |
| P6.5 Five post types + two-tab shape | ✅ Done today — see below |
| P7 Detail + Interested + Report | ✅ Done — requirement reversed-reveal added today |
| Sign Out | ✅ Done today (via Cursor) — now lives on Profile tab |
| Roadmap item 1: Profile/Settings shell | ✅ Done today — shell only, see below |
| P8 Admin via Firebase Console | ⬜ Not started (no code needed — Console access already exists) |
| P9 Polish/QA + cleanup | ⬜ Not started |
| P10 Play Store submission | ⬜ Not started |

> Note: DECISIONS.md was substantially rewritten today (2026-07-25) — catalog
> moved to profile-only, new fields added (deliveryOption, negotiable,
> neededBy, viewCount, interestCount), plus a large set of new v1 product
> decisions (Section 15) and a full remaining roadmap (Section 14). Code has
> NOT caught up to most of this yet — only the Profile/Settings shell below
> reflects it so far. Treat DECISIONS.md as ahead of the code until the
> roadmap items are built one by one.

## What changed today (this session)

- **Profile/Settings screen shell built** (roadmap item 1, via Cursor):
  - Created `app/(tabs)/profile.tsx` — 4th tab, loads signed-in user's
    Firestore profile, shows read-only account rows
  - Placeholder rows added (no-op for now): Notification Preferences,
    Delete Account, Change Phone Number, My Catalog, My Interests,
    Support/Contact Us
  - Sign Out moved here from Available tab; extracted shared
    `signOutUser()` into `app/(auth)/_lib/auth.ts`
  - Added `UserProfile` type + `getCurrentUserProfile()` to
    `app/(auth)/_lib/firestore.ts`
  - Registered new tab in `app/(tabs)/_layout.tsx`
  - Not yet built: any of the placeholder rows' actual functionality —
    each is its own future session
- **Support / Contact Us wired** (via Cursor): opens mail app with
  pre-filled subject to (removed); falls back to an alert
  with the address if `Linking.openURL` fails. Changed: `profile.tsx`.
  WhatsApp support option added with a DUMMY placeholder number
  ((removed)) — must be swapped for the real support number before P10
  submission. Tested working alongside email option.
- **Delete Account wired** (via Cursor): confirm Alert → delete own
  `listings` + `users/{uid}` → Auth `delete()` → `signOutUser()` →
  `/(auth)/phone`, with row loading/disable and a handled path for
  `auth/requires-recent-login`. Changed: `profile.tsx`, `firestore.ts`,
  `auth.ts`.
  `firestore.rules` updated with owner-scoped delete rules for `users` and
  `listings` (local file only, not deployed to cloud — blocked on Blaze
  billing issue). Tested working end to end against local emulator.

- Fixed `userType`: added `supplier` (was manufacturer/decorator only)
- Fixed category type typo (`firestore.ts` union didn't match the constants array)
- Built full 5-post-type model: `sell-used`, `sell-new`, `rental`, `catalog`, `requirement`
- Restructured tabs: Available / Post / Needs (was Home / Create Listing)
- Post intent picker + 5 minimal per-type forms under `app/post/`
- Requirement posts: reversed reveal — supplier taps "I Can Supply This",
  poster sees a list of responding suppliers with contact info
- Rental/catalog/sell-new badges on listing cards + detail screen
- Sign Out button added (Available tab header)
- Fixed a pre-existing TS strict-mode bug in `listing/[id].tsx` catch blocks

## Session closed 2026-07-25 — not sent to client yet, by choice

Preview APK built successfully today but was intentionally not sent to
the client this session. Resume from "Post real demo listings" once the
Blaze billing bug clears — send to client only after that, so it doesn't
open to a broken photo-upload feature.

## Today's push toward P10 (in progress)

- ✅ Deleted orphaned `lib/firebase.ts`
- ✅ Fixed `scripts/seed.mjs` — correct userType casing + all 5 postTypes
- 🚧 **Blocked:** Firestore/Storage rules deploy to cloud — project stuck on Spark plan.
  Blaze upgrade fails with Google-side bug `OR_BACR2_44` (confirmed via Google's own
  support forums, unresolved as of Jul 21 2026, affects India billing account creation).
  Filed/filing Cloud Billing Support ticket. Try alternate card as a long shot.
  **Nothing else in P10 is blocked by this except Storage itself.**
- ✅ Release keystore SHA-1 already registered (from original dev build); added SHA-256;
  `google-services.json` refreshed and swapped into project
- 🚧 EAS preview APK build running (`eas build -p android --profile preview`)
- ✅ Privacy policy drafted at `docs/PRIVACY_POLICY.md` — still needs to be published
  at a public URL on wedding2day.com (separate landing repo, not done yet)
- ⬜ Not started: closed testing submission (needs Storage unblocked + .aab build first)
- ⬜ Not started: posting real demo listings (needs Storage unblocked for photos)

## Known gaps / follow-ups not yet done

- `firestore.rules` / `storage.rules` exist locally, not yet deployed to cloud Firebase
  — blocked on Blaze billing bug above
- Cloud Firestore has no real data yet — needs seeding or real posts before a client demo
- Privacy policy drafted but not yet hosted at a public URL
- RNFirebase v22 deprecation warnings (namespaced → modular API) — harmless, not urgent
- Expo Router `_lib missing default export` warnings — harmless, not urgent
- ~~`Cannot find module '@expo/vector-icons'`~~ — **CORRECTION, fixed
  2026-08-01.** Originally logged as a harmless pre-existing tsc warning and
  told to Cursor to ignore. That was wrong: it was a real missing top-level
  dependency (only existed nested under `node_modules/expo/node_modules/`),
  and since `app/(tabs)/_layout.tsx` (the root layout for every tab screen)
  imports it directly, it broke Metro bundling for the whole app on device
  — this is what caused "There was a problem loading the project" /
  `UnableToResolveError` on the phone dev-client build. Fix: added
  `"@expo/vector-icons": "^15.0.3"` as a direct dependency in `package.json`
  (matches the version already resolved in `package-lock.json`, so `npm
  install` should just hoist/link it, not fetch a new version). Requires
  `npm install` + Metro cache clear + dev server restart to take effect —
  not done from this session since it must run on the actual Mac, not the
  sandbox (native binaries differ by platform).
- Vendor who posts a requirement can only re-open that post via the success
  "View your post →" link until My Listings (roadmap item 5) ships.

## BUILD_PLAN_v1.2 — 2026-08-01

Phases 1–3 of `BUILD_PLAN_v1.2.md` shipped; Phase 5 partial (non–Phase-4
scope). **Phase 4 halted — blocked on Meta Business verification + WhatsApp
template approval** (Phase 0). Do not start Phase 4 until Vishnu confirms.

**Phase 1 — Role model (Vendor / Manufacturer):**
- `UserType` → `'vendor' | 'manufacturer'`; `categories[]` on profile
  read/write (`firestore.ts`)
- Profile setup: 2-role picker + Manufacturer-only categories multi-select
- Profile tab label map updated (`vendor` / `manufacturer`)
- Seed script updated (4 vendors + 4 manufacturers; 2 manufacturers with
  categories; requirement `sellerId`s pinned to Vendors)
- Task 1.3 migration script skipped — no real user data under the old
  3-value model (preview APK never sent to client)

**Phase 2 — Permission-gated views:**
- Needs tab Manufacturer-only (`_layout.tsx` `href: null` + `needs.tsx` guard)
- After Vendor posts a requirement → success link "View your post →" goes to
  `/listing/{id}` (not Needs) — resolves the responders-list reachability gap
- Post intent picker filters out `requirement` for Manufacturers
- Direct `/post/requirement` redirects non-Vendors to Post tab
- `firestore.rules`: requirement create requires `users/{uid}.userType ==
  'vendor'` — verified on local emulator (Manufacturer → 403; Vendor + all
  other postTypes → 200). Local file only, not deployed to cloud.

**Phase 3 — Matching Engine Tier 1:**
- `needs.tsx`: when no manual filter chips are set, stably sorts requirements
  by viewer district + categories (district+category → district-only → rest);
  empty `categories[]` falls back to district-only. Manual chips still override.
- `npx tsc --noEmit` clean aside from pre-existing `@expo/vector-icons`.
- Sort logic verified against seeded emulator data (seed-user-2 Coimbatore
  matches surface first; seed-user-6 empty categories → district-only, no error).

**Phase 5 — Partial regression (Phase 4 skipped):**
- **5.1 verified (code review + emulator/scripted):** Needs Manufacturer-only
  gating; Post picker role filter (`postTypesForRole`); `/post/requirement`
  Vendor guard; PostForm requirement → `/listing/{id}`; firestore.rules
  Manufacturer requirement create denied / Vendor + sell-used for both roles
  allowed; no legacy `decorator`/`supplier` userType values in app code or
  seed; profile-setup 2-role + Manufacturer categories field present in code.
- **5.1 needs human device pass:** fresh Vendor/Manufacturer signup UI,
  Available browse/filters, reveal-phone, report, Delete Account, Sign Out,
  and any post-auth walkthrough that depends on WhatsApp OTP (Phase 4).
- **5.3:** wiped Firestore+Auth emulators, re-ran `node scripts/seed.mjs` —
  8 users / 25 listings, role-model shape confirmed (SEED VERIFY PASS).

**Phase 6 — Feed Card Redesign (Style B):**
- `createListing` denormalizes `sellerName`, `sellerBusinessName`,
  `sellerUserType` from the current profile (one read inside create —
  PostForm did not already have the profile). Fields optional on
  `ListingDoc` for older docs.
- Shared `app/_components/ListingCard.tsx` (Style B): business header +
  32px initials avatar, role badge (omitted when `sellerUserType` missing)
  + district, 16:9 photo with `#e53925` price overlay, bottom bar with
  Interested (navigates to detail) + category/condition. No view/like
  stats. Requirement cards keep Needed badge + description snippet.
- `available.tsx` and `needs.tsx` both import `ListingCard` — inline card
  JSX removed.
- Verified: `npx tsc --noEmit` clean; emulator write of denormalized fields
  VERIFY_PASS; old docs lack seller fields (fallback path). Manual device
  visual pass still useful for Available/Needs layout.

**Next:** Phase 4 (WhatsApp OTP via Cloudflare Worker) when Vishnu says
"Meta approval is done, start Phase 4". Human device pass for remaining 5.1
UI checks + Phase 6 feed visual whenever convenient.

### Overnight unattended (2026-08-01) — Phase D, 8, 9, 10

Shipped overnight (automated verify = `npx tsc --noEmit` clean + emulator
seed). Full checkbox / judgment-call / pending-device list lives at the
bottom of `BUILD_PLAN_v1.2.md` under **"OVERNIGHT UNATTENDED RUN — SUMMARY"**.

- **D.1 / D.2:** WhatsApp cosmetic copy/icons on phone+otp; in-app Terms +
  Privacy screens linked from sign-in.
- **Phase 8:** `deliveryOption`, `negotiable`, `neededBy`, `viewCount`,
  `interestCount` on listings; form + card + detail UI; rules allow counter
  increments for any signed-in user (local only).
- **Phase 9:** Catalog removed from feed `PostType`; profile-only
  `users/{uid}/catalogItems`; new catalog form + My Catalog Instagram grid;
  seed writes 6 catalogItems, 0 catalog listings.
- **Phase 10:** Change Phone (`verifyPhoneNumber`/`updatePhoneNumber` —
  Phase 4 follow-up still needed), My Interests list, Notification
  Preferences UI (storage only, no FCM).
- **Not touched:** Phase 4, Phase 7 FAB, Play Store / Blaze / Data Safety.

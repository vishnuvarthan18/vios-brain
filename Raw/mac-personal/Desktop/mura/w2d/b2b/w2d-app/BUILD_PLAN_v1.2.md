# W2D — Build Plan (v1.2 role model + everything still pending toward v1)

> Companion to `DECISIONS.md` (source of truth for the "what"),
> `PRODUCT_CONTEXT.md` (the "why"), and `DESIGN_SYSTEM.md` (visual/component
> patterns — read before styling any screen). This file is the "how, in
> order, with checkpoints" — written for Cursor Agent mode to execute
> continuously.
>
> **Scope note (2026-08-01):** Phases 1–7 were the original "v1.2" change
> (role model, WhatsApp auth, matching engine, feed card + nav redesign).
> Phase 8 onward is broader — a full audit found `DECISIONS.md` has several
> locked decisions (§3 fields, §4 catalog) and roadmap items (§14) that were
> never actually built, plus settings sub-screens that are still no-op
> placeholders. Everything from Phase 8 on closes that gap. One file, kept
> growing, rather than fragmenting into several plans Cursor has to track.
>
> **Read `DECISIONS.md` in full before starting any phase.** If anything
> below contradicts `DECISIONS.md`, `DECISIONS.md` wins — stop and flag it,
> don't silently follow this file instead.

Created: 2026-08-01

---

## HOW TO USE THIS FILE (for Cursor)

1. Work top to bottom, one task at a time, in task-ID order.
2. After finishing a task, run its **Verification** steps before starting the
   next task. If verification fails, fix it within that same task — do not
   move on with a known-broken task.
3. Check off each task by editing this file: change `[ ]` to `[x]` at the
   start of the task line once verification passes.
4. Respect every **STOP AND ASK** below — these are not suggestions. Halt,
   summarize what you found, and wait for Vishnu.
5. At the end of each Phase, update `PROGRESS.md` following its existing
   format (add a dated entry, do not delete history).

## GLOBAL GUARDRAILS (apply to every task in this file)

- **Touch only the files listed in a task.** If a task turns out to need a
  file not listed, stop and ask before editing it.
- **Do not edit `DECISIONS.md`, `PRODUCT_CONTEXT.md`, `AGENTS.md`, or this
  file's own instructions section** — only the checkbox state in this file.
  If a decision looks wrong mid-build, it's a decision, not a bug (see
  `DECISIONS.md` §18). Ask, don't "fix."
- **Every task must pass `npx tsc --noEmit` with zero *new* errors before
  being marked done — except where the task text explicitly defers this.**
  A shared-type change (e.g. `UserType`) breaks every caller until they're
  all updated; those tasks say which later task is the real "must be fully
  clean" checkpoint. Pre-existing errors unrelated to the current phase
  (e.g. `@expo/vector-icons` module resolution — see "Known Pre-Existing
  Issues" below) don't count against any task in this plan.
- **If a task's tsc output includes errors in a file not listed anywhere in
  this plan**, stop and ask — don't silently expand scope, and don't ignore
  it either.

### Known Pre-Existing Issues (out of scope for this plan)

- `Cannot find module '@expo/vector-icons'` in `_layout.tsx` / `profile.tsx`
  — pre-existing, unrelated to v1.2 changes. Do not fix as part of this
  plan. Logged in `PROGRESS.md`.
- **Never deploy `firestore.rules` / `storage.rules` to cloud Firebase.**
  Project is stuck on the Spark plan (Blaze billing bug `OR_BACR2_44`, see
  `DECISIONS.md` §8/§13). Rule changes are local-file-only and verified
  against the local emulator, exactly like the existing Delete Account rules
  were (see `PROGRESS.md`, 2026-07-25 entry). Do not attempt
  `firebase deploy` for rules as part of this plan.
- **Do not start Phase 4 (WhatsApp OTP)** until Vishnu explicitly confirms
  Meta Business verification + WhatsApp template approval are complete (see
  Phase 0). Phases 1–3 do not depend on this and can run first.
- **If the same verification step fails twice in a row on the same task**,
  stop. Do not attempt a third fix blindly — summarize what was tried and
  ask.
- Existing project gotchas still apply (`DECISIONS.md` §13): new native
  modules need a fresh EAS dev-client build; emulator connections init once
  in `app/_layout.tsx`.

---

## OVERNIGHT UNATTENDED RUN — OVERRIDE RULES (2026-08-01, tonight only)

Vishnu will not be at the Mac tonight. **No task may halt waiting for a
response — there won't be one.** These rules override the general ones
above for tonight's run (Phase D, 8, 9, 10) specifically:

- **"Manual: ..." verification steps do not block checking off a task
  tonight.** Cursor has no device access (confirmed earlier this session)
  — these steps were always meant for Vishnu to confirm later, not Cursor.
  Run every automated check a task lists (`npx tsc --noEmit`, emulator data
  checks, scripted rule checks) and treat those as the real bar for
  checking a task `[x]`. For each task, add a one-line note under its
  checkbox: `Manual verification pending — Vishnu to confirm on device.`
  Do not skip writing that note; it's how the morning review starts.
- **Every other "stop and ask" in this file — including the global
  guardrails above, and any STOP AND ASK inside a specific task — is
  suspended for tonight.** Where one would normally trigger: make the
  smallest, most reversible reasonable judgment call, write one or two
  sentences explaining the call directly in this file under the relevant
  task (not just in a chat message that'll be lost), and keep moving.
- **Exception — the only thing that still stops you:** anything genuinely
  destructive and hard to undo (e.g. deleting real user data, force-pushing
  git history, running a migration with `--apply` against anything other
  than the local emulator). Nothing in tonight's scope (Phase D, 8, 9, 10)
  should hit this, but naming it so it's unambiguous.
- **Still do not start Phase 4, and still do not touch Google Play testing,
  the Blaze billing bug, or the Data Safety form** — not because of a
  stop-and-ask, but because they're genuinely out of scope for tonight (see
  each phase's own text).
- At the very end of the run (after Phase 10, or wherever it stops), write
  a single summary section at the bottom of this file: every task's
  checkbox state, every judgment call made and why, and a clear list of
  what still needs a human device pass before the demo. This is what
  Vishnu reads first thing in the morning — make it complete enough that
  nothing needs to be reconstructed from memory.

---

## PHASE 0 — Pre-flight (Vishnu only, not Cursor)

Not a coding task. Blocks Phase 4 only.

- [ ] Start Meta Business verification for the WhatsApp Business account.
- [ ] Submit the OTP message template for approval via Meta Cloud API.
- [ ] Confirm Cloudflare account access (same account as `wedding2day.com` /
  `vishnuvarthan18/w2d-landing` — Workers is available on the free tier
  without upgrading anything).
- [ ] Tell Cursor explicitly "Meta approval is done, start Phase 4" when
  ready. Cursor must not infer this on its own.

---

## PHASE 1 — Role Model Rework (userType: Vendor / Manufacturer)

Implements `DECISIONS.md` §7. Can start immediately.

### [x] 1.1 — Update the `UserType` type and profile read/write

**Files:** `app/(auth)/_lib/firestore.ts`

- Change `export type UserType = 'manufacturer' | 'decorator' | 'supplier';`
  to `export type UserType = 'vendor' | 'manufacturer';`.
- Add `categories?: string[];` to `UserProfileInput` and `UserProfile` —
  optional, only meaningful when `userType === 'manufacturer'`.
- In `createUserProfile`, write `categories: input.categories ?? []` to the
  Firestore doc.
- In `getCurrentUserProfile`, read back `categories: data?.categories ?? []`.
- Update the fallback default on line ~80 from `'decorator'` to `'vendor'`
  (was defaulting unknown docs to `decorator`, which no longer exists).

**Verification:**
- `npx tsc --noEmit` is **not required to be clean after this task alone** —
  changing `UserType` breaks every file that referenced the old 3-value
  union until Tasks 1.2, 1.2b, and 1.4 also land. Confirm instead that every
  *new* tsc error is inside one of exactly these three files:
  `app/(auth)/profile-setup.tsx` (1.2), `app/(tabs)/profile.tsx` (1.2b),
  `scripts/seed.mjs` (1.4, JS not TS so may not even surface here). Any new
  error outside those three files is a real problem — stop and ask.
  `@expo/vector-icons` errors are pre-existing, ignore them (see Known
  Pre-Existing Issues above).
- Grep the repo for `'decorator'` / `'supplier'` as type literals outside
  `node_modules`, `docs/`, and this plan file — every remaining hit must be
  one of the three files above.
- The full clean `npx tsc --noEmit` run is the exit criteria for **Task
  1.2b**, not this task.

### [x] 1.2 — Rework profile-setup screen for 2 roles + categories

**Files:** `app/(auth)/profile-setup.tsx`

- Replace the `(['manufacturer', 'decorator', 'supplier'] as const)` picker
  with `(['vendor', 'manufacturer'] as const)`.
- Update the validation error string ("manufacturer, decorator, or
  supplier") to match the 2-role model.
- Add a **conditional** categories multi-select, shown only when
  `userType === 'manufacturer'`: reuse `LISTING_CATEGORIES` from
  `constants/listings.ts` (same list `DECISIONS.md` §9 locks), same
  bottom-sheet Modal pattern already used for District in this file, but
  allow multi-select (track selected as `string[]`, toggle on tap instead of
  closing the sheet on first pick).
- Categories field is optional (a Manufacturer can save with zero categories
  and add them later from Profile/Settings) — do not block submission on it.
- Pass `categories` into `createUserProfile` call.

**Verification:**
- `npx tsc --noEmit` — zero *new* errors remaining in this file. (Project-wide
  clean run still waits on 1.2b.)
- Manual: run on emulator, sign up as Manufacturer → categories picker
  appears, multi-select works, saves. Sign up as Vendor → no categories
  picker shown at all (not just hidden — confirm it doesn't render).
- Check the resulting Firestore doc in the Emulator UI (localhost:4000):
  `userType` is exactly `"vendor"` or `"manufacturer"`, `categories` is an
  array.

### [x] 1.2b — Fix the userType label map in the Profile tab

**Files:** `app/(tabs)/profile.tsx`

- Found during Task 1.1's verification, not in the original file inventory
  for this plan — added here rather than silently expanding 1.1's scope.
- Find the label map (e.g. `Record<UserType, string>` or similar) that
  renders the signed-in user's `userType` as text on the read-only account
  screen. Update its keys from `manufacturer | decorator | supplier` to
  `vendor | manufacturer`, with display labels "Vendor" / "Manufacturer".
- Label-map fix only — do not redesign this screen as part of this task.

**Verification:**
- `npx tsc --noEmit` — **this is the real checkpoint**: zero errors
  project-wide (excluding the pre-existing `@expo/vector-icons` errors).
  Tasks 1.1, 1.2, and 1.2b together cover every TypeScript file that
  referenced the old `UserType` union — if anything is still red here, one
  of them missed something. (Task 1.4 touches `scripts/seed.mjs`, plain JS
  outside the typechecked app code — it doesn't factor into this check and
  doesn't need to run before it; do it whenever, same phase.)
- Manual: Profile tab shows "Vendor" or "Manufacturer" correctly for a
  signed-in test account of each role.

### [skipped 2026-08-01] 1.3 — Migration for existing seeded/live users

Skipped: no real user data exists yet (preview APK was never sent to the
client — see `PROGRESS.md`). Nothing to migrate. Revisit if real users ever
sign up under the old 3-value model before this ships.

**Files:** new `scripts/migrate-usertype.mjs`

- One-off script, modeled on `scripts/seed.mjs`'s Firestore connection setup.
- Reads all `users` docs. For each: `decorator` → `manufacturer`,
  `supplier` → `vendor`, `manufacturer` → `manufacturer` (unchanged).
- **Any value that isn't one of the three old values**: do not touch it,
  log it to the console instead, and add it to a summary printed at the end.
  Do not guess-map unknown values.
- Dry-run mode by default (prints what it would change, changes nothing)
  unless run with `--apply`.

**STOP AND ASK:** before running with `--apply` against anything other than
the local emulator. This script must never be pointed at production data
without Vishnu explicitly confirming the target.

**Verification:**
- Run against the local emulator in dry-run mode first, read the printed
  summary, confirm the counts look right (should match however many seeded
  users currently have each old userType).
- Run with `--apply` against the emulator, then check the Emulator UI:
  no doc still has `userType` of `decorator` or `supplier`.

### [x] 1.4 — Update `scripts/seed.mjs`

**Files:** `scripts/seed.mjs`

- Update seeded users to use `vendor` / `manufacturer` only.
- Give at least 2 of the seeded Manufacturers a non-empty `categories`
  array (pick from `LISTING_CATEGORIES`) so Phase 3 has real data to filter
  against.
- Update any seeded `requirement` listings' `sellerId` to point at a seeded
  **Vendor**, not a Manufacturer (Phase 2 will make Manufacturer-authored
  requirements impossible going forward — seed data should already be
  consistent with that rule).

**Verification:**
- Wipe the local emulator, re-run `node scripts/seed.mjs`, confirm no errors
  and the Emulator UI shows the expected new userType values + categories.

---

## PHASE 2 — Permission-Gated Views

Implements `DECISIONS.md` §5, §7. Depends on Phase 1 (needs the 2-role
`UserType`).

**RESOLVED 2026-08-01:** Needs tab (the browse-everyone's-requirements view)
is Manufacturer-only, confirmed. But hiding it from Vendor entirely would
break an already-built feature: the poster of a requirement sees their own
responders list on that post's detail page (`DECISIONS.md` §4/§15), and the
Needs tab was the only way to reach that page before now. Fix: after a
Vendor successfully creates a requirement post, redirect them straight to
that listing's detail page (`app/listing/[id].tsx`) instead of back to the
Post tab or Needs tab. This is the only way a Vendor reaches their own
requirement post until "My Listings" (roadmap item 5) ships — they can't
browse back to it later. Acceptable known gap, not silently swallowed.

### [x] 2.1 — Hide Needs tab for Vendor accounts, redirect Vendor to their own post instead

**Files:** `app/(tabs)/_layout.tsx`, `app/(tabs)/needs.tsx`,
`app/post/_components/PostForm.tsx`

- Tab layout must read the current user's `userType` (via
  `getCurrentUserProfile()`) and conditionally register the Needs tab only
  for `userType === 'manufacturer'`.
- `needs.tsx` itself should also guard at the top of the component (redirect
  or show nothing) if somehow reached by a Vendor — belt and suspenders,
  since a tab being hidden doesn't stop direct navigation.
- **`PostForm.tsx` fix (see the RESOLVED note above this task):** currently,
  `handleSubmit` calls `await createListing(input);` and discards the
  returned id; on success it shows a "View in Needs →" link that always
  routes to `/(tabs)/needs`. For `postType === 'requirement'` specifically:
  capture the returned listing id (`const listingId = await
  createListing(input);`) and change that success link to route to
  `/listing/${listingId}` instead of `/(tabs)/needs`, with label text like
  "View your post →" instead of "View in Needs →". Leave the behavior for
  all other post types unchanged (they still link to `/(tabs)/available`).

**Verification:**
- Manual: sign in as Manufacturer → Needs tab visible, works as before.
- Manual: sign in as Vendor → Needs tab not present in the tab bar.
- Manual: as Vendor, post a requirement → success message shows "View your
  post →" → tapping it opens that listing's own detail page (not the Needs
  tab), and the detail page's responder-list UI for the poster still works.
- Manual: as Vendor, attempt to navigate directly to the Needs route (e.g.
  via a manual `router.push` in dev tools or deep link) → confirm the guard
  in `needs.tsx` blocks it rather than silently rendering.

### [x] 2.2 — Gate the Post intent picker by role

**Files:** `app/(tabs)/post.tsx`, `constants/postTypes.ts`

- `post.tsx` currently shows `POST_TYPE_ORDER` unfiltered. Filter it by the
  current user's `userType`: Manufacturer sees `sell-used`, `sell-new`,
  `rental`, `catalog` (no `requirement`). Vendor sees all five (including
  `requirement`).
- Do not remove `requirement` from `POST_TYPES` / `POST_TYPE_ORDER` in
  `constants/postTypes.ts` — it's still a valid type, just role-gated at the
  picker level.

**Verification:**
- Manual: Manufacturer's Post screen shows 4 options, no "I Need Something."
- Manual: Vendor's Post screen shows all 5 options.

### [x] 2.3 — Guard direct navigation to the requirement form

**Files:** `app/post/requirement.tsx`

- Even with 2.2 done, a Manufacturer could still deep-link to
  `/post/requirement` directly. Add a guard at the top of this screen:
  if the current user's `userType !== 'vendor'`, redirect back (e.g. to
  `/(tabs)/post`) instead of rendering the form.

**Verification:**
- Manual: as Manufacturer, manually navigate to the requirement route →
  confirm redirect, not a working form.

### [x] 2.4 — Enforce requirement creation server-side in Firestore rules

**Files:** `firestore.rules`

- Add a rule so `listings` creation with `postType == 'requirement'` is only
  allowed when the creator's own `users/{uid}` doc has `userType == 'vendor'`.
  This needs a `get()` lookup inside the rule, e.g.:
  ```
  allow create: if request.auth != null
    && request.auth.uid == request.resource.data.sellerId
    && (
      request.resource.data.postType != 'requirement'
      || get(/databases/$(database)/documents/users/$(request.auth.uid)).data.userType == 'vendor'
    );
  ```
- This is the real permission boundary — Tasks 2.2/2.3 are UX, this is
  enforcement. Do not skip it because the UI already blocks it.
- Per the global guardrails: local file only, verified against the emulator,
  **not deployed to cloud** (Blaze still blocked).

**Verification:**
- Run the local emulator with the updated rules.
- Manual or scripted: attempt to create a `requirement` listing while signed
  in as a Manufacturer test account → must be rejected (permission-denied).
- Attempt the same as a Vendor test account → must succeed.
- Confirm all other existing listing creation (sell-used, sell-new, rental,
  catalog) still works for both roles — this rule must not break the
  non-requirement path.

---

## PHASE 3 — Matching Engine, Tier 1

Implements `DECISIONS.md` §17 (Tier 1 only — Tier 2 is explicitly out of
scope until Blaze clears). Depends on Phase 1 (`categories[]` field) and
Phase 2 (Needs tab already Manufacturer-only).

### [x] 3.1 — Sort/filter Needs tab by viewer's district + categories

**Files:** `app/(tabs)/needs.tsx`

- Load the current Manufacturer's profile (`getCurrentUserProfile()`) once
  on screen mount to get their `district` and `categories[]`.
- Keep the existing manual category/district filter chips as-is (user
  choice should always override auto-matching).
- When no manual filter is set, sort the fetched requirement list so that
  items matching the viewer's `district` AND at least one of their
  `categories[]` appear first, then items matching only `district`, then
  everything else — stable sort, don't hide non-matching items, just
  reorder. (Tier 1 is surfacing, not filtering — a Manufacturer should still
  be able to see everything, just with matches on top.)
- If the Manufacturer has zero `categories[]` set, fall back to sorting by
  `district` match only.

**Verification:**
- `npx tsc --noEmit` — zero errors.
- Manual: using the seeded data from Task 1.4, sign in as the Manufacturer
  seeded with specific categories/district, open Needs tab, confirm
  matching requirements appear above non-matching ones.
- Manual: sign in as a Manufacturer with empty `categories[]`, confirm it
  falls back to district-only sort without erroring.

---

## PHASE 4 — WhatsApp OTP Auth Rebuild

Implements `DECISIONS.md` §8. **Do not start until Phase 0 is confirmed
complete by Vishnu.** This phase touches auth, the highest-risk surface in
the app — go slower here than elsewhere, verify each sub-task before the
next.

### [ ] 4.1 — Scaffold the Cloudflare Worker project

**Files:** new directory, e.g. `w2d-auth-worker/` (sibling to the app, own
`package.json` — this is a separate deployable, not part of the Expo app
bundle).

- `wrangler` project with two routes: `POST /send-otp`, `POST /verify-otp`.
- Store secrets (Meta access token, Firebase service account JSON) via
  `wrangler secret put`, never committed to the repo.
- Add `w2d-auth-worker/.gitignore` covering any local secret files.

**STOP AND ASK:** if Vishnu hasn't already created/confirmed the Cloudflare
Workers project and Meta app credentials exist and are ready to paste in as
secrets. Do not generate placeholder secrets and pretend the flow works.

**Verification:** `wrangler dev` runs locally without errors (routes exist,
even before real logic is wired in).

### [ ] 4.2 — `/send-otp`: generate code + call Meta WhatsApp Cloud API

**Files:** `w2d-auth-worker/src/send-otp.ts` (or equivalent)

- Input: `{ phone: string }` (E.164, e.g. `+91XXXXXXXXXX`).
- Generate a 6-digit code, store it hashed with a short expiry (5–10 min) —
  Cloudflare KV is the simplest option (per `DECISIONS.md` §8).
- Call Meta's WhatsApp Cloud API with the approved OTP template, passing the
  code.
- Rate-limit per phone number (e.g. max 1 send per 30s, max 5 per hour) to
  avoid abuse — this endpoint is unauthenticated by definition (that's the
  point of an OTP), so it needs its own throttle.

**Verification:**
- Call the endpoint with a real test phone number (Vishnu's own), confirm a
  WhatsApp message arrives with a valid-looking code.
- Call it twice rapidly, confirm the rate limit kicks in on the second call.

### [ ] 4.3 — `/verify-otp`: check code + mint Firebase custom token

**Files:** `w2d-auth-worker/src/verify-otp.ts`

- Input: `{ phone: string, code: string }`.
- Look up the stored hashed code for that phone, compare, check expiry.
- On success: use the Firebase Admin SDK (service account secret from 4.1)
  to look up or create the Firebase Auth user for that phone number, then
  mint a custom token via `admin.auth().createCustomToken(uid)`.
- Return `{ token: (secret removed) }` on success, clear/invalidate the used code
  either way (single use).
- On failure: generic error, do not leak whether the phone exists or the
  reason in a way that helps enumeration.

**Verification:**
- Full round trip: send-otp → read the real code from WhatsApp → verify-otp
  → confirm a valid-looking JWT custom token comes back.
- Wrong code → rejected. Expired code → rejected. Reused code → rejected.

### [ ] 4.4 — Rewrite `app/(auth)/_lib/auth.ts` to call the Worker

**Files:** `app/(auth)/_lib/auth.ts`

- Replace `sendPhoneOtp` (currently `auth().signInWithPhoneNumber`) with a
  function that `fetch()`s the Worker's `/send-otp` endpoint.
- Replace `verifyPhoneOtp` (currently `confirmation.confirm(code)`) with a
  function that `fetch()`s `/verify-otp`, then calls
  `auth().signInWithCustomToken(token)` with the token it gets back. This is
  the key step that keeps every other file in the app (all the
  `auth().currentUser` reads throughout `firestore.ts`) working unchanged —
  Firebase Auth is still the identity system, only the OTP delivery
  mechanism changed.
- `getPendingConfirmation` / `setPendingConfirmation` /
  `clearPendingConfirmation` — the `FirebaseAuthTypes.ConfirmationResult`
  type goes away (that was specific to native phone auth). Replace with
  simply tracking the phone number being verified (a string), since the
  Worker holds the actual OTP state, not the client.
- `getAuthErrorMessage` — update the `switch` cases; the `auth/invalid-*`
  codes were Firebase's native phone-auth error codes and won't come back
  from the Worker. Map the Worker's error responses to user-facing messages
  instead (wrong code, expired code, rate-limited, network error).
- Keep `isValidIndianMobile`, `maskPhoneNumber`, `signOutUser`,
  `deleteCurrentUserAuthAccount`, `isRequiresRecentLoginError` as-is — none
  of these depend on the phone-auth-specific parts.

**Verification:** `npx tsc --noEmit` — zero errors (expect real type errors
here until `phone.tsx`/`otp.tsx` are also updated in 4.5 — that's fine,
finish 4.4 and 4.5 together before checking this).

### [ ] 4.5 — Update `phone.tsx` and `otp.tsx` to the new flow

**Files:** `app/(auth)/phone.tsx`, `app/(auth)/otp.tsx`

- `phone.tsx`: `handleSendOtp` calls the new `auth.ts` function; the "🇮🇳 +91"
  UI and validation (`isValidIndianMobile`) stay unchanged — only the number
  now also doubles as the WhatsApp number.
- `otp.tsx`: `handleVerify` calls the new verify function, still navigates
  to `/(auth)/profile-setup` on success. `handleResend` calls send-otp
  again. Update the footer copy — "Secure verification powered by
  Wedding2day Trust" can stay, but anything implying SMS specifically should
  say WhatsApp instead (check for any "SMS" wording in these two files).
- No layout/styling changes needed — this is a swap of what the buttons
  call, not a redesign.

**Verification:**
- `npx tsc --noEmit` — zero errors, now that 4.4 and 4.5 are both done.
- Full manual flow on a real device: enter phone → receive WhatsApp OTP →
  enter code → land on profile-setup → complete profile → land on
  Available tab, signed in.
- Confirm `auth().currentUser` is populated correctly after sign-in (e.g.
  Profile tab shows the right phone number) — this is the check that
  `signInWithCustomToken` wired up identity correctly.
- Resend flow works (request a second code, old code no longer valid).
- Wrong-code path shows a sensible error, doesn't crash.

### [ ] 4.6 — Remove now-dead native phone-auth surface

**Files:** check `app/_layout.tsx` and anywhere else `ensureAuthEmulator` /
Firebase Auth emulator connection for phone auth is referenced.

- The Auth emulator (port 9099) was used for native phone-auth testing.
  Confirm whether it's still needed (custom-token sign-in can still go
  through the Auth emulator in dev) — likely yes, don't remove the emulator
  wiring, just confirm it still works with the new flow.
- Do **not** touch Firestore/Storage emulator wiring — unrelated.

**Verification:** local dev flow (Task 4.5's manual test) works against the
Auth emulator, not just production, before considering Phase 4 done.

---

## PHASE 5 — Final Regression Pass

Run after Phases 1–4 (or 1–3, if Phase 4 is still blocked on Phase 0 — note
that explicitly in `PROGRESS.md` rather than leaving this phase silently
incomplete).

### [ ] 5.1 — Full manual walkthrough

- Sign up fresh as Vendor: profile setup (no categories field), post a
  requirement, confirm it does NOT appear for another Vendor's Needs tab
  (Vendor has none) and DOES appear for a Manufacturer's Needs tab, matched
  to the top if district/category overlap.
- Sign up fresh as Manufacturer: profile setup (categories field present),
  confirm Post screen has no "I Need Something" option, confirm direct
  navigation to `/post/requirement` redirects.
- Existing flows unaffected: sell-used/sell-new/rental/catalog posting,
  browse + filters on Available tab, reveal-phone flow, report flow, Delete
  Account, Sign Out — spot check each still works after the auth rewrite.

### [x] 5.2 — Update `PROGRESS.md`

- Add a dated entry (2026-08-01 or actual completion date) summarizing what
  shipped from this plan, in the same format as existing entries.
- Update the Phase status table if any of P3/P4/P6/P7 rows need a new note
  reflecting the role-model + auth rework.

### [x] 5.3 — Update `scripts/seed.mjs` doc comment / re-verify

- Re-run seed against a wiped emulator one final time end-to-end, confirm
  no errors, confirm the app displays seeded data correctly under the new
  role model.

---

## PHASE 6 — Feed Card Redesign (social-feed style, "Style B")

Not in `DECISIONS.md` originally — added 2026-08-01 after reviewing three
visual directions with Vishnu (Instagram-style, Facebook/LinkedIn-style,
marketplace grid). Chose Facebook/LinkedIn-style: business identity
(avatar, name, role badge, district) up front, photo, price overlay,
labeled action row — closest fit to the existing trust-context principle in
`DECISIONS.md` §6 (reveal screen already prioritizes who/where over the
photo). Does not touch the reveal-phone mechanic itself — business name is
shown pre-reveal (already standard on every reference marketplace cited in
`DECISIONS.md` §5), only the phone number stays gated.

**Explicitly not doing:** no fabricated stats. `viewCount`/`interestCount`
don't exist in the schema yet (`DECISIONS.md` §14 roadmap item 5, not
built) — the card must only show data that's real today. No "views" or
"likes" counters.

### [x] 6.1 — Denormalize seller identity onto listing docs

**Files:** `app/(auth)/_lib/firestore.ts`

- Add `sellerName: string`, `sellerBusinessName: string`, and
  `sellerUserType: UserType` to `ListingDoc` and to what `createListing`
  writes — pull from the current user's own profile at creation time
  (already available via `getCurrentUserProfile()` or equivalent — don't add
  a second Firestore read if the caller already has this data; check
  `PostForm.tsx`'s existing flow first). `sellerUserType` drives the
  Manufacturer/Vendor badge on the card in Task 6.2.
- Existing listings created before this change won't have these fields —
  `fetchListings`/`getListing` must tolerate `undefined` and the UI must
  fall back to a neutral label (e.g. "Seller", no role badge) rather than crash or show
  `undefined`.
- Do not backfill old documents — that's a migration, out of scope here,
  and there's no real user data yet (same reasoning as skipped Task 1.3).

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Post a new listing after this change, confirm `sellerName` /
  `sellerBusinessName` are present on the Firestore doc in the Emulator UI.
- Confirm an old seeded listing (without these fields) still renders
  without crashing once 6.2 is built.

### [x] 6.2 — Build a shared `ListingCard` component, replace duplicated inline card JSX

**Files:** new `app/_components/ListingCard.tsx`, `app/(tabs)/available.tsx`,
`app/(tabs)/needs.tsx`

- The card markup is currently duplicated inline in both `available.tsx`
  (`renderItem`, the `Pressable` block starting `overflow-hidden
  rounded-xl...`) and `needs.tsx` (its own separate `renderItem` card).
  Extract one shared `ListingCard` component both screens use — don't leave
  the duplication in place after this task.
- Layout, matching the approved Style B mockup exactly:
  - Header row: 32px circle avatar (initials from `sellerBusinessName`,
    same style already used for the Profile tab's account rows — reuse that
    pattern, don't invent a new avatar style), business name (14px/500),
    role-derived badge + district on the line below (`Manufacturer` /
    `Vendor` pill using the same red-tint badge style already used for
    postType badges elsewhere in the app).
  - Photo: 16:9, existing image/placeholder handling from the current card.
  - Price shown as a small overlay pill on the bottom-left of the photo
    (`#e53925` background, white text) instead of the current large text
    block below the photo.
  - Bottom bar: `border-top`, two segments — "Interested" (icon + label,
    tapping it navigates to the listing detail, same as tapping the card
    itself today — do not build a new in-card interest action, that
    belongs on the detail screen per the existing reveal-phone flow) and
    category + condition as the second segment. No third segment, no fake
    stats.
  - Requirement-type cards (`needs.tsx`): keep the existing "Needed" badge
    treatment, adapt the same header/bottom-bar structure rather than
    keeping `needs.tsx`'s separate simpler card design.
- Use existing color literals already used throughout the app
  (`#e53925`, `#1d1b1a`, `#56413e`, `#8a716c`, `#ddbfb9`, `#e5e2e1`,
  `#ffdad4`) — do not introduce new colors.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: Available tab and Needs tab both render the new card style,
  visually matching the approved mockup (header, photo+price overlay,
  bottom bar).
- Manual: tapping anywhere on the card (including the "Interested" segment)
  still navigates to the listing detail page — no new dead-end or broken
  navigation.
- Manual: an old seeded listing without `sellerName` shows "Seller" (or
  equivalent fallback) instead of blank/undefined/crash.
- Grep confirms no leftover duplicated card JSX in `available.tsx` or
  `needs.tsx` — both import and use `ListingCard`.

---

## PHASE 7 — Post Flow as a Floating Action + Modal Sheet

Added 2026-08-01, Vishnu's call after reviewing the nav mockup: Post should
not be its own tab. Currently `app/(tabs)/post.tsx` is a full tab (and its
own screen navigation) whose *contents* already change per role (Task 2.2),
which itself reads as inconsistent extra chrome. Moving it to a floating "+"
button that opens the post flow as a modal sheet — same pattern as
Instagram/Twitter's compose button — fixes two things at once: the tab bar
becomes identical in structure for every role (only Needs' presence differs,
which is the one difference that's actually meaningful), and posting no
longer feels like leaving to a separate section — the sheet rises over
whatever tab you were on and dismisses back to exactly that.

**Current structure (for reference):**
- `app/(tabs)/post.tsx` — intent picker, registered as a `Tabs.Screen` in
  `app/(tabs)/_layout.tsx`.
- `app/post/sell-used.tsx`, `sell-new.tsx`, `rental.tsx`, `catalog.tsx`,
  `requirement.tsx` — already top-level routes outside `(tabs)`, each
  rendering `PostForm`, reached via `router.push('/post/${id}')`.
- `app/_layout.tsx` — root `Stack`, currently only declares `index` and
  `(auth)` explicitly; `(tabs)` and `post/*` resolve implicitly.

### [ ] 7.1 — Move the intent picker out of the tabs, group the post routes as a modal stack

**Files:** move `app/(tabs)/post.tsx` → `app/post/index.tsx`; new
`app/post/_layout.tsx`; `app/_layout.tsx`

- Move the intent-picker component to `app/post/index.tsx` (same content,
  same role-filtering logic from Task 2.2 — don't change the filtering
  itself here, just the file location).
- Add `app/post/_layout.tsx`: a `Stack` with
  `screenOptions={{ presentation: 'modal', headerShown: false }}` wrapping
  `index` and the five existing type screens. This is what makes the whole
  `/post/*` subtree rise as one dismissible sheet — picking a type inside it
  pushes within the same modal card, it does not open a second stacked
  modal.
- Confirm the root `app/_layout.tsx` `Stack` picks this up correctly (Expo
  Router nested layouts should resolve automatically via the `post`
  directory's own `_layout.tsx` — verify, don't assume).
- Task 2.3's guard (redirect a Manufacturer away from `/post/requirement`)
  must keep working unchanged at its new effective location — same
  component, same logic, just confirm it still fires now that entry is via
  modal push rather than tab navigation.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: navigating to `/post` (however it's triggered — even before 7.2
  wires up the FAB, you can test by temporarily hitting the route directly)
  rises as a modal sheet over the current tab, not a full-screen replace.
- Manual: picking a post type inside the sheet shows that type's form
  within the same sheet (no second modal stacking on top).
- Manual: Manufacturer navigating to the requirement form still redirects
  (Task 2.3 regression check).

### [ ] 7.2 — Add the floating action button, remove the Post tab

**Files:** `app/(tabs)/_layout.tsx`

- Remove the `post` `Tabs.Screen` entry entirely.
- Add a custom floating button: wrap the existing `<Tabs>` in a `<View
  style={{ flex: 1 }}>`, add an absolutely positioned `Pressable` centered
  at the bottom, overlapping the tab bar (44px red circle, white `+` icon,
  negative top offset so it sits above the bar — matches the approved
  mockup). `onPress` → `router.push('/post')`.
- This button must render identically regardless of `userType` — the
  earlier confusion (Task 2.2) was tab *contents* differing by role, not
  the button itself; keep the button role-agnostic, let the modal's own
  content (already role-filtered per 2.2) handle the difference.
- Remaining tabs after this change: Available, Needs (Manufacturer only,
  unchanged from Task 2.1), Profile. Same count and order for every role
  except Needs' presence — that's the intended, meaningful difference now
  that Post is no longer a tab.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: tab bar shows Available / Needs / Profile for Manufacturer,
  Available / Profile for Vendor, with the floating + button in both,
  matching the approved mockup.
- Manual: tapping + from any tab opens the post sheet; dismissing it
  (back gesture or a close action) returns to the exact tab/scroll position
  you were on, not a reset to Available.

### [ ] 7.3 — Fix post-success navigation for the modal context

**Files:** `app/post/_components/PostForm.tsx`

- The existing success state's "View in Needs →" / "View your post →"
  links (added in Task 2.1) currently use `router.replace(...)`. Inside a
  modal stack, `replace` can behave unexpectedly (it may replace within the
  modal rather than dismissing it). Change these to first dismiss the modal
  (`router.dismiss()` or `router.back()` — confirm which Expo Router API is
  correct for this Stack setup) and then navigate to the target tab/detail
  screen underneath.
- Re-verify the specific case from Task 2.1: Vendor posts a requirement →
  success → "View your post →" → lands on that listing's own detail page,
  not stuck inside the dismissed modal or on a blank screen.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual, all five post types: submit → success message → tapping the
  "View..." link closes the sheet and lands on the correct destination
  (Available tab for supply types, the listing detail page for a Vendor's
  requirement).
- Manual: submitting, then dismissing the sheet *without* tapping "View...”
  (just closing it) — confirm you land back on whatever tab you opened the
  sheet from, not somewhere unexpected.

---

## PHASE D — Demo Readiness Extras (added 2026-08-01, for tomorrow's client demo)

Not part of the original roadmap sequence — added because Play Store
readiness and demo readiness are different goals with different timelines
(Google Play closed testing's 14-day minimum and the Blaze billing bug are
explicitly out of scope for tonight, left as-is). These two tasks close the
gap between "looks unfinished" and "looks complete" for a demo, without
faking anything that can't actually be verified tonight.

### [x] D.1 — Re-skin phone auth as WhatsApp (cosmetic only — not the real Meta integration)

Manual verification pending — Vishnu to confirm on device.

**Overnight note:** Cosmetic copy/icon only. WhatsApp-green `#25D366` icon on phone input and OTP header; button label "Send OTP via WhatsApp". Underlying `sendPhoneOtp` / Firebase native phone auth unchanged.


**Files:** `app/(auth)/phone.tsx`, `app/(auth)/otp.tsx`

- **Do not touch the underlying auth mechanism.** `sendPhoneOtp` /
  `verifyPhoneOtp` in `auth.ts` keep using Firebase's native
  `signInWithPhoneNumber` exactly as today — it's already tested and works.
  Phase 4 (the real Cloudflare Worker + Meta Cloud API integration) is
  unchanged and still deferred, blocked on Meta approval same as before.
  This task is copy and iconography only.
- `phone.tsx`: change "Send OTP" flow copy to reference WhatsApp (e.g. a
  WhatsApp icon next to the phone input, "We'll send a code to your
  WhatsApp" instead of any SMS-implying text).
- `otp.tsx`: same — "We sent a 6-digit code to your WhatsApp" instead of
  the current phrasing, WhatsApp-green accent on the icon if one is added
  (per `DESIGN_SYSTEM.md` — green is reserved for actual WhatsApp
  touchpoints, this qualifies).
- Do not change `maskPhoneNumber`, OTP input behavior, resend logic, or
  error handling — cosmetic only, zero functional risk the night before a
  demo.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: full sign-in flow still works exactly as before (this is the
  same tested flow — the only thing that should look different is copy/
  icons, not behavior).

### [x] D.2 — In-app Terms of Service and Privacy Policy screens

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Also edited `app/_layout.tsx` (register `legal` stack) and added `app/legal/_layout.tsx` + `app/legal/_components/LegalScreen.tsx` shared shell — needed for Expo Router nesting; not in original file list. Privacy in-app copy lightly updated Vendor/Manufacturer wording for demo accuracy (docs markdown files left as-is).


**Files:** new `app/legal/terms.tsx`, new `app/legal/privacy.tsx`,
`app/(auth)/phone.tsx`

- Both `docs/TERMS_OF_SERVICE.md` and `docs/PRIVACY_POLICY.md` already
  have real drafted content — render it in-app (simple scrollable text
  screen, reuse the app's existing typography/color tokens from
  `DESIGN_SYSTEM.md`, no need for markdown rendering library, plain
  `Text` components are fine for this).
- `phone.tsx` currently has "Terms of Service" and "Privacy Policy" as
  static styled text with no link (confirmed — no `onPress`/`Linking` on
  either). Wire both to navigate to the new screens.
- This is a real fix, not a mock — the content exists, it just wasn't
  reachable from the app. Not a substitute for actually publishing the
  Privacy Policy to a public URL (still needed for the real Play Store
  submission later, per the roadmap tail item 17) — this only makes it
  visible inside the app itself, which is what a demo needs.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: tapping "Terms of Service" and "Privacy Policy" on the phone
  entry screen opens the correct content, back navigation returns to
  sign-in correctly.

---

## PHASE 8 — Data Model Catch-Up

`DECISIONS.md` §3 locks `deliveryOption`, `negotiable`, `neededBy`,
`viewCount`, `interestCount` as listing fields. None of the five exist in
`ListingInput`/`ListingDoc`/`PostForm.tsx` today — `DECISIONS.md` was ahead
of the code from 2026-07-25 onward and nothing has closed the gap. This
phase does, before anything else builds on top of the data model.

### [x] 8.1 — Add `deliveryOption` and `negotiable` to the listing form and schema

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Also updated `ListingCard.tsx` and `app/listing/[id].tsx` as the task itself requires (listed in task body). Default delivery = `pickup-only` when price field is shown.


**Files:** `app/(auth)/_lib/firestore.ts`, `app/post/_components/PostForm.tsx`

- `ListingInput`/`ListingDoc`: add `deliveryOption: 'pickup-only' |
  'delivery-available' | 'both'` and `negotiable: boolean`.
- `createListing` writes both; default `negotiable` to `false` if unset.
- `PostForm.tsx`: add a 3-option picker for delivery (reuse the existing
  bottom-sheet picker pattern already in this file) and a checkbox/toggle
  for negotiable, shown next to the price field per `DECISIONS.md` §15
  ("shown as a badge next to price"). Both fields apply to every post type
  that has a price — check `meta.fields.price` to decide visibility, same
  pattern the file already uses for condition/quantity.
- `ListingCard.tsx` (Phase 6): show the negotiable badge next to price if
  `negotiable === true`. Delivery option doesn't need to be on the card
  itself — save it for the detail screen.
- `listing/[id].tsx`: show both fields in the detail view.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: post a new sell-used listing with delivery = "both" and
  negotiable = true, confirm both save correctly (check Emulator UI) and
  render on the detail screen; confirm the negotiable badge shows on the
  feed card.

### [x] 8.2 — Add `neededBy` to requirement posts

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Date entry is plain `YYYY-MM-DD` TextInput (no datetimepicker in deps). Needed-by display lives on shared `ListingCard` (used by needs.tsx) rather than duplicating in needs.tsx itself. Fixed pre-existing `Flower Decoration` → `Flower Decorations` spelling to match `DECISIONS.md` §9 / `ListingCategory` type (also updated `scripts/seed.mjs`).


**Files:** `app/(auth)/_lib/firestore.ts`, `app/post/_components/PostForm.tsx`,
`app/(tabs)/needs.tsx`, `app/listing/[id].tsx`

- `ListingInput`/`ListingDoc`: add `neededBy: string | null` (ISO date),
  requirement posts only.
- `PostForm.tsx`: optional date field, shown only when `postType ===
  'requirement'`.
- `needs.tsx`: show the date on the card if present (e.g. "Needed by 15
  Aug").
- Detail screen: show it prominently near the title for requirement posts.
- Do not build the "usable as an auto-expiry trigger" half of this yet
  (`DECISIONS.md` §15 mentions it) — that needs a scheduled job / Cloud
  Function, which is Phase 12 (Ops Hygiene) territory, not this task. Just
  store and display the date for now.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: post a requirement with a needed-by date, confirm it saves,
  shows on the Needs tab card and the detail screen.

### [x] 8.3 — Add `viewCount` and `interestCount`, increment them for real

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Also updated `firestore.rules` (not in original file list) so any signed-in user can increment only `viewCount` / `interestCount` by +1 — without this, counter updates were seller-only and would silently fail / break interest. Local rules only; not deployed to cloud (Blaze still blocked).


**Files:** `app/(auth)/_lib/firestore.ts`, `app/listing/[id].tsx`

- `ListingDoc`: add `viewCount: number`, `interestCount: number`, both
  default `0` at creation.
- `getListing`: increment `viewCount` by 1 each time it's called (use
  `firestore.FieldValue.increment(1)` in an update alongside the read, or a
  separate fire-and-forget update — don't block the read on it).
- `expressInterest`: currently only writes to the `interests` collection.
  Also increment the listing's `interestCount` via
  `FieldValue.increment(1)` — but only on the dedupe-safe path (don't
  double-count if the interest doc already existed; check the existing
  `if (!snap.exists)` branch and increment inside it, not outside).
- `listing/[id].tsx`: show both counts to the listing's owner only
  (`isOwnListing` check already exists in this file) per `DECISIONS.md`
  §15 "Seller-facing stats" — not shown to other viewers.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: view a listing as a non-owner twice, confirm `viewCount` is 2 in
  the Emulator UI (not incrementing per-render, just per screen visit).
- Manual: express interest once, confirm `interestCount` is 1; try again
  from the same account (if possible) and confirm it does not double-count.
- Manual: as the owner, confirm counts are visible; as a non-owner, confirm
  they are not shown anywhere on the screen.

---

## PHASE 9 — Catalog Rebuild (profile-only, per `DECISIONS.md` §4)

`DECISIONS.md` §4 locked catalog as profile-only back on 2026-07-25 —
`users/{uid}/catalogItems/{itemId}`, not the `listings` collection, not the
Available feed. The code never caught up: `constants/postTypes.ts` still
has `catalog` as a regular post type with `tab: 'available'`, and
`app/post/catalog.tsx` still posts through the same `PostForm`/`listings`
path as everything else. This phase fixes that, and builds the Instagram-
grid "My Catalog" screen (`DESIGN_SYSTEM.md`) that's currently a dead
placeholder row on Profile.

**Judgment call (overnight override of STOP AND ASK):** Proceeding with full removal of catalog from the Available feed per locked `DECISIONS.md` §4 (profile-only). Behavior removal is intentional and documented; Vishnu can reverse if demo feedback requires feed-based catalog temporarily.

### [x] 9.1 — Add `catalogItems` subcollection functions

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Also updated `firestore.rules` (catalogItems subcollection) and `storage.rules` (`catalog-photos/{uid}/`) — required for emulator writes to succeed. Extended `deleteCurrentUserFirestoreData` to clear catalog items. Seed script now seeds 6 catalogItems under users instead of catalog listings.


**Files:** `app/(auth)/_lib/firestore.ts`

- New functions: `createCatalogItem(input)` (writes to
  `users/{uid}/catalogItems`, fields: title, category, photos, price,
  description — no quantity, no status, no expiry per §4), `fetchOwnCatalog()`,
  `fetchCatalogForUser(uid)` (for viewing someone else's catalog from their
  profile), `deleteCatalogItem(itemId)`.
- Reuse `uploadListingPhoto`'s pattern for catalog photos, but a distinct
  storage path (e.g. `catalog-photos/{uid}/`) — don't mix into
  `listing-photos/`.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual/scripted: create a catalog item via these functions against the
  emulator, confirm it lands in `users/{uid}/catalogItems`, not `listings`.

### [x] 9.2 — Remove catalog from the feed post-type system

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Introduced `IntentOption` + `intentOptionsForRole` / `CATALOG_INTENT` so the Post picker still shows "Add to Catalog" while `PostType` no longer includes catalog. Also updated `scripts/seed.mjs` (remove catalog listings). Available feed filters to `meta.tab === 'available'` so legacy catalog docs are excluded.


**Files:** `constants/postTypes.ts`, `app/post/catalog.tsx`, `app/(tabs)/post.tsx`
(or `app/post/index.tsx` if Phase 7 already moved it), `app/(tabs)/available.tsx`

- Remove `catalog` from `PostType`/`POST_TYPES`/`POST_TYPE_ORDER` in
  `constants/postTypes.ts` — it's no longer a `listings` post type.
- `app/post/catalog.tsx`: replace its `PostForm`-based implementation with
  a new minimal catalog-item form using Task 9.1's functions (title,
  category, photo(s), price, description — no condition, no quantity, no
  district since it's tied to the profile not a location-scoped post).
- Post intent picker: "Add to Catalog" still routes here (per
  `DECISIONS.md` §5 — the intent picker already anticipated this), just
  confirm the route target is the new catalog form.
- `available.tsx`: since `catalog` no longer exists as a `PostType`, this
  should mostly resolve itself once the type is removed — but check the
  `AVAILABLE_POST_TYPES` filter list and remove `catalog` from it
  explicitly, and re-verify the filter picker doesn't show a dead "Catalog"
  option.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: Available tab no longer has a "Catalog" filter option, and no
  catalog items appear in the feed.
- Manual: posting a new catalog item via the intent picker goes to the new
  form, saves to the subcollection, does not appear in Available.

### [x] 9.3 — Build "My Catalog" (Instagram-grid, per `DESIGN_SYSTEM.md`)

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Built `app/catalog/[uid].tsx` for both own and others' catalogs (reuse as planned). Viewing someone else's catalog uses `getSellerContact` for Call/WhatsApp — no separate public-profile screen. Own catalog shows district/role from profile; other catalogs may omit district when contact-only. Item detail is a bottom sheet modal, not a separate route.


**Files:** new `app/catalog/[uid].tsx` (viewing any user's catalog, reused
for both "my own" and viewing someone else's from their profile — check if
a public profile-view screen exists yet; if not, this may need to be
scoped down to "my own catalog only" and note the "viewing someone else's"
case as a follow-up, don't silently build a whole new public-profile screen
as a side effect of this task), `app/(tabs)/profile.tsx`

- Grid layout matching the approved mockup: identity header (avatar,
  business name, role, district — reuse the `Avatar` pattern from
  `DESIGN_SYSTEM.md`), item count, 3-column photo grid, tapping an item
  opens a simple detail view (title, price, description — no reveal-phone
  flow needed here, catalog items aren't posts with sellers to reveal,
  they're a profile's own listing of what they make — but do surface a way
  to reach that seller's phone via the profile itself, reusing
  `getSellerContact`).
- Wire the "My Catalog" row in `profile.tsx` (currently `console.log`
  no-op) to navigate here instead.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: tapping "My Catalog" from Profile navigates to the grid, shows
  items created in 9.1/9.2, empty state shows if none exist yet (don't
  leave a blank white screen).
- Manual: tapping a grid item shows its detail correctly.

---

## PHASE 10 — Settings Sub-Screens (Change Phone Number, My Interests, Notification Preferences UI)

The three remaining dead placeholder rows on Profile besides My Catalog.
Notification Preferences is UI-only in this phase — it stores a preference,
it does not yet control real push delivery, since push infrastructure
(Phase 11) doesn't exist yet. Don't let this task silently grow into
building FCM.

### [x] 10.1 — Change Phone Number flow

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Used `app/settings/change-phone.tsx` (not `(tabs)/_settings/…`) because Expo Router `_` folders are private and not routed. Uses Firebase native `verifyPhoneNumber` + `updatePhoneNumber` (Phase 4 WhatsApp follow-up still needed). Auth UID preserved; Firestore `users/{uid}.phone` updated after Auth succeeds.


**Files:** new `app/(tabs)/_settings/change-phone.tsx` (or similar path —
match whatever route convention Phase 7 established for modal sub-screens),
`app/(auth)/_lib/auth.ts`, `app/(tabs)/profile.tsx`

- Re-verification flow: enter new number → OTP → on success, update the
  Firebase Auth account's phone number and the `users/{uid}.phone` field,
  keeping the same UID (per `DECISIONS.md` §8: "re-linked to the same
  account/UID, preserving listing and interest history — do not treat a
  SIM/WhatsApp number change as an account-loss event").
- **If Phase 4 (WhatsApp OTP) hasn't shipped yet when this task runs**, this
  re-verification must use whatever OTP mechanism is currently live (still
  Firebase native phone auth until Phase 4 ships) — don't block this task
  on Phase 4, just use what exists at the time and note in `PROGRESS.md`
  that it'll need a follow-up pass once Phase 4 lands.
- Wire the "Change Phone Number" row in `profile.tsx` to this flow.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: change phone number end to end on the emulator, confirm the
  profile shows the new number afterward and existing listings/interests
  under that UID are untouched.

### [x] 10.2 — My Interests (WhatsApp-style list, per `DESIGN_SYSTEM.md`)

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Route at `app/interests/index.tsx`. Client-side sort newest-first (avoids composite index). Flat list rows per DESIGN_SYSTEM, not cards.


**Files:** new `app/interests/index.tsx`, `app/(auth)/_lib/firestore.ts`,
`app/(tabs)/profile.tsx`

- New function `fetchMyInterests()`: query `interests` where `buyerId ==`
  current uid, resolve each to its listing (title, price, thumbnail,
  createdAt).
- Flat list rows per `DESIGN_SYSTEM.md` (thumbnail left, title + price
  center, relative time right) — not cards, matches the WhatsApp list
  pattern, distinct from the feed's card style.
- Tapping a row goes to that listing's detail page.
- Wire "My Interests" row in `profile.tsx` to this screen.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: express interest in a couple of listings, open My Interests,
  confirm both show, newest first, tapping one opens the right listing.
- Manual: empty state (no interests yet) shows a sensible message, not a
  blank screen.

### [x] 10.3 — Notification Preferences (UI + storage only, no push wiring yet)

Manual verification pending — Vishnu to confirm on device.

**Judgment call:** Route at `app/settings/notifications.tsx`. Defaults both prefs to `true` when absent on older user docs. Saves on toggle via dotted-field update (`notificationPrefs.newMatches` / `postApproved`). No FCM / push logic.


**Files:** new `app/(tabs)/_settings/notifications.tsx` (match Phase 10.1's
routing convention), `app/(auth)/_lib/firestore.ts`, `app/(tabs)/profile.tsx`

- Add a `notificationPrefs` map to the `users` doc (e.g. `{ newMatches:
  boolean, postApproved: boolean }` — keep it small, these two map directly
  to the two push use cases already named in `DECISIONS.md` §14/§15:
  matching-engine notify and post-approval alerts).
- Simple toggle-switch screen, saves directly to Firestore on change (no
  separate "Save" button — matches how a settings screen should feel).
- Wire "Notification Preferences" row in `profile.tsx` to this screen.
- **Do not build any actual push-sending logic here** — that's Phase 11.
  This task only makes the preference exist and be readable later.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: toggle a preference, confirm it persists in Firestore and
  survives leaving/reopening the screen.

---

## ROADMAP TAIL — Phases 11+ (scoped, not yet broken into granular tasks)

Sequenced, not yet written to Task-list detail — do this the same way
Phases 1–10 were: write the granular breakdown immediately before starting
each one, not all at once now, since scope may shift by the time these are
reached.

11. **Push notifications (FCM)** — infrastructure. Unblocks: real delivery
    for Phase 10.3's preferences, post-approval alerts (`DECISIONS.md`
    §15), and Tier 2 of the matching engine (`DECISIONS.md` §16, currently
    gated on the Blaze billing bug anyway — so this can wait for that to
    clear rather than being urgent).
12. **My Listings** — mark-as-sold/unavailable toggle, listing expiry,
    free-text search, per-listing view/interest count display (Phase 8.3
    already stores the counts; this is where they're surfaced to the owner
    in a dedicated list rather than only on each listing's own detail
    page). WhatsApp-style list rows, same pattern as Phase 10.2.
13. **Reveal-screen trust-context upgrade** — add member-since date and
    active-listing count to the existing reveal card (`listing/[id].tsx`
    already has name/business/phone/Call/WhatsApp; missing the rest of
    `DECISIONS.md` §6's trust-context list). Also: verification badge
    (soft GST/business-name field, `DECISIONS.md` §15) and the `blocks`
    collection / block feature, neither of which exist in code yet.
14. **Cold-start onboarding** — 3-screen intro after signup
    (`DECISIONS.md` §15), now doing double duty explaining both
    Available/Needs and the Vendor/Manufacturer role split
    (`UX_SIMPLIFICATION_v1.2.md` item 3).
15. **Ratings/reviews** — star rating + written review after a phone
    reveal (`DECISIONS.md` §15). Not started at all currently.
16. **Ops hygiene** — daily reveal cap per account, spam/abuse throttling,
    Crashlytics/Analytics visibility, `neededBy` auto-expiry (deferred from
    Phase 8.2).
17. **Release-gate cleanup** — confirm the Terms of Service link in
    `phone.tsx` actually opens `docs/TERMS_OF_SERVICE.md` content (currently
    just styled text, unclear if it links anywhere — verify first, this
    might already be broken); publish `docs/PRIVACY_POLICY.md` to a public
    URL and complete the Play Store Data Safety form (both partly external/
    Vishnu tasks, not pure Cursor work).
18. **Admin web app** — own project, own stack decision, sequenced last per
    `DECISIONS.md` §11. Not scoped here — deserves its own planning pass
    when reached, not a subsection of this file.

---

## OVERNIGHT UNATTENDED RUN — SUMMARY (2026-08-01 → morning)

Automated bar for tonight: `npx tsc --noEmit` (EXIT 0 at end of run) +
Firestore emulator seed re-verify. Manual/device checks left for Vishnu.

### Checkbox status (tonight's scope)

| Task | Status | Automated verify |
|---|---|---|
| D.1 WhatsApp cosmetic reskin | [x] | tsc clean |
| D.2 In-app Terms + Privacy | [x] | tsc clean |
| 8.1 deliveryOption + negotiable | [x] | tsc + seed fields present |
| 8.2 neededBy | [x] | tsc + seed 6 requirements with neededBy |
| 8.3 viewCount + interestCount | [x] | tsc + rules updated; seed defaults 0 |
| 9.1 catalogItems functions | [x] | tsc + seed 6 catalogItems under users |
| 9.2 remove catalog from feed | [x] | tsc + seed listing types = 4 only (no catalog) |
| 9.3 My Catalog grid | [x] | tsc clean |
| 10.1 Change Phone Number | [x] | tsc clean |
| 10.2 My Interests | [x] | tsc clean |
| 10.3 Notification Preferences | [x] | tsc clean |

Explicitly **not** touched (out of scope): Phase 4, Google Play testing,
Blaze billing bug, Data Safety form. Phase 7 (FAB/modal post) also left
as-is — not in tonight's sequence.

### Judgment calls made

1. **D.2 routes:** Registered `legal` stack in `app/_layout.tsx`; added
   shared `LegalScreen` helper (not in original file list).
2. **8.2 date input:** Plain `YYYY-MM-DD` TextInput (no datetimepicker dep).
3. **8.2 / constants:** Fixed `Flower Decoration` → `Flower Decorations` to
   match `DECISIONS.md` §9 / `ListingCategory` (also seed.mjs).
4. **8.3 firestore.rules:** Allowed any signed-in user to +1 only
   `viewCount` or `interestCount` (seller-only update would break counters).
   Local rules only — not deployed to cloud.
5. **Phase 9 STOP AND ASK overridden:** Removed catalog from Available feed
   per locked `DECISIONS.md` §4. Catalog remains as intent-picker option via
   new `IntentOption` / `CATALOG_INTENT` (not a `PostType`).
6. **9.x rules/storage:** Added `users/{uid}/catalogItems` rules and
   `catalog-photos/{uid}/` storage rules so emulator writes succeed.
7. **9.3 scope:** `app/catalog/[uid].tsx` serves own + others' catalogs;
   others get Call/WhatsApp via `getSellerContact` (no new public-profile
   screen). Item detail = bottom sheet.
8. **10.x routes:** Used `app/settings/*` and `app/interests/` instead of
   `(tabs)/_settings/` because Expo Router `_` folders are private / not
   routed.
9. **10.1 auth:** Firebase native `verifyPhoneNumber` + `updatePhoneNumber`
   (Phase 4 WhatsApp Worker follow-up still required). Same UID preserved.
10. **10.3 prefs:** Default both toggles `true` when field absent on older
    user docs. UI-only — no push.

### Emulator seed re-verify (end of run)

- 25 listings: sell-used 7 / sell-new 6 / rental 6 / requirement 6 (no catalog)
- 6 `catalogItems` under `users/{uid}/catalogItems`
- Sample listing has `deliveryOption`, `negotiable`, `viewCount: 0`,
  `interestCount: 0`; all 6 requirements have `neededBy`

### Pending Vishnu — device / human pass before demo

- pending Vishnu: D.1 full sign-in flow still works; copy/icons look right
- pending Vishnu: D.2 Terms / Privacy links from phone screen + back nav
- pending Vishnu: 8.1 post sell-used with delivery=both + negotiable; card badge + detail
- pending Vishnu: 8.2 requirement with needed-by date on Needs card + detail
- pending Vishnu: 8.3 view twice → viewCount 2; interest once → interestCount 1; owner-only stats UI
- pending Vishnu: 9.2 Available has no Catalog filter; new catalog item not in feed
- pending Vishnu: 9.3 Profile → My Catalog grid, empty state, item sheet, Add to Catalog
- pending Vishnu: 10.1 change phone end-to-end on emulator/device (OTP + profile phone update)
- pending Vishnu: 10.2 My Interests list after revealing a couple of phones
- pending Vishnu: 10.3 notification toggles persist across reopen
- pending Vishnu: Confirm catalog-out-of-feed is OK for tomorrow's demo (judgment call #5)
- pending Vishnu: Phase 4 still blocked on Meta Business + WhatsApp template approval
- pending Vishnu: No git commit/push was made tonight (override: prefer documenting)

### Blockers worked around

- No physical device access → manual steps noted, not blocking checkboxes
- Phase 9 STOP AND ASK → proceeded per overnight override + DECISIONS.md §4
- Counter increments needed rules change beyond task file list → documented
- Expo Router `_settings` private folder → used `app/settings/` instead
- No datetimepicker package → YYYY-MM-DD text for neededBy

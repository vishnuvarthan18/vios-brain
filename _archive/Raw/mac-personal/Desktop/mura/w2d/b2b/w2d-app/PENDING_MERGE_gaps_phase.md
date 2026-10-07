# Staged — merge into BUILD_PLAN_v1.2.md once the overnight run is confirmed done

> Do not hand this to Cursor until the current overnight run (Phase D, 8, 9,
> 10) is finished and `BUILD_PLAN_v1.2.md` is confirmed safe to edit again.
> Once merged, this becomes "Phase 11" (renumber the existing roadmap tail
> down by one, or append as the new next phase — whichever keeps the
> existing numbering least disrupted at merge time).

Source: `GAPS_AUDIT.md`. Only the genuinely Cursor-buildable items — see
`VISHNU_TASKS.md` for everything that isn't code.

---

## PHASE — Infrastructure & Launch-Readiness Gaps

### [ ] G.1 — Paginate the listings feed

**Files:** `app/(auth)/_lib/firestore.ts`, `app/(tabs)/available.tsx`,
`app/(tabs)/needs.tsx`

- `fetchListings()` currently has no `.limit()` — fetches the entire
  collection every load. Add cursor-based pagination: `limit(20)` +
  `startAfter(lastDoc)`, return the last document alongside the results so
  callers can request the next page.
- `available.tsx`/`needs.tsx`: wire `onEndReached` on the existing
  `FlatList`s to fetch the next page, append rather than replace. Existing
  client-side filters (category/district/price) still apply to whatever
  page is loaded — do not attempt server-side filtering as part of this
  task, that's a bigger change, out of scope here.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: with more than 20 seeded listings, confirm the feed loads the
  first page, then loads more on scroll, without duplicating or skipping
  items.

### [ ] G.2 — Compress photos before upload

**Files:** `app/(auth)/_lib/firestore.ts` (or a new
`app/(auth)/_lib/image.ts`), `app/post/_components/PostForm.tsx`

- Add `expo-image-manipulator` (new dependency — flag it, don't add
  silently) to resize/compress images client-side before
  `uploadListingPhoto` runs (e.g. max 1600px on the long edge, JPEG quality
  ~0.7).
- Apply to every photo upload path (post creation, catalog items once
  Phase 9 exists).

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: upload a large photo (multi-MB camera original), confirm the
  uploaded file in Storage is meaningfully smaller, image still looks
  correct in the app.

### [ ] G.3 — Terms of Service consent record

**Files:** `app/(auth)/_lib/firestore.ts`, `app/(auth)/profile-setup.tsx`

- Add `termsAcceptedAt: Timestamp` to the user profile, set at profile
  creation (`createUserProfile`).
- `profile-setup.tsx`: add a real checkbox ("I agree to the Terms of
  Service and Privacy Policy", linking to the screens from Phase D.2) —
  required to submit, not just decorative link text.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: cannot complete signup without checking the box; the resulting
  profile doc has `termsAcceptedAt` set.

### [ ] G.4 — Crash reporting

**Files:** `app.json`, `app/_layout.tsx`, `package.json`

- Add Firebase Crashlytics (`@react-native-firebase/crashlytics` — new
  dependency, flag it) or Sentry, whichever fits the existing Firebase-
  centric stack better — default to Crashlytics for consistency with the
  rest of the Firebase usage unless there's a reason not to.
- Wire basic initialization only — this task is "crashes get reported
  somewhere," not a full observability build-out.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: trigger a test crash/error, confirm it shows up in the
  Crashlytics dashboard.

### [ ] G.5 — Deep-link sharing for a listing

**Files:** `app/listing/[id].tsx`, `app.json` (Expo Router deep linking is
mostly automatic, but confirm the scheme/config supports opening a shared
link back into this exact screen)

- Add a "Share" action on the listing detail screen — uses React Native's
  `Share` API to share a link (e.g. `https://wedding2day.com/listing/{id}`
  or an Expo Router deep link, whichever the app is actually configured
  to handle) with a short pre-filled message, targeting WhatsApp share
  naturally since that's the share sheet default on most Indian Android
  phones.
- Confirm opening that link (from outside the app) actually lands on the
  correct listing — this requires the deep link to be handled, not just
  generated. If deep linking isn't currently configured at all, this task
  may need to expand into that setup — flag it if so rather than silently
  shipping a share button that doesn't deep-link anywhere useful.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: share a listing, send the resulting link to yourself via
  WhatsApp, tap it, confirm it opens the app directly to that listing.

### [ ] G.6 — Basic rate limiting on posting

**Files:** `firestore.rules`, `app/post/_components/PostForm.tsx`

- Simplest version: track `lastPostAt` on the user doc, client-side check
  before allowing a new post within some short window (e.g. 60 seconds) —
  full server-side enforcement would need Cloud Functions, which is
  Blaze-gated same as everything else Cloud-Function-based in this
  project, so keep this client-side-only for now and note that as a known
  limitation, not attempt to work around Blaze here.

**Verification:**
- `npx tsc --noEmit` — zero new errors.
- Manual: attempt to post twice in rapid succession, confirm the second
  attempt is blocked with a clear message, not silently dropped.

---

## Explicitly not staged here (see VISHNU_TASKS.md instead)

Git backup, Play Store assets, Meta verification, Privacy Policy public
hosting, Data Safety form, account recovery policy decision — none of
these are Cursor tasks.

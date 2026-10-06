# W2D — Play Console "Data Safety" form: draft answers

> **This is a drafting artifact, not a submission.** The Data Safety form lives
> in Play Console and can only be filled there by a human. Paste these answers
> across; nothing here writes to Play.
>
> Drafted 2026-09-01 (overnight Batch 8) from the code as it stands at that
> date. **Every row is evidenced against a file below.** Where W2D does *not*
> collect something, it is listed in §4 with the reason — over-declaring is a
> real risk on this form, and several of the near-misses here are things a
> reasonable person would assume the app does.
>
> Companion documents: `docs/PRIVACY_POLICY.md` (generated from
> `app/legal/_content/privacy.ts`) and `docs/PLAY_CONTENT_RATING.md`. The
> policy is hosted at `https://wedding2day-a99ea.web.app/privacy` once Batch 7
> is deployed; the deletion URL is `.../delete-account`.

---

## 1. The three form-wide answers

| Question | Answer | Why |
|---|---|---|
| Does your app collect or share any of the required user data types? | **Yes** | Phone auth, business profile, posts, photos, and automatic Analytics/Crashlytics. |
| Is all of the user data collected by your app encrypted in transit? | **Yes** | Every write goes through Firebase SDKs (Firestore, Auth, Storage, Analytics, Crashlytics) over TLS. There is no custom network layer in the app — no `fetch`/`axios` to any endpoint of ours. |
| Do you provide a way for users to request that their data be deleted? | **Yes** | In-app: Profile → Delete Account, OTP-verified (`deleteCurrentUserFirestoreData()`). Web: `https://wedding2day-a99ea.web.app/delete-account`. Both routes are in Privacy Policy §8. |

**On "Sharing": every row below answers NO.** Play excludes transfers to a
service provider acting on the developer's behalf, and Firebase is the only
recipient of any data. W2D has no ads SDK, no attribution SDK, no analytics
other than Firebase, and no payment processor — confirmed by `package.json`,
which lists exactly seven `@react-native-firebase/*` packages (app, auth,
firestore, storage, messaging, analytics, crashlytics) and nothing else that
talks to a network. Confirmed by grep: there is no `fetch`, `axios` or
`XMLHttpRequest` call anywhere in `app/`.

---

## 2. Data types to declare as COLLECTED

### Personal info → Name

| Field | Answer |
|---|---|
| Collected | Yes |
| Shared | No |
| Processing | Not ephemeral (stored) |
| Required or optional | **Required** — signup cannot complete without it |
| Purposes | App functionality; Account management |
| Deletable | Yes |

Two names, both required at signup: the contact person's `name` and the
`businessName`. Evidence: `createUserProfile()` in
`app/(auth)/_lib/firestore.ts`. Both are shown to other businesses on feed
cards and on the reveal screen (§6), and `businessName` is shown publicly on
the profile page (§6a).

### Personal info → Phone number

| Field | Answer |
|---|---|
| Collected | Yes |
| Shared | No |
| Processing | Not ephemeral (stored) |
| Required or optional | **Required** — it is the only sign-in credential |
| Purposes | App functionality; Account management |
| Deletable | Yes |

Firebase phone auth is the entire login (§8 — WhatsApp OTP is a plan, not
built). The number is stored at `users/{uid}/contact/info`, readable only by
the owner, an admin, or the holder of a per-pair reveal grant (§6).

**Declare this one carefully:** if the business creates a public profile, its
phone number is published on a page reachable **with no login at all** (§6a,
`app/p/[slug].tsx`, `profiles` has a public read rule). That is deliberate and
is disclosed in Privacy Policy §3. Play has no checkbox for "published
publicly", so the disclosure lives in the policy — but do not describe this
data as internal-only anywhere in the listing.

### Personal info → User IDs

| Field | Answer |
|---|---|
| Collected | Yes |
| Shared | No |
| Processing | Not ephemeral (stored) |
| Required or optional | Required |
| Purposes | App functionality; Account management; Analytics; Crash reporting |
| Deletable | Yes |

The Firebase Auth uid. It is also set as the Crashlytics user identifier —
`setCrashUser()` in `app/(auth)/_lib/telemetry.ts` calls
`crashlytics().setUserId(uid)` and passes nothing else.

### Photos and videos → Photos

| Field | Answer |
|---|---|
| Collected | Yes |
| Shared | No |
| Processing | Not ephemeral (stored) |
| Required or optional | **Optional** |
| Purposes | App functionality |
| Deletable | Yes |

Max 3 per post/catalog item (§9). Three Storage paths, and the access levels
differ — `storage.rules`:
- `listing-photos/{sellerId}/` — signed-in read
- `catalog-photos/{uid}/` — signed-in read
- `profile-photos/{uid}/` — **public read**, because they render on the
  no-login profile page

Camera / photo-library permission is requested only at the moment the user
chooses to add a photo (`expo-image-picker`).

### App activity → App interactions

| Field | Answer |
|---|---|
| Collected | Yes |
| Shared | No |
| Processing | Not ephemeral (stored) |
| Required or optional | Required (automatically collected) |
| Purposes | Analytics |
| Deletable | Yes |

Firebase Analytics. The complete event list is the `AnalyticsEvent` union in
`app/(auth)/_lib/telemetry.ts` — `trade_interest`, `listing_created`,
`public_profile_viewed`, `public_profile_shared`, `public_profile_created`,
`signup_completed`, `rate_limit_hit`.

**The parameters carry no personal data, by rule.** Verified at every call
site: `post_type`, `category`, `district`, `action`, `role`. No phone number,
no business name, no person's name, and deliberately not the profile `slug`
(which is the shareable secret in §6a).

### App activity → Other user-generated content

| Field | Answer |
|---|---|
| Collected | Yes |
| Shared | No |
| Processing | Not ephemeral (stored) |
| Required or optional | Optional |
| Purposes | App functionality |
| Deletable | Yes |

Listing titles and descriptions, catalog entries, and report reasons —
`listings`, `users/{uid}/catalogItems`, `reports`.

### App info and performance → Crash logs

| Field | Answer |
|---|---|
| Collected | Yes · Shared No · Required · Purpose: Crash reporting (Analytics) · Deletable Yes |

Firebase Crashlytics, wired in `app.json` plugins and used in
`telemetry.ts` (`recordHandledError`, `setCrashUser`).

### App info and performance → Diagnostics

| Field | Answer |
|---|---|
| Collected | Yes · Shared No · Required · Purpose: Crash reporting; Analytics · Deletable Yes |

Automatic Crashlytics/Analytics device and performance metadata.

### Device or other IDs → Device or other IDs

| Field | Answer |
|---|---|
| Collected | Yes · Shared No · Required · Purposes: App functionality; Analytics · Deletable Yes |

Two, both real:
- **FCM registration token** — `messaging().getToken()` in
  `app/(auth)/_lib/push.ts`, stored per-device at `users/{uid}/devices/{token}`.
- **Firebase Analytics app instance ID** — collected automatically by the
  Analytics SDK.

---

## 3. Business category, role and district — where they go on the form

`category` (one of 29, §9), `role` (Vendor/Manufacturer, §7) and `district`
(one of 38, §9) are required at signup and shown publicly.

**Play has no "business profile" data type.** Declare them under **App
activity → Other user-generated content** together with listing content, which
is where user-entered profile attributes of this kind belong.

> **Do NOT declare `district` under Location.** Play's Location types mean
> *device* location — approximate or precise. `district` is a value the user
> picks from a hardcoded 38-item list. The app requests no location permission,
> holds no location SDK, and never reads device position. Privacy Policy §6
> states this explicitly. Declaring it as Location would be a false positive
> that invites a review question and implies a permission the app does not have.

---

## 4. Data types to declare as NOT collected — with the reason

Each of these is something a reviewer (or a cautious founder) might assume is
collected. Each was checked against the code.

| Play data type | Why not |
|---|---|
| **Location** (approximate / precise) | No location permission in `app.json`, no location SDK, no device-position read anywhere. `district` is a picker value — see §3. "Near me" sorting is listed in §15 as *not yet built*. |
| **Email address** | Nothing collects one. Auth is phone-only, and Google Sign-In is rejected (§8, §12). The support address in the policy is *ours*, not the user's. |
| **Physical address** | Not a field. `district` is the only geography. |
| **Financial info / payment info** | No payments in the app at all (§6, §10). No payment SDK in `package.json`. |
| **Messages (emails, SMS, in-app)** | No in-app chat or inbox on the trade side — rejected in §6/§10/§12, and there is no messages collection. The in-app notification centre holds system notices, not user-to-user messages. |
| **Contacts** | No contacts permission. "Invite friends" is listed in §15 as *not yet built*. |
| **Calendar, Files and docs, Music, Health, SMS/Call log** | No permissions, no SDKs, no code paths. `app.json` explicitly `blockedPermissions` `WRITE_EXTERNAL_STORAGE`, `SYSTEM_ALERT_WINDOW` and `RECORD_AUDIO`. |
| **Purchase history / Search history** | Not collected. Feed search runs entirely in memory (`app/(auth)/_lib/search.ts`) and is never written or logged. |
| **Race/ethnicity, political/religious beliefs, sexual orientation** | Not collected. Note the category list includes *Iyer* (a priest service) — that is a **business trade category**, not a declaration about the person, and must not be declared as religious belief. |
| **Ratings / reviews content** | §15 describes full reviews, but **they are not built** — no `reviews` collection exists in `firestore.rules` and no screen writes one. The ToS mentions them conditionally ("where reviews are available"). Do not declare until built. |
| **GST / business registration number** | §15 describes an optional GST field for the soft verification badge. **Not built** — no UI field writes one, and `profiles` rules actively reject `gstNumber`. Do not declare until built. |

---

## 5. Things to double-check in Play Console before submitting

1. **Only one permission is requested:** `POST_NOTIFICATIONS`. Camera and
   photo-library access are handled by `expo-image-picker` at point of use.
2. **The account-deletion URL must be live** before submitting — it exists only
   after `firebase deploy --only hosting` (Batch 7). The form rejects an
   unreachable URL.
3. **This draft has a date.** If reviews, GST verification, "near me" sorting or
   "invite friends" ship (all listed as unbuilt in §15), the corresponding row
   in §4 moves to §2 and this file needs revisiting. Data Safety declarations
   are a standing obligation, not a one-time form.
4. Regenerate and re-read the hosted policy (`npm run docs:legal`) so the form
   and the policy say the same thing — `npm run test:legal` enforces that the
   policy text and the repo copies agree, but not that this file agrees with
   either.

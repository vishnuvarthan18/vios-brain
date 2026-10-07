> ⚠️ SUPERSEDED — describes old v1 scope. See DECISIONS.md at project root for current truth.

# W2D — Execution Roadmap

## Phase Status Overview

| Phase | Name | Status |
|-------|------|--------|
| 1 | Project setup | ✅ Done |
| 2 | Toolchain verification | ✅ Done |
| 3 | Authentication (Phone OTP + Google) | 🔜 Next |
| 4 | Profile creation | ⬜ Pending |
| 5 | Create listing | ⬜ Pending |
| 6 | Browse feed + filters | ⬜ Pending |
| 7 | Listing detail + "I'm Interested" | ⬜ Pending |
| 8 | Admin approval flow | ⬜ Pending (Firebase Console only) |
| 9 | Polish & QA | ⬜ Pending |
| 10 | Play Store submission | ⬜ Pending |

---

## Phase 1 — Project Setup ✅ DONE

- [x] Firebase project `wedding2day-a99ea` created
- [x] `google-services.json` downloaded from Firebase Console
- [x] `app.json` configured with `"googleServicesFile": "./google-services.json"` inside `"android"` block
- [x] Expo project initialized at `~/Desktop/w2d`
- [x] `package.json` `main` field set to `expo-router/entry`
- [x] Expo Router scaffolded (`app/_layout.tsx`, `app/index.tsx`)
- [x] NativeWind installed and configured
- [x] `lib/firebase.ts` created (Auth + Firestore, emulator connections under `__DEV__`)

## Phase 2 — Toolchain Verification ✅ DONE

- [x] EAS dev-client build succeeded (package `com.w2d.app`)
- [x] Dev-client APK installed on Vishnu's real Android phone
- [x] Phone connected to Mac dev server via `npx expo start --dev-client`
- [x] Firebase Local Emulator Suite installed and running
  - [x] Firebase CLI v15.22.4 installed globally
  - [x] Java (openjdk via Homebrew) installed + symlink fix applied
  - [x] Emulators: Auth (9099), Firestore (8080)
- [x] End-to-end toolchain verified: Expo + EAS + Firebase config + dev client + phone + Mac

---

## Phase 3 — Authentication 🔜 NEXT

### Pre-work (blocker)
- [ ] Decide: upgrade Firebase project to Blaze plan for production Phone OTP?
  - Check India per-SMS pricing in Firebase Console first
  - Set budget alert in Google Cloud Console
  - Development uses emulator — no billing impact during dev

### Milestones

**3.1 — Phone number entry screen**
- [ ] Create `app/(auth)/phone.tsx`
- [ ] Input field for phone number (numeric, with +91 prefix shown)
- [ ] "Send OTP" button → calls `signInWithPhoneNumber(auth, phone)`
- [ ] Loading state during OTP send
- [ ] Error state (invalid number, network error)
- [ ] Store `verificationId` in state for next screen

**3.2 — OTP verification screen**
- [ ] Create `app/(auth)/verify.tsx`
- [ ] 6-digit OTP input field
- [ ] "Verify" button → creates `PhoneAuthCredential` → calls `signInWithCredential(auth, credential)`
- [ ] Loading state during verification
- [ ] Error state (wrong OTP, expired)
- [ ] On success → check if `users/{uid}` document exists in Firestore

**3.3 — Auth routing logic**
- [ ] If profile exists → navigate to `/(tabs)/browse`
- [ ] If profile does not exist → navigate to `/(onboarding)/profile`

**3.4 — Auth persistence & guard**
- [ ] Root `app/_layout.tsx` listens to `onAuthStateChanged`
- [ ] Unauthenticated users redirected to `/(auth)/phone`
- [ ] Authenticated users without profile redirected to `/(onboarding)/profile`

**3.5 — Google Sign-In (secondary)**
- [ ] Add Google Sign-In button to phone screen or as separate entry
- [ ] Configure Google Sign-In in Firebase Console (add SHA-1 fingerprint from EAS)
- [ ] Test on device

**3.6 — Test on emulator**
- [ ] Open Emulator UI at `http://localhost:4000`
- [ ] Sign in using test phone number (emulator provides OTP automatically)
- [ ] Confirm user appears in Auth emulator

---

## Phase 4 — Profile Creation

**4.1 — Profile form screen**
- [ ] Create `app/(onboarding)/profile.tsx`
- [ ] Fields: name (text), businessName (text), userType (picker: manufacturer/decorator), district (picker: TN_DISTRICTS), phone (pre-filled from auth, read-only)
- [ ] All fields required — inline validation
- [ ] "Save Profile" button → writes to `users/{uid}` in Firestore

**4.2 — Data write**
- [ ] On submit: `setDoc(doc(db, 'users', auth.uid), { ...profileData, createdAt: serverTimestamp() })`
- [ ] On success → navigate to `/(tabs)/browse`
- [ ] On error → show error message, do not navigate

**4.3 — Profile read (for header/account screen)**
- [ ] Create `hooks/useProfile.ts` → fetches `users/{uid}` document
- [ ] Display name and userType in account tab

---

## Phase 5 — Create Listing

**5.1 — Listing form screen**
- [ ] Create `app/(tabs)/create.tsx`
- [ ] Fields: title (text), category (picker), condition (picker: used/new), price (numeric), quantity (numeric), district (picker: TN_DISTRICTS), description (multiline text)
- [ ] Photo picker: `expo-image-picker` → multi-select up to 5 images
- [ ] All required fields validated inline

**5.2 — Image upload**
- [ ] Generate Firestore doc ref before upload: `const listingRef = doc(collection(db, 'listings'))`
- [ ] Upload each image to `listing-photos/{listingRef.id}/{filename}` in Firebase Storage
- [ ] Collect download URLs into array

**5.3 — Firestore write**
- [ ] Write listing document: `setDoc(listingRef, { ...formData, imageUrls, sellerId: auth.uid, status: 'pending', createdAt: serverTimestamp() })`
- [ ] On success → navigate to browse tab + show success toast
- [ ] On error → show error, preserve form state

**5.4 — Categories**
- [ ] Decide and hardcode the category list (e.g., Mandap, Backdrop, Lighting, Floral, Props, Fabric, Other)
- [ ] Store in `constants/categories.ts`

---

## Phase 6 — Browse Feed + Filters

**6.1 — Listings fetch**
- [ ] Create `hooks/useListings.ts` → queries `listings` where `status == 'approved'`
- [ ] Real-time listener (`onSnapshot`) or one-time fetch (`getDocs`) — recommend `getDocs` with pull-to-refresh for simplicity

**6.2 — Listing card component**
- [ ] Create `components/ListingCard.tsx`
- [ ] Show: first image thumbnail, title, price, district, condition badge
- [ ] Tappable → navigates to `listing/[id]`

**6.3 — Browse screen**
- [ ] Create `app/(tabs)/browse.tsx`
- [ ] `FlatList` of `ListingCard` components
- [ ] Pull-to-refresh
- [ ] Empty state when no listings

**6.4 — Filter bar**
- [ ] Create `components/FilterBar.tsx`
- [ ] Horizontal scrollable chips: Category, District, Condition, Price
- [ ] Tapping a chip opens a modal or dropdown to select value
- [ ] Active filter shown as highlighted chip with clear (×) option
- [ ] Filters applied to `useListings` query or filtered client-side

**6.5 — Composite index**
- [ ] If multiple filters used simultaneously, Firebase will throw an error with a direct link to create the required index in Firebase Console — click it and create

---

## Phase 7 — Listing Detail + "I'm Interested"

**7.1 — Dynamic route**
- [ ] Create `app/listing/[id].tsx`
- [ ] Fetch listing by `id` param: `getDoc(doc(db, 'listings', id))`
- [ ] Display all listing fields: image carousel, title, category, condition, price, quantity, district, description

**7.2 — Interest button**
- [ ] Create `components/InterestButton.tsx`
- [ ] Button visible at bottom of screen
- [ ] On tap:
  1. Write to `interests`: `{ listingId: id, buyerId: auth.uid, createdAt: serverTimestamp() }`
  2. Fetch seller document: `getDoc(doc(db, 'users', listing.sellerId))`
  3. Reveal seller phone number
  4. Show WhatsApp button: `Linking.openURL('https://wa.me/91' + sellerPhone)`

**7.3 — Phone reveal state**
- [ ] After interest logged, show seller phone as tappable `tel:` link and WhatsApp button
- [ ] Do NOT show phone before interest is tapped

**7.4 — Report button**
- [ ] Add "Report" button on listing detail
- [ ] v1 implementation: opens email client with pre-filled subject (`Report listing: {title}`) to Vishnu's email
- [ ] No in-app report management in v1

---

## Phase 8 — Admin Approval (Firebase Console)

No code to write. This phase is documentation and process:

- [ ] Vishnu logs in to Firebase Console → `wedding2day-a99ea` → Firestore
- [ ] Navigate to `listings` collection
- [ ] Filter/scan for documents where `status == "pending"`
- [ ] Click document → edit `status` field → change to `"approved"` → Save
- [ ] Listing is now visible in the browse feed

**Optional QoL:** Set up a Firestore index on `status` field to make filtering in Console easier.

---

## Phase 9 — Polish & QA

**9.1 — Error handling audit**
- [ ] Every network call has a try/catch
- [ ] Every loading state has a spinner or skeleton
- [ ] Every empty state has a message and action

**9.2 — Form validation audit**
- [ ] All required fields enforced client-side
- [ ] Numeric fields reject non-numeric input
- [ ] District and category are always from the hardcoded list (picker, not free text)

**9.3 — Auth edge cases**
- [ ] App opened while logged in → skip auth screens
- [ ] App opened while logged in but no profile → go to profile screen
- [ ] Session expiry handled gracefully

**9.4 — Performance**
- [ ] `FlatList` with `keyExtractor` and `getItemLayout` if list is long
- [ ] Images: use `expo-image` with caching

**9.5 — Firestore Security Rules**
- [ ] Deploy final security rules (see `3_integrations_and_apis.md`)
- [ ] Test rules in Firebase Console → Rules Playground

**9.6 — Storage Security Rules**
- [ ] Deploy final storage rules
- [ ] Test photo upload and read on real device

**9.7 — Device testing**
- [ ] Test all flows on Vishnu's Android phone
- [ ] Test with slow network (Android Developer Options → network throttle)

---

## Phase 10 — Play Store Submission

### Pre-submission checklist
- [ ] Privacy policy page published at a public URL (required for Play Store)
- [ ] App icon: 512×512 PNG, no transparency
- [ ] Feature graphic: 1024×500 PNG
- [ ] Screenshots: minimum 2, phone screenshots

### Build
- [ ] `eas build --platform android --profile production` → produces `.aab`
- [ ] Download `.aab` from EAS dashboard

### Play Console steps
- [ ] Upload `.aab` to closed testing track
- [ ] Add 15–20 testers (need 12+ to complete 14 days of testing)
- [ ] Publish to closed testing → wait for Google review (~1–3 days for testing track)
- [ ] Run closed testing for 14 consecutive days
- [ ] After 14 days + 12+ testers → eligible to apply for production

### Production review
- [ ] Apply for production release in Play Console
- [ ] ~7-day review for first-time apps
- [ ] Respond to any policy queries from Google

### Known blockers
- Privacy policy URL must exist before submission
- Google Play Console registration must be fully approved (ID verification, if requested)
- Blaze plan must be active for Phone OTP in production build

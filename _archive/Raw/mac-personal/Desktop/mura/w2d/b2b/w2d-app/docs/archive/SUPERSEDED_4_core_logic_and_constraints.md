> ⚠️ SUPERSEDED — describes old v1 scope. See DECISIONS.md at project root for current truth.

# W2D — Core Logic & Constraints

## Auth Flow Logic

### First-Time User Flow
1. User enters phone number → Firebase sends OTP SMS
2. User enters OTP code → Firebase Auth creates account → `auth.currentUser` is set
3. App checks Firestore for existing `users/{uid}` document
4. If document does NOT exist → redirect to `/onboarding/profile` screen
5. User completes profile → document created in `users/{uid}` → redirect to main app (tabs)

### Returning User Flow
1. User enters phone number → OTP → Firebase Auth restores session
2. App checks for `users/{uid}` document
3. Document exists → redirect directly to browse feed (tabs)

### Session Persistence
- Firebase Auth persists session automatically on device
- On app open, check `auth.onAuthStateChanged` — if user exists and profile exists, skip auth screens entirely

### Auth Guard Pattern
All screens inside `(tabs)/` and `listing/[id].tsx` must check:
- Is user authenticated? If not → redirect to `/(auth)/phone`
- Does user have a profile document? If not → redirect to `/(onboarding)/profile`

## Listing Status Lifecycle

```
[User submits listing] → status: "pending"
        ↓
[Vishnu opens Firebase Console → Firestore → listings]
        ↓
[Manually edits document: status → "approved"]
        ↓
[Listing visible in browse feed]
```

**Browse feed query:** Only fetch listings where `status == "approved"`. Never show `pending` listings to other users.

## "I'm Interested" Logic

1. User taps "I'm Interested" on listing detail screen
2. App writes a document to `interests` collection: `{ listingId, buyerId: auth.uid, createdAt }`
3. App reveals seller's phone number (from `users/{sellerId}` document) and a WhatsApp deep link
4. WhatsApp deep link format: `https://wa.me/91{phoneNumber}` (prepend India country code)

**Idempotency consideration:** Decide whether to allow a buyer to express interest multiple times. Simplest v1 approach — allow it (each tap creates a new `interests` document). This inflates the metric slightly but keeps the code simple. Revisit in v2.

**Seller phone reveal:** Only revealed after interest is logged. Do NOT show seller phone number on the listing card or in the listing detail before the interest tap.

## Filters Logic (Browse Feed)

Filters applied client-side OR as Firestore compound queries. Compound queries require composite indexes in Firebase Console — create them as needed.

| Filter | Firestore Query |
|--------|----------------|
| Category | `where('category', '==', selectedCategory)` |
| District | `where('district', '==', selectedDistrict)` |
| Condition | `where('condition', '==', selectedCondition)` |
| Price (max) | `where('price', '<=', maxPrice)` |
| Status | `where('status', '==', 'approved')` — always applied |

Multiple simultaneous filters require composite indexes. Create them in Firebase Console → Firestore → Indexes when Firestore throws an index error (it provides a direct link to create the index).

## Image Upload Logic (Listing Creation)

1. User taps photo picker → `expo-image-picker` opens device gallery
2. Selected images → upload each to Firebase Storage at `listing-photos/{listingId}/{filename}`
3. Get download URL for each uploaded image
4. Store array of URLs in `listings.imageUrls[]`
5. On listing detail screen, display images from `imageUrls[]`

**Upload order:** Upload images first → get URLs → then write the Firestore document. This avoids a listing document with empty `imageUrls`.

**Listing ID for storage path:** Generate the Firestore document ID before uploading (`doc(db, 'listings')` returns a ref with an auto-generated ID you can use before writing).

## Business Constraints

### District Validation
- `district` field on both `users` and `listings` must be one of the 38 hardcoded `TN_DISTRICTS` values
- Use a dropdown/picker — never a free-text field for district

### UserType
- `userType` must be exactly `"manufacturer"` or `"decorator"` — no other values
- Set during profile creation, not editable in v1

### Listing Ownership
- Only the listing creator (`sellerId == auth.uid`) can edit or delete their listing — enforce in Firestore security rules AND in UI (hide edit/delete buttons for non-owners)

### Admin is Vishnu in Firebase Console
- There is no admin app in v1
- There are no admin roles in Auth in v1
- Status changes happen only via direct Firestore document edits in Firebase Console
- Never build an admin screen inside the app for v1

## Known Bugs & Solved Problems

### `google-services.json` must be explicitly referenced in `app.json`
**Problem:** EAS Prebuild fails silently if `google-services.json` is not referenced.  
**Solution:** The `"android"` block in `app.json` must contain:
```json
"googleServicesFile": "./google-services.json"
```
This is not auto-detected. Its absence is the most common build-breaking mistake.

### React version mismatch breaks EAS builds
**Problem:** If `react-dom` version doesn't match `react`, EAS build fails.  
**Solution:**
```bash
npx expo install react react-dom --fix
```

### `127.0.0.1` doesn't work for emulator on physical device
**Problem:** When running the app on a real Android phone (not a simulator), `127.0.0.1` points to the phone itself, not the Mac.  
**Solution:** Use the Mac's LAN IP address (e.g., `192.168.31.16`) in `lib/firebase.ts` as the emulator host.

### Wrong directory causes `ConfigError: package.json not found`
**Problem:** Running Expo or npm commands from `~/W2D` (stray folder) instead of `~/Desktop/w2d`.  
**Solution:** Always `cd ~/Desktop/w2d` before any command. Confirm with `pwd` if unsure.

### Expo Go cannot run native Firebase Phone Auth
**Problem:** Firebase Phone OTP is a native module that Expo Go cannot load.  
**Solution:** Always use a dev-client build installed via EAS (`eas build --platform android --profile development`). Expo Go is permanently blocked for this project.

### Java required for Firebase Emulator
**Problem:** Firebase Emulator Suite requires Java — not pre-installed on all Macs.  
**Solution:** Install via Homebrew (`brew install openjdk`) and apply the required symlink. This has already been done on Vishnu's Mac.

## Security Constraints

- Never expose seller phone number before "I'm Interested" is tapped
- All Firestore reads require `request.auth != null` — unauthenticated reads are blocked
- Users can only write to their own `users/{uid}` document
- Listings can only be updated/deleted by the `sellerId`
- `interests` can be created by any authenticated user but not deleted (immutable log for metric)
- Never store Firebase credentials in source code — use `google-services.json` (auto-injected at build time)

## Blaze Plan Blocker (Phase 3)

Firebase Phone OTP in production requires the Blaze (pay-as-you-go) plan. Steps before enabling production Phone Auth:
1. Upgrade Firebase project `wedding2day-a99ea` to Blaze in Firebase Console
2. Check India SMS pricing (Firebase Console → Authentication → Sign-in method → Phone)
3. Set a budget alert in Google Cloud Console to avoid unexpected charges
4. Only then enable Phone Auth provider in Firebase Console

During development, the local emulator bypasses this entirely.

> ⚠️ SUPERSEDED — describes old v1 scope. See DECISIONS.md at project root for current truth.

# W2D — Integrations & APIs

## Firebase (Primary Backend)

**Project ID:** `wedding2day-a99ea`  
**Account:** `(removed)`  
**Plan:** Spark (free) → must upgrade to **Blaze (pay-as-you-go)** before enabling Phone OTP in production

### Firebase Authentication

**Method:** Phone OTP (primary), Google Sign-In (secondary)

- Phone OTP requires Blaze plan — India per-SMS pricing must be verified in Firebase Console before enabling
- During development: Firebase Local Emulator bypasses real SMS entirely (no billing)
- Google Sign-In: can be added on Spark plan — no SMS cost involved
- Auth UID becomes the `id` field in the `users` Firestore document

**Emulator setup:**
```bash
firebase emulators:start
# Auth emulator: http://localhost:9099
# Firestore emulator: http://localhost:8080
# Emulator UI: http://localhost:4000
```

**Emulator host on physical device:** Use Mac's LAN IP (e.g., `192.168.31.16`), never `127.0.0.1`.

### Firestore

- NoSQL document database
- Collections: `users`, `listings`, `interests`
- Security rules: must restrict reads/writes to authenticated users
- Admin access: Vishnu approves listings manually in Firebase Console (no admin SDK or app in v1)

**Firestore Security Rules (v1 minimum viable):**
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }

    match /listings/{listingId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update, delete: if request.auth != null &&
        request.auth.uid == resource.data.sellerId;
    }

    match /interests/{interestId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
    }
  }
}
```

### Firebase Storage

- Bucket path: `listing-photos/{listingId}/{filename}`
- Used for listing photo uploads
- Download URLs saved to `listings.imageUrls[]`
- Storage security rules: allow authenticated users to upload; allow all authenticated users to read

**Storage Security Rules (v1 minimum viable):**
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /listing-photos/{allPaths=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
  }
}
```

### Firebase Cloud Messaging (FCM)

- Scaffolded but not a primary v1 feature
- Config is included in `google-services.json`
- Do not build notification triggers in v1 — only set up the config

### Firebase Local Emulator Suite

**Firebase CLI version:** v15.22.4 (installed globally)  
**Logged in as:** `(removed)`  
**Java requirement:** openjdk via Homebrew (symlink fix already applied)

Emulator start command:
```bash
cd ~/Desktop/w2d
firebase emulators:start
```

Emulator ports:
| Service | Port |
|---------|------|
| Auth | 9099 |
| Firestore | 8080 |
| Emulator UI | 4000 |

## EAS (Expo Application Services)

**Account:** `vishnu18`  
**Package name:** `com.w2d.app`

### Build Profiles (in `eas.json`)

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "production": {
      "android": {
        "buildType": "app-bundle"
      }
    }
  }
}
```

### Build Commands

| Purpose | Command |
|---------|---------|
| Dev client build (install on phone) | `eas build --platform android --profile development` |
| Production `.aab` for Play Store | `eas build --platform android --profile production` |
| Submit to Play Store | `eas submit --platform android` |

**Note:** Expo Go cannot run this app — native Firebase Phone Auth requires a dev-client or production build.

## Google Play Console

- Registration started: contact details submitted, $25 paid
- Runs passively in background
- Only act if Google emails requesting ID verification documents
- Package name locked: `com.w2d.app`
- Play Store submission requires:
  - Closed testing: 12+ testers × 14 days (recruit 15–20 as buffer)
  - ~7-day production review after closed testing
  - Privacy policy URL (public-facing URL required at submission time — not yet created)

## NativeWind

- Tailwind-style utility classes for React Native
- Applied via `className` prop on RN components
- No `StyleSheet.create()` — NativeWind replaces it
- Requires `tailwind.config.js` with `content` paths pointing to `app/` and `components/`

## Node.js / npm

- Node.js managed via `nvm` (v24+)
- npm for package management
- Always confirm working directory is `~/Desktop/w2d` before running any npm or Expo command
- Common directory mistake: accidentally running commands from `~/W2D` (a stray empty folder under home — unrelated junk, never use it)

## Previously Used (Abandoned — Reference Only)

| Service | Status | Notes |
|---------|--------|-------|
| FlutterFlow | Abandoned | Old frontend — fully replaced by Expo/React Native |
| Supabase | Abandoned | Old backend — fully replaced by Firebase. Ref `hodrckzswjdugfeukczg` retained only as schema reference |
| Message Central | Abandoned | Was the OTP provider for old stack |
| Twilio | Abandoned | Listed in original system prompt but never implemented |

Do not reference or revive any of the above services.

> ⚠️ SUPERSEDED — describes old v1 scope. See DECISIONS.md at project root for current truth.

# W2D — Architecture & Data

## Tech Stack (Locked)

| Layer | Technology | Notes |
|-------|-----------|-------|
| Framework | React Native (Expo managed workflow) | |
| Router | Expo Router (file-based) | `app/` directory |
| Styling | NativeWind | Tailwind classes in RN |
| Auth | Firebase Authentication | Phone OTP + Google Sign-In |
| Database | Firestore (NoSQL) | |
| File Storage | Firebase Storage | Listing photos |
| Push Notifications | Firebase Cloud Messaging (FCM) | Scaffolded, not primary v1 |
| Build | EAS (Expo Application Services) | EAS account: `vishnu18` |
| Local Dev | Firebase Local Emulator Suite | Auth: 9099, Firestore: 8080 |

## Project Identity

- **Expo package name:** `com.w2d.app`
- **Firebase project ID:** `wedding2day-a99ea`
- **Firebase account:** `(removed)`
- **EAS account:** `vishnu18`
- **Active project folder on Mac:** `~/Desktop/w2d` (lowercase — not `~/W2D`)

## Folder Structure

```
~/Desktop/w2d/
├── app/
│   ├── _layout.tsx              # Root layout (Expo Router)
│   ├── index.tsx                # Entry / redirect screen
│   ├── (auth)/
│   │   ├── _layout.tsx
│   │   ├── phone.tsx            # Phone number entry (OTP step 1)
│   │   └── verify.tsx           # OTP code entry (OTP step 2)
│   ├── (onboarding)/
│   │   └── profile.tsx          # Profile creation after first login
│   ├── (tabs)/
│   │   ├── _layout.tsx          # Tab navigator definition
│   │   ├── browse.tsx           # Listings feed + filters
│   │   ├── create.tsx           # Create listing form
│   │   └── account.tsx          # User profile / settings
│   └── listing/
│       └── [id].tsx             # Listing detail (dynamic route)
├── components/
│   ├── ListingCard.tsx
│   ├── FilterBar.tsx
│   ├── InterestButton.tsx
│   ├── StatusBadge.tsx
│   └── ...
├── lib/
│   └── firebase.ts              # Firebase init + emulator connection
├── hooks/
│   ├── useAuth.ts
│   ├── useListings.ts
│   └── useProfile.ts
├── constants/
│   └── districts.ts             # 38 Tamil Nadu districts (hardcoded array)
├── types/
│   └── index.ts                 # TypeScript interfaces for User, Listing, Interest
├── google-services.json         # Firebase Android config (do not edit manually)
├── app.json                     # Expo config — includes googleServicesFile reference
├── package.json
└── tsconfig.json
```

## Firebase Configuration (`lib/firebase.ts`)

```typescript
import { initializeApp } from 'firebase/app';
import { getAuth, connectAuthEmulator } from 'firebase/auth';
import { getFirestore, connectFirestoreEmulator } from 'firebase/firestore';

const firebaseConfig = {
  // values from google-services.json / Firebase Console
};

const app = initializeApp(firebaseConfig);
export const auth = (secret removed)
export const db = getFirestore(app);

if (__DEV__) {
  // Use Mac's LAN IP (e.g., 192.168.31.16) when testing on physical device
  // Use 127.0.0.1 ONLY when running on simulator
  const EMULATOR_HOST = '192.168.31.16'; // update if IP changes
  connectAuthEmulator(auth, `http://${EMULATOR_HOST}:9099`);
  connectFirestoreEmulator(db, EMULATOR_HOST, 8080);
}
```

**Critical:** `app.json` must contain this inside the `"android"` block:
```json
"googleServicesFile": "./google-services.json"
```
Without this, Prebuild silently fails — this is the most common build-breaking mistake.

## Firestore Data Model (Locked)

### Collection: `users`

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | = Firebase Auth UID (document ID) |
| `createdAt` | timestamp | Set on profile creation |
| `name` | string | Full name |
| `businessName` | string | |
| `userType` | string | `"manufacturer"` or `"decorator"` |
| `district` | string | One of 38 Tamil Nadu districts |
| `phone` | string | Phone number used for OTP |

### Collection: `listings`

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | Auto-generated Firestore ID (document ID) |
| `createdAt` | timestamp | |
| `sellerId` | string | = Auth UID of the creating user |
| `title` | string | |
| `category` | string | Decoration category |
| `condition` | string | `"used"` or `"new"` |
| `price` | number | In INR |
| `quantity` | number | |
| `district` | string | One of 38 Tamil Nadu districts |
| `description` | string | |
| `status` | string | Default: `"pending"`. Admin sets to `"approved"` in Firebase Console |
| `imageUrls` | string[] | Firebase Storage download URLs |

### Collection: `interests`

| Field | Type | Notes |
|-------|------|-------|
| `listingId` | string | |
| `buyerId` | string | = Auth UID |
| `createdAt` | timestamp | |

No document ID needed — auto-generated. This collection is the source of truth for the primary metric (interests per listing).

### Firebase Storage

- **Bucket path:** `listing-photos/{listingId}/{filename}`
- Images are uploaded during listing creation.
- Download URLs stored in `listings.imageUrls[]`.

## Constants

### Tamil Nadu Districts (38 — hardcoded, NOT a Firestore collection)

```typescript
// constants/districts.ts
export const TN_DISTRICTS = [
  "Ariyalur", "Chengalpattu", "Chennai", "Coimbatore", "Cuddalore",
  "Dharmapuri", "Dindigul", "Erode", "Kallakurichi", "Kancheepuram",
  "Kanyakumari", "Karur", "Krishnagiri", "Madurai", "Mayiladuthurai",
  "Nagapattinam", "Namakkal", "Nilgiris", "Perambalur", "Pudukkottai",
  "Ramanathapuram", "Ranipet", "Salem", "Sivaganga", "Tenkasi",
  "Thanjavur", "Theni", "Thoothukudi", "Tiruchirappalli", "Tirunelveli",
  "Tirupathur", "Tiruppur", "Tiruvallur", "Tiruvannamalai", "Tiruvarur",
  "Vellore", "Villupuram", "Virudhunagar"
];
```

## State Management

No global state library (Redux, Zustand, etc.) in v1. State handled by:
- **React local state** (`useState`, `useReducer`) for form state and UI state
- **Custom hooks** (`useAuth`, `useListings`, `useProfile`) for Firestore data fetching
- **Expo Router** for navigation state

This is intentional — keep complexity minimal for a solo non-technical founder to maintain.

## TypeScript Types

```typescript
// types/index.ts

export interface User {
  id: string;
  createdAt: Date;
  name: string;
  businessName: string;
  userType: 'manufacturer' | 'decorator';
  district: string;
  phone: string;
}

export interface Listing {
  id: string;
  createdAt: Date;
  sellerId: string;
  title: string;
  category: string;
  condition: 'used' | 'new';
  price: number;
  quantity: number;
  district: string;
  description: string;
  status: 'pending' | 'approved';
  imageUrls: string[];
}

export interface Interest {
  listingId: string;
  buyerId: string;
  createdAt: Date;
}
```

## Build Configuration

- **Dev:** `npx expo start --dev-client` — requires EAS dev-client build installed on phone
- **Build command:** `eas build --platform android --profile development`
- **Production build:** `eas build --platform android --profile production` → outputs `.aab` for Play Store
- **Expo Go:** Cannot be used — native Firebase Phone Auth requires dev-client or production build

# W2D — LOCKED DECISIONS

> **READ THIS FIRST before changing any code.**
> This file is the single source of truth for product and architecture decisions.
> If code contradicts this file, the FILE is right and the code needs fixing.
> If you (an AI assistant) think something here is a bug — it is NOT. It is a decision.
> Do not "fix" anything listed here. Ask the human first.

Last updated: 2026-07-21

---

## 1. WHAT THIS PRODUCT IS

**Wedding2day (W2D)** — a B2B **trade connection platform** for the Tamil Nadu wedding
industry. It connects decorators, manufacturers and suppliers so they can buy, sell,
rent and source wedding decoration materials.

- **Current scope = v1.1** (trade side only)
- **NOT couple-facing.** A full couple-facing wedding directory is the long-term end
  vision, but is explicitly OUT of scope now.
- Region: Tamil Nadu (38 districts)
- Primary metric: interests per post

---

## 2. STACK (LOCKED — DO NOT SUGGEST ALTERNATIVES)

| Layer | Choice |
|---|---|
| App | React Native + Expo (Expo Router, managed workflow) |
| Styling | NativeWind |
| Backend | Firebase — Auth, Firestore, Storage, FCM |
| IDE | Cursor Pro |
| Firebase project | `wedding2day-a99ea` |
| Android package | `com.w2d.app` |

**Old stack (FlutterFlow / Supabase / Twilio) is fully abandoned.** Never reference it.

---

## 3. DATA MODEL

### Collection name: `listings` — THIS IS DELIBERATE

**DECISION (2026-07-21): The main collection stays named `listings`.**

A rename to `posts` was considered and **rejected** — the rework cost across runtime
code, seed script, security rules and docs was not worth the semantic gain.

> ⚠️ The `listings` collection holds ALL post types, including requirements
> ("I need X"). The name is historical. **This is not a bug. Do not rename it.**

### Collections

| Collection | Purpose |
|---|---|
| `users` | id = uid, createdAt, name, businessName, userType, district, phone |
| `listings` | all posts (all 5 post types — see below) |
| `interests` | listingId, buyerId, createdAt |
| `reports` | user reports on posts |

### `listings` document fields

createdAt, sellerId, **postType**, title, category, condition, price, quantity,
district, description, imageUrls[], status

- `status` defaults to `"pending"`
- `price` and `quantity` are stored as **numbers**, not strings
- Storage path: `listing-photos/{sellerId}/` — 5MB max, JPEG/PNG/WebP

---

## 4. POST TYPES (5) — via the `postType` field

All five live in the single `listings` collection, distinguished by `postType`.

| postType | Direction | Meaning |
|---|---|---|
| `sell-used` | Supply | Offload surplus / old stock |
| `sell-new` | Supply | Sell new product |
| `rental` | Supply | Item available for hire |
| `catalog` | Supply | Permanent product range, no quantity limit, does not expire |
| `requirement` | **Demand** | "I need X" — suppliers respond |

### Rules per type

- **Rental is a LABEL ONLY.** The post is tagged "for rent"; phone is revealed and
  dates/deposit/return are handled off-app on WhatsApp.
  **There is NO in-app booking, calendar, or availability system.** Do not build one.
- **Catalog** = permanent product, no quantity limit, does not expire like surplus.
- **Requirement** = the demand side. Response is reversed: a supplier taps
  "I can supply this" → their phone is revealed to the poster.

---

## 5. APP SHAPE — TWO TABS

| Tab | Holds |
|---|---|
| **Available** | `sell-used`, `sell-new`, `rental`, `catalog` — filterable by type |
| **Needs** | `requirement` only |

**Why two tabs:** deliberate anti-clutter decision. Evidence: IndiaMART and Facebook
separate supply from demand; OLX's single feed drew clutter complaints. **Do not merge
these into one feed.**

**Post button** asks intent once, then routes to the right form.

**Forms:** a separate, minimal form per post type (progressive disclosure).
**Do NOT build one universal form for all five types.**

---

## 6. CONNECTION MECHANIC — REVEAL PHONE

For **all** post types: tap to reveal the other party's phone / WhatsApp, then they
connect directly off-app.

- **NO in-app chat. NO in-app inbox. NO in-app payments.** Do not build these.
- This is a deliberate pre-launch "ship faster" bet. The known risk (deal leakage /
  disintermediation) is accepted and deferred to the scaling stage.
- Every reveal logs an interest event to the `interests` collection (with dedupe).

---

## 7. USER TYPES — LABEL ONLY

`userType` is **single-select**, one of:

- Manufacturer
- Decorator
- Supplier

**It gates NOTHING.** Everyone can post any post type and respond to anything.
There is no buyer/seller distinction and no separate home screen per type.
It is purely a profile label.

Multi-select was considered and **rejected**.

---

## 8. AUTH

**Phone OTP only.**

Google Sign-In was considered and **rejected**: phone verification is mandatory
regardless of sign-in method (the core mechanic needs a verified phone/WhatsApp),
so Google added a second auth surface, native-module rebuild overhead, and recurring
SHA-1/token bugs without removing any required step. **Do not re-add it.**

---

## 9. LOCKED CONSTANTS

**Categories:** Mandap/Stage Structures · Backdrops & Panels · Flower Decoration
(Artificial) · Lighting · Pillars & Entrance Decor · Furniture (Chairs/Sofas/Thrones) ·
Carpets & Flooring · Fabric & Drapes · Props & Standees · Other

**Conditions:** New · Used - Like New · Used - Good · Used - Fair

**Districts:** 38 Tamil Nadu districts — hardcoded constant, NOT a Firestore collection.

**Photos:** max 3 per post.

---

## 10. EXPLICITLY OUT OF SCOPE — DO NOT BUILD

- Full couple-facing wedding directory (end vision, not now)
- The ~29 wedding service categories (that belongs to the directory vision)
- In-app chat / inbox / messaging
- In-app rental booking, calendar, or availability
- In-app payments
- Dedicated admin app (admin is done via Firebase Console directly)
- Tamil search, AI listings, social features

---

## 11. REJECTED IDEAS (do not re-propose)

| Idea | Status |
|---|---|
| Rename `listings` → `posts` | Rejected — rework cost too high |
| Multi-select userType | Rejected |
| Catalog as profile-only section | Rejected — chose feed post |
| Catalog as a checkbox on a sell post | Rejected — chose its own post type |
| Google Sign-In | Rejected |
| In-app rental booking | Rejected — rental is a label only |
| In-app chat/inbox | Rejected — reveal phone instead |

---

## 12. KNOWN INFRASTRUCTURE GOTCHAS

1. **Any new native module requires a fresh EAS dev-client build.**
   "Cannot find native module" = rebuild needed, NOT a code fix.
2. **Emulator connections must be initialised exactly once**, at module level in
   `app/_layout.tsx` inside the `__DEV__` block.
   "Cannot call useEmulator() after instance already initialized" = this rule broken.
   Silent failure mode: writes go to real cloud Firestore → permission-denied.
3. **Emulator ports:** Auth 9099 · Firestore 8080 · Storage 9199 · UI 4000.
   Host `192.168.31.16` (Mac local IP, for physical device testing).
   Startup must show all 3 emulators — photo uploads hang if Storage is missing.
4. `firestore.rules` and `storage.rules` live at project root and are referenced in
   `firebase.json`.
5. Dev seeding: `scripts/seed.mjs` seeds 8 sellers + 25 listings into the emulator.

---

## 13. HOW TO WORK ON THIS PROJECT (for AI assistants)

1. **Read this file before changing anything.**
2. If something in the code looks wrong but is listed here — it is a DECISION,
   not a bug. Do not "fix" it. Ask first.
3. Change one feature or screen per prompt.
4. Never touch files outside the stated scope of the request.
5. Never make architecture or product decisions autonomously — surface options and ask.
6. If a request contradicts this file, say so instead of silently complying.

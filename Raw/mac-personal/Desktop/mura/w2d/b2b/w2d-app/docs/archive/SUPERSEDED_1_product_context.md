> ⚠️ SUPERSEDED — describes old v1 scope. See DECISIONS.md at project root for current truth.

# Wedding2day (W2D) — Product Context

## Business Overview

**Product name:** Wedding2day (W2D)  
**Type:** B2B mobile marketplace  
**Geography:** Tamil Nadu, India (all 38 districts)  
**Founder:** Vishnu (solo, Vellore-based), 10+ years in wedding stage decoration manufacturing  
**Platform:** Android-first (iOS deferred post-v1)

## Problem Being Solved

Wedding decoration manufacturers and decorators in Tamil Nadu have no structured channel to buy and sell used or surplus wedding decoration materials. Transactions happen informally, creating price opacity and wasted inventory.

## Solution

A mobile marketplace where:
- **Manufacturers** list used/surplus decoration stock for resale
- **Decorators** browse and contact sellers directly via phone/WhatsApp

No in-app transactions. No escrow. No delivery logistics. Pure lead-generation / connection platform in v1.

## Target Users

| User Type | Role | Primary Action |
|-----------|------|----------------|
| Manufacturer | Seller | Create listings for surplus stock |
| Decorator | Buyer | Browse listings, express interest, call seller |

Both types sign up through the same app. `userType` field on profile distinguishes them.

## Primary Success Metric

**Interests per listing** — the number of times buyers tap "I'm Interested" on a given listing. This is the north-star metric for v1.

## Business Model (v1)

- Free to use in v1 — no monetization layer yet.
- Revenue model TBD post-validation.

## V1 Feature Scope (LOCKED)

1. Phone OTP sign-up and login
2. Profile creation: name, business name, userType (manufacturer/decorator), district, phone
3. Create listing: title, category, condition (used/new), photos, price, quantity, district, description — status defaults to `pending`
4. Browse all listings with filters: category, district, condition, price
5. Listing detail page
6. "I'm Interested" button → reveals seller's phone number and WhatsApp link so buyer can contact directly
7. Admin approves listings by changing status in Firebase Console — no admin app in v1

## What Is Explicitly Out of Scope for V1

- In-app messaging or chat
- In-app payments or escrow
- iOS build
- Admin mobile/web app
- Delivery or logistics tracking
- Reviews or ratings
- Notifications (FCM scaffolded but not a primary v1 feature)
- Negotiation tools

## User Journeys

### Seller (Manufacturer)
1. Download app → sign up with phone OTP → create profile (name, business, district, userType = manufacturer)
2. Tap "Create Listing" → fill form (title, category, condition, photos, price, qty, district, description) → submit
3. Listing status = `pending` until admin approves in Firebase Console
4. Once approved, listing is visible in the browse feed

### Buyer (Decorator)
1. Download app → sign up with phone OTP → create profile (userType = decorator)
2. Browse feed → apply filters (category, district, condition, price range)
3. Tap a listing → view full detail
4. Tap "I'm Interested" → seller's phone number and WhatsApp link revealed
5. Contact seller directly via phone or WhatsApp outside the app

### Admin (Vishnu)
1. Log in to Firebase Console
2. Go to Firestore → `listings` collection
3. Find listings with `status: pending` → manually set `status: approved`

## UI/UX Design System

### Colors (to be finalized — use these as defaults until Vishnu specifies brand palette)
- **Primary:** Deep maroon / gold-adjacent (wedding industry aesthetic)
- **Background:** Off-white / light neutral
- **Text primary:** Near-black
- **Text secondary:** Medium grey
- **CTA:** High-contrast accent color (gold or saffron tone)
- **Error:** Standard red

### Typography
- Use system fonts via React Native defaults — no custom font loading in v1 to reduce complexity.
- Heading sizes: large for screen titles, medium for card titles, small for labels.

### Component Conventions
- **ListingCard:** thumbnail image (left or top), title, price, district, condition badge, category label
- **FilterBar:** horizontal scrollable chips for quick filter access
- **InterestButton:** prominent CTA at bottom of listing detail screen
- **ProfileForm / ListingForm:** `KeyboardAvoidingView` wrapper, label above each input, validation inline below input
- **StatusBadge:** color-coded pill — pending (yellow), approved (green)

### NativeWind Usage
- All styling via `className` prop using Tailwind-equivalent utility classes
- No `StyleSheet.create()` anywhere
- No inline `style` objects unless a NativeWind limitation forces it (document the reason in a comment)

## Positioning

W2D is not a consumer marketplace. It is a trade-only tool for Tamil Nadu's wedding decoration industry. The UX should feel professional and utilitarian, not flashy — fast to load, easy to browse, minimal friction to post a listing.

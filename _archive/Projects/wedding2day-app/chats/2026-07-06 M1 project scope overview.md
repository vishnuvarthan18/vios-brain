---
tags: chat
date: 2026-07-06
source: Claude personal account
uuid: cf1b674a-590d-4834-bc71-3ec5721f6b4c
---
# M1 project scope overview

## Summary
**Conversation Overview**

The person is working on a project called Wedding2day (W2D), a marketplace mobile app for the wedding industry targeting Tamil Nadu. The conversation focused on retrieving the v1 (M1) feature scope of the project. The person's goal was to obtain a clean, presentation-ready scope document to share with external companies for quotation purposes.

Claude initially provided the scope with technical stack and tool details included, but the person explicitly requested a version with no stack or tool names mentioned — just the functional scope. Claude iterated accordingly and produced a final clean table covering seven features: phone OTP and Google login authentication, user profile creation (name, business name, user type as manufacturer or decorator, district, phone), listing creation with photos and details, browsable listing feed with filters, single listing detail view, a contact reveal mechanism via an "I'm Interested" button, and backend manual approval for admin without a dedicated admin app. The platform is Android (Play Store). Out-of-scope items for v1 were also listed: rental listings, in-app chat, in-app payments, a dedicated admin app, Tamil-language search, AI-generated listings, and social or forum features.

A recurring pattern in this conversation is that the person communicates in brief, direct messages and prefers output tailored for a specific external audience, indicating they want clean deliverables stripped of internal technical context when sharing externally.

## Chat

**Vishnu:** give me the m1 full scope of the proejct

**Claude:** **M1 = v1 launch scope**, per your project doc:

| Feature | Detail |
|---|---|
| Auth | Sign up / log in with OTP |
| Profile | Name, business name, type (manufacturer/decorator), district, phone |
| Create listing | Title, category, condition (used/new), photos, price, quantity, district, description |
| Browse listings | All listings, filters by category/district/condition/price |
| Listing detail | Full view of a single listing |
| Contact reveal | "I'm interested" → reveals seller's phone/WhatsApp for direct connection |
| Admin | Manual approval via Supabase dashboard — no admin app built for v1 |

**Not in M1** (explicitly deferred): rental listings, in-app chat/payments, admin web app, full UI redesign, Tamil search, AI-generated listings.

One conflict to flag: your uploaded doc says **Twilio** for OTP, but memory says Twilio/DLT was **abandoned June 27** in favor of **Message Central (Verify Now)** — already built and confirmed working. Memory is the current state; the doc is stale on that point.

Current status against this scope:

| Phase | Status |
|---|---|
| Auth (OTP + Google) | ✅ Done |
| Profile creation | ~85% (phone field pending, Create Profile action pending) |
| Create listing | Not started |
| Browse + filters | Not started |
| Listing detail | Not started |
| Contact reveal | Not started |
| Admin (Supabase dashboard) | No build needed — usable as-is |

Want to resume Phase 4 (profile) where you left off, or something else?

**Vishnu:** no need what is done and the date jsut pluet hte scope of the porejct i need to give to a comoay and ask quatations

**Claude:** Rule 10 says stack is Twilio/FCM — but memory shows Twilio was **abandoned June 27** for Message Central due to DLT delays, and that's locked. Flagging the conflict since you're using this for external quotations — I'll use the actual current stack below, not Rule 10's.

**Wedding2day (W2D) — v1 Scope for Vendor Quotation**

| Category | Detail |
|---|---|
| Platform | Android (Play Store), mobile app |
| Frontend | FlutterFlow |
| Backend | Supabase (Postgres DB, Auth, Storage, Edge Functions) |
| OTP/SMS | Message Central (Verify Now) |
| Push notifications | Firebase Cloud Messaging |
| Admin | Supabase dashboard (manual approvals) — no separate admin app in v1 |

**Features:**

| # | Feature | Details |
|---|---|---|
| 1 | Auth | Sign up / log in via phone OTP + Google OAuth |
| 2 | Profile | Name, business name, user type (manufacturer/decorator), district (Tamil Nadu), phone |
| 3 | Create listing | Title, category, condition (used/new), photos (up to 5MB, JPEG/PNG/WebP), price, quantity, district, description |
| 4 | Browse listings | Full listing feed with filters: category, district, condition, price |
| 5 | Listing detail | Single listing view, all fields + photos |
| 6 | Contact reveal | "I'm Interested" button reveals seller's phone/WhatsApp |
| 7 | Admin | Manual approval of users/listings via Supabase dashboard, no dedicated app |

**Out of scope for v1:** rental listings, in-app chat, in-app payments, admin web app, Tamil-language search, AI-generated listings, social/forum features.

**Vishnu:** dont mention adny stack adn any tools names jsut need scope

**Claude:** **Wedding2day (W2D) — v1 Scope for Vendor Quotation**

| # | Feature | Details |
|---|---|---|
| 1 | Auth | Sign up / log in via phone OTP + Google login |
| 2 | Profile | Name, business name, user type (manufacturer/decorator), district (Tamil Nadu), phone |
| 3 | Create listing | Title, category, condition (used/new), photos (up to 5 images), price, quantity, district, description |
| 4 | Browse listings | Full listing feed with filters: category, district, condition, price |
| 5 | Listing detail | Single listing view, all fields + photos |
| 6 | Contact reveal | "I'm Interested" button reveals seller's phone/WhatsApp |
| 7 | Admin | Manual approval of users/listings from backend — no dedicated admin app in v1 |

**Platform:** Android app (Play Store)

**Out of scope for v1:** rental listings, in-app chat, in-app payments, admin web app, Tamil-language search, AI-generated listings, social/forum features.

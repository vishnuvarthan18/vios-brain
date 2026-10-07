---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Wedding2day (W2D) — wedding2day.app

## 1. What this project is
- **Goal:** A B2B trade app for the Tamil Nadu wedding industry. Businesses buy, sell, rent, and ask for wedding materials and services. Every business also gets a free public profile page it can share.
- **Who it is for:** Wedding businesses in Tamil Nadu (38 districts), split into two roles:
  - **Manufacturer** — makes or supplies physical items (decor materials, garlands, furniture, invitations, rental stock). Posts items for sale/rent, keeps a Catalog, answers requirements.
  - **Vendor** — gives a wedding-day service (photography, DJ, catering, makeup, venue, priest, etc.). Posts requirements ("I need X").
- **Why it exists:** Built by [[People/Vishnu]]. (The old wedding-decoration experience line was pitch text only, not a real fact — confirmed 2026-10-06.) Sourcing is done by calls, visits and [[Tools/WhatsApp]] — slow and scattered. Founder interviews (Aug 2026) found the main pain is **"not enough business"**, so the public profile layer was added to help businesses get customers.
- **Long-term vision:** A full couple-facing wedding platform (venues, decor, photo, catering, honeymoon). **Not in scope now.**
- **Platform:** Android app ([[Tools/Google Play Console|Play Store]]). English UI only.

## 2. Status now (as of 2026-10-06)
- Paused. Last recorded work: **2026-09-02** (last app commit: EAS preview profile pinned to APK; admin last commit 2026-09-01). No work since.
- Code on the personal Mac at `~/Desktop/mura/w2d/b2b/` — 3 repos: `wedding2day-app` (Expo + Firebase), `w2d-admin` (Vite + React ops dashboard, same Firebase project), `w2d-landing`.
- **What works:**
  - Login with phone OTP ([[Tools/Firebase]] phone auth). Testers can log in using [[Tools/Firebase]] test phone numbers.
  - Profile setup with 29 categories + role (role pre-filled from category).
  - Post types: sell-used, sell-new, rental (Manufacturer only), requirement (Vendor only). Catalog (Manufacturer only, profile-only).
  - Two tabs: Available + Needs. Reveal-phone connection with daily cap, block, report.
  - Public profile page (`app/p/[slug].tsx`), share link via `w2d://` deep link only (no web URL yet).
  - Settings, account deletion, phone-number change, My Listings, expiry, search, in-app notification center, ToS screen.
  - Tier 1 matching (category beats district).
  - **Firestore rules — two records, state both:**
    - LIVE: the 2026-09-02 session log says rules (with the role lock fix) were deployed to production.
    - LOCAL: the overnight run (09-01/02) fixed 4 Firestore permission holes (e.g. `users.role` was self-writable); `E2E_FINDINGS` says these fixes are local only until `firebase deploy --only firestore` runs. 275 app tests + 29 admin rule tests pass. Not sure if the 09-02 deploy included all 4 fixes — running the deploy again is the safe single next action. See E2E_FINDINGS (archived: Raw/mac-personal/Desktop/mura/w2d/b2b/w2d-app/E2E_FINDINGS.md).
  - Preview APK built with [[Tools/EAS]] on 2026-09-02 (expired 2026-09-16).
  - Automated test suite (`npm test` with emulator wrapper).
  - `w2d-admin` web dashboard exists ([[Tools/Vite]] / [[Tools/React]]).
- **What does not work / not done:**
  - [[Tools/Gluestack UI]] migration (overnight batches 12–15) not visually checked on a device. Blocked by [[Tools/JDK]] 17 install + new dev-client build.
  - Privacy Policy and account-deletion pages not hosted at a real URL ([[Tools/Firebase]] Hosting deploy prepared, not done) — both needed for Play Store.
  - Android dev build needs openjdk 17.
  - [[Tools/Google Play Console|Play Store]] Data Safety form not done.
  - Push notifications cannot send (needs [[Tools/Firebase]] Blaze plan; Blaze blocked by [[Companies/Google]] billing bug).
  - [[Tools/WhatsApp]] OTP not built.
  - Founder said the category → role table is "too wrong" — not yet discussed.

## 3. Next steps
0. Run `firebase deploy --only firestore` so the 4 local permission fixes are surely live.
1. Ask founder exactly which category/role rows (DECISIONS.md §9) are wrong. Do not guess. Fix table + log in §17.
2. Founder decision D1: public profile shows phone openly vs reveal cap elsewhere — keep or change?
3. Install [[Tools/JDK]] 17, make a new [[Tools/EAS]] dev-client build, then visual QA of the [[Tools/Gluestack UI]] migration.
4. Build a fresh preview APK for testers (old one expired).
5. Deploy Privacy Policy to [[Tools/Firebase]] Hosting → get a real URL.
6. Fill the [[Tools/Google Play Console|Play Store]] Data Safety form.
7. Run the contacts migration to production (details not sure).
8. Find or recreate `W2D_Test_Plan.xlsx` (D4), then run the 48 test cases.
9. Give `W2D_SignUp_Flow` doc to one real Vendor and one real Manufacturer; collect corrections.
10. Run [[Tools/Maestro]] E2E tests (plan drafted 2026-08-21, not run).
11. Audit `w2d-admin` for role-model gaps.
12. Recruit 15–20 testers for [[Tools/Google Play Console|Play Store]] closed testing (14-day wait).
13. [[Tools/Google Play Console|Play Store]] submission.

## 4. Decisions
- 2026-06-03 — Start with B2B used/surplus decor resale in Tamil Nadu; seed 1–2 hub cities first — narrow niche first. #decision
- 2026-06-03 — Primary success metric = interests per listing — shows marketplace liquidity. #decision
- 2026-06-03 — Connection = reveal phone, no in-app chat — ship faster, users already use [[Tools/WhatsApp]]. #decision
- 2026-06-03 — Social/forum layer deferred; test with [[Tools/WhatsApp]] group first — avoid scope creep. #decision
- 2026-06-04 — Brand colors red + white (from logo). #decision
- 2026-06-06 — No video uploads in v1; photos only, 5MB, JPEG/PNG/WebP — slow internet in small districts, storage cost. #decision
- 2026-06-27 — Phone field = simple capture (no extra OTP) — save time. #decision
- 2026-07-06 — Stack pivot: full rebuild in [[Tools/React Native]] + [[Tools/Expo]] + [[Tools/NativeWind]] + [[Tools/Firebase]] + [[Tools/Cursor]] Pro. Old build abandoned. Old deadlines voided. Build fully first, launch in last phase. #decision
- 2026-07-06 — 11-phase build plan locked (Phase 0 toolchain → Phase 10 [[Tools/Google Play Console|Play Store]]). #decision
- 2026-07-06 — Districts = hardcoded constant, not a database table. #decision
- 2026-07-06 — [[Tools/Cursor]] rules: [[Tools/Claude]] writes prompts, one feature per prompt, new chat per phase, review diffs. #decision
- 2026-07-07 — [[Tools/Firebase]] Firestore region asia-south1 (Mumbai) — low latency. #decision
- 2026-07-08 — Use [[Tools/Firebase]] local emulators instead of real OTP during dev — Blaze plan on hold. #decision
- 2026-07-14 — [[Companies/Google]] Sign-In dropped. Phone OTP only — phone must be verified anyway for reveal. #decision
- 2026-07-14 — One new [[Tools/Cursor]] chat per phase — long chats burn tokens. #decision
- 2026-07-15 — 10 decor categories, 4 conditions, max 3 photos per listing; new listings start as `status: pending`. #decision
- 2026-07-15 — Emulator connections set once in `app/_layout.tsx` inside `__DEV__`. #decision
- 2026-07-16 — Pre-launch landing page is a separate side project. #decision
- 2026-07-21 — Collection stays named `listings` (rename to `posts` rejected) — rework not worth it. #decision
- 2026-07-25 — v1.1 scope: trade platform, not resale-only — supply chain logic; resale-only had low repeat use + cold start. #decision
- 2026-07-25 — `postType` field in `listings`; two tabs (Available / Needs); separate small form per post type. #decision
- 2026-07-25 — Rental = label only, no booking. No chat, no payments. #decision
- 2026-07-25 — Catalog moved to profile-only (not in feed). #decision
- 2026-07-25 — Admin web app added back to v1 roadmap (built last); [[Tools/Firebase]] Console until then. #decision
- 2026-07-25 — Extra product rules: soft GST badge, star reviews after reveal, manual reports, [[Tools/WhatsApp]]/email support, push on approve/reject, mark as sold, trust info on reveal screen, daily reveal cap, block, draft autosave, delivery tag, negotiable badge, neededBy date, seller stats. #decision
- 2026-07-25 — `DECISIONS.md` + `PRODUCT_CONTEXT.md` + `AGENTS.md` created — stop [[Tools/Cursor]] from "fixing" deliberate decisions. #decision
- 2026-08-01 — Plan: [[Tools/WhatsApp]] OTP via [[Tools/Cloudflare]] Worker + [[Companies/Meta]] WhatsApp API (not built). #decision
- 2026-08-01 — Vendor/Manufacturer role split added (old model). #decision
- 2026-08-01 — Matching: Tier 1 (cheap, client-side) first; Tier 2 ([[Tools/Firebase]] Cloud Function push) later, needs Blaze. #decision
- 2026-08-04 — Draft (not adopted): social layer + 3-tab Feeds/Market/Profile nav. Still parked. #decision
- 2026-08-18 — Dev paused for research; research said reveal-phone mechanic is weak and incumbents exist. #decision
- 2026-08-19 — 29-category list locked (founder supplied). #decision
- 2026-08-19 — Public, no-login, shareable profile added; phone visible openly — answers "not enough business". #decision
- 2026-08-19 — Public profile shows identity, contact, photos only — never Catalog or price. #decision
- 2026-08-19 — Role split dropped (reversed next day). #decision
- 2026-08-19 — Profile ID = slug + random suffix. Public profile photo cap = 20. #decision
- 2026-08-19 — Share link = app deep link `w2d://p/<slug>`, labelled honestly — no web version yet. #decision
- 2026-08-19 — Overnight runs allowed with zero stop conditions; agent logs its own decisions. #decision
- 2026-08-20 — Role split restored (new model): role comes from category (service = Vendor, supply = Manufacturer). One role per business. Symmetric gating. 5 "Both" categories default to Manufacturer. #decision
- 2026-08-20 — Catalog = Manufacturer only. Requirement = Vendor only. Needs tab = Manufacturers only. #decision
- 2026-08-20 — Category + role migration on one combined screen; role pre-filled, must be confirmed. #decision
- 2026-08-20 — Kept: daily caps (25 reveals / 20 posts / 10 reports), expiry (60d supply; requirement neededBy+3d or 30d), sold toggle only on approved listings. #decision
- 2026-08-20 — Phone-enumeration fix + full UI revamp done unattended in overnight run #2 (risk accepted). #decision
- 2026-08-21 — Use [[Tools/Maestro]] for automated E2E testing. #decision
- 2026-09-02 — Role field bound to category (not fully blocked) so old accounts are not stranded. #decision
- 2026-09-02 — Ship with SMS OTP for now; [[Tools/WhatsApp]] OTP = parallel, non-blocking track. #decision
- 2026-09-02 — Testers use [[Tools/Firebase]] test phone numbers (no real SMS needed). #decision

## 5. Timeline
- 2026-06-02 — Claude Pro and Claude Code started (2-3 Jun); build began on FlutterFlow + Supabase (Supabase project "W2D").
- 2026-06-03 — Strategy chat: MVP scope, competitor check, budget.
- 2026-06-04 — Feature plan, interactive UI prototype, design handoff doc.
- 2026-06-06 — Old-stack backend + app setup.
- 2026-06-19 to 06-27 — Old build: [[Companies/Google]] login, SMS OTP, profile setup (~40–45% done).
- 2026-07-04 — Cost analysis; [[Tools/Cloudflare]] migration idea dropped.
- 2026-07-06 — Stack pivot; 11-phase plan; [[Tools/Google Play Console]] registration started ($25 paid).
- 2026-07-07 — Phase 0 toolchain + Phase 1 scaffold; first [[Tools/EAS]] build.
- 2026-07-08 — Phase 2 done (app on real Android phone). [[Tools/Expo Router]] + emulators set up.
- 2026-07-14 — Phase 3 (phone OTP) + Phase 4 (profile) done. [[Companies/Google]] Sign-In removed.
- 2026-07-15 — Phase 5 (create listing + photos) done.
- 2026-07-16 — Landing page live on wedding2day.com ([[Tools/Cloudflare]] Pages); A4 QR poster made.
- 2026-07-21 — Blaze billing bug starts (`OR_BACR2_44`). Collection rename attempted and cancelled.
- 2026-07-23 — Roadmap check: git backup, userType, post-type picker, two tabs.
- 2026-07-25 — v1.1 scope locked; DECISIONS.md / PRODUCT_CONTEXT.md created; P0–P7 marked complete.
- 2026-08-01 / 08-04 — Role split, [[Tools/WhatsApp]] OTP plan, matching tiers; social layer draft.
- 2026-08-18 — Dev paused; desk + AI research.
- 2026-08-19 — Founder interviews; reconciliation; Batch 1 + 2 (category migration, rules, public profile); `w2d-admin` repo created on [[Tools/GitHub]]; overnight run #1.
- 2026-08-20 — Overnight #1 reviewed (143 tests passing, 5 big bugs fixed); rules deployed; role split restored; overnight run #2.
- 2026-08-21 — [[Tools/Maestro]] E2E plan drafted; test plan built.
- 2026-09-01 — `W2D_Test_Plan.xlsx` + `W2D_SignUp_Flow.docx` delivered; code audit found role self-write hole.
- 2026-09-01 — Overnight run fixed 4 Firestore permission holes (local); 275 app tests + 29 admin rule tests.
- 2026-09-02 — Overnight run (16 batches, 15 done); rules deployed to prod; preview APK built; tester login fixed; role mapping concern raised.

## 6. Key facts
- **People:** [[People/Vishnu]] (Vishnuvarthan V) — solo founder, non-technical, Tamil Nadu. [[Tools/GitHub]] `vishnuvarthan18`.
- **Tech stack:** [[Tools/React Native]] + [[Tools/Expo]] ([[Tools/Expo Router]], managed workflow), [[Tools/NativeWind]], [[Tools/Firebase]] (Auth, Firestore, Storage, FCM, Crashlytics, Analytics), [[Tools/EAS]] builds, [[Tools/Cursor]] Pro, [[Tools/Claude]] for planning and prompts. [[Tools/Gluestack UI]] added in Sept overnight run (not sure if final). [[Tools/Maestro]] planned for E2E.
- **[[Tools/Firebase]] project:** `wedding2day-a99ea` (asia-south1).
- **Android package:** `com.w2d.app`. App scheme `w2d`.
- **[[Tools/Expo]] account:** `vishnu18` (project `w2d`).
- **Repos (private, [[Tools/GitHub]]):**
  - https://github.com/vishnuvarthan18/wedding2day-app — main app
  - https://github.com/vishnuvarthan18/w2d-admin — admin dashboard
  - https://github.com/vishnuvarthan18/w2d-landing — landing page
- **Local folders:** `~/Desktop/mura/w2d/b2b/` (w2d-app, w2d-admin, w2d-landing) (older path `~/Desktop/w2d`; `~/W2D` is a stray empty folder).
- **Domain:** wedding2day.com (landing page, [[Tools/Cloudflare]] Pages; also https://w2d-landing.pages.dev/). Separate stack — do not reuse its code in the app.
- **Old stack (June 2026):** FlutterFlow + Supabase (abandoned 2026-07-06).
- **Cost tracker:** a Google Keep note tracks Wedding2day costs (Claude, FlutterFlow spend).
- **Chats index:** INDEX (archived: Projects/wedding2day-app/chats/INDEX.md). Project docs copied to `docs/`.
- **Landing signups:** [[Tools/Google Sheets]] "W2D Pre-Launch Registrations" via [[Tools/Google Apps Script]].
- **Emulator ports:** Auth 9099, Firestore 8080, Storage 9199, UI 4000. Mac local IP `192.168.31.16`.
- **Last preview build:** https://expo.dev/accounts/vishnu18/projects/w2d/builds/3f8bb6ba-30d9-470e-a774-c8cec17b4dc2 (commit `6edaa41`, expired 2026-09-16).
- **Collections:** `users`, `listings`, `interests`, `reports`, `blocks`, `profiles`, `users/{uid}/catalogItems`, reveal grants (exact name not sure).
- **Competitors noted:** [[Companies/IndiaMART]], [[Companies/TradeIndia]], [[Companies/Justdial]], [[Companies/Event Material Hub]], [[Companies/Evento.rent]], [[Companies/Sulekha]]; B2C: [[Companies/WedMeGood]], [[Companies/WeddingWire]], [[Companies/Meragi]].

## 7. Files and documents
- `DECISIONS.md` — locked rules, single source of truth (updated 2026-08-20).
- `PRODUCT_CONTEXT.md` — reasons behind decisions (updated 2026-08-19; outdated on role split).
- `v1` — early v1 scope note (June, old).
- `claude/RESEARCH_PHASE1_DESK.md` — desk research: incumbents, why B2B marketplaces fail.
- `claude/RESEARCH_PHASE1_AI_RUN_FULL.md` — full research: competitors, market size, 5 alternative directions (D1–D5), interview guides.
- `claude/DEV_PROMPTS_BATCH_1.md` — [[Tools/Cursor]] prompts for Tasks 1–5.
- `claude/SESSION_LOG_2026-08-19.md` — Batch 1 + 2, overnight #1 queue.
- `claude/SESSION_LOG_2026-08-20.md` — role split reversal, overnight #2 plan.
- `claude/SESSION_LOG_2026-08-21.md` — [[Tools/Maestro]] E2E plan.
- `claude/TEST_PLAN_2026-08-21.md` — notes on 48-case test plan.
- `claude/SESSION_LOG_2026-09-01.md` — test plan + sign-up flow docs.
- `claude/AUDIT_2026-09-01_ROLE_SECURITY.md` — code audit vs DECISIONS.md, fix order.
- `claude/SESSION_LOG_2026-09-02.md` — overnight run, APK, tester login fix, role concern.
- In repo only: `AGENTS.md`, `OVERNIGHT_NOTES.md`, `OVERNIGHT_RUN_LOG.md`, `E2E_FINDINGS.md`, `TEST_SHEET_RUN.md`, `MAESTRO_SETUP.md`, `docs/PRIVACY_POLICY.md`, `docs/TERMS_OF_SERVICE.md`, `firestore.rules`, `storage.rules`, `scripts/seed.mjs`.
- Delivered files: `W2D_Test_Plan.xlsx` (location unknown), `W2D_SignUp_Flow.docx` + page images, `W2D-build-plan.md`, A4 QR poster (PNG/PDF).
- Stale duplicates: `files/DECISIONS.md`, `files/PRODUCT_CONTEXT.md` (2026-07-21) — ignore.

## 8. Open questions and problems
- Category → role table is "too wrong" (founder, 2026-09-02) — which rows? Not discussed yet.
- D1: public profile phone is open and can be harvested; reveal cap guards trade side only. Keep or gate?
- **Blocker — Blaze plan:** [[Companies/Google]] billing bug `OR_BACR2_44` since 2026-07-21. Blocks [[Tools/Firebase]] Cloud Functions, push sending, Tier 2 matching.
- **Blocker — [[Tools/WhatsApp]] OTP:** needs [[Companies/Meta]] Business verification + template approval (Vishnu-only task). Status unknown.
- Vishnu-only tasks: Meta verification + WhatsApp template, public privacy policy URL, Play Store Data Safety form and listing assets, account recovery policy.
- Are the 09-01 Firestore permission fixes live? (session log says deployed 09-02; E2E_FINDINGS says local until deploy)
- **Blocker — [[Tools/Google Play Console|Play Store]]:** 12+ testers opted in for 14 days; ~7-day review; privacy policy URL; Data Safety form.
- [[Tools/Gluestack UI]] vs locked [[Tools/NativeWind]] styling — conflict? (not sure)
- Visual QA of new UI not done.
- `W2D_Test_Plan.xlsx` not found on dev machine.
- Old profiles can become orphans if slug pointer changes (needs Cloud Function).
- Available feed has no query-level role filter (skipped on purpose) — verify client filter.
- Research says reveal-phone-only mechanic is risky (leakage, low frequency). Accepted for now.
- Tamil UI rejected for v1, but research says it is the clearest gap vs competitors.
- No real web link for public profiles (only `w2d://`).
- Social layer / 3-tab nav — still draft.
- PRODUCT_CONTEXT.md not updated for 2026-08-20 role restore.
- No locked launch date.

## 9. All chats in this project
- B2B manufacturer-service provider platform- MVP to scale strategy (archived: Projects/wedding2day-app/chats/2026-06-02 B2B manufacturer-service provider platform- MVP to scale strategy.md) — 2026-06-02
- Apps and websites (archived: Projects/wedding2day-app/chats/2026-06-04 Apps and websites.md) — 2026-06-04
- Designing UI-UX wireframes (archived: Projects/wedding2day-app/chats/2026-06-05 Designing UI-UX wireframes.md) — 2026-06-05
- Flutter Flow v1 scope setup steps (archived: Projects/wedding2day-app/chats/2026-06-06 Flutter Flow v1 scope setup steps.md) — 2026-06-06
- Reviewing recent conversation highlights (archived: Projects/wedding2day-app/chats/2026-06-08 Reviewing recent conversation highlights.md) — 2026-06-08
- Canceling unwanted FlutterFlow subscription before debit (archived: Projects/wedding2day-app/chats/2026-06-17 Canceling unwanted FlutterFlow subscription before debit.md) — 2026-06-17
- Completing the process next steps (archived: Projects/wedding2day-app/chats/2026-06-19 Completing the process next steps.md) — 2026-06-19
- Project status update (archived: Projects/wedding2day-app/chats/2026-06-19 Project status update.md) — 2026-06-19
- Completing version 1 for client delivery (archived: Projects/wedding2day-app/chats/2026-06-26 Completing version 1 for client delivery.md) — 2026-06-26
- Current location status (archived: Projects/wedding2day-app/chats/2026-06-27 Current location status.md) — 2026-06-27
- Starting the next step (archived: Projects/wedding2day-app/chats/2026-06-27 Starting the next step.md) — 2026-06-27
- Migration to Cloudflare and cost analysis (archived: Projects/wedding2day-app/chats/2026-07-04 Migration to Cloudflare and cost analysis.md) — 2026-07-04
- Changed decision announcement (archived: Projects/wedding2day-app/chats/2026-07-06 Changed decision announcement.md) — 2026-07-06
- M1 project scope overview (archived: Projects/wedding2day-app/chats/2026-07-06 M1 project scope overview.md) — 2026-07-06
- Wedding2day MVP launch strategy and tech stack (archived: Projects/wedding2day-app/chats/2026-07-06 Wedding2day MVP launch strategy and tech stack.md) — 2026-07-06
- Final steps in a process (archived: Projects/wedding2day-app/chats/2026-07-07 Final steps in a process.md) — 2026-07-07
- Starting a new conversation (archived: Projects/wedding2day-app/chats/2026-07-07 Starting a new conversation.md) — 2026-07-07
- Project knowledge migration to Cursor IDE (archived: Projects/wedding2day-app/chats/2026-07-08 Project knowledge migration to Cursor IDE.md) — 2026-07-08
- Starting phase 3 (archived: Projects/wedding2day-app/chats/2026-07-08 Starting phase 3.md) — 2026-07-08
- Next steps (archived: Projects/wedding2day-app/chats/2026-07-13 Next steps.md) — 2026-07-13
- Creating a listing (archived: Projects/wedding2day-app/chats/2026-07-14 Creating a listing.md) — 2026-07-14
- QR code for customer data collection (archived: Projects/wedding2day-app/chats/2026-07-16 QR code for customer data collection.md) — 2026-07-16
- Long day, back again (archived: Projects/wedding2day-app/chats/2026-07-20 Long day, back again.md) — 2026-07-20
- W2D project development roadmap (archived: Projects/wedding2day-app/chats/2026-07-23 W2D project development roadmap.md) — 2026-07-23
- Next steps and current status (archived: Projects/wedding2day-app/chats/2026-07-25 Next steps and current status.md) — 2026-07-25
- Project summary for viOS (archived: Projects/wedding2day-app/chats/2026-10-02 Project summary for viOS.md) — 2026-10-02
- (2026-07-16 "QR code for customer data collection" is the pre-launch landing page work.)
- Note: work from 2026-08-18 to 2026-09-02 was done in other sessions (Cowork/[[Tools/Cursor]]). It exists only in the session logs (docs/), not as chats.

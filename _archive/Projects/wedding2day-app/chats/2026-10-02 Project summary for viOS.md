---
tags: chat
date: 2026-10-02
source: Claude personal account
uuid: 2f091df8-e697-4e46-a715-8db3abea8987
---
# Project summary for viOS

## Summary
**Conversation overview**

The person is Vishnu (Vishnuvarthan V), a solo non-technical founder in Tamil Nadu with 10+ years in wedding stage decoration manufacturing, building Wedding2day (W2D) — a B2B trade app for the Tamil Nadu wedding industry. The conversation centered on migrating the entire project history into his personal knowledge system (viOS, a note-taking tool reached via local device MCP tools), which Claude uses as a "second brain" for cross-session continuity.

Claude read all project chats and files, then produced a structured SUMMARY.md covering project goals, current status, decisions (dated, oldest first), timeline, key facts (tech stack, repos, Firebase project IDs), files, open problems, and a full chat list. The first save attempt to viOS timed out; Claude showed the full document as a fallback and successfully saved it on retry. Vishnu then asked Claude to make the project "connected" in viOS by adding YAML frontmatter (tags, status, owner), converting every person/company/tool/project mention into wiki-style links, and tagging decision lines with #decision. Claude created 31 new linked pages (1 person, 11 companies, 19 tools) since none existed yet, and logged the work in viOS's daily log per its RULES.md. Vishnu confirmed status should be "paused" and asked Claude to also create STATE.md (current snapshot) and LOG.md (dated history) per viOS conventions, which Claude completed, cross-linking all three files (SUMMARY, STATE, LOG).

Key people/entities in the project: Vishnu (founder), with competitors IndiaMART, TradeIndia, Justdial, Event Material Hub, Evento.rent, Sulekha, WedMeGood, WeddingWire, Meragi; tech stack includes React Native, Expo, NativeWind, Firebase, Cursor, Gluestack UI, Maestro, EAS, GitHub, Cloudflare.

**Tool knowledge**

For viOS (a local note-taking system reached via `mcp__claude-device__lcl-` prefixed tools), the required tools (write_note, read_note, list_notes, append_note) are not always present in context and must be discovered via `ToolSearch` with a `select:` query listing exact tool names (e.g., `select:mcp__claude-device__lcl-viOS-write_note,mcp__claude-device__lcl-viOS-list_notes`) — this pattern successfully surfaced the tools when they weren't initially available. `write_note` calls can time out (one attempt failed after 4 minutes with no response) and should be retried; always have a fallback of showing the full document in a code block so the person isn't blocked.

viOS has a `RULES.md` file at the root that defines house conventions — Claude read this before restructuring the project, and it specified that every project needs a `STATE.md` (current snapshot/next-steps) and `LOG.md` (dated history, newest-first) in addition to any existing summary, and that finished work should be appended to a `Daily log/<date>.md` file via `append_note`.

The person's viOS folder structure uses `People/`, `Companies/`, `Tools/`, and `Projects/<project-name>/` as top-level categories, with wiki-link syntax `[[Category/Name]]` and pipe-aliasing `[[Tools/Google Play Console|Play Store]]` to preserve natural reading text while linking to the canonical page name. New entity pages follow a strict template: YAML frontmatter with a single `tags:` field (`person`, `company`, or `tool`), an H1 title, one descriptive line, and a `- Used in: [[Projects/...]]` backlink — Claude created all 31 pages in this exact format in one batch of `write_note` calls before reporting back.

## Chat

**Vishnu:** You are helping me move this project into my personal second brain (viOS).

Step 1. Search and read ALL chats in this project, from the very first to the latest. Also read the project instructions and project files.

Step 2. Write ONE Markdown document in this exact format. Use simple English and short points. Do not skip anything important. If you are not sure about something, write "(not sure)".

# PROJECT: <project name>

## 1. What this project is
- Goal, who it is for, why it exists

## 2. Status now (as of <today's date>)
- Where it stands today
- What works, what does not

## 3. Next steps
- Ordered list of what to do next

## 4. Decisions
- <date> — <what we decided> — <why>
(every decision, oldest first)

## 5. Timeline
- <date> — <what happened>

## 6. Key facts
- People, companies, tools, tech stack
- Links, GitHub repos, servers, domains (NO passwords or secret keys)

## 7. Files and documents
- Name — what it contains

## 8. Open questions and problems

## 9. All chats in this project
- <chat title> — <date>
(list EVERY chat you can find in this project)

Step 3. If the "viOS" tool is connected, save this document with write_note to: Projects/<project-name-in-lowercase-with-dashes>/SUMMARY.md and tell me it is saved. If viOS is not connected, show the full document in one code block so I can copy it.

**Claude:** I'll go through every chat and project file in Wedding2day, build the summary, then save it to viOS.

_[tool: Projects]_

_[tool: mcp__claude_ai__recent_chats]_

_[tool: ToolSearch]_

_[tool: mcp__claude_ai__recent_chats]_

_[tool: mcp__claude_ai__recent_chats]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: mcp__claude_ai__conversation_search]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: Projects]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

viOS did not respond (the save timed out after 4 minutes), so the note is **not saved**. Here is the full document to copy:

````markdown
# PROJECT: Wedding2day (W2D) — wedding2day.app

## 1. What this project is
- **Goal:** A B2B trade app for the Tamil Nadu wedding industry. Businesses buy, sell, rent, and ask for wedding materials and services. Every business also gets a free public profile page it can share.
- **Who it is for:** Wedding businesses in Tamil Nadu (38 districts), split into two roles:
  - **Manufacturer** — makes or supplies physical items (decor materials, garlands, furniture, invitations, rental stock). Posts items for sale/rent, keeps a Catalog, answers requirements.
  - **Vendor** — gives a wedding-day service (photography, DJ, catering, makeup, venue, priest, etc.). Posts requirements ("I need X").
- **Why it exists:** The founder (Vishnu) has 10+ years in wedding stage decoration manufacturing. Sourcing is done by calls, visits and WhatsApp — slow and scattered. Founder interviews (Aug 2026) found the main pain is **"not enough business"**, so the public profile layer was added to help businesses get customers.
- **Long-term vision:** A full couple-facing wedding platform (venues, decor, photo, catering, honeymoon). **Not in scope now.**
- **Platform:** Android app (Play Store). English UI only.

## 2. Status now (as of 2026-10-02)
- Last recorded work: **2026-09-02**. No logged work for about 1 month.
- **What works:**
  - Login with phone OTP (Firebase phone auth). Testers can log in using Firebase test phone numbers.
  - Profile setup with 29 categories + role (role pre-filled from category).
  - Post types: sell-used, sell-new, rental (Manufacturer only), requirement (Vendor only). Catalog (Manufacturer only, profile-only).
  - Two tabs: Available + Needs. Reveal-phone connection with daily cap, block, report.
  - Public profile page (`app/p/[slug].tsx`), share link via `w2d://` deep link only (no web URL yet).
  - Settings, account deletion, phone-number change, My Listings, expiry, search, in-app notification center, ToS screen.
  - Tier 1 matching (category beats district).
  - Firestore rules (with role lock fix) deployed to production.
  - Preview APK built on 2026-09-02 (expired 2026-09-16).
  - Automated test suite (`npm test` with emulator wrapper).
  - `w2d-admin` web dashboard exists (Vite/React).
- **What does not work / not done:**
  - Gluestack UI migration (overnight batches 12–15) not visually checked on a device. Blocked by JDK 17 install + new dev-client build.
  - Privacy Policy not hosted at a real URL (Firebase Hosting deploy prepared, not done).
  - Play Store Data Safety form not done.
  - Push notifications cannot send (needs Blaze plan; Blaze blocked by Google billing bug).
  - WhatsApp OTP not built.
  - Founder said the category → role table is "too wrong" — not yet discussed.

## 3. Next steps
1. Ask founder exactly which category/role rows (DECISIONS.md §9) are wrong. Do not guess. Fix table + log in §17.
2. Founder decision D1: public profile shows phone openly vs reveal cap elsewhere — keep or change?
3. Install JDK 17, make a new EAS dev-client build, then visual QA of the Gluestack UI migration.
4. Build a fresh preview APK for testers (old one expired).
5. Deploy Privacy Policy to Firebase Hosting → get a real URL.
6. Fill the Play Store Data Safety form.
7. Run the contacts migration to production (details not sure).
8. Find or recreate `W2D_Test_Plan.xlsx` (D4), then run the 48 test cases.
9. Give `W2D_SignUp_Flow` doc to one real Vendor and one real Manufacturer; collect corrections.
10. Run Maestro E2E tests (plan drafted 2026-08-21, not run).
11. Audit `w2d-admin` for role-model gaps.
12. Recruit 15–20 testers for Play Store closed testing (14-day wait).
13. Play Store submission.

## 4. Decisions
- 2026-06-03 — Start with B2B used/surplus decor resale in Tamil Nadu; seed 1–2 hub cities first — narrow niche first, founder's real experience.
- 2026-06-03 — Primary success metric = interests per listing — shows marketplace liquidity.
- 2026-06-03 — Connection = reveal phone, no in-app chat — ship faster, users already use WhatsApp.
- 2026-06-03 — Social/forum layer deferred; test with WhatsApp group first — avoid scope creep.
- 2026-06-04 — Brand colors red + white (from logo).
- 2026-06-06 — No video uploads in v1; photos only, 5MB, JPEG/PNG/WebP — slow internet in small districts, storage cost.
- 2026-06-27 — Phone field = simple capture (no extra OTP) — save time.
- 2026-07-06 — Stack pivot: full rebuild in React Native + Expo + NativeWind + Firebase + Cursor Pro. Old build abandoned. Old deadlines voided. Build fully first, launch in last phase.
- 2026-07-06 — 11-phase build plan locked (Phase 0 toolchain → Phase 10 Play Store).
- 2026-07-06 — Districts = hardcoded constant, not a database table.
- 2026-07-06 — Cursor rules: Claude writes prompts, one feature per prompt, new chat per phase, review diffs.
- 2026-07-07 — Firestore region asia-south1 (Mumbai) — low latency.
- 2026-07-08 — Use Firebase local emulators instead of real OTP during dev — Blaze plan on hold.
- 2026-07-14 — Google Sign-In dropped. Phone OTP only — phone must be verified anyway for reveal.
- 2026-07-14 — One new Cursor chat per phase — long chats burn tokens.
- 2026-07-15 — 10 decor categories, 4 conditions, max 3 photos per listing; new listings start as `status: pending`.
- 2026-07-15 — Emulator connections set once in `app/_layout.tsx` inside `__DEV__`.
- 2026-07-16 — Pre-launch landing page is a separate side project.
- 2026-07-21 — Collection stays named `listings` (rename to `posts` rejected) — rework not worth it.
- 2026-07-25 — v1.1 scope: trade platform, not resale-only — supply chain logic; resale-only had low repeat use + cold start.
- 2026-07-25 — `postType` field in `listings`; two tabs (Available / Needs); separate small form per post type.
- 2026-07-25 — Rental = label only, no booking. No chat, no payments.
- 2026-07-25 — Catalog moved to profile-only (not in feed).
- 2026-07-25 — Admin web app added back to v1 roadmap (built last); Firebase Console until then.
- 2026-07-25 — Extra product rules: soft GST badge, star reviews after reveal, manual reports, WhatsApp/email support, push on approve/reject, mark as sold, trust info on reveal screen, daily reveal cap, block, draft autosave, delivery tag, negotiable badge, neededBy date, seller stats.
- 2026-07-25 — `DECISIONS.md` + `PRODUCT_CONTEXT.md` + `AGENTS.md` created — stop Cursor from "fixing" deliberate decisions.
- 2026-08-01 — Plan: WhatsApp OTP via Cloudflare Worker + Meta WhatsApp API (not built).
- 2026-08-01 — Vendor/Manufacturer role split added (old model).
- 2026-08-01 — Matching: Tier 1 (cheap, client-side) first; Tier 2 (Cloud Function push) later, needs Blaze.
- 2026-08-04 — Draft (not adopted): social layer + 3-tab Feeds/Market/Profile nav. Still parked.
- 2026-08-18 — Dev paused for research; research said reveal-phone mechanic is weak and incumbents exist.
- 2026-08-19 — 29-category list locked (founder supplied).
- 2026-08-19 — Public, no-login, shareable profile added; phone visible openly — answers "not enough business".
- 2026-08-19 — Public profile shows identity, contact, photos only — never Catalog or price.
- 2026-08-19 — Role split dropped (reversed next day).
- 2026-08-19 — Profile ID = slug + random suffix. Public profile photo cap = 20.
- 2026-08-19 — Share link = app deep link `w2d://p/<slug>`, labelled honestly — no web version yet.
- 2026-08-19 — Overnight runs allowed with zero stop conditions; agent logs its own decisions.
- 2026-08-20 — Role split restored (new model): role comes from category (service = Vendor, supply = Manufacturer). One role per business. Symmetric gating. 5 "Both" categories default to Manufacturer.
- 2026-08-20 — Catalog = Manufacturer only. Requirement = Vendor only. Needs tab = Manufacturers only.
- 2026-08-20 — Category + role migration on one combined screen; role pre-filled, must be confirmed.
- 2026-08-20 — Kept: daily caps (25 reveals / 20 posts / 10 reports), expiry (60d supply; requirement neededBy+3d or 30d), sold toggle only on approved listings.
- 2026-08-20 — Phone-enumeration fix + full UI revamp done unattended in overnight run #2 (risk accepted).
- 2026-08-21 — Use Maestro for automated E2E testing.
- 2026-09-02 — Role field bound to category (not fully blocked) so old accounts are not stranded.
- 2026-09-02 — Ship with SMS OTP for now; WhatsApp OTP = parallel, non-blocking track.
- 2026-09-02 — Testers use Firebase test phone numbers (no real SMS needed).

## 5. Timeline
- 2026-06-03 — Strategy chat: MVP scope, competitor check, budget.
- 2026-06-04 — Feature plan, interactive UI prototype, design handoff doc.
- 2026-06-06 — Old-stack backend + app setup.
- 2026-06-19 to 06-27 — Old build: Google login, SMS OTP, profile setup (~40–45% done).
- 2026-07-04 — Cost analysis; Cloudflare migration idea dropped.
- 2026-07-06 — Stack pivot; 11-phase plan; Play Console registration started ($25 paid).
- 2026-07-07 — Phase 0 toolchain + Phase 1 scaffold; first EAS build.
- 2026-07-08 — Phase 2 done (app on real Android phone). Expo Router + emulators set up.
- 2026-07-14 — Phase 3 (phone OTP) + Phase 4 (profile) done. Google Sign-In removed.
- 2026-07-15 — Phase 5 (create listing + photos) done.
- 2026-07-16 — Landing page live on wedding2day.com; A4 QR poster made.
- 2026-07-21 — Blaze billing bug starts (`OR_BACR2_44`). Collection rename attempted and cancelled.
- 2026-07-23 — Roadmap check: git backup, userType, post-type picker, two tabs.
- 2026-07-25 — v1.1 scope locked; DECISIONS.md / PRODUCT_CONTEXT.md created; P0–P7 marked complete.
- 2026-08-01 / 08-04 — Role split, WhatsApp OTP plan, matching tiers; social layer draft.
- 2026-08-18 — Dev paused; desk + AI research.
- 2026-08-19 — Founder interviews; reconciliation; Batch 1 + 2 (category migration, rules, public profile); `w2d-admin` repo created; overnight run #1.
- 2026-08-20 — Overnight #1 reviewed (143 tests passing, 5 big bugs fixed); rules deployed; role split restored; overnight run #2.
- 2026-08-21 — Maestro E2E plan drafted; test plan built.
- 2026-09-01 — `W2D_Test_Plan.xlsx` + `W2D_SignUp_Flow.docx` delivered; code audit found role self-write hole.
- 2026-09-02 — Overnight run (16 batches, 15 done); rules deployed to prod; preview APK built; tester login fixed; role mapping concern raised.

## 6. Key facts
- **People:** Vishnu (Vishnuvarthan V) — solo founder, non-technical, Tamil Nadu. GitHub `vishnuvarthan18`.
- **Tech stack:** React Native + Expo (Expo Router, managed workflow), NativeWind, Firebase (Auth, Firestore, Storage, FCM, Crashlytics, Analytics), EAS builds, Cursor Pro. Gluestack UI added in Sept overnight run (not sure if final). Maestro planned for E2E.
- **Firebase project:** `wedding2day-a99ea` (asia-south1).
- **Android package:** `com.w2d.app`. App scheme `w2d`.
- **Expo account:** `vishnu18` (project `w2d`).
- **Repos (private):**
  - https://github.com/vishnuvarthan18/wedding2day-app — main app
  - https://github.com/vishnuvarthan18/w2d-admin — admin dashboard
  - https://github.com/vishnuvarthan18/w2d-landing — landing page
- **Local folders:** `~/Desktop/mura/w2d/b2b/w2d-app`, `~/Desktop/mura/w2d/b2b/w2d-admin` (older path `~/Desktop/w2d`; `~/W2D` is a stray empty folder).
- **Domain:** wedding2day.com (landing page, Cloudflare Pages; also https://w2d-landing.pages.dev/). Separate stack — do not reuse its code in the app.
- **Landing signups:** Google Sheet "W2D Pre-Launch Registrations" via Google Apps Script.
- **Emulator ports:** Auth 9099, Firestore 8080, Storage 9199, UI 4000. Mac local IP `192.168.31.16`.
- **Last preview build:** https://expo.dev/accounts/vishnu18/projects/w2d/builds/3f8bb6ba-30d9-470e-a774-c8cec17b4dc2 (commit `6edaa41`, expired 2026-09-16).
- **Collections:** `users`, `listings`, `interests`, `reports`, `blocks`, `profiles`, `users/{uid}/catalogItems`, reveal grants (exact name not sure).
- **Competitors noted:** IndiaMART, TradeIndia, Justdial, Event Material Hub, Evento.rent, Sulekha; B2C: WedMeGood, WeddingWire, Meragi.

## 7. Files and documents
- `DECISIONS.md` — locked rules, single source of truth (updated 2026-08-20).
- `PRODUCT_CONTEXT.md` — reasons behind decisions (updated 2026-08-19; outdated on role split).
- `v1` — early v1 scope note (June, old).
- `claude/RESEARCH_PHASE1_DESK.md` — desk research: incumbents, why B2B marketplaces fail.
- `claude/RESEARCH_PHASE1_AI_RUN_FULL.md` — full research: competitors, market size, 5 alternative directions (D1–D5), interview guides.
- `claude/DEV_PROMPTS_BATCH_1.md` — Cursor prompts for Tasks 1–5.
- `claude/SESSION_LOG_2026-08-19.md` — Batch 1 + 2, overnight #1 queue.
- `claude/SESSION_LOG_2026-08-20.md` — role split reversal, overnight #2 plan.
- `claude/SESSION_LOG_2026-08-21.md` — Maestro E2E plan.
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
- **Blocker — Blaze plan:** Google billing bug `OR_BACR2_44` since 2026-07-21. Blocks Cloud Functions, push sending, Tier 2 matching.
- **Blocker — WhatsApp OTP:** needs Meta Business verification + template approval (external wait). Status unknown.
- **Blocker — Play Store:** 12+ testers opted in for 14 days; ~7-day review; privacy policy URL; Data Safety form.
- Gluestack UI vs locked NativeWind styling — conflict? (not sure)
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
- B2B manufacturer-service provider platform: MVP to scale strategy — 2026-06-03
- Apps and websites — 2026-06-04
- Flutter Flow v1 scope setup steps — 2026-06-06
- Project status update — 2026-06-19
- Completing the process next steps — 2026-06-21
- Completing version 1 for client delivery — 2026-06-27
- Current location status — 2026-06-27
- Starting the next step — 2026-06-27
- Migration to Cloudflare and cost analysis — 2026-07-04
- M1 project scope overview — 2026-07-06
- Changed decision announcement — 2026-07-07
- Starting a new conversation — 2026-07-07
- Final steps in a process — 2026-07-08
- Starting phase 3 — 2026-07-08
- Project knowledge migration to Cursor IDE — 2026-07-08
- Next steps — 2026-07-14
- Creating a listing — 2026-07-15
- QR code for customer data collection — 2026-07-16
- W2D project development roadmap — 2026-07-23
- Long day, back again — 2026-07-25
- Next steps and current status — 2026-07-25
- Note: work from 2026-08-18 to 2026-09-02 was done in other sessions (Cowork/Cursor). It exists only in the session logs, not as chats in this project.
````

Notes:
- I left out secrets on purpose: the Apps Script endpoint URL, the SHA-1 fingerprint, and the test phone OTP code.
- To retry the save, make sure the viOS app is running on your Mac, then say "save it". I'll write it to `Projects/wedding2day-app/SUMMARY.md`.

**Vishnu:** Please try to save the document again now with the viOS write_note tool.

**Claude:** _[tool: mcp__claude-device__lcl-viOS-write_note]_

Saved to `Projects/wedding2day-app/SUMMARY.md` in viOS.

**Vishnu:** Now make this project connected in viOS:

1. Read Projects/wedding2day-app/SUMMARY.md.
2. Add this at the very top of the file (frontmatter):
---
tags: project
status: <active / paused / done>
owner: "[[People/Vishnu]]"
---
3. In the text, turn every person, company, tool, and other project into a link:
   - People: [[People/Name]]
   - Companies: [[Companies/Name]]
   - Tools and tech: [[Tools/Name]] (e.g. [[Tools/Firebase]])
   - Other projects: [[Projects/project-name]]
   - Put the tag #decision at the end of every decision line.
4. Save the updated SUMMARY.md with write_note.
5. For each person, company and tool you linked, if the page does not exist yet, create a short page with write_note:
   ---
   tags: person (or company, or tool)
   ---
   # Name
   - One line about who/what it is
   - Used in: [[Projects/wedding2day-app/SUMMARY]]
6. Tell me the list of pages you created.

**Claude:** _[tool: ToolSearch]_

_[tool: mcp__claude-device__lcl-viOS-read_note]_

_[tool: mcp__claude-device__lcl-viOS-list_notes]_

_[tool: mcp__claude-device__lcl-viOS-read_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: ToolSearch]_

_[tool: mcp__claude-device__lcl-viOS-append_note]_

SUMMARY.md is updated with the frontmatter, links and #decision tags. I created 31 new pages (none existed before).

**People (1)**
- `People/Vishnu.md`

**Companies (11)**
- `Companies/Google.md`
- `Companies/Meta.md`
- `Companies/IndiaMART.md`
- `Companies/TradeIndia.md`
- `Companies/Justdial.md`
- `Companies/Event Material Hub.md`
- `Companies/Evento.rent.md`
- `Companies/Sulekha.md`
- `Companies/WedMeGood.md`
- `Companies/WeddingWire.md`
- `Companies/Meragi.md`

**Tools (19)**
- `Tools/Firebase.md`
- `Tools/React Native.md`
- `Tools/Expo.md`
- `Tools/Expo Router.md`
- `Tools/EAS.md`
- `Tools/NativeWind.md`
- `Tools/Gluestack UI.md`
- `Tools/Cursor.md`
- `Tools/Claude.md`
- `Tools/Maestro.md`
- `Tools/JDK.md`
- `Tools/Vite.md`
- `Tools/React.md`
- `Tools/GitHub.md`
- `Tools/Cloudflare.md`
- `Tools/WhatsApp.md`
- `Tools/Google Play Console.md`
- `Tools/Google Sheets.md`
- `Tools/Google Apps Script.md`

A few things to check:
- **Status:** I set it to `paused` because the last recorded work was 2026-09-02. Change it to `active` if you are still building.
- **Other projects:** I made no project links. The landing page and admin dashboard are parts of W2D, not separate viOS projects.
- **"Play Store" links:** these use `[[Tools/Google Play Console|Play Store]]`, so the text still reads "Play Store".
- **Daily log:** your RULES.md says to log finished work, so I added one line to `Daily log/2026-10-02.md`.
- **Missing files:** RULES.md says every project needs `STATE.md` and `LOG.md`. `Projects/wedding2day-app/` does not have them yet. Say if you want me to create them from the summary.

**Vishnu:** Status: paused is correct (change to active if I am still building). Yes, create STATE.md and LOG.md from the summary.

**Claude:** _[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-write_note]_

_[tool: mcp__claude-device__lcl-viOS-append_note]_

I created both files and added a line to today's daily log.

- **`Projects/wedding2day-app/STATE.md`:** where the project stands now, the 12 next steps, the 3 blockers and the key repo links. It has the same frontmatter as SUMMARY.md, including `status: paused`.
- **`Projects/wedding2day-app/LOG.md`:** 23 dated entries from 2026-06-03 to today, newest on top.

Both files link to each other and to SUMMARY.md. When you start building again, change `status` to `active` in SUMMARY.md and STATE.md.

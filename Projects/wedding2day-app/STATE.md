---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: Wedding2day (W2D)

Full history: [[Projects/wedding2day-app/SUMMARY]] · Log: [[Projects/wedding2day-app/LOG]]

## Where we are (as of 2026-10-06)
- Status: **paused**. Last work 2026-09-02 (app commit); admin last commit 2026-09-01.
- App mostly built: phone OTP login, profile with 29 categories + role (role follows category), Available/Needs tabs, role-gated posting, Catalog, reveal-phone, public profile, settings, My Listings, Tier 1 matching.
- **Firestore rules:**
  - Live: rules with the role lock fix were deployed to production on 2026-09-02 (session log).
  - Local: the overnight run fixed 4 permission holes (e.g. self-writable `users.role`); E2E_FINDINGS says these are local until `firebase deploy --only firestore` runs. Not sure all 4 are in the 09-02 deploy.
- Tests: 275 app tests + 29 admin rule tests.
- Testers log in with [[Tools/Firebase]] test phone numbers. Last preview APK expired 2026-09-16.
- New [[Tools/Gluestack UI]] look built but not checked on a phone.

## Next steps
1. Run `firebase deploy --only firestore` (makes the local fixes surely live).
2. Ask Vishnu which category → role rows (DECISIONS.md §9) are wrong; decide D1 (phone open on public profile?).
3. Install openjdk 17 → new [[Tools/EAS]] dev-client build → visual check; fresh preview APK.
4. Hosting deploy: privacy policy + account deletion URLs; run production contact migration.
5. Play Store Data Safety form; find/recreate `W2D_Test_Plan.xlsx`; run 48 tests and [[Tools/Maestro]] E2E.
6. Recruit 15–20 testers (14-day closed test), then submit.

## Blockers
- [[Tools/Firebase]] Blaze plan blocked by [[Companies/Google]] billing bug `OR_BACR2_44` (since 2026-07-21) — blocks Cloud Functions, push, Tier 2 matching.
- Vishnu-only: Meta Business verification + WhatsApp template (WhatsApp OTP), privacy URL, Data Safety form, listing assets, account recovery policy.

## Key places
- Local: `~/Desktop/mura/w2d/b2b/` (wedding2day-app, w2d-admin, w2d-landing)
- Repos: github.com/vishnuvarthan18/wedding2day-app, /w2d-admin, /w2d-landing
- Source of truth: `DECISIONS.md` in the repo root; findings: [[Raw/mac-personal/Desktop/mura/w2d/b2b/w2d-app/E2E_FINDINGS]]

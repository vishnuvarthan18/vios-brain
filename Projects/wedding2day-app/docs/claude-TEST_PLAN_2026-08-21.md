# W2D — Full Test Plan (2026-08-21)

Delivered as `W2D_Test_Plan.xlsx` (3 tabs). Built from DECISIONS.md (last
locked 2026-08-20, role split restored §0b/§7/§9) and PRODUCT_CONTEXT.md —
not from the Maestro plan drafted in `claude/SESSION_LOG_2026-08-21.md`
(that log's plan is code-execution via a dev agent in Cursor; this is the
manual/QA test-case layer, complementary, not a replacement).

## Tabs

1. **Vendor User Stories** (19 stories, V1–V19) — auth, category+role
   confirm, migration prompt, requirement posting, gating off supply
   listings/Catalog, no general Needs feed, responder list, reveal
   (outbound), Available tab browsing, My Listings, My Interests, public
   profile (contact visible, no catalog), block/report, ratings, settings,
   role-aware onboarding framing.
2. **Manufacturer User Stories** (21 stories, M1–M21) — mirrors Vendor
   plus: supply-listing posting (3 types), rental-is-label-only, Catalog
   (Manufacturer-exclusive), Needs tab (Tier 1 category-over-district
   ranking), requirement response (reversed reveal direction), seller
   stats, daily reveal cap + the known phone-enumeration gap (flagged,
   unresolved as of 2026-08-20 — do not assume fixed).
3. **Full Test Sheet** (48 test cases, TC-001–TC-048, 9 sections) —
   covers auth/onboarding, post creation + role gating (incl. negative/
   direct-Firestore-write attempts), Catalog gating, Available/Needs tab
   gating, reveal mechanic + trust-context fields + the known reveal-cap
   gap, public profile, My Listings/settings/ToS+Privacy release gates,
   explicit out-of-scope negative checks (no chat/booking/payments/Tamil
   UI/social layer), and an admin (w2d-admin) role-migration verification
   item. Status column has a dropdown (Not Run/Pass/Fail/Blocked/N/A).

## Flagged while building this (needs founder/dev verification, not yet re-confirmed)

- TC-030 / M18: phone-enumeration gap in §6 — DECISIONS.md says unresolved
  as of 2026-08-20; test sheet treats this as expected-fail until
  reverified, not as a new bug.
- TC-045: Privacy Policy + Play Store Data Safety form — §14 flags this as
  NOT done (RELEASE GATE). Test sheet surfaces it as a blocking check.
- TC-048: w2d-admin's second (role) migration — §14 item 0b/§11 describe
  this as needed; status unverified, sheet asks to check rather than
  assume.
- TC-028: per-business reveal dedupe across multiple listings from the
  same seller — same nuance the 2026-08-21 Maestro plan flagged; worth
  confirming actual behavior once, then keeping both plans in sync.

## Next step

Run this manually (or hand to QA) against the emulator + seed data,
alongside/after the Maestro suite once that's built per
`SESSION_LOG_2026-08-21.md`. Fold any real bugs found here into
`E2E_FINDINGS.md` rather than a separate file, so there's one findings log.

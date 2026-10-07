# W2D — logic & permissions validation run

**Date:** 2026-09-01 (overnight run, Batch 10)
**Against:** Firebase emulator (Firestore 8080 / Auth 9099 / Storage 9199) with
`w2d-app/scripts/seed.mjs` (8 users, 25 listings, 6 catalog items) and
`w2d-admin/scripts/seed-admin.mjs` (admin allowlist, role fixtures, contact docs).

---

## READ THIS FIRST — what this document is, and is not

Batch 10 asked for `W2D_Test_Plan.xlsx`'s Full Test Sheet, 48 cases,
TC-001–TC-048.

**That file does not exist.** It is not in this repo, not in `w2d-admin`, not
in `docs/` or `docs/archive/`, and not anywhere under
`/Users/vishnuvarthanv/Desktop/mura`. The only `.xlsx` on this machine in that
tree is `w2d/d2c/wedding2day.com/data/wedding2day-ops-call-sheets.xlsx`, which
belongs to a different product and which DECISIONS.md §10 and §19.8 forbid
drawing on. The only mention of `W2D_Test_Plan.xlsx` anywhere is
`OVERNIGHT_RUN_PROMPT.md` itself.

**So the case IDs below are mine, not that sheet's.** They are `TL-` prefixed
specifically so nobody later mistakes them for TC-001–TC-048 and assumes the
sheet was run. Reconstructing 48 case IDs somebody else wrote and reporting
Pass against them would be a fabricated result — the exact failure mode
DECISIONS.md §19.10 records a previous session being *right* to refuse.

What this run does deliver is the batch's stated purpose: *"this run validates
LOGIC/PERMISSIONS correctness before the visual layer changes."* Every
functional area DECISIONS.md defines is exercised against the emulator and seed
data, and the result is recorded per area with Pass/Fail/Blocked.

---

## Result: 266 assertions across 27 areas, 0 failures

Plus, in `w2d-admin` against the same emulator and the same rules: **29 client-SDK
rules cases and 5 admin checks, all PASS** (see Batch 9).

| ID | Area | DECISIONS.md | Assertions | Result |
|---|---|---|---|---|
| TL-001 | Tier 1 matching + Available-feed role filter | §16, §5, §7 | 21 | **PASS** |
| TL-002 | Listing expiry + free-text search | §15, §14.5 | 20 | **PASS** |
| TL-003 | Daily caps + UTC day key (pure logic) | §6, §14.6 | 50 | **PASS** |
| TL-004 | `users` reads, writes, role lock, phone lock | §3, §6, §7, §9 | 24 | **PASS** |
| TL-005 | Role gate — all six combinations | §4, §7 | 10 | **PASS** |
| TL-006 | `listings` create/update/delete + `postType` lock | §4, §7 | 6 | **PASS** |
| TL-007 | Account deletion permissions | §14.2 (RELEASE GATE) | 4 | **PASS** |
| TL-008 | Reveal grants — the enumeration fix | §6 | 21 | **PASS** |
| TL-009 | `interests` + doc-id/payload binding | §6 | 10 | **PASS** |
| TL-010 | `catalogItems` Manufacturer-exclusivity | §4, §6a | 3 | **PASS** |
| TL-011 | `profiles` public showcase + field allowlist | §6a | 11 | **PASS** |
| TL-012 | In-app notification centre permissions | §15 | 10 | **PASS** |
| TL-013 | FCM device tokens | §14.3 | 2 | **PASS** |
| TL-014 | Daily caps against the emulator | §6, §14.6 | 14 | **PASS** |
| TL-015 | Reports & blocks | §15 | 2 | **PASS** |
| TL-016 | Storage rules (3 photo paths) | §3, §6a | 5 | **PASS** |
| TL-017 | Phone change keeps the public profile in step | §6a | 4 | **PASS** |
| TL-018 | Business-details edit + denormalization backfill | §3 | 4 | **PASS** |
| TL-019 | Combined category+role migration, end to end | §3, §7 | 3 | **PASS** |
| TL-020 | Catalog editor | §4 | 3 | **PASS** |
| TL-021 | My Listings: status, expiry, delete | §15 | 6 | **PASS** |
| TL-022 | Role journeys, end to end (both roles) | §4, §7 | 5 | **PASS** |
| TL-023 | Requirement responders | §7, §15 | 2 | **PASS** |
| TL-024 | Reveal grants, replayed as the client runs them | §6 | 8 | **PASS** |
| TL-025 | Reveal cap, replayed as the client runs it | §6 | 3 | **PASS** |
| TL-026 | Notification lifecycle | §15 | 8 | **PASS** |
| TL-027 | Account deletion sequence, end to end | §14.2 (RELEASE GATE) | 7 | **PASS** |

Reproduce with `npm test` in `w2d-app` and `npm run verify` in `w2d-admin`,
with the emulator running (§13.3) and both seed scripts applied.

---

## BLOCKED — cannot be verified from this environment

These are not failures. They are cases no emulator run can settle, listed so
the human knows exactly what this pass did *not* cover.

| ID | Area | Why blocked |
|---|---|---|
| TL-B01 | Real SMS OTP delivery | The emulator never sends SMS (§13.10) — the code is read from the Emulator UI. Only a real device on a real project settles this. |
| TL-B02 | Push notification **delivery** | Sending needs Cloud Functions, which need Blaze, which is blocked by billing bug `OR_BACR2_44` (§8, §13). Token registration is covered (TL-013); delivery cannot be. |
| TL-B03 | Composite index behaviour | §13.4: the emulator does **not** enforce indexes. Any index-dependent query is unverified until `firebase deploy --only firestore`. |
| TL-B04 | Native module behaviour (Analytics, Crashlytics, Messaging) | Loaded defensively and no-op without a dev-client build (§13.1, §13.8). Nothing here exercises the native path. |
| TL-B05 | Visual / layout correctness | No screenshot or device tooling in this environment. This is Batch 16's subject, and it is blocked there too — see `OVERNIGHT_RUN_LOG.md`. |
| TL-B06 | Production migration state | §6 and §14.6 are only closed in production once `npm run migrate:contacts -- --production` has been run. Not run — Run Rule 7 forbids production actions. |

---

## Note on re-running after the UI batches

Batch 10 asks that relevant cases be re-run after Batch 16. Every area above is
verified through the data and rules layers, not through screens, so a component
library swap cannot change any of these results — but the suite is one command
and the re-run is recorded in `OVERNIGHT_RUN_LOG.md` under Batch 16 regardless.

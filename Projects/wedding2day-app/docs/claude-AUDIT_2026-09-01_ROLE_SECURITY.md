# Code Audit vs DECISIONS.md — 2026-09-01

Run in Cursor agent (9 finders + adversarial verifiers + completeness critic) against commit `a50a4f8`. Checks whether §14 item 0b "DONE" markers (overnight 2026-08-21) hold in actual code. Nothing modified — read-only audit.

## Verdict summary

| # | Item | Verdict |
|---|---|---|
| 1 | role field in users/listings/profiles (§3,§7) | BUILT |
| 2 | Rules: role gate on listings create + catalogItems | PARTIALLY BUILT |
| 3 | One-time confirm-role prompt (§14-0b) | BUILT (with pre-fill bug) |
| 4 | profile-setup.tsx role pre-fill | BUILT |
| 5 | Needs/Post-picker/Available role filter (§5) | PARTIALLY BUILT — Available feed NOT filtered |
| 6 | Tier 1 matching factors role (§16) | BUILT (matches §16 literal scope) |
| 7 | seed.mjs role fixtures | BUILT |
| 8 | Phone-enumeration gap (§6) | FIXED — with 2 caveats |
| 9 | Privacy Policy + Play Data Safety form | PARTIALLY — policy text yes, hosted URL no, Data Safety form NOT BUILT |

## Critical security findings (not in original 9 rows)

**A. `users/{uid}.role` is self-writable via client update/create — no field allowlist in firestore.rules (create: line 237, owner-update allowlist at 240-247 covers only status/verified, not role).** This breaks every role gate in row 2 — `roleOf()` reads this exact field. Directly contradicts DECISIONS.md locked text: "a business does not choose its own role freely" and "role corrections are an ADMIN action... no role picker anywhere." No test covers writing role directly. **Highest-priority fix.**

**B. `interests` doc id not bound to payload** (rules.rules:649-653) — no check that `interestId == listingId + '_' + buyerId`. A vendor can write an interest doc keyed to a requirement while payload points at a supply listing, passing the response gate and forging the notification proof-of-interaction.

**C. §6 vs §6a scope collision** — public `profiles/{slug}.phone` is openly readable and reachable via `listings.sellerId → profileSlug → profiles/{slug}`, uncapped by the §6 reveal cap. App never does this walk itself; rules permit it. This is a product decision (§6a says don't harmonize with §6 without asking) — needs founder call, not a silent code fix.

## Other real gaps

- postType not immutable on listing update — create-then-mutate defeats the row-2 gate symmetry (flows.test.mjs:585-611 asserts this is allowed).
- Phone field can still be written back onto `users/{uid}` — no allowlist blocks it (dead code path, not a live leak since nothing writes it, but a standing permission).
- Available feed has zero role filtering, even client-side — full collection fetched (firestore.ts:846-849).
- confirm-business.tsx doesn't pre-fill existing category for accounts migrating — blind re-pick risk, and writes users only (no listings/profiles backfill).
- Play Store Data Safety form: 0 artifacts in repo — RELEASE GATE, not done.
- Privacy Policy text exists (privacy.ts, ~1650 words) but not hosted anywhere (no URL in firebase.json/app.json).
- w2d-admin (sibling repo) not audited — 3 of §14-0b's 10 bullets ungraded, incl. the only sanctioned place to set role per §17:745.

## Recommended fix order (before any new feature work)

1. Lock down `users.role` — add to owner-update allowlist as blocked, and to create validation (must equal category's locked role from §9 table, not client-supplied). This is the load-bearing fix — everything else in row 2 depends on it.
2. Bind `interests` doc id to payload (mirror the `revealGrants` pattern already in the codebase).
3. Founder decision on §6/§6a collision — either accept public numbers are always harvestable (update §6's wording) or gate profile phone display too.
4. Make `postType` immutable on update.
5. Available feed: add role filter (decide if query-level or client-level is acceptable per §16 scope).
6. confirm-business.tsx: pre-fill existing category; backfill listings/profiles on role confirm.
7. Play Store Data Safety form (release gate) + host the Privacy Policy at a real URL.
8. Audit w2d-admin separately for the same role-model gaps.

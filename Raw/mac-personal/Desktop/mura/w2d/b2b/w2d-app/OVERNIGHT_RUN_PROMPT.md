# W2D Overnight Full Run — Cursor Agent Prompt

Paste this whole thing into Cursor's agent chat, in the `wedding2day-app` repo, on a clean working tree at commit `a50a4f8` (or later if you've since fixed something manually — check `git log` first).

---

## RUN RULES — READ FIRST

You are running UNATTENDED overnight. No human will answer questions. Follow these rules strictly:

1. **Never stop to ask a question.** If something is ambiguous, make the most conservative choice (the one that changes the least, breaks the least, and is easiest to reverse), write down what you chose and why in `OVERNIGHT_RUN_LOG.md`, and keep going.
2. **Read `DECISIONS.md` in full before starting.** It is the single source of truth. If anything below conflicts with it, `DECISIONS.md` wins — log the conflict and skip that step rather than guess.
3. **Work batch by batch, in the exact order below.** Do not skip ahead. Do not merge batches.
4. **After every batch:** run the existing test suite (`npm test` or whatever the repo's test script is), commit with a clear message (`git add -A && git commit -m "batch N: <what>"`), and append a short entry to `OVERNIGHT_RUN_LOG.md` — what you did, what you found, what tests passed/failed, anything flagged for human review.
5. **If a batch's tests fail and you can't fix it after 2 real attempts**, do not block the whole run — commit what you have on a separate note in the log as "BLOCKED", revert that batch's changes if they're unsafe to leave half-done, and move to the next batch.
6. **Never touch code outside a batch's stated scope.**
7. **Do not deploy anything to production** (no `firebase deploy`, no Play Store actions). All work stays local/committed, ready for the human to review and deploy themselves.
8. **If a batch requires a product/business decision (not a technical one), do not decide it yourself.** Flag it clearly in the log under a `## NEEDS HUMAN DECISION` section and move on without making that change.

---

## BATCH 1 — Lock down `role` field (CRITICAL SECURITY FIX)

**Problem:** `users/{uid}.role` is currently self-writable by the client — no field allowlist blocks it in `firestore.rules` (create at ~line 237, owner-update allowlist at ~240-247 covers only `status`/`verified`, not `role`). This breaks every role-based permission gate in the app, since `roleOf()` reads this exact field.

**Fix:**
- Add `role` to the fields blocked from owner self-update (same treatment as `status`/`verified`).
- On `create`, validate that the submitted `role` matches the locked role for the submitted `category`, per the table in `DECISIONS.md` §9 (for "Both" categories, validate against the founder-assigned default listed there — Manufacturer in all 5 cases as of the current table).
- Add/extend `rules.test.mjs` with a test that directly attempts `update(users/{uid}, {role: 'manufacturer'})` as a non-admin owner and asserts it's rejected.
- Add a test that attempts `create` with a `role` that doesn't match its `category`'s locked role and asserts rejection.

## BATCH 2 — Bind `interests` doc id to payload

**Problem:** `interests` creation (~firestore.rules:649-653) checks `buyerId`, `mayRespondToListing(payload.listingId)`, and the daily cap — but never verifies `interestId == listingId + '_' + buyerId`, which is the format the client always uses. This lets a doc id and its payload disagree.

**Fix:** Mirror the `revealGrants` rule's id-binding pattern (already in the codebase — find it and copy the approach) onto the `interests` create rule. Add a rules test that writes `interests/{listingA_id}_{uid}` with a payload pointing at a different listing and asserts rejection.

## BATCH 3 — Make `postType` immutable on listing update

**Problem:** `flows.test.mjs` (~585-611) currently asserts owner updates to a listing are unrestricted, meaning `postType` can be changed after creation — defeating the role-gate-on-create logic entirely (create as an allowed type, then mutate to a forbidden one).

**Fix:** Add a rule that blocks `postType` from changing on update (compare `request.resource.data.postType == resource.data.postType`). Update the test at flows.test.mjs:585-611 to reflect the new restriction (other fields stay updatable; only `postType` locks). Confirm no other legitimate flow in the app currently relies on changing postType post-creation — grep the app code for any `.update(..., { postType`.

## BATCH 4 — Block phone re-add on `users/{uid}`

**Problem:** Contact info was moved to `users/{uid}/contact/info` (per the §6 fix), but nothing in the rules explicitly blocks a client from writing a `phone` field back onto `users/{uid}` directly. No current code path does this, but it's a standing permission gap.

**Fix:** Add `phone` to the blocked-fields list on both `users` create and update rules. Add a test asserting a write of `{phone: '...'}` to `users/{uid}` is rejected.

## BATCH 5 — Available feed role filter

**Problem:** The Available feed has no role filtering anywhere — not in the Firestore query (fetches the whole `listings` collection per `firestore.ts` ~846-849) and not client-side. Per DECISIONS.md §16, Tier 1 matching/filtering should account for role where relevant.

**Fix:** Add a query-level filter so the Available feed only shows listings from Manufacturer-role sellers (mirroring the existing `sellerRole` denormalized field already on listing docs — do not add a new field). Update `matching.test.ts` with role-based test cases (it currently has zero, per the audit). Do not change ranking logic (district+category) — only the audience/filter layer, consistent with how the Needs tab was already scoped in `matching.ts`.

## BATCH 6 — Fix confirm-business.tsx pre-fill + backfill

**Problem:** `confirm-business.tsx` does not pre-fill the account's existing `category` (it's `useState(null)` with no read of the existing profile) — migrating users have to blind-repick from 29 rows, and a mistake silently rewrites their category. It also only writes to `users` — not `listings` or `profiles` — so a role/category correction here doesn't propagate. It also fails open on a read error.

**Fix:**
- Load the existing `category` (and `role`, if already set) from the user's profile on mount and pre-fill the form.
- On confirm, if `category` changed, backfill the denormalized `sellerCategory`/`sellerRole` fields on the user's own existing `listings` docs, and the `profiles` doc.
- On a read error, fail CLOSED (block progress, show a retry state) instead of failing open.
- Add/update a test covering the pre-fill and the backfill.

## BATCH 7 — Host Privacy Policy at a real URL

**Problem:** Privacy Policy text exists (`privacy.ts`, ~1650 words, current) but is not hosted anywhere — no URL in `firebase.json` or `app.json`. This is a Play Store RELEASE GATE requirement (a live URL is mandatory).

**Fix:** Set up Firebase Hosting (if not already configured) to serve the rendered privacy policy at a stable path (e.g. `wedding2day-a99ea.web.app/privacy` or similar — check if a custom domain is already configured in the repo first, use that if so). Include a working account-deletion-request URL/section, since Play Store requires this too if the app supports account deletion (it does, per DECISIONS.md §14 item 2). Do NOT deploy to production — prepare the hosting config and rendered output, commit it, and log the exact `firebase deploy` command the human needs to run manually.

## BATCH 8 — Draft Play Store Data Safety form answers

**Problem:** The Play Store "Data Safety" form (separate from Privacy Policy text) has zero artifacts in the repo — it must be filled in Play Console manually, but nobody has drafted the answers.

**Fix:** Write a new file `PLAY_DATA_SAFETY_ANSWERS.md` that maps every data type W2D actually collects (phone number, name, business name, category/role, district, photos, listing content, device identifiers if FCM/Crashlytics/Analytics collect them — check `app.json`/relevant SDK configs for what's actually wired in) to Play's Data Safety form categories (collected/shared, purpose, optional/required, encrypted in transit, user can request deletion). Base this only on what's actually in the code — do not guess at data collection that isn't implemented. This is a drafting task, not a form submission — the human pastes these answers into Play Console themselves.

## BATCH 9 — Audit + fix w2d-admin role parity

**Problem:** The audit didn't cover the sibling `w2d-admin` repo. Per DECISIONS.md §11/§17, `w2d-admin` is the only sanctioned place to correct a business's role — this matters even more now that Batch 1 has locked role against client self-write.

**Fix:** If `w2d-admin` is available as a sibling directory or separate clone, run the same audit questions against it: does `seed-admin.mjs` include role fixtures matching §9? Do `verify-rules.mjs`/`verify-admin.mjs` correctly assert the new Batch 1-4 rules (not the old ones)? Does the admin UI have a working role-correction action for a business (this is now the ONLY way to fix a wrong role, per Batch 1)? If `w2d-admin` is not accessible from this working directory, log this clearly under NEEDS HUMAN DECISION/ACTION and skip.

## BATCH 10 — Run the full test sheet

Run `W2D_Test_Plan.xlsx`'s Full Test Sheet (48 cases, TC-001–TC-048) against the Firebase emulator + seed data. Since Batches 12-16 (UI) come after this and will touch the same screens, this run validates LOGIC/PERMISSIONS correctness before the visual layer changes — do not skip re-running relevant cases after Batch 16 if time allows, but the primary pass happens here. Log Pass/Fail/Blocked per case in a copy of the sheet or a markdown table.

## BATCH 11 — Write E2E_FINDINGS.md

Consolidate every bug found in Batches 1-10 into a single `E2E_FINDINGS.md` (per the existing plan from `SESSION_LOG_2026-09-01.md` — don't create scattered files). Explicitly flag the §6/§6a public-profile-phone-harvestable issue here as NEEDS HUMAN DECISION (do not fix it — it's a product call about whether public profiles should stay reveal-free).

## BATCH 12 — Install Gluestack UI v2

Install Gluestack UI v2 (`@gluestack-ui/*` + its NativeWind integration) per its official Expo setup docs. Wire up the provider and map the theme tokens (colors, spacing, radii) to match W2D's current brand colors — inspect current `tailwind.config.js`/NativeWind theme extension for the existing palette and carry it over exactly, do not invent new brand colors. Confirm the app still builds and boots after install before moving on.

## BATCH 13 — Migrate core UI primitives

Migrate the shared/reused components first: buttons, text inputs, cards, badges (role badge, verification badge), the tab bar. Keep visual output as close to current as possible — this is a component-library swap, not a redesign. Screenshot before/after for each primitive if a screenshot tool is available in this environment; if not, just do the migration and note in the log that visual QA needs a human pass.

## BATCH 14 — Migrate high-traffic screens

Migrate: Available feed, Needs feed, Post intent picker, listing detail/reveal screen — using the Batch 13 primitives. Do not change functionality, layout structure, or copy — only the underlying component implementation.

## BATCH 15 — Migrate remaining screens

Migrate: Profile, Settings, Catalog editor, public profile route (`p/[slug].tsx`), auth screens (phone/OTP/confirm-business/profile-setup). Same constraint as Batch 14 — visual parity, not redesign.

## BATCH 16 — Visual QA pass

Re-run the app, go through every migrated screen, and log anything that looks visually broken (misaligned, wrong color, overflow, missing icon) in the log under NEEDS HUMAN DECISION — do not attempt to "improve" layout choices, only flag actual breakage. If a screenshot tool is available, capture each screen.

---

## FINAL STEP

Write a top-level summary at the end of `OVERNIGHT_RUN_LOG.md`: total batches completed, total batches blocked, full list of everything under NEEDS HUMAN DECISION across all batches, and the single next action the human should take when they wake up.

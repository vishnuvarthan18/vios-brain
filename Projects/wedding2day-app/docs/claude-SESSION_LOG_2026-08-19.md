# W2D — Session Log, 2026-08-19 (Batch 1 + Batch 2 dev run)

Companion to `DEV_PROMPTS_BATCH_1.md`. This is the execution log — what
was actually run, verified, and pushed, plus the overnight queue kicked
off at end of session. Read this alongside `DECISIONS.md` (source of
truth) and `DEV_PROMPTS_BATCH_1.md` (the prompts).

---

## Environment fix (before Task 1)

The app folder had a broken name: `w2d-ad\pp` (stray backslash, shell-
escaping accident). This broke Node's ESM resolver — `node scripts/seed.mjs`
could not run at all. Renamed to `w2d-app`. Confirmed path:
`~/Desktop/mura/w2d/b2b/w2d-app`. Sibling: `~/Desktop/mura/w2d/b2b/w2d-admin`
(separate git repo, no shared remote with w2d-app).

## Repos

- `wedding2day-app` — https://github.com/vishnuvarthan18/wedding2day-app
  (private). Existing repo, had real history before today.
- `w2d-admin` — https://github.com/vishnuvarthan18/w2d-admin (private).
  **Did not exist before today** — created fresh mid-session (no remote
  configured locally until then). Now linked and pushed.

---

## BATCH 1 — Foundation work (DECISIONS.md §14 item 0) — ALL 5 TASKS DONE

| Task | What | Commit(s) | Status |
|---|---|---|---|
| 1 | Read DECISIONS.md/PRODUCT_CONTEXT.md, full codebase audit, no code changes | — | ✅ Done (audit output captured below) |
| 2 | 29-category schema/constants migration (`BusinessCategory`, additive alongside `userType`) | `b9ba427`, cleanup `4ed4f92` | ✅ Done, pushed |
| 3 | Removed Vendor-only requirement-create gate in `firestore.rules`; added 4 rule tests to `verify-rules.mjs` | `153a6ab` | ✅ Done, pushed. All 4 new + 3 pre-existing tests pass. Negative control confirmed (old gate → case 1 fails; new gate → passes). |
| 4 | `profiles` collection + public security rules + storage rules for public photos | `6c77918` | ✅ Done, pushed. 14/14 rules tests pass, incl. anonymous-can-read-phone and anonymous-cannot-enumerate. |
| 5 | `w2d-admin` category migration (Users screen, Dashboard tile, `verify-admin.mjs`) | `9c64a20` (w2d-admin repo, first-ever push — repo created this session) | ✅ Done, pushed |

### Task 1 audit findings (kept for reference, not re-run)
- No `profiles` collection/route/public screen existed before today (confirmed by grep — zero hits).
- No anonymous-readable Firestore path existed anywhere before Task 4 — every rule required `isSignedIn()`.
- Vendor-only requirement gate confirmed present at `firestore.rules:71-81`, exactly as DECISIONS.md §7 described.
- Two stale duplicate docs exist at `files/DECISIONS.md` and `files/PRODUCT_CONTEXT.md`, dated 2026-07-21, describing the pre-pivot single-select `userType` model. **Not deleted — still there, still stale. Flag to any future agent that greps and finds these: wrong file, use root `DECISIONS.md`.**

### Key decisions made during Batch 1 (all confirmed by user)
- Profile doc ID: slug + random suffix (not owner uid, not bare slug) — keeps it unguessable/share-only.
- `verified` badge: excluded from public profile (not in §6a's exposed-field list).
- Admin (`w2d-admin`) can update/delete profile docs — matches existing pattern on listings/reports.
- Seed data: mixed — 4 of 8 seed users get a category, 4 deliberately left `null` to exercise both the populated and the "missing category" UI paths.
- Public profile photo cap: 20 (agent's own defensive-hygiene number, not in DECISIONS.md — flagged as easy to raise later).
- `w2d-admin` `category` field: NOT written by `seed-admin.mjs` — `seed.mjs` (in w2d-app) remains sole owner of category fixture data, avoids two scripts writing the same field.

### Bugs caught and fixed during Batch 1
- `updatePublicProfile` originally used `set()` without merge — would have silently wiped `createdAt` on every edit. Fixed to `update()`.
- A failed profile-create after the `profileSlug` pointer was written would have permanently locked a business out of retrying. Fixed: create now checks whether the pointed-at doc actually exists before treating it as "already have a profile."

### Known limitation, not fixed (flagged for later)
One-profile-per-business is enforced only via the `users/{uid}.profileSlug` pointer. If a business rewrites that pointer to a new slug, the old profile becomes an orphan (still live, still public, nothing garbage-collects it). Needs a transaction or Cloud Function to close properly — Cloud Functions are Blaze-blocked (§8/§13). Relevant when the profile-editing UI (Batch 2 Task 7) is exercised for real.

---

## BATCH 2 — Public profile UI + migration prompt (§14 item 1 prep)

| Task | What | Commit | Status |
|---|---|---|---|
| 6 | Public-facing profile screen `app/p/[slug].tsx`, no-auth route, `scheme: "w2d"` added to `app.json` | `627ee32` | ✅ Done, pushed |
| 7 | Profile-editing screen `app/settings/public-profile.tsx` (authenticated), 20-photo cap enforced, share-link via `w2d://p/<slug>` (labelled honestly — no real domain exists yet) | `74cf882` | ✅ Done, pushed |
| 8 | One-time "pick your category" migration prompt for legacy accounts + **profile-setup.tsx migrated to collect real 29-item category at signup** (scope expanded per user decision — closes the gap rather than leaving new signups to hit the prompt too) | Not yet committed as of end-of-session — **first thing the overnight run must do is commit this** | ✅ Built and verified, ⚠️ commit pending |

### Key decisions made during Batch 2
- Link mechanism for Task 6: "Route + add scheme" — `app/p/[slug].tsx` + `scheme: "w2d"` in `app.json`. Real web URL (browser-openable, no app required) explicitly NOT built — would require `react-native-web` + Firebase JS SDK alongside RNFirebase (native-only), a stack decision §2 doesn't cover. Scoped as a future separate task.
- Share button behavior (Task 7): "App deep link, labelled honestly" — shares `w2d://p/<slug>`, UI states plainly it only opens for people who already have the app. Isolated in `buildProfileShareUrl()` — one-line swap once a real domain exists.
- Task 8 scope: **expanded** beyond the original prompt (which only said "flag the profile-setup gap, don't fix it") to actually fix `profile-setup.tsx`. Reason found: it was writing `category: null` for literally every new signup — this wasn't a hypothetical gap, it was already live and broken. User chose to fix it same-task rather than leave it.

### Bugs found and fixed during Batch 2
- **`.exists` bug (RNFirebase v25):** `if (!doc.exists)` negates a *method reference* (always false) instead of calling it — `doc.exists()`. This silently broke "not found" detection everywhere it appeared. Fixed in `fetchPublicProfile` and `createPublicProfile` (Task 6). **Five more instances remain unfixed on purpose** — in `getCurrentUserProfile` and ~4 other call sites (~lines 110, 409, 445, 468, 706 in `app/(auth)/_lib/firestore.ts`). Not fixed because `getCurrentUserProfile()` currently returns a hollow object instead of `null` when no doc exists, and other auth-flow screens may be silently depending on that. This is flagged as now **actively shaping other design decisions** (e.g. Task 8's category-prompt gate had to guard on route segments instead of a null-check, specifically because of this). **Recommend a dedicated audit/fix task for this cluster soon — it's no longer just latent risk, it's already influencing how new code is written.**
- Task 8: fixed a loop risk between the root-level gate hook and the picker screen (different hook instances) via an explicit `notifyCategoryChosen()` module-level notifier called before navigation.

---

## OVERNIGHT RUN — kicked off end of session, 2026-08-19 night

User explicitly requested **zero stop conditions** — agent was instructed
not to pause for approval under any circumstance, and instead to make its
own call on any decision that would normally warrant asking, log the
decision + reasoning to `OVERNIGHT_NOTES.md`, and keep going. This is a
deliberate, informed trade-off the user chose (flagged clearly before
proceeding) — full autonomy in exchange for zero interruption.

**Queue, in order:**
1. Commit Task 8 (pending from end of live session)
2. Task 9 — Profile/Settings screen (§14 item 1): notifications, account
   deletion flow, in-app phone number change, catalog editor, link to
   Task 7's public-profile editor, sign-out
3. Task 10 — Terms of Service + in-app account deletion (§14 item 2,
   RELEASE GATE)
4. Task 11 — Push notifications/FCM infra (§14 item 3) — instructed to
   log and skip whatever needs Blaze (billing bug, §8/§13), not bypass it
5. Task 12 — Matching engine, Tier 1 ONLY (§14 item 4, §16) — Tier 2
   explicitly forbidden (Blaze-gated)
6. Task 13 — My Listings + expiry + free-text search (§14 item 5)
7. Task 14 — Ops hygiene: reveal cap, spam throttling, Crashlytics/
   Analytics visibility (§14 item 6)
8. Task 15 (research only) — full legacy `userType`/old-category
   reference audit across both repos
9. Task 16 (research only) — Play Store readiness audit (Data Safety
   form prep, permissions justifications, §6a public-contact disclosure)

**Rules given:** work in order, no skipping; testing pass (edge cases,
empty states, error paths) after each task's own build verification;
commit+push each task individually before moving to the next; do NOT
start the full UI revamp (§14 item 7) or admin app (item 8) — both stay
deliberately parked; stop only for a fatal environment failure (e.g.
emulator process dying) — everything else gets a self-made decision
logged to `OVERNIGHT_NOTES.md`.

**⚠️ FIRST THING NEXT SESSION:** read `OVERNIGHT_NOTES.md` in full before
touching anything else. It is the substitute review checkpoint for the
live approvals skipped overnight — treat every self-made decision logged
there as unreviewed until read and confirmed, not as already-approved.

---

## Standing environment notes

- Emulator ports: Auth 9099 · Firestore 8080 · Storage 9199 · UI 4000
  (per DECISIONS.md §13). Must be running for any rules-verification work.
- `firestore-debug.log` and `firebase-debug.log` are gitignored — if either
  shows as modified in `git status`, it's emulator noise, not a real change.
- `w2d-admin/src/lib/types.ts` had a pre-existing uncommitted comment-only
  edit to `UserStatus` (not from any task in this log) — harmless, gets
  swept into whatever commit runs next, flagged here so it's not mistaken
  for new work.

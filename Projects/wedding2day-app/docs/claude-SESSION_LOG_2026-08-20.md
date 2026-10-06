# W2D — Session Log, 2026-08-20 (Role split reversal + overnight run #2)

Continuation of `claude/SESSION_LOG_2026-08-19.md`. Read that first for
full history through the end of the first overnight run. This log covers
everything that happened today, 2026-08-20, up to kicking off tonight's
(second) overnight run.

---

## Morning: reviewed overnight run #1 results

Read `OVERNIGHT_NOTES.md` from the 2026-08-19→20 run. All 9 tasks (9-16)
shipped clean — 143 automated tests passing. Five headline bugs found and
fixed: `expressInterest` never wrote a document (RNFirebase v25 `.exists`
is a method, not a property — silently broke My Interests/responders
list/interest counts everywhere), account deletion destroyed data before
checking if the Auth delete would succeed, Tier 1 matching could never
have matched (two disjoint category taxonomies), `users` collection was
listable by any signed-in account (reveal cap was "guarding a door with no
wall"), push notifications were dead on arrival (no `POST_NOTIFICATIONS`
declared).

**Actions taken on the 4 flagged decisions — all confirmed keep-as-is:**
- Daily caps (25 reveals / 20 posts / 10 reports per day) — kept
- Listing expiry windows (60d supply, neededBy+3d or 30d requirements,
  30d renewal) — kept
- Tier 1 ranking (category outranks district) — kept
- Sold/unavailable toggle only on `approved` listings — kept

**Deploy steps completed:**
- `firebase deploy --only firestore` — rules + indexes live on real
  Firebase (not just emulator)
- EAS dev-client rebuild — completed successfully, app now loads on
  physical device
- Confirmed OTP login works (emulator Auth doesn't send real SMS — read
  the code from the Emulator UI at `127.0.0.1:4000/auth` during local dev,
  noted as a standing gotcha)

**One item deliberately deferred, not fixed yet at the time:** the
phone-number enumeration gap (§6) — any signed-in account can still read
other users' phone numbers one at a time by walking `listings` for
`sellerId`s, bypassing the daily reveal cap. User originally said "fix
before deploy," then later in the day said include it in tonight's
overnight run instead (see below) — accepting unattended risk on a
security-sensitive change.

---

## Midday: the Vendor/Manufacturer role-split reversal

User asked for "separate login" for manufacturer/wholesaler vs supplier —
initially ambiguous, clarified through several rounds of questions into a
concrete, confirmed model. **This directly reverses DECISIONS.md §7**,
which had removed the role split on 2026-08-19. The reversal is a genuine,
deliberate founder decision — not a misunderstanding to walk back.

### The confirmed model

- **Vendor** = service-side wedding businesses (photographers, decorators,
  DJs, caterers, makeup artists, priests, musicians, event hosts, venues,
  counselors, financial services, travel booking)
- **Manufacturer** = wholesale/raw-material suppliers who sell TO vendors
  (physical decor materials, garlands, furniture, invitations, ornamental
  stock, rental equipment)
- Every business has BOTH a `category` (one of 29, unchanged list) AND a
  new `role` field (`vendor` | `manufacturer`) — role is NOT free-choice,
  it's derived from category via a locked lookup table
- **Gating is fully SYMMETRIC and confirmed stricter than the original
  2026-08-01 model:** Manufacturers can ONLY post `sell-used`/`sell-new`/
  `rental`/catalog; Vendors can ONLY post `requirement`. Neither can cross
  into the other's post types at all (the old model only restricted
  requirement creation).
- **One role per business, even for the 5 ambiguous "Both" categories**
  (Welcome Entrance, Stage Decoration, Costume Rental Service, Audio and
  Lighting, LED Wall) — no dual-role registration. Accepted tradeoff: these
  5 categories lose access to 4 of 5 post types under their assigned
  default role.
- **Catalog is now Manufacturer-exclusive** (was open to all as of
  2026-08-19)
- **The two migration prompts (category, then role) are COMBINED into ONE
  screen** — not sequential. Role pre-fills live from category selection,
  always requires explicit confirmation.

### The category → role table (drafted by AI, accepted directionally by
founder on 2026-08-20 — NOT validated against real interviews, correctable
later)

| Category | Role | Category | Role |
|---|---|---|---|
| Banana Tree | Manufacturer | Ice Cream and Beeda | Manufacturer |
| Green Panthal | Manufacturer | Return Gift | Manufacturer |
| Welcome Entrance | **Both → Manufacturer** | Honeymoon Trip | Vendor |
| Welcome Girls | Vendor | Psychiatrist for Marriage Counseling | Vendor |
| Welcome Toys | Manufacturer | Financial Support | Vendor |
| Plate Decors | Manufacturer | Costume Rental Service | **Both → Manufacturer** |
| Stage Decoration | **Both → Manufacturer** | Furniture for Wedding | Manufacturer |
| Photography and Videos | Vendor | Wedding Dress Materials | Manufacturer |
| DJ | Vendor | Ornamental Rental Service | Manufacturer |
| Catering | Vendor | New Ornamental Sales | Manufacturer |
| Bridal Makeup | Vendor | Mahal or Mandabam | Vendor |
| Maalai | Manufacturer | Invitation | Manufacturer |
| Iyer | Vendor | Audio and Lighting | **Both → Manufacturer** |
| Mangala Vathiyam | Vendor | LED Wall | **Both → Manufacturer** |
| RJ | Vendor | | |

Full table with reasoning is in DECISIONS.md §9.

### DECISIONS.md updated and pushed

**Important process note:** the first attempt to update DECISIONS.md wrote
it to the claude.ai Project's document store only — NOT to the actual git
repo. The dev agent correctly caught this (verified via `git log`, `git
diff HEAD origin/main`, file mtimes — found zero evidence of the update in
the real repo) and refused to proceed rather than guess at a 29-row
role-mapping table that didn't actually exist in the file it could read.
This was the right call by the agent. Fixed by: writing the file locally,
delivering it via SendUserFile, user downloaded and copied it into the
real `DECISIONS.md`, committed as `3db79bc` on the `wedding2day-app` repo,
pushed successfully.

**New/changed sections in DECISIONS.md:**
- **§0b (NEW)** — explains the reversal, full reasoning, what does/doesn't
  change, confirms the foundation work from 2026-08-19/20 is NOT wasted
  (only the role-gate removal from overnight commit `6f479b3` is reversed;
  category-migration work stands)
- **§7 (REWRITTEN)** — the restored role model, symmetric gating, "Both"
  category handling, explicit note that this reverses `6f479b3` specifically
- **§9 (UPDATED)** — added the full role column/table to the 29 categories
- **§3, §4, §5, §6, §6a, §11, §14, §15, §16, §17** — all updated to
  reference `role` where relevant
- **§12** — added new rejected-ideas rows (asymmetric gating, two sequential
  migration prompts, dual-role registration — all considered and rejected
  in favor of the confirmed model above)

---

## Evening: scoping the second overnight run

User wants the app "complete soon" — decided to use tonight's overnight
capacity for much more than just the role-split task, given last night's
run only used ~3-4 hours of a full night.

**Scope decisions made (all via explicit confirmation):**
- Zero stop conditions again (same as night 1) — even though tonight's
  work includes a live security-mechanic redesign (§6's reveal-grant fix)
- Include the phone-enumeration security fix tonight, unattended — user
  explicitly accepted this risk trade-off after it being named plainly
- Include Play Store content-prep items that don't need live review
  (Privacy Policy rewrite, permission strip, tagline fix)
- Include the FULL UI/UX revamp (§14 item 7) tonight too — normally
  sequenced last, now pulled forward
- **Critical sequencing constraint, confirmed by user:** the UI revamp
  MUST run only after the role-split (Phase 1) and security/content work
  (Phase 2) are built AND self-verified by the agent — not in parallel,
  not before. Reason: revamp touches the NEW screens Phase 1 builds
  (the combined migration screen, restored role-gated tabs); revamping
  before they're settled risks redoing visual work.

---

## Tonight's overnight run — kicked off, in progress

**Structure: 3 phases, 9 tasks, run inside one long unattended session.**

### Phase 1 — Role split (must complete + self-verify before Phase 2)
1. Restore Vendor/Manufacturer role split: `role` field on
   users/profiles/listings, `BUSINESS_ROLE_MAP` constant (verbatim from
   §9), combined category+role migration screen, symmetric
   `firestore.rules` gates (listings create + catalogItems create),
   restore the 6 sites overnight commit `6f479b3` removed (revised to use
   `role` not `userType`), update Tier 1 matching, update both seed
   scripts, invert `verify-admin.mjs`/`verify-rules.mjs` assertions, new
   rules tests (6 allow/deny combinations).
2. w2d-admin UI for role (Users screen, Dashboard tile, filtering).
3. Explicit VERIFY step — full test suite + manual trace of every gated
   action for both roles + full migration-screen flow, logged PASS/FAIL,
   before Phase 2 is allowed to start.

### Phase 2 — Hardening + content (after Phase 1 verified)
4. **Fix the phone-enumeration gap (§6)** — redesign contact-info storage,
   e.g. a per-pair "reveal grant" document checked via `exists()` in the
   `users` read rule, created only on a legitimate (cap-respecting)
   reveal. Most security-sensitive change of the night — instructed to be
   thorough with rules tests here specifically.
5. ToS role-model consistency pass (`app/legal/terms.tsx`).
6. Privacy Policy rewrite (`app/legal/privacy.tsx` + `docs/PRIVACY_POLICY.md`),
   fix `docs/TERMS_OF_SERVICE.md` (contradicts real in-app ToS), strip
   unused `WRITE_EXTERNAL_STORAGE` permission, prep content-rating answers
   doc (report-only — cannot submit itself).
7. Signup tagline fix (`app/(auth)/phone.tsx:72`).
8. Explicit VERIFY step again before Phase 3 is allowed to start.

### Phase 3 — Full UI/UX revamp (only after Phase 1+2 verified clean)
9. Establish a proper design system (spacing/typography/color scale,
   consistent components) if one doesn't exist; apply across EVERY screen
   built so far — both tabs, Post flow, Profile/Settings + sub-screens, My
   Listings, the new combined migration screen, public profile screen +
   editor, listing detail/reveal screen, notification center, legal
   screens. Special attention to empty/loading/error states, the reveal
   screen (§6: "highest-trust moment"), and the public profile screen
   (§6a: the one screen non-users see). Must NOT change data model,
   business logic, navigation structure, or permission gating — visual/
   interaction pass only; log any restructuring urges to
   OVERNIGHT_NOTES.md instead of acting on them. Re-run full test suite
   after — should not break anything if truly visual-only.

**Estimated total time: 7-11 hours** (Phase 1: 2-3h, Phase 2: 2-3h,
Phase 3: 3-5h+) — expected to use most or all of a full night, unlike
night 1 which finished in ~3-4 hours.

---

## ⚠️ FIRST THING NEXT SESSION

1. Read `OVERNIGHT_NOTES.md` in full — it now has TWO nights of entries
   (2026-08-19/20 run, then tonight's run appended after). Read tonight's
   entries specifically; do not assume anything from night 1 changed.
2. Check whether Phase 3 (UI revamp) actually ran, or whether the agent
   stopped/ran out of time during Phase 1 or 2 — the run may not have
   completed everything given the expanded scope.
3. **Given zero stop conditions were used again, and this run includes a
   live security-mechanic redesign (§6 reveal-grant) done unattended**,
   review that specific change with extra care before trusting it — it
   was flagged as needing thorough rules-test coverage, verify that
   coverage is real and not just claimed.
4. If Phase 3 ran, this is the first visual output from the whole project
   — review it as a human, not just via test-pass output, since "does this
   look like a proper product" isn't something automated tests can confirm.

---

## Standing environment notes (carried from 2026-08-19 log, still true)

- Folder is `~/Desktop/mura/w2d/b2b/w2d-app` (renamed from the broken
  `w2d-ad\pp` on 2026-08-19).
- Repos: `wedding2day-app` (main app) and `w2d-admin` (admin dashboard,
  created 2026-08-19) — both on GitHub, both private, separate repos with
  separate remotes.
- Emulator ports: Auth 9099 · Firestore 8080 · Storage 9199 · UI 4000. Must
  be running for any rules-verification work — start with
  `firebase emulators:start` from `w2d-app`, leave the terminal open.
- `firestore-debug.log` / `firebase-debug.log` are gitignored — modified
  status on these is emulator noise, not a real change.
- Dev-client build requires `expo start --dev-client` (Metro) running
  separately for the app to load any JS on device.
- Firebase emulator Auth does not send real SMS — read OTP codes from
  `http://127.0.0.1:4000/auth` during local dev.

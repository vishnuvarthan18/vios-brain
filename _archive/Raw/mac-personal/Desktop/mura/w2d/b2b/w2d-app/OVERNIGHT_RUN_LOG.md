# W2D — Overnight Run Log (2026-09-01 → 2026-09-02)

Unattended run per `OVERNIGHT_RUN_PROMPT.md`, starting from a clean tree at
`a50a4f8`. `DECISIONS.md` read in full before Batch 1 (Run Rule 2).

**Baseline before any change:** 8 suites, 226 tests, all green.
`firebase emulators:start --only firestore,auth,storage` running throughout.

---

## BATCH 1 — Lock down `role` (CRITICAL SECURITY FIX) — DONE

**Premise verified against code, not taken on trust (§19.7).** Both halves of
the reported hole were real:
- `firestore.rules` `users` create was `isSignedIn() && request.auth.uid == userId`
  — no field validation of any kind.
- the owner-update allowlist blocked `['status', 'verified']` only, so `role`
  was freely self-writable.

`roleOf()` reads exactly this field, so all six §7 role gates — which the
existing suite covered and which passed — could be walked through by writing
yourself the other role first. The gates were real permissions; their input
was self-service.

### DEVIATION FROM THE BATCH TEXT (deliberate, logged per Run Rule 1)

The batch says: *"Add `role` to the fields blocked from owner self-update
(same treatment as `status`/`verified`)."* **A flat block is not
implementable, and I did not implement it.** Two shipped, mandatory flows are
owner-side client writes of `role`:

1. `setCurrentUserCategoryAndRole()` — the §3 / §14-item-0b **combined
   "confirm your category & role" screen**. This is the mandatory, blocking
   migration for every account created 2026-08-19–20. Those accounts have a
   category and no role, and under §7's symmetric gates they can create
   *nothing at all*. This one client write is their only way out.
2. `updateBusinessDetails()` — the profile editor, where changing category
   moves role with it by §7's own design.

There is no Cloud Function to perform either write instead (Blaze blocked, §8 /
§13). A flat block would therefore **permanently strand every pre-role
account** with no recovery path — a worse outcome than the hole.

**What I did instead — strictly stronger where it counts, and it closes the
actual escalation:** a written `role` must equal the role §9 locks to the
`category` on the same document. §7 already states "a business does not choose
its own role freely; it is determined by category", so this enforces the
documented invariant rather than inventing one. An owner can no longer *pick* a
role. Both shipped flows keep working, because both write the §9 map's own
answer.

Two carve-outs, each tested:
- **Absent role passes.** That is the pre-migration state, and such an account
  can create nothing — it is exactly who the confirm screen exists for.
- **An unchanged `role` is exempt on update.** Without this, §17's admin
  role-correction would be undone: a "Both"-category business corrected by ops
  to the non-default role would be unable to edit even its own name.

### Changes
- `firestore.rules` — added `vendorCategories()` (12 rows), `manufacturerCategories()`
  (17 rows), `lockedRoleFor(category)` and `submittedRoleMatchesCategory()`;
  wired into `users` create and the owner branch of `users` update.
- `tests/rules.test.mjs` — 11 new tests, including both cases the batch names
  explicitly (owner `update` to `{role:'manufacturer'}` rejected; `create` with
  a role contradicting its category rejected), plus the reverse direction, a
  junk role, an off-taxonomy category, and **regression guards for the confirm
  screen and profile editor** so a future flat-block attempt fails loudly.
- `tests/roles.test.ts` — 3 new tests. `firestore.rules` cannot import
  `BUSINESS_ROLE_MAP`, so Batch 1 had to transcribe §9's table a second time.
  Two hand-maintained copies of a permission table drift; these parse the rules
  file as text and assert it agrees with the map row for row. **Mutation-tested**
  (injected a wrong row; both guards failed as intended, then restored).

### Tests
240/240 across 8 suites (was 226). rules 95→106, roles 19→22.

### Flagged for human review
- Role is now a pure function of category, and **category remains
  user-selectable**. So a business can still move its own role by changing its
  category (Catering → Banana Tree drags Vendor → Manufacturer). That is §7's
  design, not a bypass, and §15's "soft" verification + manual moderation is
  the stated backstop — but if the intent was that role never moves without ops,
  the *category* field is where that has to be enforced. See NEEDS HUMAN
  DECISION at the end of this log.

---

## BATCH 2 — Bind `interests` doc id to payload — DONE

**Premise verified.** `interests` create checked `buyerId`,
`mayRespondToListing(payload.listingId)` and the daily cap, and nothing else —
no id binding. Confirmed the `{listingId}_{buyerId}` format is what every
writer actually uses before pinning it: `expressInterest()` in
`app/(auth)/_lib/firestore.ts:1046`, and `w2d-admin/scripts/seed-admin.mjs:70`
(Admin SDK, bypasses rules anyway). No other writer exists.

**Why it mattered more than a tidiness fix.** The id is load-bearing:
`expressInterest()` dedupes by *reading that exact id*, and the rules make
interests immutable and undeletable specifically so one response stays one
response. With the id unbound, a buyer could write `interests/{listingB}_{uid}`
carrying a payload naming listing A, repeat it once per listing id it knows,
and have every write pass the dedupe check (a different document each time)
while charging listing A a fresh `interestCount` increment and another row in
its responders list. §4's Manufacturer-only response gate reads the *payload's*
listing, so one permitted response could be replayed as many.

### Changes
- `firestore.rules` — `interests` create now requires
  `interestId == payload.listingId + '_' + payload.buyerId`, mirroring the
  `revealGrants` rule at `firestore.rules:722` as the batch directed.
- `tests/rules.test.mjs` — 4 new tests: the id/payload listing mismatch the
  batch names, the **replay exploit run end to end** (one honest response
  succeeds, the replays under fresh ids fail), the buyer-half mismatch, and a
  payload with no `listingId`.

### Tests
244/244. rules 110 (was 106). No other suite touched.

### Flagged
Nothing. No product decision involved.

---

## BATCH 3 — `postType` immutable on listing update — DONE

**Premise verified.** The `listings` update rule had no field constraints on
the owner branch at all, so `postType` was freely mutable after creation. Since
§7's role gate runs on **create only**, that defeated the entire role model:
create the one type your role allows, then edit it into one it does not.

**Grep for legitimate post-creation writers, as the batch required — none
exist.** `postType` is written exactly once, at `PostForm.tsx:236`, inside the
create. `setListingAvailability()` (`firestore.ts:924`) writes `status` only,
and there is no listing-edit screen (`app/listing/` holds only `[id].tsx`, a
viewer). `w2d-admin` reads `postType` in `Dashboard.tsx`, `Listings.tsx` and
`Requirements.tsx` and never writes it.

**Applied to admins too**, on that evidence: exempting ops would add a
capability nobody uses, on the one field that decides who may post what.
Compared both sides with `get('postType', '')` so a legacy listing that lacks
the field stays editable while still refusing to have one grafted on.

### Changes
- `firestore.rules` — `listings` update now requires
  `request.resource.data.get('postType','') == resource.data.get('postType','')`,
  hoisted above the actor branches so it binds every path.
- `tests/flows.test.mjs` — updated the test the batch names (now at
  `flows.test.mjs:585`, "the role gate does NOT block reads, edits, deletes or
  counter bumps"): its "each owner still fully controls their own post" claim
  is now qualified, with an added assertion that title/price stay updatable.
  New test alongside it covering both escalation directions, a same-role type
  swap, and re-sending the same value with a real edit (so a client that writes
  the whole document back is not broken).
- `tests/rules.test.mjs` — the admin half of that assertion, placed here
  because `flows.test.mjs` has no admin fixture and adding one would have been
  out of scope.

### Tests
250/250. rules 111, flows 50.

### Flagged
Nothing.

---

## BATCH 4 — Block phone re-add on `users/{uid}` — DONE

**Premise verified, and it was a live gap rather than a theoretical one.** The
`users` match block has carried this instruction since 2026-08-20:

> DO NOT ADD `phone` (or any other contact field) BACK TO THIS DOCUMENT. Doing
> so silently reopens the enumeration hole, and no test would fail.

That was literally accurate — it was a comment, not a permission, and no test
failed. `get` on `users/{uid}` is open to every signed-in account, so a single
`phone` field turns the whole `listings` collection back into a phone book,
which is the §6 hole the contact subcollection and reveal grants exist to close.

### Shape of the fix, and why not the obvious one
Written as **"this write may not add or change a phone"**, not "the result may
not contain one". The stricter form would have broken the migration in both
directions:
- `migrateOwnContactIfNeeded()` and `updateCurrentUserPhone()` both call
  `update({ phone: FieldValue.delete() })`. Removing the field is the *fix*,
  not the attack, and must stay allowed.
- an account whose silent migration has not run yet still carries a stale
  `phone`, and must still be able to edit its name and district. Denying that
  would lock unmigrated users out of their own profile over a field they did
  not write.

Both are pinned by tests. Applied **outside the actor branches so it binds
admins too**: nothing writes this field anywhere (the app has two
`FieldValue.delete()` calls and no setter; `w2d-admin`'s Users screen reads
numbers from the `contact` collection group), so an ops exemption would be
capability nobody uses on the field that reopens §6's hole.

### Three pre-existing tests were asserting the hole
Not anticipated by the batch, found by running the suite. `flows.test.mjs` had
three tests writing `phone` onto `users/{uid}` and asserting it **succeeded** —
a shape the app stopped producing on 2026-08-20. They were stale, not correct:
- `updating users.phone then profiles.phone both succeed` → rewritten as
  `a phone change writes the contact doc and the public profile`, using
  `users/{uid}/contact/info`, which is what `updateCurrentUserPhone()` writes.
- `phone change with no public profile is a no-op` → same correction.
- `re-running signup would wipe profileSlug` → dropped `phone` from the signup
  payload, since `createUserProfile()` has not sent one since 2026-08-20.

A fourth, `re-running signup is rejected outright for a verified account`,
still **passed** — but for the wrong reason: its `assertFails` would now trip on
the phone field before ever reaching the `verified` check it exists to prove.
Removed the field so the assertion still isolates its own subject.

### Changes
- `firestore.rules` — `doesNotAddPhone()` helper; `users` create rejects a
  `phone` key outright, `users` update rejects adding or changing one.
- `tests/rules.test.mjs` — 6 new tests: direct re-add, smuggled beside a
  legitimate edit, at create time, by an admin, plus the two carve-outs
  (deletion still works; an unmigrated account can still edit its profile but
  cannot quietly swap the number it carries).
- `tests/flows.test.mjs` — the four fixture corrections above.

### Tests
256/256. rules 117, flows 50.

### Flagged
Nothing new. Note the standing §6 dependency is unchanged: the gap is only
fully closed in production once `npm run migrate:contacts -- --production`
has been run.

---

## BATCH 5 — Available feed role filter — DONE (with a deliberate deviation)

**Premise verified.** `fetchListings()` (`firestore.ts:842`) fetches the whole
`listings` collection and `available.tsx` filtered only by `postType`. §5 fixes
the Available tab as "`sell-used`, `sell-new`, `rental` — **Manufacturer-posted
only**", and nothing enforced that on the read side. `matching.test.ts` had
zero role coverage, as reported.

### DEVIATION: implemented as a client-side filter, not a query-level one

The batch says *"Add a query-level filter"*. I did not, and the batch's own
next clause is why: it also says to stay *"consistent with how the Needs tab
was already scoped in `matching.ts`"* — and that scoping is client-side and
absent-tolerant. Three concrete reasons a `where('sellerRole','==','manufacturer')`
would have been wrong:

1. **It would retroactively empty the feed.** §3: listings created before the
   role migration are **not backfilled**, so every pre-2026-08-20 listing has
   no `sellerRole` at all. Firestore cannot express "equals X *or* field
   missing" in a `where`, so a query filter silently deletes the entire back
   catalogue from the feed. `matching.ts` documents avoiding exactly this trap
   on the Needs side ("no retroactive emptying"), and §3 says the UI must
   "fall back gracefully when absent".
2. **It contradicts the function's stated design.** `fetchListings()` already
   filters `status` and expiry client-side, with the reason in its own
   docstring: *"Client-side filtering also avoids a composite index on the feed
   query."*
3. **It would have passed the tests and broken production.** §13.4: the
   emulator does not enforce indexes. A new composite index requirement would
   go undetected locally.

So: `isManufacturerPostedSupply` / `filterAvailableFeed` in `matching.ts`, the
exact mirror of `isVendorPostedRequirement` / `filterNeedsFeed`, hiding only an
explicit `'vendor'`. Applied in `available.tsx` after the `postType` pass, in
the same two-filter shape `needs.tsx` uses. **No new field** (uses the existing
denormalized `sellerRole`) and **no ranking change**, as instructed.

Rows this actually removes: supply listings posted by Vendors during the
2026-08-19–20 window when §4's supply gate was not yet a real permission — the
same window, and the same kind of small set, that the Needs-side filter removes.

### Changes
- `app/(auth)/_lib/matching.ts` — `isManufacturerPostedSupply`,
  `filterAvailableFeed`.
- `app/(tabs)/available.tsx` — wired in, mirroring `needs.tsx:112`.
- `tests/matching.test.ts` — 6 role-based tests (the suite had none): the
  filter itself, untagged/null/empty staying visible, a junk value treated as
  untagged, order preservation and input immutability, and a partition check
  that no tagged row can show on both tabs.

### Tests
262/262. matching 15→21. `npx tsc --noEmit` clean.

### Flagged
If the human does want a query-level filter, it needs a **`sellerRole`
backfill migration over existing listings first** (the same shape as
`scripts/migrate-contacts.mjs`), plus a composite index deployed with
`firebase deploy --only firestore`. Listed under NEEDS HUMAN DECISION.

---

## BATCH 6 — `confirm-business.tsx` pre-fill + backfill — DONE

**All three premises verified.** The screen started at
`useState<BusinessCategory | null>(null)` and read nothing; it wrote only
`users/{uid}` via `setCurrentUserCategoryAndRole()`; and the fail-open was in
`useBusinessProfileGate.refresh()` (`catch { setNeedsConfirm(false) }`), since
the screen itself did no read to fail.

### 1. Pre-fill
Reads `getCurrentUserProfile()` on mount and pre-selects the existing category.
Only pre-selects a value still in `BUSINESS_CATEGORIES` — a legacy document may
carry a value from the retired taxonomy (§9), and pre-selecting something absent
from the list would render a form with no visible selection and a working
Continue button. **The confirmation tick is deliberately not pre-armed**: §3
requires explicit confirmation "always", so pre-filling the answer is a
convenience but pre-filling the consent would delete the step §3 mandates.

### 2. Backfill
`setCurrentUserCategoryAndRole()` now re-denormalizes `sellerCategory` /
`sellerRole` onto the business's own listings and `category` / `role` onto its
`profiles` document. Two details worth recording:
- **Backfill runs BEFORE the `users` write.** `users/{uid}` is what the gate
  reads to decide whether the screen still appears, so writing it last means a
  failure part-way through leaves the screen pending and the sequence retried.
  The other order marks the migration done and strands the copies permanently.
  Same reasoning `migrateOwnContactIfNeeded()` gives for its two writes.
- **Best-effort, recorded not thrown.** This screen is mandatory with no way
  out, so a business whose backfill kept failing would be locked out of the app
  entirely. The cost of skipping is a stale denormalized field, which §3
  already requires every reader to tolerate and which both feed filters treat
  as visible. Being stuck is worse than being stale.

It matters most for exactly the population the screen exists for: accounts from
2026-08-19–20 whose listings have no `sellerRole` at all, because the field did
not exist when they posted. This is the first moment that value can be written.

### 3. Fail closed — done at the screen, deliberately NOT at the gate
- **Screen (done):** if the pre-fill read fails, the screen blocks with an
  `ErrorState` + retry and Continue is disabled. This is where the harm the
  batch describes actually lives: without it, a failed read shows a blank form,
  the user picks whatever looks right, and silently overwrites their category —
  and now, its copies on every listing they have posted.
- **Gate (changed, but not to fail-closed):** it no longer *decides* on a
  failed read. It used to `setNeedsConfirm(false)`; it now records the error and
  leaves the previous value. Failing open was never the safe half of that trade
  — under §7's symmetric gates an account with no role can create nothing, so
  waving it through hands the user an app where every post returns a bare
  permission error. But failing **closed** at the root layout would route every
  user with a flaky connection into a mandatory blocking screen, including
  fully-migrated ones. Not deciding does neither: a known-needed confirmation is
  never cleared by a blip, and nobody is yanked in by one.

  **The root-layout hard block is a product call about offline behaviour, not a
  technical one — flagged under NEEDS HUMAN DECISION rather than decided here
  (Run Rule 8).**

### Changes
- `app/(auth)/confirm-business.tsx` — pre-fill, loading state, fail-closed
  retry.
- `app/(auth)/_lib/firestore.ts` — `setCurrentUserCategoryAndRole()` reads
  before writing and skips the backfill when nothing changed; new
  `backfillSellerDenormalization()`.
- `app/(auth)/_lib/useBusinessProfileGate.ts` — no longer decides on a failed
  read.
- `tests/flows.test.mjs` — new group `confirm category & role migration (§3, §7)`,
  3 tests: the full sequence against a no-role account with an untagged listing
  and a stale profile; a corrected category propagating (including that the
  backfill cannot launder a `postType` past Batch 3's lock while it is in there);
  and that it cannot touch another business's listings or profile.

### Tests
265/265. flows 50→53. `npx tsc --noEmit` clean.

### Flagged
- Root-layout hard-block on a failed profile read (see above) — NEEDS HUMAN
  DECISION.
- `updateBusinessDetails()` (the profile editor) backfills listings but **not**
  the `profiles` document — the same gap this batch just fixed on the confirm
  screen. Out of this batch's stated scope (Run Rule 6), so not touched.
  Recorded in `E2E_FINDINGS.md` at Batch 11.

---

## BATCH 7 — Privacy Policy at a real URL — DONE (prepared, NOT deployed)

**Premise verified.** `firebase.json` had no `hosting` block, `app.json` carries
no URL, and `.firebaserc` names only the default project. **No custom domain is
configured anywhere in the repo**, so the URL is the default site,
`wedding2day-a99ea.web.app`, which `firebase hosting:channel:list` confirms
exists (read-only check — nothing was deployed).

### Approach: generate the pages, don't hand-write them
`scripts/render-legal-docs.ts` already generates `docs/*.md` from the same
content modules the in-app screens render, with `npm run test:legal` failing on
drift — built precisely because the 2026-08-19 audit found the hand-maintained
copies had come to contradict the shipped screens. A hosted page is the *third*
copy, and it is the one a Play reviewer reads, so it is generated from those
same modules rather than written again.

Pages, served at clean URLs:
- `/privacy` — `public/privacy.html`
- `/terms` — `public/terms.html`
- `/delete-account` — `public/delete-account.html`
- `/` — a small index, so the hosting root is not a 404 a reviewer hits first

The pages are plain, self-contained HTML with inline CSS: no scripts, no
external fonts, no third-party requests. A privacy policy that pulls a font from
a third party is making a third-party request on behalf of someone reading about
what we do not send to third parties. A test asserts this.

### The account-deletion URL
Play requires one because the app supports in-app deletion (§14 item 2). It is
built from the Privacy Policy's **own section 8 blocks**, extracted
programmatically, plus section 11's contact address rendered as a working
`mailto:` link — not rewritten. §14 item 2 already ties section 8 to
`deleteCurrentUserFirestoreData()` line for line, and a second, subtly different
account of what deletion removes would be worse than no page. Both documented
routes are on it: self-serve (Profile → Delete Account, OTP-verified) and the
written request for anyone who no longer holds the registered number. A test
asserts the page carries every section-8 block verbatim.

### Verified locally, not deployed (Run Rule 7)
Ran the **hosting emulator** and fetched every route: `/`, `/privacy`, `/terms`,
`/delete-account` all return 200 with the right `<title>`, so `cleanUrls` is
doing what the Play listing will depend on.

### >>> COMMAND FOR THE HUMAN TO RUN <<<
```
cd w2d-app && npm run docs:legal && firebase deploy --only hosting
```
Then the Play listing URLs are:
- Privacy Policy: `https://wedding2day-a99ea.web.app/privacy`
- Account deletion: `https://wedding2day-a99ea.web.app/delete-account`

### Changes
- `scripts/render-legal-docs.ts` — `renderLegalHtml()`, `sectionBlocks()`,
  `HOSTED_PAGES`, `INDEX_PAGE`; the writer now emits `public/` too.
- `firebase.json` — `hosting` block: `public`, `cleanUrls`, and
  `Cache-Control` / `X-Content-Type-Options` / `Referrer-Policy` headers.
- `public/*.html` — generated, checked in.
- `tests/legal.test.ts` — 5 new tests: HTML drift, the deletion page carrying
  section 8 verbatim, self-containment, cross-links on every page, and that
  `firebase.json` really does serve clean URLs.

### Tests
270/270. legal 17→22.

### Flagged
Nothing blocking. If a custom domain is ever added, the two URLs above change
and the Play listing needs updating with them.

---

## BATCH 8 — Play Data Safety draft answers — DONE

**Premise verified.** No Data Safety artifact existed in the repo.
`docs/PLAY_CONTENT_RATING.md` covers content rating and target audience only,
and §14 item 2 says outright that the Data Safety form "is still NOT done, and
cannot be done from the repo".

Wrote `PLAY_DATA_SAFETY_ANSWERS.md` (233 lines). Every row is evidenced against
a specific file, per the batch's "base this only on what's actually in the
code". What the evidence gathering actually turned up:

**Declared as collected (8 types):** Name, Phone number, User IDs, Photos, App
interactions, Other user-generated content, Crash logs, Diagnostics, Device or
other IDs. Two real device identifiers, not one: the FCM registration token
(`push.ts`, stored at `users/{uid}/devices/{token}`) and the Firebase Analytics
app instance ID.

**Sharing is NO on every row**, and the file says why rather than asserting it:
`package.json` has exactly seven `@react-native-firebase/*` packages and
nothing else that talks to a network, and a grep confirms there is no `fetch`,
`axios` or `XMLHttpRequest` anywhere in `app/`. No ads, attribution or payment
SDK exists to share with.

### The three findings worth the human's attention

1. **`district` must NOT be declared under Location.** Play's Location types
   mean *device* location. `district` is a value picked from a hardcoded
   38-item list; the app requests no location permission and never reads device
   position. Declaring it would be a false positive that implies a permission
   the app does not have. Filed under App activity → Other user-generated
   content instead, with the reasoning written out.
2. **The phone number is published publicly** for any business with a public
   profile (§6a — `profiles` has a public read rule and `app/p/[slug].tsx` needs
   no auth). Play has no checkbox for "published publicly", so the file flags
   that the listing must not describe this data as internal-only anywhere.
3. **Two things §15 describes are NOT built, and must not be declared:** full
   reviews (no `reviews` collection exists in `firestore.rules`, no screen
   writes one; the ToS mentions them conditionally) and the optional GST
   verification field (no UI writes one, and `profiles` rules actively reject
   `gstNumber`). Over-declaring unbuilt collection is the specific failure mode
   this batch was told to avoid.

Also called out: the category list contains *Iyer*, a priest service. That is a
**business trade category**, not a statement about the person, and must not be
declared as religious belief.

### Tests
No code changed. 270/270 unaffected.

### Flagged
The account-deletion URL must be **live** before the form is submitted — it
only exists after Batch 7's `firebase deploy --only hosting`. Play rejects an
unreachable URL. Carried into NEEDS HUMAN DECISION.

---

## BATCH 9 — w2d-admin role parity — DONE

**`w2d-admin` IS accessible** as a sibling directory (`../w2d-admin`), a
separate git repo at `1b8d889`. Audited and changed there; **committed
separately in that repo** (`e97d5f1`), since it has its own history.

### The three audit questions, answered

**1. Does `seed-admin.mjs` include role fixtures matching §9?** **Yes,
already.** Three fixtures, and the categories are correct against §9:
`Furniture for Wedding` → Manufacturer (row 22), `Catering` → Vendor (row 10),
plus a deliberate `null`-role, `null`-category row standing in for a
pre-migration account. No change needed.

**2. Do the verify scripts assert the new Batch 1-4 rules?** They did not — but
the more useful finding came first: **all 23 existing checks PASS unchanged
against the new rules.** So Batches 1-4 do not break ops. Ran them against the
live emulator serving the new `firestore.rules`, which makes this a real
integration test of the app's rules from the admin client, not a reading.

Then added the missing coverage — 6 new cases in `verify-rules.mjs`
(client SDK, so genuinely rules-checked; `verify-admin.mjs` uses the Admin SDK
and bypasses rules, so it cannot assert them):
- case 14 — role is not self-writable, **and the §3 migration write still is**
- case 15 — an interest id must match its payload
- case 16 — `postType` is immutable, ordinary edits are not
- case 17 — `phone` cannot be re-added to `users/{uid}`
- case 18 — **ops CAN still correct a role (§17)** — see below
- case 18b — ops is still bound by the `postType` and `phone` locks

**3. Does the admin UI have a working role-correction action?** **Yes** —
`src/pages/Users.tsx:224-265`, with a confirm dialog that names §9's implied
role. It **still works under Batch 1**, because that rule exempts the
`isAdmin()` branch. Case 18 now asserts this explicitly and asserts the value
took, because Batch 1 makes this the *only* remaining way to fix a wrong role:
if it ever breaks, a wrong role becomes unfixable without console access.

### Two findings beyond the batch's questions

**A fourth hand-maintained copy of §9's table.**
`w2d-admin/src/lib/categories.ts` has its own `BUSINESS_ROLE_MAP`. Counting
Batch 1's addition, §9's table is now transcribed by hand in four places:
DECISIONS.md, `constants/businessRoles.ts`, `firestore.rules`, and this. **I
diffed all three code copies: all 29 rows agree today**, but nothing enforced
it. Admin's copy drives the Users screen's *role-mismatch filter* — the view
ops uses to find businesses whose stored role disagrees with their category —
so drift there invents mismatches that are not real, or hides real ones. That
is worse than not having the filter: a wrong answer presented as an audit.
Added check `A5` to `verify-admin.mjs`, which skips cleanly when `../w2d-app`
is not checked out beside it (§13.5 — separate clones). **Mutation-tested**:
flipped `Catering` in the admin copy, the check failed naming the row; restored.

**A pre-existing flake in `verify-admin.mjs`, which actually bit during this
run.** `A2.1` asserts ≥1 pending listing, but this same script approves and
rejects pending listings further down (`A4.1`) and never restores one — so a
second run against the same emulator failed, while `A4.1`'s own reset fallback
sat forty lines later and never got the chance to run. Fixed by self-healing in
`A2.1`. This is the same flake shape the file's own comments complain about
twice ("exactly the kind of flake that gets a verification script ignored").
**Verified idempotent over three consecutive runs: 34 PASS each time.**

### Result
`npm run verify` in `w2d-admin`: **29 rules cases + 5 admin checks, all PASS**,
three runs running. `npm run typecheck` clean.

### Flagged
Nothing blocking. The four-copy role table is inherent to the locked stack (no
shared package, and rules cannot import) — the two guards now make drift a
failing check in both repos rather than a silent permission bug.

---

## BATCH 12 — Install Gluestack UI v2 — DONE (build confirmed; boot BLOCKED)

### Installed
- `@gluestack-ui/core@5.0.15`, `@gluestack-ui/utils@5.0.6`,
  `@gluestack-ui/nativewind-utils@1.0.28`
- peers via `npx expo install` so the versions are SDK-54-compatible:
  `react-native-svg@15.12.1`, `react-native-web@^0.21.0`

Gluestack v2 already builds on NativeWind, which is the locked styling layer
(§2), so this is a component library on top of the existing stack rather than a
change to it. Nothing in §10 or §12 rules it out.

**Provider written by hand, not by `npx gluestack-ui init`** — that CLI is
interactive and this run is unattended (Run Rule 1). The file it generates is
what `app/_components/ui/gluestack/GluestackUIProvider.tsx` now contains: a
`View` carrying the theme variables, with `OverlayProvider` and `ToastProvider`
inside it. Mounted once in `app/_layout.tsx`, wrapping the whole `Stack`.

### Theme: W2D's own tokens, and no new colours
`app/_components/ui/gluestack/config.ts` supplies Gluestack's CSS variables
from `tailwind.config.js`'s values. Two things worth recording:

- **Ramp steps repeat, deliberately.** Gluestack expects an 11-step ramp per
  colour. W2D has exactly three reds. Interpolating eight more would be
  inventing brand colours, which the batch explicitly forbids — so each step
  takes the nearest existing token and neighbours share values. The visible
  consequence: Gluestack components will read flatter than the library's demos,
  because W2D's palette genuinely has fewer steps than the library assumes.
  That is the right trade against twelve reds nobody designed.
- **`info` resolves to brand.** W2D has no info colour. Rather than introduce a
  blue, a component reaching for `info` renders brand — visibly wrong at review
  rather than quietly adding a thirteenth hue.
- **Light only.** `tailwind.config.js` closes by explaining that dark mode is
  deliberately not switched on because it "needs to be LOOKED at on a device",
  and this run has no device either. A Gluestack dark ramp would light up dark
  styling on components whose contrast nobody has checked.

### New guard: `npm run test:tokens`
The palette is now written out in **three** places — `tailwind.config.js`,
`colors.ts`, and the Gluestack config. `colors.ts` has carried "KEEP THE TWO IN
STEP" as a comment since 2026-08-21; that is now a test. Five cases, including
one that fails if the Gluestack theme contains **any** hex that is not somewhere
in `tailwind.config.js` — a direct check on the batch's "do not invent new brand
colours". **Mutation-tested** (changed one digit of `brandStrong`; two cases
failed naming the token; restored). Wired into `npm test`.

### "Confirm the app still builds and boots"
- **Builds: CONFIRMED.** `npx expo export --platform android` completes clean —
  a full Metro bundle with the provider mounted, 5.2 MB. `npx tsc --noEmit`
  clean. Full suite 275/275.
- **Boots: BLOCKED, and this is a real gate for Batches 13-16.**
  `@gluestack-ui/core` peer-requires **`react-native-svg`, a native module**, and
  §13.1 is explicit: *"Any new native module requires a fresh EAS dev-client
  build."* Until that build exists, the app will fail to start with "Cannot find
  native module" on any existing dev client. I did not run an EAS build — it is
  a remote, quota-consuming action outside this run's local-only mandate
  (Run Rule 7), and §13.8 records that the last such rebuild was a human step.

### >>> COMMAND FOR THE HUMAN <<<
```
cd w2d-app && eas build --profile development --platform android
```
Install that build before opening the app. Nothing in Batches 13-16 is
observable on a device until it exists.

### Tests
275/275 (8 suites → 9, tokens added).

### Flagged
The EAS dev-client rebuild — carried into NEEDS HUMAN DECISION as a required
action, not a decision.

---

## BATCH 13 — Migrate core UI primitives — DONE (visual QA needs a human)

**The public API did not move.** Every screen still imports `Button`,
`TextField`, `Checkbox`, `Badge`, `Card` from `app/_components/ui` with exactly
the same props. That is the constraint that makes this reviewable without a
device: **every class string is carried over unchanged**, so anything that moved
would show up in the diff.

### What was actually migrated, and to what

| Primitive | Migrated to | Note |
|---|---|---|
| **Button** | Gluestack `createButton` + `tva` | Root is `withStyleContext(Pressable)`, so the button's pressed/disabled state is published to its descendants instead of being re-derived by a lookup map at the call site. Three `Record<ButtonVariant, string>` maps became two `tva` recipes. |
| **TextField** | Gluestack `createInput` + `tva` | The root owns `isDisabled` / `isInvalid`. Border-on-focus behaviour preserved exactly, including the original's documented reason for it (elevation is unreliable across Android versions, a border is not). |
| **Badge** | `tva` | See below. |
| **Card** | `tva` | The pressable variant is now part of the recipe rather than a second `cn()` at the bottom of the component. |

`tva` is Gluestack v2's own styling primitive (`tailwind-variants`), not a
substitute for it — v2's CLI generates its components exactly this way.

**What the recipes buy over parallel lookup maps:** the variants cannot drift
apart. Adding a sixth `ButtonVariant` to `CONTAINER` and forgetting `LABEL` used
to compile fine and render invisible text.

### Two things I did NOT migrate, and why

- **Badge stays a plain `View`.** Gluestack v2's core ships no badge — its CLI
  generates one as precisely what this already is, a `tva`-styled View. There is
  nothing to delegate to, and wrapping a View in a View to be able to report it
  as migrated would add a layout node for no behaviour. It got the `tva`
  treatment, which is the real half of the change.
- **The tab bar stays on Expo Router's `Tabs`.** Gluestack has no tab-bar
  primitive, and the tab bar here *is* React Navigation — which §2 locks
  ("Expo Router"). Replacing it would be a navigation change, not a component
  swap, and it would break the role-gated Needs-tab visibility in
  `(tabs)/_layout.tsx` that §5/§7 depend on. Left alone deliberately.

Recording both plainly rather than quietly claiming five of five.

### Verification
- `npx tsc --noEmit` clean.
- `npx expo export --platform android` clean — 5.3 MB (was 5.2 MB before the
  migration; +0.1 MB is the Gluestack/react-aria runtime now actually reached
  from the component tree).
- Full suite 275/275, including `test:tokens`.

### >>> VISUAL QA IS OUTSTANDING <<<
**No screenshots were taken — there is no screenshot or device tooling in this
environment,** which the batch anticipated ("if not, just do the migration and
note in the log that visual QA needs a human pass"). Type-checking and bundling
prove the tree compiles and links; they prove nothing about pixels. Specifically
unverified, and worth looking at first on a device:
1. button press states across all five variants (the pressed style now comes
   through Gluestack's Root rather than a bare `Pressable`);
2. the text field's focus border, and the `leading` slot (country code on the
   phone screen);
3. multiline fields — `textAlignVertical` is passed through a Gluestack
   component now, and that is the sort of prop a wrapper can swallow.

This also cannot happen until the **EAS dev-client rebuild** from Batch 12 —
`react-native-svg` is a native module (§13.1), so the app will not start on an
existing client no matter how the components look.

---

## BATCH 14 — Migrate high-traffic screens — DONE

**The four named screens were already fully composed from the design system**,
so Batch 13 migrated most of this batch by construction. Audited each one
directly rather than assuming — `available.tsx`, `needs.tsx`, `post.tsx` and
`listing/[id].tsx` between them contain:

- **0** raw `Pressable` / `TouchableOpacity`
- **0** hardcoded hex values
- **0** raw `TextInput`
- **0** hand-rolled duplicates of a primitive's class recipe

That is §14 item 7's revamp having done its job ("Zero hardcoded colour values
remain in `app/`"), and it means every button, field, card and badge on those
screens is now Gluestack-backed through Batch 13's primitives without the screen
files changing at all. Worth stating explicitly, because "no diff" and "not
done" look identical in a commit log.

### What genuinely needed migrating: `ListingCard`
`app/_components/ListingCard.tsx` — **the single most-rendered component in the
app**, since both feeds are lists of it — was hand-rolling a card: a bare
`Pressable` carrying `overflow-hidden rounded-card border border-line
bg-surface`, which is the `Card` recipe, copied. It is not one of the four
screens by name, but it is the substance of two of them, and migrating the
screens without it would have been a migration in name only.

Now uses the `Card` primitive.

**One pixel-level change, stated plainly:** press feedback moves from
`active:opacity-95` to `Card`'s `active:opacity-90`. I chose to accept the 5%
rather than special-case it. `cn()` is a plain join with no tailwind-merge, so
an override would have emitted both classes and left the winner to NativeWind's
specificity — and a 5% divergence in press opacity is exactly the "not a design
decision, copy-paste history" drift `tailwind.config.js` was written to remove.
Noted here so it is a recorded choice rather than a surprise in visual QA.

### Also fixed
`app/listing/[id].tsx` imported `Pressable` and never used it — dead since the
2026-08-21 revamp. Removed.

### Not changed, as instructed
No functionality, layout structure or copy was touched on any of the four
screens.

### Verification
`npx tsc --noEmit` clean, `npx expo export --platform android` clean (5.3 MB),
full suite 275/275. **Visual QA still outstanding** — same as Batch 13, and
still gated on the EAS dev-client rebuild.

---

## BATCH 15 — Migrate remaining screens — DONE

Audited all thirteen: Profile, the six Settings screens, Catalog editor, the
public profile route, and the four auth screens. Same measurement as Batch 14.

**Eleven of thirteen were already fully on the design system** — zero raw
pressables, zero hex, zero raw `TextInput` — so Batch 13's primitives migrated
them without their files changing.

### What genuinely needed migrating
`app/(auth)/confirm-business.tsx` was hand-rolling a **byte-identical copy of
`SearchBar`** — same container classes, same magnifier icon, same clear button.
The comment sitting there explained why it was not a `TextField` (that component
draws its own bordered container; nesting the two would draw two borders) —
which is true, and is exactly why `SearchBar` exists. It just was not reached
for.

Now uses `SearchBar`. Two improvements fall out, neither a visual change:
- the clear button gets a real 28px target instead of a bare icon with
  `hitSlop`;
- the keyboard shows a Search return key here as it already did on the feeds.

`SearchBar` gained one optional prop, `accessibilityLabel`, defaulting to its
current hardcoded `"Search"`. Purely additive. Without it the migration would
have downgraded this screen's label from `"Search categories"` to `"Search"` —
a real screen-reader regression on a **blocking** screen listing 29 rows.

### Four raw controls left alone, each for a reason
Recorded rather than force-fitted, because wrapping these in a primitive that
does not fit would be worse than leaving them:
1. **`otp.tsx` — the six-digit code boxes.** Per-digit refs, key-press handling
   for backspace traversal, `selectTextOnFocus`. `TextField` cannot express any
   of it, and this is the screen where getting input handling wrong locks people
   out of the app.
2. **`catalog/[uid].tsx` — the photo grid cell.** Measured pixel `width`/`height`
   from a computed `CELL` constant; not a `Card`.
3. **`public-profile.tsx` — the photo remove badge.** An absolutely-positioned
   28px circle overlapping its thumbnail's corner.
4. **`public-profile.tsx` — the add-photo tile.** A dashed-border drop target.
   `Button` would give it a filled or outlined pill, which is the opposite of
   what a dashed placeholder communicates.

### Also cleaned
Dead `TextInput` import removed from `confirm-business.tsx` after the swap.

### Not changed, as instructed
No functionality, layout structure or copy touched.

### Verification
`npx tsc --noEmit` clean, `npx expo export --platform android` clean (5.3 MB),
full suite 275/275. **Visual QA outstanding** — Batch 16.

---

## BATCH 16 — Visual QA pass — BLOCKED (with the intent delivered another way)

### Why it is blocked — two concrete, independent prerequisites

Batch 16 asks to "re-run the app". I could not, and it is worth being exact
about why, because the blockers are fixable and the human should know which.

1. **No JDK the build can use.** An Android SDK *is* present, `adb` works, and an
   AVD (`Pixel_10`) exists — so I checked properly rather than assuming. But the
   only installed JDK is **26.0.1**, and React Native 0.81 / AGP needs JDK 17.
   A local `npx expo run:android` cannot succeed. Installing a second JDK is a
   machine-level change nobody asked for, so I did not make one.
2. **The native module gate from Batch 12 is still open.** `react-native-svg`
   arrived with Gluestack, and §13.1 is explicit that a new native module needs
   a fresh EAS dev-client build. Even with a working JDK, the app will not start
   on an existing client.

So there are **no screenshots**, and none of Batches 13-15 has been seen
rendering. Stating that plainly rather than implying a visual pass happened.

### What I did instead: verify visual parity mechanically

"Keep visual output as close to current as possible" is, for a NativeWind
codebase, a claim about class strings — and that is checkable against git.
I extracted every Tailwind class token from every `.tsx` under `app/` at the
run's baseline commit `a50a4f8` and at HEAD, and diffed the multisets.

**Result: exactly one class changes anywhere in the app.**

`active:opacity-95` → `active:opacity-90`, on `ListingCard`'s press feedback —
the single delta already recorded under Batch 14. Every other token that
disappeared from a *file* still renders, because it moved into the shared
primitive that file now composes (`ListingCard`'s card classes into `Card`,
`confirm-business`'s pill classes into `SearchBar`); each was confirmed still
present in `app/` by count. Nothing else was added or removed.

That is a real result: it rules out the whole class of migration bug where a
class is silently dropped in a refactor. It says nothing about how the tree
*renders*, which is the part that still needs eyes.

### Targeted list for the human's device pass
Not "check everything" — these are the specific places where compiling and
bundling clean would not catch a break:

1. **Button press states, all five variants.** The pressed style now arrives
   through Gluestack's `withStyleContext` Root rather than a bare `Pressable`.
   If the Root does not forward `className`, buttons render unstyled — the
   loudest possible failure, and the first thing to look at.
2. **Text field focus border and the `leading` slot** (country code on the phone
   screen). The container is Gluestack's `Input` root now.
3. **Multiline fields** (post description, catalog description).
   `textAlignVertical` is passed through a Gluestack component, and that is
   exactly the kind of prop a wrapper can swallow — text would centre vertically
   instead of starting at the top.
4. **The feeds**, since every row is the migrated `ListingCard`.
5. **`confirm-business.tsx`'s search pill**, now `SearchBar` — check the clear
   button and that the keyboard's Search key behaves.

### Batch 10 re-run, as it asked
Re-ran everything after the UI batches: **app 275/275 across 9 suites**,
**`w2d-admin` 35 checks PASS**, `npx tsc --noEmit` clean,
`npx expo export --platform android` clean. No result moved — expected, since
every area in `TEST_SHEET_RUN.md` is verified through the data and rules layers,
which a component swap cannot reach.

### >>> COMMANDS FOR THE HUMAN <<<
```
brew install openjdk@17          # RN 0.81 needs 17; only 26 is installed
eas build --profile development --platform android
# install that build, then walk the five points above
```

---
---

# FINAL SUMMARY

**Run:** 2026-09-01 overnight, unattended, from a clean tree at `a50a4f8`.
**Result: 16 of 16 batches attempted. 15 completed. 1 blocked (Batch 16).**
No batch was skipped, and nothing was left half-applied.

**Tests: 226 → 275** in `w2d-app` (8 suites → 9), plus `w2d-admin` 23 → 29
rules cases and 4 → 5 admin checks. All green. `npx tsc --noEmit` clean;
`npx expo export --platform android` clean.

**Commits:** 16 in `w2d-app`, 1 in `w2d-admin` (separate repo — `e97d5f1`).

| Batch | Outcome |
|---|---|
| 1 · lock `role` | **DONE** — deviated from the literal instruction; see below |
| 2 · bind `interests` id | DONE |
| 3 · `postType` immutable | DONE |
| 4 · block phone re-add | DONE |
| 5 · Available feed role filter | **DONE** — client-side, not query-level; see below |
| 6 · confirm-business pre-fill/backfill | DONE |
| 7 · host Privacy Policy | DONE — prepared, **not deployed** (Rule 7) |
| 8 · Data Safety answers | DONE |
| 9 · w2d-admin parity | DONE |
| 10 · full test sheet | **DONE against a missing artifact**; see D4 |
| 11 · E2E_FINDINGS.md | DONE |
| 12 · install Gluestack v2 | DONE — builds; **boot needs a dev-client rebuild** |
| 13 · migrate primitives | DONE — visual QA outstanding |
| 14 · high-traffic screens | DONE |
| 15 · remaining screens | DONE |
| 16 · visual QA | **BLOCKED** — no usable JDK, no dev client |

**Two deliberate deviations, both logged where they happened and both because
the literal instruction was not implementable:**
- **Batch 1** asked to block `role` from owner self-update. That would strand
  every pre-role account permanently, because §3's mandatory confirm screen is
  itself an owner-side client write of `role` and there is no server to do it
  instead. Bound the value to `category` instead — an owner can no longer *pick*
  a role, and both shipped write paths keep working.
- **Batch 5** asked for a query-level feed filter. Firestore cannot express
  "equals X or field missing", and §3 does not backfill `sellerRole`, so that
  would have silently emptied the feed of its entire back catalogue — and the
  emulator does not enforce indexes (§13.4), so it would not have shown up
  locally.

**Two defects found that no batch predicted:** three tests that were asserting
the phone hole Batch 4 closed (they passed by asserting the wrong outcome), and
a pre-existing flake in `w2d-admin/scripts/verify-admin.mjs` that bit during
this run. Both fixed. Full defect list in `E2E_FINDINGS.md`.

---

## NEEDS HUMAN DECISION

Genuine product/business calls. Run Rule 8 says an agent does not take these,
so none of them was taken.

**D1 — §6a: the public profile publishes a phone number, reveal-free.**
*(the item Batch 11 was told to flag)*
§6 spent a whole overnight pass bounding phone harvesting behind a per-pair
reveal grant and a daily cap. §6a then publishes the same number on a page that
needs no login. **Both are correct as written** — §6a says "Do not 'harmonize'
with §6 without asking", and §12 records reveal-gating the public profile as
already considered and rejected. The showcase layer does have a bound (a slug is
unguessable, and `profiles` is `get`-open but never `list`-open, which tests and
`verify-rules.mjs` case 9 both assert), it is just a *different* bound from §6's.
The question is how much reach is worth how much exposure. Three options, laid
out in `E2E_FINDINGS.md` D1. **No code was changed for this.**

**D2 — should the Available feed filter become query-level?** Needs a
`sellerRole` backfill migration first, then a composite index deployed. Worth
doing if the feed grows; not worth doing blind.

**D3 — should the app hard-block at launch when the profile read fails?**
The screen now fails closed and the gate no longer fails open. Making the *root
layout* hard-block is a call about offline behaviour that DECISIONS.md does not
cover.

**D4 — `W2D_Test_Plan.xlsx` does not exist.** A search of the entire home
directory to depth 6 found no such file (one unrelated hit, inside a VS Code
extension); it is in neither repo and is referenced nowhere but the overnight
prompt. Most likely it lives on another machine or in cloud storage. `TEST_SHEET_RUN.md`
delivers Batch 10's stated purpose with `TL-` numbering, prefixed so it can never
be mistaken for TC-001–TC-048. **If the real sheet exists, it still needs running.**

**D5 — role is now a pure function of `category`, and `category` is still
user-selectable.** So a business can still move its own role by changing its
category. That is §7's design ("role is determined by category") and §15's soft
verification is the stated backstop — but if the intent was that role never moves
without ops, `category` is where that has to be enforced.

**D6 — visual QA has not happened.** Batches 13-15 have never been seen
rendering. The class-level parity check rules out dropped classes; it says
nothing about pixels.

### Required actions that are not decisions
- `npm run migrate:contacts -- --production` — §6/§14.6 are only closed in
  production once this has run. Not run: Rule 7 forbids production actions.
- `firebase deploy --only firestore` — **the rules changes in Batches 1-4 are
  local only.** None of the four security fixes is live until this runs.
- `firebase deploy --only hosting` — the Privacy Policy and deletion URLs do not
  exist until this runs, and Play rejects an unreachable deletion URL.
- `brew install openjdk@17`, then
  `eas build --profile development --platform android` — nothing from Batches
  12-16 is observable on a device without it.

---

## THE SINGLE NEXT ACTION

**Deploy the security rules.**

```
cd w2d-app && npm test && firebase deploy --only firestore
```

Batches 1-4 fixed four live permission holes, the first of which — `users.role`
being self-writable — made every role gate in the app decorative, since any
account could write itself the other role and walk straight through them. All
four are committed, tested (275 app tests, 29 admin rules cases) and **inert
until this command runs.** Everything else on this list can wait a day; this
cannot.

Read `E2E_FINDINGS.md` next — it is the whole defect list in one file — and
answer D1, which is the only finding deliberately left unfixed.

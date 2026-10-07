# OVERNIGHT_NOTES — unattended run, night of 2026-08-19 → 2026-08-20

Append-only log. Each task gets: what was built, decisions made without
asking (with reasoning), and the testing pass results.

Context read in full before starting: `DECISIONS.md` (744 lines),
`PRODUCT_CONTEXT.md` (480 lines), plus the existing app source.

Tasks in scope tonight: 9, 10, 11, 12, 13, 14, and research-only 15 and 16.
Task 8 was already done (commit `450a702`). **Not** in scope by explicit
instruction: §14 item 7 (full UI/visual revamp) and §14 item 8 (admin app).

---

## Session setup decisions (before Task 9)

**Decision S1 — added `@firebase/rules-unit-testing` as a devDependency.**
There was no automated security-rules test in the repo (commit `153a6ab`
says "add rule tests" but its diff contains none). Every task tonight
touches `firestore.rules`, and §6a/§7 both say explicitly that a rules
change must be *verified in the rules file*, not assumed from the doc.
Reasoning: a dev-only dependency is the cheapest way to make that
verification real and repeatable; it does not ship in the app bundle and
does not touch the locked stack (§2, which governs runtime choices).
Tests live in `tests/` and run with `npm run test:rules` against the
already-required local emulator (§13.3).

**Decision S2 — emulator host for tests is `127.0.0.1`, not `192.168.31.16`.**
§13.3 pins `192.168.31.16` because a *physical device* needs the Mac's LAN
IP. Node tests run on the Mac itself. `firebase.json` already binds the
emulators to `0.0.0.0`, so both hosts reach the same process. No app code
changed; `app/_layout.tsx` still uses the LAN IP for device testing.

---

## TASK 9 — Profile / Settings screen (§14 item 1)

### What I found first

Most of §14 item 1 already existed from commit `e61c186`: notification
preferences, the phone-change screen, My Interests, and sign-out. What was
missing or broken was more interesting than what was absent, so the bulk of
this task is corrections plus the two genuinely-missing pieces (catalog
editing, business-details editing).

### Built / changed

**1. Fixed all five remaining `.exists` property misuses** in
`app/(auth)/_lib/firestore.ts`. In `@react-native-firebase` v25 `exists` is a
**method**; `!doc.exists` negates a function reference, which is always
`false`, so the "document missing" branch never runs. Sites and real impact:

| Line | Function | Impact of the bug |
|---|---|---|
| ~110 | `getCurrentUserProfile` | returned a hollow profile (all fields `''`) instead of `null` for a missing user doc |
| ~434 | `getListing` | returned a hollow listing for a bad id — the detail screen's "Listing not found" state could never render |
| ~470 | `expressInterest` | **`interests` docs were never written and `interestCount` never incremented** |
| ~493 | `getSellerContact` | a deleted seller yielded a contact card with a blank name and blank phone instead of "unavailable" |
| ~731 | `getCatalogItem` | hollow item instead of null |

The `expressInterest` one is the severe one: every §6 feature downstream of
the `interests` collection — My Interests, the requirement responders list,
seller-facing interest counts — has been reading a permanently empty
collection. Tasks 12, 13 and 14 all depend on that collection, so this was a
hard prerequisite, not a tidy-up.

**Decision D9.1 — I fixed all five rather than raising them first.** My own
prior session note on this said to ask before touching them, because fixing
line 110 flips `getCurrentUserProfile()` from "hollow object" to `null` and
auth-path screens might lean on the accidental non-null behaviour. I traced
every caller instead: `(tabs)/_layout`, `(tabs)/post`, `(tabs)/needs`,
`(auth)/otp`, `post/requirement`, `catalog/[uid]`, `settings/public-profile`,
`useCategoryPrompt` and `useSuspendedLockout` all already use `profile?.x` or
an explicit `if (!profile)`. `(tabs)/profile` shows a retry state instead of a
blank card, which is strictly better. `useCategoryPrompt` becomes *more*
correct: a half-signed-up user with no doc now reads as `null` rather than as
a legacy account needing migration. Reasoning: leaving a known
data-corruption bug in place for another day was worse than the migration
risk, and the risk turned out to be nil. Added an `EXISTS-IS-A-METHOD` block
comment at the top of the file so it does not come back — `tsc` only catches
the truthy form (TS2774), never the negated one.

**2. Account deletion now actually deletes everything.** It previously
removed `listings` + `catalogItems` + `users/{uid}` and stopped there.

- **Added: the public `profiles/{slug}` document.** This is the important
  one. That document is WORLD-READABLE and holds a phone number (§6a), so
  the old flow kept publishing the contact details of an account that no
  longer existed. That is the exact opposite of what deleting an account
  means and would not survive a Play Store data-deletion review (§14.2).
- **Added: Storage objects.** Deleted by download URL (`refFromURL`) rather
  than by listing a folder, because `listAll()` needs read permission on the
  path prefix and every image is already recorded as a URL on the document
  being deleted. Best-effort: an orphaned image is a cost problem, a throw
  would abort a deletion the user is entitled to complete.
- URLs are collected **before** the documents that reference them are
  deleted; `users/{uid}` is deleted **last** because the rules'
  `isNotSuspended()` helper reads it.

**Decision D9.2 — `interests`, `reports` and `blocks` are deliberately NOT
deleted.** `interests` are immutable by rule and by design (§6), and
`listings.interestCount` on *other* businesses' posts is derived from them —
deleting them would corrupt other people's data. `reports` are admin-owned
moderation records (§15). `blocks` naming this user are the other party's
setting. All three hold only a uid that now resolves to nothing, no personal
data. Documented in the function's doc comment so the next reader does not
"fix" it. Flagged for the Data Safety form in Task 16.

**3. `storage.rules`: split `allow write` into `create, update` + `delete`.**
On a delete `request.resource` is null, so a combined rule that reads
`request.resource.size` throws and denies. **I verified this against the old
rules text before changing anything** — replaying a delete under the previous
rules returns `storage/unauthorized`. So account deletion could never have
removed a single image, including the public portfolio photos. Size and
content-type limits are unchanged for uploads.

**4. `firestore.rules`: `delete` is no longer suspension-gated** for
`listings` and `users/{uid}/catalogItems`. `create`/`update` still are.

**Decision D9.3 — reasoning:** §14.2 makes in-app account deletion a Play
Store release gate, and a suspended user is exactly the person most likely to
want out. Under the old rules a suspended account could not delete its own
content, so the deletion flow would half-fail. Removing your own posts is
never the abuse that suspension guards against.

**5. Catalog editor (§4).** `deleteCatalogItem` existed but nothing called
it, and there was no edit path at all. Added `updateCatalogItem` and wired
Edit + Delete into the item modal on `app/catalog/[uid].tsx` (owner only).

**Decision D9.4 — edit reuses `app/post/catalog.tsx` via `?id=<itemId>`
rather than a second screen.** A catalog item has exactly one shape (no
status, quantity or expiry per §4), so a parallel edit screen would be the
same fields twice and would drift. In edit mode the form uploads only local
URIs and passes through existing `https://` URLs, so re-saving does not
duplicate Storage objects.

**6. New `app/settings/business-details.tsx`.** `name`, `businessName`,
`district` and `category` were write-once at signup with no way to fix a
typo, and `category` could only ever be set by the one-time migration picker.

**Decision D9.5 — editing business details backfills the denormalized copies
on that business's own listings** (`sellerName`, `sellerBusinessName`,
`sellerCategory`). §3 denormalizes these so feed cards render without a
per-card user lookup; without a backfill, renaming a business would leave
every card it ever posted showing the old name and old category badge. §16's
Tier 1 matching also reads `category`, so a stale copy would mis-sort the
Needs tab. Batched and chunked at 500. Phone is deliberately excluded — it
needs OTP re-verification and has its own screen (§8).

**Decision D9.6 — a phone change now also rewrites `profiles/{slug}.phone`.**
The public profile is the one number a couple outside the app can see, and it
is a denormalized copy. Leaving it stale hands real customers a dead number,
which defeats the whole reason §6a shows contact details directly instead of
reveal-gating them. `whatsapp` is deliberately NOT touched — it is a separate
number the business entered on purpose, not a mirror of the login number. The
profile write is best-effort: the Auth number has already changed by then, so
failing the whole call would leave `users/{uid}.phone` behind the Auth record,
a worse inconsistency than a stale public copy the owner can fix in the editor.

**7. Profile screen moved off `userType`.** It showed a "Vendor" /
"Manufacturer" badge and a "Type" row — a field §7 deleted. Now shows the
29-item `category`, with `"Category not set"` when absent (§3 forbids
inferring one from `userType`). Also added the §15 soft-verification badge
(admin-toggled, hidden when unset), reordered the settings rows so
destructive Delete Account sits last instead of third, and switched the
screen to `useFocusEffect` so edits made on sub-screens show on return.

**8. Copy fix** in `settings/notifications.tsx`: "your district or
categories" → "your district and category" (the plural `categories[]` field
is gone — §7).

### Testing pass

Build/verify: `npx tsc --noEmit` clean; `npx expo export --platform android`
bundles successfully (4.72 MB, no unresolved imports); all three suites green.

New test infrastructure, run against the live emulator:
- `npm run test:rules` — 33 tests
- `npm run test:storage` — 5 tests
- `npm run test:flows` — 15 tests (multi-step client sequences replayed
  against the real rules — the failure mode where every individual permission
  is fine but the sequence still breaks)

Edge cases, empty states and error paths exercised — all pass:

| Case | Result |
|---|---|
| Delete a brand-new account with zero listings/catalog/profile | passes (no empty-batch crash) |
| Delete an account whose `profileSlug` pointer dangles (create failed partway) | passes, does not block deletion |
| Delete while **suspended** | passes (this is what D9.3 fixed) |
| Suspended account tries to CREATE listing/catalog | still denied |
| Deletion attempts to touch another business's listing / user doc | denied |
| Deletion attempts to delete another user's `interests` record | denied (correct — §6) |
| Public profile readable by anon *after* account deletion | gone |
| Partial `profiles` update (phone only) vs. the `hasAll()` field allowlist | passes — rules evaluate the merged doc, so untouched required fields still satisfy it |
| Phone change smuggling a non-allowlisted field (`gstNumber`) onto the public profile | denied |
| Clearing `whatsapp` with `deleteField()` | allowed |
| Phone change when the user has no public profile | no-op, not an error |
| Business-details edit with zero listings (empty backfill) | passes |
| Backfill reaching another business's listings | denied |
| Owner edit trying to set `verified` / `status` | denied |
| Editing another business's catalog item | denied |
| Suspended: catalog edit denied, catalog delete allowed | as designed |
| Storage: oversized (>5 MB) and non-image (`application/pdf`) uploads | denied |
| Storage: another user uploading/deleting your photos | denied |
| Storage: anon reading `profile-photos/` vs `listing-photos/`/`catalog-photos/` | public / denied respectively (§6a) |
| Storage delete under the **old** rules text | `storage/unauthorized` — confirms the bug was real, not theoretical |

Not covered by automated tests (no RN runtime in node): the React screens
themselves. Typecheck plus a successful Metro bundle is the available
verification there.
---

## TASK 10 — Terms of Service + in-app account deletion (§14 item 2, RELEASE GATE)

Privacy Policy content and the Data Safety form were **not** touched, per the
explicit instruction and §14.2's own note that they are parked separately.
Their current state is audited in Task 16 instead.

### Built / changed

**1. Terms of Service rewritten** (`app/legal/terms.tsx`). The previous text
described a product that no longer exists: "wedding decoration industry",
"manufacturers, decorators, and suppliers" — i.e. decoration-only with the
Vendor/Manufacturer role split §7 deleted. More seriously, it never mentioned
the public profile at all.

**Decision D10.1 — the public profile gets its own prominent ToS section
(section 5), written in plain language.** §6a publishes a business's phone
number, business name, category, district and portfolio photos on a page
anyone can open with no account. Terms that never say so are not disclosure,
and this is the single most consequential thing the app does with user data.
The section states what becomes public, that it is optional, that the link can
be forwarded onward and we cannot control that, that the page is not listed or
searchable but *is* public, and that catalog and pricing are never on it. This
is also the disclosure the Data Safety form will have to match (Task 16).

New/rewritten sections also cover: the 29 categories (using real names from
§9 — I did not invent any), the two-layer product shape, that changing your
phone number is not account loss (§8), the reversed requirement-response
direction (§6), rentals being label-only with no booking (§4), the daily
reveal cap and a no-scraping clause (§6/§15 — the cap itself is Task 14), a
content-licence paragraph, and a deletion section that mirrors
`deleteCurrentUserFirestoreData()` exactly, including what is retained and
why. Added a doc comment tying section 11 to that function so the two do not
drift.

**2. New `app/settings/delete-account.tsx`** — a real deletion flow rather
than a one-tap alert.

**Decision D10.2 — the order is re-authenticate → delete data → delete Auth,
which is the reverse of what the old code did, and this was a live data-loss
bug.** Firebase rejects `user.delete()` with `auth/requires-recent-login` once
sign-in is more than a few minutes old — which it always is by the time
someone walks to Settings. The old `runDeleteAccount` deleted all Firestore
data **first** and only then hit that rejection, then told the user "your
account is still active". So the realistic path was: listings, catalog and
public profile destroyed, account still live, nothing recoverable. Re-auth
first means the irreversible part only starts once the delete is certain to
be accepted.

**Decision D10.3 — always re-authenticate, rather than trying the delete
first and re-authenticating only on failure.** The try-first approach cannot
work here without the data-loss ordering problem above (you would have to
attempt the Auth delete before touching data, and a successful Auth delete
leaves no credentials to delete the data with). Always re-verifying also
matches Google's own guidance for destructive account actions. Cost is one
SMS per deletion, which is rare and well inside the free tier.

The screen also spells out what is deleted and what is retained, in the same
terms as the ToS, and offers an email-support fallback for the case where the
registered number can no longer receive SMS — Play expects a route to
deletion even when the in-app one cannot complete.

**3. Suspended accounts can now delete themselves.** `useSuspendedLockout`
pinned a suspended user to `/(auth)/suspended` with no way to reach Settings,
so in-app deletion did not exist for them at all. Added a "Delete my account"
action on the suspended screen and exempted the `delete-account` segment from
the lockout redirect. This pairs with the rules change in Task 9 (D9.3).

**4. Fixed returning-user routing after OTP** (`app/(auth)/otp.tsx`). This
turned out to be a real bug, not just a Task 10 convenience.

`otp.tsx` sent **every** successful sign-in to `/(auth)/profile-setup`, which
calls `createUserProfile()` → `.set()` — a whole-document replace. For an
existing account that means:
  - `profileSlug` is dropped, orphaning the §6a public profile: the editor
    falls back to create-mode, and the rules then reject the create because
    the pointer no longer matches. The old public page keeps serving the
    user's phone number with nothing in the app pointing at it.
  - `notificationPrefs` is dropped.
  - For a **verified** account the write is rejected outright (owners may not
    affect `verified`), so signup dies on a bare error message.

Both are pinned by new tests. A returning user now goes straight to the tabs.
This check only became possible after Task 9 made `getCurrentUserProfile()`
return `null` for a missing document instead of a hollow object — before that,
a returning user and a brand-new one were indistinguishable.

**5. Profile screen**: Delete Account now routes to the new screen (styled
destructively, last in the list) instead of running inline; added "Terms of
Service" and "Privacy Policy" rows. Play reviewers look for both in-app, and
previously they were only reachable from the pre-signup phone screen.

### Testing pass

Build/verify: `tsc --noEmit` clean; `expo export --platform android` bundles;
rules 33 ✓, storage 5 ✓, flows 18 ✓.

Edge cases and error paths exercised:

| Case | Result |
|---|---|
| Suspended account runs the FULL deletion batch (listings + catalog + profile + user doc) | passes — this is what D9.3 + the lockout exemption unlock |
| Re-running signup over an existing account | confirmed to drop `profileSlug` and orphan the public profile (test pins the damage the routing fix avoids) |
| Re-running signup over a **verified** account | rejected by rules, as expected |
| Deletion with a dangling `profileSlug`, and with an empty account | both pass (from Task 9) |
| OTP entered with fewer than 6 digits | blocked client-side before any Auth call |
| Wrong OTP during re-auth | caught, mapped through `getAuthErrorMessage`, step stays on OTP, **no data deleted** — the ordering guarantee |
| Account with no `phoneNumber` on the Auth record | `sendReauthOtp` rejects with a support-contact message rather than throwing raw |
| Cancel from the OTP step | clears the pending verification id, returns to the confirm step |

Not verifiable without a device/emulator build: the OTP round-trip itself and
the suspended-screen navigation. Both are thin wrappers over the same
`verifyPhoneNumber` listener pattern already used by the working
change-phone screen.

### Flagged, not fixed (deliberately out of Task 10 scope)

- `app/legal/privacy.tsx` is **stale in the same way the old ToS was** —
  "wedding decoration", "Business type (Vendor or Manufacturer)", no mention
  of the public profile. It is parked per instruction; it must be rewritten
  before submission. Detailed in Task 16.
- `SUPPORT_WHATSAPP_NUMBER` and the ToS contact address are still a personal
  gmail/number with a `TODO` in `support.ts`. Also in Task 16.
---

## TASK 11 — Push notifications infrastructure / FCM (§14 item 3)

### What Blaze blocks, precisely

Asked for explicitly, so stated precisely. The billing bug (`OR_BACR2_44`,
§8/§13) blocks the **Blaze plan**, and therefore **Cloud Functions**. What
that does and does not cost us:

| Piece | Needs Blaze? | Status tonight |
|---|---|---|
| `@react-native-firebase/messaging` client SDK | No | **Built** |
| Notification permission prompt (Android 13 POST_NOTIFICATIONS) | No | **Built** |
| Getting an FCM device token and storing it | No | **Built** |
| Handling a received message / notification tap | No | **Built** |
| In-app notification centre (§15) | No — plain Firestore | **Built** |
| **Actually SENDING a push** | **Yes** (needs a server) | **BLOCKED** |
| Post-approval status alert on a Firestore `status` trigger (§15) | **Yes** | **BLOCKED** — rules + data model ready, no writer |
| Tier 2 push-on-match Cloud Function (§16) | **Yes** | **BLOCKED**, and explicitly out of scope tonight anyway |

So: a device can now be fully prepared to receive pushes, and nothing exists
that can send one. The FCM *protocol* is free on Spark — what is missing is a
trusted server to call it from, since a send needs credentials that cannot
ship in the app. §8 already establishes the escape hatch (a Cloudflare Worker
on the free tier, as planned for WhatsApp OTP); whichever sender arrives, it
reads the `users/{uid}/devices` tokens written here and needs no client
change.

**Decision D11.1 — I built the in-app notification centre as a real,
working, server-free feature rather than a stub waiting on Blaze.** §15
describes it as "fallback for denied permissions or missed pushes", but with
no sender it is currently the *only* delivery path, which makes it the most
valuable buildable half of item 3. The design that makes this work without a
server: **the client that causes an event writes the notification document**,
and the recipient reads it. No trigger, no function, no Blaze.

**Decision D11.2 — one account writing into another account's inbox is a spam
channel unless the rules stop it, so they do.** Two constraints carry it:
1. The actor must already have an `interests/{listingId}_{actorUid}` document
   — a notification can only ever follow a real, recorded interaction. This
   also fixes the write ordering: `listing/[id].tsx` must notify *after*
   `expressInterest()` resolves, never before.
2. No displayed string is free text. `listingTitle` must equal the listing's
   real title, and `actorBusinessName` must equal the actor's own live
   `users/{uid}.businessName`. So there is no field to inject
   "WIN A FREE IPHONE" or "Wedding2day Official Support" through. Both are
   tested.
The recipient may only ever flip `read`; `create` is rejected if it arrives
pre-read; suspended accounts cannot send; nobody can read an inbox but its
owner, and an unscoped `list` is denied.

**Decision D11.3 — `@react-native-firebase/messaging` is loaded through a
guarded require, not a top-level import.** §13.1 is explicit that a new native
module needs a fresh EAS dev-client build. A plain import would hard-crash
every existing dev client from the moment this lands until that rebuild
happens. Instead `getMessaging()` requires the package *and probes it* — RNFB
only throws when the module factory is invoked, not on require, so checking
the require alone would report push as available on exactly the builds where
it is not. Without the native module every function is a no-op,
`isPushAvailable()` returns false, and the notification centre (plain
Firestore) keeps working. The centre shows an honest banner in that state
rather than implying pushes work.

> ⚠️ **ACTION FOR YOU:** `@react-native-firebase/messaging` and its config
> plugin are now in `package.json` / `app.json`. **A fresh EAS dev-client
> build is required** before push does anything (§13.1). Nothing breaks
> before then — it degrades to the in-app centre.

**Decision D11.4 — added a third notification preference, `newInterest`.**
The existing two (`newMatches`, `postApproved`) did not cover "someone
responded to your post", which is the only event the app can actually
generate today. Defaults to true, absent-safe on old documents.

**Decision D11.5 — permission is requested on first sign-in, not at app
launch.** Asking a stranger for notification permission before they have an
account is the standard way to get it denied permanently, and Android 13 gives
you one prompt. The token is registered regardless of the answer, since a user
may enable notifications in OS settings later without passing back through
that code path.

**Decision D11.6 — added `firestore.indexes.json` and wired it into
`firebase.json`.** The two inbox queries (`recipientId` + `orderBy createdAt`,
and `recipientId` + `read`) need composite indexes. The emulator does not
enforce indexes, so this would have passed every local test and then failed in
production on first use. Verified with `firebase deploy --only firestore
--dry-run`.

### Also built

- `users/{uid}/devices/{token}` for FCM tokens — a subcollection, not an array
  on `users/{uid}`, because that document is readable by every signed-in user
  and device tokens do not belong in it. The token *is* the document id, which
  makes re-registration idempotent (tested: registering twice leaves one row,
  so a future sender cannot double-push one phone).
- Token dropped on sign-out from both the Profile and suspended screens, so
  the next person to use the phone does not inherit the previous account's
  notifications.
- Account deletion now also removes `users/{uid}/devices` and the user's
  inbox.
- Bell + unread badge in both feed headers, a "Notifications" row in Profile,
  notification-tap deep routing to the listing, and a foreground-message
  handler (Android draws no system notification while the app is foregrounded,
  so without it a message arriving mid-use would be invisible).
- `expressInterest()` now returns whether it actually created the record, so
  reopening a listing you already responded to does not re-notify the owner.

### Testing pass

Build/verify: `tsc --noEmit` clean; `expo export --platform android` bundles
(confirms the guarded require does not break Metro); rules 45 ✓, storage 5 ✓,
flows 26 ✓. Rules also validated with `firebase deploy --dry-run`.

Edge cases, empty states and error paths — all pass:

| Case | Result |
|---|---|
| Notification written *before* the interest record | denied — pins the required call order |
| Notification with no interest record at all | denied (the core anti-spam gate) |
| Notification addressed to someone other than the listing owner | denied |
| Injected `listingTitle` ("WIN A FREE IPHONE — CALL NOW") | denied |
| Injected `actorBusinessName` ("Wedding2day Official Support") | denied |
| A business that renamed itself reusing its OLD name | denied — the rule reads the live `users` doc |
| Spoofed `actorId`; pre-set `read: true`; extra field (`deepLink`) | all denied |
| Suspended business sending a notification | denied |
| Actor trying to read the notification back | denied (recipient-only) |
| Anonymous read of a notification | denied |
| Unscoped `list` of the collection vs. the recipient-scoped query | denied / allowed |
| Recipient editing anything but `read` | denied |
| Mark-all-read batch reaching another user's rows | denied |
| Empty inbox | reads cleanly, empty state renders |
| Registering the same device token twice | one row, not two |
| Another business reading/writing your device tokens (incl. an admin) | denied |
| Admin `post-status` notification with no interest record | allowed (§15) |
| Admin trying an `interest`-type notification without one | denied |
| Account deletion removing tokens + inbox | passes |
| Notification write failing (offline) | swallowed — the reveal the user asked for still completes (§6) |
| Build without the messaging native module | every push call no-ops, centre still works, banner shown |

### Known gap, by design

Post-approval alerts (§15) have their data model and rules in place but **no
writer**: only an admin can change `status`, and the admin app is §14 item 8,
explicitly out of scope tonight. The rule already permits
`isAdmin() && type == 'post-status'`, so wiring it later is a single write in
`w2d-admin`'s approve/reject handler — no schema or rules change needed.
---

## TASK 12 — Matching engine, TIER 1 ONLY (§14 item 4, §16)

**Tier 2 (push-on-match Cloud Function) was NOT built** and no groundwork for
it was laid. §16 gates it on the Blaze billing bug clearing (§8, §13), and
tonight's brief repeats that.

### The prerequisite nobody had done yet

Tier 1 compares a requirement's `category` against the viewing business's own
`category`. That comparison **could never have been true**: `users.category`
came from the 29-item list (§9), while every category picker in the app —
`PostForm`, the catalog form, and both feed filters — still wrote and filtered
on the retired 10-item decor list. The two lists share **zero values**.

So Tier 1 was not a sort tweak on top of working data; the taxonomy migration
was a hard prerequisite, and I did it as part of this task.

**Decision D12.1 — migrate every picker to `BUSINESS_CATEGORIES` and delete
the `LISTING_CATEGORIES` constant outright.** Keeping it would have left a
loaded gun: any picker still pointing at it silently writes values that can
never match anything. The `LegacyListingCategory` *type* stays, because
existing documents carry those values and must keep type-checking on read.
`ListingInput.category` and `CatalogItemInput.category` were narrowed to
`BusinessCategory` (writes), while `ListingDoc.category` stays wide (reads) —
so the compiler now enforces that nothing writes a legacy value again. That
narrowing is what surfaced every remaining call site.

**Decision D12.2 — legacy-category listings degrade, they are not remapped.**
An old listing still displays, still opens, still reveals a phone; it just
never matches a filter or a Tier-1 band. §3 explicitly forbids guessing a
mapping from the old taxonomy to the new one, and a wrong auto-mapping would
be worse than no match. Pinned by a test.

**Decision D12.3 — a pre-migration catalog item clears its category on edit.**
The 29-item picker cannot display a legacy value, so showing it would mean
rendering an option that is not in the list. The field is cleared instead,
which forces a real choice on save rather than an inferred one.

### The role gates §7 removed but the code still enforced

§7 has been "locked" since 2026-08-19, but the app was still enforcing the
dropped model in six places. All removed:

| Where | Gate |
|---|---|
| `(tabs)/_layout.tsx` | Needs tab hidden unless `userType === 'manufacturer'` |
| `(tabs)/needs.tsx` | redirected non-Manufacturers away from the tab |
| `(tabs)/post.tsx` | `intentOptionsForRole` withheld `requirement` from Manufacturers |
| `post/requirement.tsx` | redirected anyone who was not a Vendor |
| `constants/postTypes.ts` | `intentOptionsForRole` / `postTypesForRole` |
| `(auth)/profile-setup.tsx` | "I am a Vendor / Manufacturer" selector + the Manufacturer-only `categories[]` multi-select |

**Decision D12.4 — signup stops writing `userType` and `categories[]`
entirely, and `createUserProfile` now requires `category`.** §3's `users`
schema lists no `userType`, and leaving signup writing a field the product
deleted would keep manufacturing pre-migration accounts forever. `UserProfile.userType`
survives as an optional deprecated field so pre-migration documents still read,
and `getCurrentUserProfile` no longer defaults it to `'vendor'` — a document
written after the split genuinely has no role, and defaulting one in would
resurrect exactly the model §7 removed. `listings.sellerUserType` likewise
stops being written; `sellerCategory` is the badge, and it is what Tier 1
matches on.

**Checked before doing this:** `w2d-admin` reads `userType` only at
`src/pages/Users.tsx:204`, guarded as `{u.userType && (…)}`, so a new account
without one simply shows no role badge. No regression there, and no admin
changes were needed (details in Task 15).

### Tier 1 itself

Extracted into `app/(auth)/_lib/matching.ts` — pure functions, no React or
Firebase imports — so the ranking is unit-testable directly instead of only
through the screen.

**Decision D12.5 — category outranks district.** §16 says "sort by district
and category" without ordering them. Four bands: both (3) > category (2) >
district (1) > neither (0). Reasoning: a requirement's `category` is what the
poster *needs*, so a viewer whose own category matches is someone who can
actually supply it — the harder constraint. District decides whether the deal
is practical, which matters, but a nearby requirement you cannot fulfil at all
is worth less than a distant one you can. The previous implementation returned
0 for any district mismatch, so a perfect category match in the next district
ranked below a useless one next door.

**Decision D12.6 — the ranking is visible, not invisible.** §16 says "matches
surface first", which on its own is a silent reorder nobody notices. Added a
green badge per card ("Your category · near you" / "Your category" / "Near
you") and a "Matches you (n)" filter chip. The chip hides itself when the
viewer has neither signal, since it could only ever produce an empty list.
Manual filter chips narrow the set *first*, then ranking runs on what is left —
the other order would let a category chip hide the matches the ranking exists
to surface. The chip's count is over the whole feed, not the filtered set, so
it advertises how many matches exist rather than how many survive the chips.

**Decision D12.7 — the ranking uses plain `string` category comparison**, not
the `BusinessCategory` union. Ranking must also be able to *look at* legacy
documents; narrowing the type would make them unrankable rather than simply
unmatched.

### Seed data

`scripts/seed.mjs` no longer writes `userType`, `categories[]` or
`sellerUserType`, and listings now draw from the same 29-item list as users —
seeding two disjoint taxonomies would have made every match test silently pass
by never matching.

**Decision D12.8 — the seed's category list has SEVEN entries, and that is
load-bearing.** Post types cycle with period 4. With the 8-item list I first
wrote, every `requirement` landed on `idx % 8 ∈ {3,7}` — two categories,
neither belonging to a seeded user, all six requirements owned by just two
accounts, and **zero** Tier-1 matches for any viewer. I only caught this by
querying the emulator after seeding. 7 is coprime with 4, districts stride by
3, and sellers use `% 7`, so everything walks its whole list.

Verified against the emulator after re-seeding:

```
users still carrying userType:            0  (want 0)
users still carrying categories[]:        0  (want 0)
listings still carrying sellerUserType:   0  (want 0)
listings with a non-29-item category:     0  (want 0)
distinct requirement posters:             6  (was 2)
users with NO category (fixture):         4  (want 4)
viewer seed-user-2 (Audio and Lighting / Coimbatore):
  requirement bands → both:0 category:1 district:3 none:2   (was 0/0/0/6)
```

### Testing pass

Build/verify: `tsc --noEmit` clean; `expo export --platform android` bundles;
matching 15 ✓, rules 45 ✓, storage 5 ✓, flows 26 ✓ (91 total). New
`npm run test:matching` runs the shipped ranking module directly via
sucrase-node.

Edge cases, empty states and boundaries — all pass:

| Case | Result |
|---|---|
| Both signals / category only / district only / neither | 3 / 2 / 1 / 0, and category **is** verified to outrank district |
| Viewer with no `category` (pre-migration account, §3) | district-only ranking, no guessed category |
| Viewer with an empty district vs. a listing with an empty district | does **not** match — guards the `'' === ''` bug |
| Viewer with no signals at all | everything ranks 0; "Matches you" chip is hidden |
| Legacy-taxonomy listing vs. a 29-item viewer category | never a category match; district still counts |
| Stability inside a rank band | newest-first order preserved |
| Full four-band ordering | `both, category, district, none` |
| `matchesOnly` with nothing matching | empty list, no crash; distinct empty-state copy |
| Empty feed | ranks to empty, count 0 |
| `countMatches` across mixed bands | counts every non-zero band |
| Labels per band | present for 3/2/1, `undefined` for 0 (no meaningless badge) |
| Input array mutation | none — `rankListings` does not reorder its argument |
| Requirement create by a non-Vendor category | allowed (rules suite, from Task 9) |
| Feed load failure | error + Retry, unchanged |

Not verifiable in node: the screens. Typecheck plus a clean Metro bundle, plus
the emulator seed check above, is the available verification.
---

## TASK 13 — My Listings + expiry + free-text search (§14 item 5)

### What already existed vs. what was missing

Two of this item's six pieces already existed but had **never actually
worked**, because of the `expressInterest` `.exists` bug fixed in Task 9: no
`interests` document was ever written, so **My Interests** was permanently
empty and the **requirement responders list** always showed "No responses
yet". Both are now live for the first time — no new code needed for either,
only the Task 9 fix. I added tests pinning them rather than rebuilding them.

The other four were genuinely absent: My Listings screen, the
sold/unavailable toggle, listing expiry, free-text search.

### Listing expiry

**Decision D13.1 — expiry is a stored `expiresAt` timestamp plus a
client-side filter. Nothing is ever deleted.** A scheduled cleanup would be a
Cloud Function → Blaze → blocked (§8, §13). Rather than skip the feature, the
filter runs where the feed is already filtered client-side. Expired posts drop
out of Available/Needs and stay in My Listings, exactly the behaviour §15
already defines for sold/unavailable — so it reuses a pattern the product has
already decided on, and no work is destroyed.

**Decision D13.2 — the windows, which DECISIONS.md never fixed.**

| Post type | Window | Why |
|---|---|---|
| `sell-used`, `sell-new`, `rental` | **60 days** | The pre-launch risk is an *empty* feed, not a stale one, so this is deliberately generous. But a six-month-old post whose stock is long gone wastes a phone reveal, and §6 calls that "the highest-trust moment in the app". ~One wedding season. |
| `requirement` with `neededBy` | **`neededBy` + 3 days** | §15 already names `neededBy` as "usable as an auto-expiry trigger". The grace period exists because the date is when the goods are needed, not when the poster stops caring — suppliers still respond around it, and weddings move. |
| `requirement` without `neededBy` | **30 days** | Demand goes stale far faster than supply. "I need 50 pillars in Madurai next week" is worthless a month later; a mandap frame for sale is not. |
| Renewal | **30 days from now** | Measured from *now*, not from the old expiry, so renewing something that lapsed a month ago gives a full fresh window rather than one already half gone. |

All four live as named constants in `app/(auth)/_lib/expiry.ts` — one edit to
change any of them.

**Decision D13.3 — a listing with NO `expiresAt` never expires.** Every
listing created before tonight lacks the field. Inferring an expiry from
`createdAt` would retroactively empty the entire existing feed the first time
someone opened the app. Pinned by a test.

**Decision D13.4 — expiry is renewable and visible, not silent.** My Listings
shows "Expires in N days" / "Expires today" / "Expired", and an expired post
gets a Renew action. A feature that quietly deletes someone's work without
telling them is worse than no feature.

### The single `status` field problem

§3 fixes one `status` field that carries **two different ideas**: moderation
state (`pending` → `approved`/`rejected`) and owner availability
(`sold`/`unavailable`). So "put it back on sale" has to know what to restore
*to*.

**Decision D13.5 — constrain the entry point instead of adding a field.** The
toggle is offered only on `approved` listings, and restoring always returns to
`approved`. This works because a `pending` post appears in no feed (marking it
sold would mean nothing) and a `rejected` one must not be restorable by its
owner at all. That removes the ambiguity rather than storing extra state to
resolve it later, and keeps §3's documented shape untouched — I did not want
to add a second field to a schema the file fixes explicitly.

### Free-text search

**Decision D13.6 — client-side, over the already-fetched feed.** Firestore has
no substring or full-text query. The real alternatives are an external index
(Algolia/Typesense — a new paid dependency and a sync job, neither in the
locked stack §2) or filtering what the client already holds. `fetchListings()`
already downloads the whole collection and filters in memory, so search adds
**zero extra reads and zero infrastructure**.

The scaling limit is stated plainly in the module: this searches only what the
device has fetched, which is correct exactly as long as `fetchListings()`
itself is viable. When that stops being true, the feed query and search have
to be solved together — search is not the piece that breaks first.

Behaviour: case-insensitive **substring** matching (so "panthal" finds "Green
Panthal") across title, description, category and district; multiple terms
**AND** together, because a second word is for narrowing — "red carpet" should
not return everything red plus every carpet. No debounce, deliberately:
nothing is being requested, so filtering an in-memory array per keystroke is
free.

### Also built

- **My Listings** (`app/my-listings/index.tsx`), reached from Profile → My
  Posts. Active/Inactive tabs; per-row status badge, expiry state, and §15's
  seller-facing `viewCount` / `interestCount`; actions for mark sold, mark
  unavailable, put back on sale, renew, delete. `pending` counts as *active* —
  it is on its way into the feed, and burying it under "Inactive" would read
  as a rejection.
- `deleteListing()` removes the listing's photos too (best-effort, same
  reasoning as account deletion).
- Composite index for `sellerId` + `createdAt DESC` added to
  `firestore.indexes.json` — the emulator does not enforce indexes, so this
  would otherwise have failed only in production. Verified with
  `firebase deploy --dry-run`.
- Distinct empty-state copy per situation: nothing posted, nothing matching
  the chips, nothing found for a specific search term, nothing in your
  category/district.

> ⚠️ Counts on listings created before tonight will read 0 views / 0
> interests. That is a genuine history gap from the `.exists` bug, not a
> display fault — the events were never recorded.

### Testing pass

Build/verify: `tsc --noEmit` clean; `expo export --platform android` bundles;
matching 15 ✓, expiry+search 20 ✓, rules 45 ✓, storage 5 ✓, flows 34 ✓ (119
total). Rules validated with `firebase deploy --dry-run`.

Edge cases, empty states, boundaries and error paths — all pass:

**Expiry**

| Case | Result |
|---|---|
| Supply vs. requirement windows | 60d / 30d, and demand is verified to be the shorter one |
| `neededBy` drives expiry + grace | correct to the millisecond |
| `neededBy` already in the past | post is already expired, as intended |
| **Malformed** `neededBy` (`'not-a-date'`, `''`, `null`) | falls back to the default window, never throws — a bad optional field must not block posting |
| Listing with **no** `expiresAt` | never expires; no label, no day count |
| Exact boundary (`expiresAt === now`, ±1 ms) | inclusive at now; correct either side |
| Day count on a part-day, and long past expiry | floors; never negative |
| Labels at each boundary | Expired / today / tomorrow / "in N days" |
| Renewing a 45-day-lapsed listing | full fresh window, not a half-gone one |

**Search**

| Case | Result |
|---|---|
| Empty and whitespace-only query | returns everything unchanged |
| Case-insensitive; substring not whole-word | "PANTHAL", "carpe" both match |
| Multiple terms AND, not OR | "red carpet" → 1 result, "red" → 2 |
| Term order | irrelevant |
| Searching category, district, description | all match |
| No results | empty list (not everything), with its own empty-state copy |
| Listings with missing title/description fields | no crash |
| Input array mutation | none |

**My Listings / rules**

| Case | Result |
|---|---|
| Owner: approved → sold → approved → unavailable | all allowed |
| Non-owner changing `status` or `expiresAt` | denied — while the §6 counter bump they legitimately need still works |
| Owner renewing an expired listing | allowed, expiry lands in the future |
| Suspended owner: toggle vs. delete | toggle denied, delete allowed (D9.3) |
| My Listings query returns all four statuses | 4/4, not just approved |
| Business with no posts | empty result, not an error |
| Responder of a **different** category on a requirement (§7) | reaches the poster's responders list, and the poster can resolve their contact |
| Responding twice | second write denied — dedupe keeps `interestCount` honest |
---

## TASK 14 — Ops hygiene (§14 item 6, §6, §15)

### Daily caps — and exactly how much they guarantee

Counters live at `users/{uid}/rateLimits/{yyyymmdd}`. The rules enforce, and I
tested, two things that are genuinely airtight:

1. a counter may only move **up, by exactly 1**, and never past its cap;
2. a protected write (`interests`, `listings`, `reports`) requires today's
   counter to exist and be within cap.

**What they do NOT guarantee — stated plainly because it matters:** rules
cannot count writes, and cannot require two documents to change together. A
client that deliberately **skips the increment call** can still exceed the cap.
There is no way to close that in rules; it needs a trusted server (Cloud
Functions → Blaze, blocked per §8/§13) or App Check. So this is a real limit
for anything using the app's own flow and a deterrent against casual scripting
— not a wall against a purpose-built one. The backstop is the one §15 already
chose: manual moderation, with counters as per-day documents an admin can read
so a harvester's pattern is visible, and suspension **is** rules-enforced.
I wrote a test that asserts this gap explicitly rather than leaving it implied.

**Decision D14.1 — the caps.** DECISIONS.md mandates a reveal cap (§6, §15)
but never fixes a number.

| Action | Cap/day | Reasoning |
|---|---|---|
| Phone reveals | **25** | Generous for a real business working a wedding season; restrictive for anyone collecting numbers. §6 calls this "the only guard against harvesting". |
| New posts | **20** | Well above any honest day's posting; low enough that flooding the feed takes many accounts, not one. |
| Reports | **10** | Reporting should be rare. A high allowance is exactly what report-bombing needs. |

Asserted in a test that reveals > posts > reports, so the ordering can't be
casually inverted later.

**Decision D14.2 — the `<=` in the cap check, which is a bug I introduced and
then caught.** The client increments *before* acting, so by the time the rule
runs the counter already includes the action being checked. I first wrote
`count < cap`, which silently made **every cap one lower than the number it
advertises** — 24 reveals, not 25. Reading the rule did not reveal it; replaying
the client's real increment-then-act sequence against the emulator did, on
iteration 25. Going over is still impossible: the counter's own rule refuses to
move past `cap`, so the 26th action can never be charged. This is the main
reason the flow suite replays sequences rather than only asserting single
permissions.

**Decision D14.3 — only a FIRST-time reveal or report is charged.** Reopening
a listing you already responded to shows a number you have already been given;
counting it again would let ordinary re-checking burn an allowance meant to
stop harvesting *new* numbers. Re-filing a report overwrites one document, so
the rules exempt `update` on `reports` from the cap entirely — otherwise
changing your mind about the reason would cost an allowance.

**Decision D14.4 — the counter document id must be TODAY.** Without that, a
user could pre-seed tomorrow's counter and hold two days' allowance at once. A
fresh counter must also start with exactly one action recorded (not
part-spent, not two at once), and counters are never deletable — deleting
today's would reset the cap.

### The scraping hole the reveal cap does not close — please read

While implementing the cap I checked what it actually protects, and found the
`users` collection was readable **and listable** by any signed-in account.
Phone numbers live in `users/{uid}`. So bulk harvesting never needed the
reveal flow at all: one `getDocs(collection('users'))` returned the entire
table, and §6's cap would have been decorative.

**Fixed tonight:** `users` is now `get`-yes / `list`-admin-only, the same shape
`profiles` already uses (§6a). No client code lists users — I checked every
`collection('users')` call site in the app; all are `.doc(id)`. `w2d-admin`
keeps its Users page because `isAdmin()` retains `list`.

**Still open, and I did not fix it:** a signed-in account can still walk
`listings` (which it must be able to read), collect `sellerId`s, and read
those user documents one at a time. Closing that means contact details cannot
live in a broadly-readable document at all — e.g. moving `phone` behind a
per-pair "reveal grant" document that the `users` read rule checks with
`exists()`. That is expressible in rules, but it is a redesign of §6's
mechanic surface and it changes how the responders list and the catalog header
read contacts. **That is a product/security decision, not ops hygiene, so I
flagged it rather than doing it unsupervised overnight.** Noted again in Task
16 as a release-gate consideration.

### Crashlytics + Analytics visibility

Both free on Spark. Added behind the same guarded-require-and-probe pattern as
messaging (§13.1 — a top-level import would hard-crash every existing dev
client), so without the native modules every call is a silent no-op.

**Decision D14.5 — the event set is §1's two metrics, kept separate.** §1 is
explicit that "interests per trade post" and "profile views/shares" must be
tracked separately and never collapsed into one number, so the events mirror
that split: `trade_interest` + `listing_created` on one side,
`public_profile_viewed` / `_shared` / `_created` on the other, plus
`signup_completed` and `rate_limit_hit` (a spike in the last one is what
harvesting looks like).

**Decision D14.6 — nothing personal is logged, as a hard rule.** Parameters
carry post type, category and district only — never a phone number, business
name, person's name, or profile slug (the slug is the shareable secret that
makes a profile unguessable, §6a). Crash reports get the uid and nothing else.
The reason is concrete: whatever is logged has to be declarable on the Play
Data Safety form (§14.2), and the cheapest way to keep that declaration honest
is to log nothing that identifies anyone.

**Decision D14.7 — the errors the app deliberately swallows now report to
Crashlytics** (`notifyListingOwner`, `deleteStorageObjectByUrl`). Those are
the failures the UI hides on purpose, which means they were invisible in ops
as well as in the app. Now only one of those is true.

> ⚠️ **ACTION FOR YOU:** `@react-native-firebase/crashlytics` and
> `/analytics` are now dependencies with config plugins. Like messaging, they
> need a **fresh EAS dev-client build** before they do anything (§13.1).
> Nothing breaks before then.

### Also

Extracted the caps and the UTC day-key formula into `constants/rateLimits.ts`,
which imports nothing, so a plain node test can verify the numbers and the
formula that **must** agree with `firestore.rules`. UTC is load-bearing there:
`request.time` in the rules is always UTC, so a local-time key would put
client and rules on different days for part of every day and deny every capped
write in that window.

### Testing pass

Build/verify: `tsc --noEmit` clean; `expo export --platform android` bundles;
matching 15 ✓, expiry/search 20 ✓, caps/day-key 6 ✓, rules 60 ✓, storage 5 ✓,
flows 37 ✓ (**143 total**). Rules validated with `firebase deploy --dry-run`.

Edge cases, boundaries and abuse paths — all pass:

| Case | Result |
|---|---|
| Full 25-reveal sequence replayed exactly as the client runs it | all 25 succeed; the 26th cannot be charged |
| The `<` vs `<=` off-by-one | caught by that replay; cap now delivers the number it advertises |
| Counter jumped by 2, moved backwards, or reset to 0 | all denied |
| Counter pushed past its cap | denied for reveals, posts and reports |
| Two counters moved in one write | denied |
| Counter created for **tomorrow** | denied (would grant two days' allowance) |
| Fresh counter created part-spent, at zero, with two actions, or missing fields | all denied |
| Another account touching your counters | denied |
| Counter deletion (would reset the cap) | denied |
| Counter read by owner / admin / another user | yes / yes / no |
| Protected write with **no** counter document at all | denied |
| Counter *above* cap (unreachable, but defence in depth) | action denied outright |
| Yesterday's allowance fully spent | today is unaffected |
| Suspended account | blocked before the cap matters; incrementing its own counter stays allowed and is harmless |
| Caps blocking reads, edits, deletes | they do not — only new actions |
| Report cap reached, then amending an existing report | still allowed (exempt by design) |
| `users` enumeration by a signed-in user / anonymous / admin | denied / denied / allowed |
| Client that skips the increment | **still succeeds** — the documented, untestable-away limitation, asserted explicitly |
| Day key: UTC midnight rollover, single-digit month/day, year end, leap day | all correct |
---

## TASK 15 — Legacy reference audit (RESEARCH ONLY, no code changed)

Swept both repos for `userType` assumptions, old 10-item category references,
and anything still assuming the pre-Task-2 model. Grouped by severity, with
`file:line`.

**No code was changed under this heading.** Two items in §A are regressions
caused by tonight's own work; they are fixed in a separate follow-up commit
after this audit, clearly labelled as such, not as part of Task 15.

---

### A. BROKEN RIGHT NOW — regressions caused by tonight's changes

Both confirmed by actually running the scripts, not by reading them.

| # | Location | What breaks |
|---|---|---|
| A1 | `w2d-admin/scripts/verify-admin.mjs:56-58,65` | Asserts `vendors.length === 4 && manufacturers.length === 4` from `users.userType`. Task 12 stopped `w2d-app/scripts/seed.mjs` writing `userType`, so this now fails: **`ASSERT: role split 4/4`**. `npm run verify` in `w2d-admin` dies here. Also `:76` prints `v=/m=` counts that are now always 0. |
| A2 | `w2d-admin/scripts/verify-rules.mjs:180-181,194,204-229` | Creates listings as a signed-in fixture user. Task 14 made `listings` create require today's `users/{uid}/rateLimits/{yyyymmdd}` counter, and these fixtures have none, so every create is denied: **`ASSERT: case 1: manufacturer creates requirement must be ALLOWED — got permission-denied`**. The script also still seeds `userType` on its fixtures and names its cases "manufacturer"/"vendor" (harmless labels now, but misleading). |

Fix for A1: assert on `category` coverage instead of the role split — the file
already computes `categorised` two lines below. Fix for A2: seed a
`rateLimits/{today}` counter alongside each fixture user, exactly as
`w2d-app/tests/flows.test.mjs` does.

---

### B. HIGH — user-facing and wrong, blocks or damages release

| # | Location | Issue |
|---|---|---|
| B1 | `app/legal/privacy.tsx:11,15` | Describes the app as "wedding **decoration**" only, connecting "manufacturers, decorators, and suppliers" — the model §7 deleted and the scope §9 widened to 29 categories. |
| B2 | `app/legal/privacy.tsx:27` | Collects "Business type (**Vendor or Manufacturer** — self-declared)". That field is no longer collected or written at all. A privacy policy naming data you do not collect is as wrong as one omitting data you do. |
| B3 | `app/legal/privacy.tsx` (whole file) | **No mention of the public profile (§6a)** — the one place the app publishes a phone number to anyone with a link. This is the single most consequential disclosure gap. The ToS was rewritten for it tonight (Task 10); the Privacy Policy is parked per instruction and must match before submission. |
| B4 | `docs/PRIVACY_POLICY.md:6-7,11,20` | The same, and **worse**: line 20 still lists the *three*-value model ("Manufacturer, Decorator, or Supplier") superseded on 2026-08-01, two models ago. |
| B5 | `docs/TERMS_OF_SERVICE.md:8,20` | Stale copy of the pre-tonight ToS ("wedding decoration industry", "manufacturers, decorators and suppliers"). Now **contradicts** the shipped `app/legal/terms.tsx`. Two ToS texts disagreeing is worse than one being out of date — whichever a reviewer finds first is the one that counts. |
| B6 | `app/(auth)/phone.tsx:72` | First screen a new user ever sees: "Buy & sell wedding **decoration materials**". Wrong for 28 of the 29 categories — a caterer or photographer reads that and assumes the app is not for them. Cheapest high-value copy fix in the codebase. |

---

### C. MEDIUM — will mislead code or hide bugs

| # | Location | Issue |
|---|---|---|
| C1 | `w2d-admin/src/lib/types.ts:37` | `UserDoc.userType: UserType` is **required**, not optional. Every account created after tonight has no `userType`, so the type asserts something false. `Users.tsx:204` happens to guard with `{u.userType && …}`, so nothing crashes — but the next person to trust the type will write unguarded code. Should be `userType?: UserType`. |
| C2 | `w2d-admin/src/lib/types.ts:44` | `categories?: string[]` — the Manufacturer-only multi-select §7 deleted. Nothing writes it. |
| C3 | `w2d-admin/src/lib/types.ts` (`ListingDoc`) | **No `expiresAt`** (added tonight, Task 13). The admin cannot see that a listing has expired, so an expired post looks "approved" and live in the moderation queue while being invisible to every user. |
| C4 | `w2d-admin/src/lib/types.ts` / all pages | No awareness of `notifications` (Task 11), `users/{uid}/devices` (Task 11) or `users/{uid}/rateLimits` (Task 14). The last one matters most for ops: the counters are the only visible signal of a harvesting pattern, and nothing surfaces them. |
| C5 | `w2d-admin/src/lib/categories.ts:1-13` | The 29-item list is a **hand-copied duplicate** of `w2d-app/constants/listings.ts` (separate repos, no workspace). Already flagged in its own header. Real drift risk if §9 ever changes. |
| C6 | `app/(auth)/_lib/firestore.ts:374-394` | `LegacyListingCategory` and the `ListingCategory` union survive. **Intentional** — old documents still carry those values and must type-check on read — but this is the last structural trace of the 10-item list, and it should be deleted once no live document uses it. |

---

### D. LOW — dead surface, correctly handled, no action needed now

| # | Location | Note |
|---|---|---|
| D1 | `app/(auth)/_lib/firestore.ts:30,84,146,515` | `UserType` type, `UserProfile.userType?`, `ListingDoc.sellerUserType?` — all `@deprecated`, all optional, read-only, never written. `getCurrentUserProfile` no longer defaults to `'vendor'` (Task 12). Correct migration shape. |
| D2 | `w2d-admin/src/pages/Users.tsx:204-205` | Renders the legacy `userType` badge, guarded — degrades to no badge on new accounts. Deliberate: it shows ops which accounts predate the migration. |
| D3 | `w2d-admin/src/pages/Dashboard.tsx:99-102` | Already migrated off the role split to "how many accounts have picked a category". Correct. |
| D4 | `w2d-admin/scripts/seed-admin.mjs:88-93` | Explicitly does *not* write `category` or `userType`, with the reasoning documented. Correct. |
| D5 | `w2d-app/tests/flows.test.mjs:788` | Uses `userType: 'vendor'` on purpose, reproducing a legacy signup payload to pin the profile-overwrite hazard. Correct. |
| D6 | `app/(auth)/pick-category.tsx:29`, `useCategoryPrompt.ts:11`, `(tabs)/_layout.tsx:8`, `post.tsx:18-19`, `ListingCard.tsx:45`, `catalog/[uid].tsx:113`, `requirement.tsx:6`, `profile-setup.tsx:29`, `matching.ts:15`, `seed.mjs:263` | Comments explaining what was removed and why. Keep them — §19's whole point is that a future session must not "fix" a deliberate decision back. |

---

### E. INFORMATIONAL — planning docs describing the old model

Not code, and not load-bearing, but a future AI session reading them cold
would be misled — which is precisely the failure `PRODUCT_CONTEXT.md` §13 says
this process exists to prevent.

| File | Hits for vendor/manufacturer/userType |
|---|---|
| `BUILD_PLAN_v1.2.md` | 80 |
| `PROGRESS.md` | 24 |
| `UX_SIMPLIFICATION_v1.2.md` | 10 |
| `ADMIN_DASHBOARD_PLAN.md` | 4 |
| `GAPS_AUDIT.md`, `VISHNU_TASKS.md`, `SESSION_SUMMARY_2026-08-01.md`, `DESIGN_SYSTEM.md` | 1 each |
| `docs/archive/SUPERSEDED_*.md` (5 files) | already marked superseded, and `AGENTS.md` tells agents to ignore `docs/archive/` |

Suggestion, not done: give the top four the same treatment `docs/archive/`
already has — a header saying superseded, or move them there. `AGENTS.md`
already instructs agents to ignore that directory, so moving them makes the
instruction do the work.

---

### F. Checked and found CLEAN

- No client code anywhere lists the `users` collection — every
  `collection('users')` call in `w2d-app` is `.doc(id)`, so Task 14's
  `list: isAdmin()` restriction breaks nothing.
- No remaining `intentOptionsForRole` / `postTypesForRole` call sites.
- No remaining `LISTING_CATEGORIES` imports; every picker is on the 29-item list.
- No listing, catalog or profile **write path** can produce a legacy category
  value — the compiler enforces it via `BusinessCategory` on the input types.
- `w2d-admin`'s runtime pages are unaffected by tonight's rules changes:
  `isAdmin()` retains `list` on `users`, and admin listing moderation still
  matches the unchanged admin branch of the `listings` update rule.
---

## FOLLOW-UP (not Task 15) — fixing the two regressions Task 15 found

Task 15 was research-only and stayed that way. These fixes are a separate
commit in the **`w2d-admin`** repo, because they repair breakage tonight's own
work caused, and leaving a knowingly-broken verification script would be worse
than the mild scope stretch. Both were confirmed broken by running them, and
both confirmed fixed the same way.

**A1 — `scripts/verify-admin.mjs`.** Replaced the
`vendors === 4 && manufacturers === 4` assertion. That read `users.userType`,
which nothing writes any more (§7), so it could only ever fail from Task 12
onward. It now asserts the thing that assertion was standing in for: a MIX of
categorised and uncategorised seed accounts, so both the populated and the
fallback UI paths get exercised. The metrics line reports
`categorised=4/8 legacyUserType=0` instead of `v=/m=`.

Also scoped the fixture-count assertions to the `seed-` id prefix. Bare
`users.size === 8` passed on a fresh emulator and failed on every run after,
because the sibling `verify-rules.mjs` leaves its own `rules-check-*` fixtures
in the same emulator and never cleans them up. That flake is what gets a
verification script quietly ignored.

**A2 — `scripts/verify-rules.mjs`.** Its fixture businesses had no
`users/{uid}/rateLimits/{today}` counter, which Task 14 made a precondition of
creating a listing, so every case was denied before reaching the rule actually
under test. Two subtleties, both found by running it rather than reading it:

1. the counter must be created with **`posts: 1`, not `0`** — the rules refuse
   a counter recording no action at all (sum must be exactly 1), because a
   client able to write a zeroed counter could also reset one; and
2. it must be created **only when absent** — a `setDoc(..., {merge: true})`
   that changes nothing has `affectedKeys().size() === 0`, which the "exactly
   one counter moved by one" rule denies. Writing it unconditionally passed
   the first run and failed every re-run.

Also moved those fixtures off `userType` onto two different `category` values.
The case labels ("manufacturer"/"vendor") are kept so the case names read the
same, but what they now prove is the actual §7 change: *any* category can
raise a requirement.

Verified after the fix: `verify-admin.mjs` → all 5 checks PASS;
`verify-rules.mjs` → all 14 cases PASS, including the §6a public-profile
cases; `tsc -b` clean.
---

## TASK 16 — Play Store readiness audit (RESEARCH ONLY, no code changed)

Everything below was checked against the actual code, `app.json`, `eas.json`,
the asset files, and the AndroidManifest files inside the installed native
modules — not assumed. Target SDK resolves to **35** (Android 15) via
`expo-modules-autolinking`, which matters for several items.

**One item here is a defect in tonight's own Task 11 work (§A1). It is fixed
in a follow-up commit after this audit, not under Task 16.**

---

### A. HARD BLOCKERS — submission will fail or the feature will not work

| # | Item | Detail |
|---|---|---|
| **A1** | **`POST_NOTIFICATIONS` is not declared anywhere** | Grepped every installed module: neither `@react-native-firebase/messaging`'s manifest, its config plugin, nor `app.json` declares it. On targetSdk 35, Android 13+ **requires** it before a notification permission prompt can appear, so `messaging().requestPermission()` cannot succeed. Push would be silently dead on every modern device. Needs `"permissions": ["android.permission.POST_NOTIFICATIONS"]` under `expo.android`. **This is a defect in Task 11 — fixed in the follow-up below.** |
| **A2** | **No reviewer sign-in path** | Every screen except `/p/[slug]` is behind phone-OTP auth. A Google reviewer has no Indian mobile number, so they see the phone screen and nothing else — a standard, very common rejection. Fix: configure a Firebase Auth **test phone number** with a fixed OTP (Firebase Console → Authentication → Sign-in method → Phone → *Phone numbers for testing*), then fill in Play Console → **App access** → "All functionality is restricted" with that number, the fixed code, and a one-line walkthrough. Nothing in the repo does this and nothing can. |
| **A3** | **No public Privacy Policy URL** | Play requires a live, publicly reachable URL in the store listing *and* in the Data Safety form. The in-app screen does not satisfy it. `w2d-landing` is a single static `index.html` with no host configuration in the repo — so there is currently nowhere to put it. |
| **A4** | **Privacy Policy content is wrong** (see Task 15 §B) | Even once hosted: `app/legal/privacy.tsx` and `docs/PRIVACY_POLICY.md` describe a decoration-only app with a role model that no longer exists, and **never mention the public profile** — the one place the app publishes a phone number to anyone with a link. Parked per tonight's instruction; it is a release gate. |
| **A5** | **No account-deletion web URL** | Play's data-deletion policy wants a web-accessible way to request deletion for people who have already uninstalled. In-app deletion now exists (Task 10), but the URL field in the Data Safety form has nothing to point at. Same hosting problem as A3. |
| **A6** | **No store listing assets** | Nothing in the repo: Play needs a 512×512 store icon, a 1024×500 feature graphic, and **at least 2 phone screenshots**. `assets/icon.png` is 1024×1024 (the app icon, not the store icon). |

---

### B. PERMISSIONS — the exact merged set, and what to justify

Enumerated from each module's own `AndroidManifest.xml`, because Play asks you
to justify what actually ships, not what you think you requested.

| Permission | Source | Justification / action |
|---|---|---|
| `CAMERA` | `expo-image-picker` | Taking photos of stock, catalog items and portfolio work. Genuine and used in three screens. |
| `READ_EXTERNAL_STORAGE` | `expo-image-picker` manifest | Choosing existing photos. On API 33+ superseded by `READ_MEDIA_IMAGES`. |
| **`WRITE_EXTERNAL_STORAGE`** | `expo-image-picker` manifest | ⚠️ **The app never writes to shared storage.** This is the permission Play scrutinises hardest, and it is being requested for nothing. Strip it in the merged manifest with `tools:node="remove"` before submission. |
| `INTERNET`, `WAKE_LOCK`, `ACCESS_NETWORK_STATE` | RNFirebase auth + messaging | Standard, no justification needed. |
| `POST_NOTIFICATIONS` | **absent** | See A1. |
| Location, Contacts, SMS | **none** | Verified absent across every dependency. §15 *plans* "near me" sorting and "invite friends", but neither is built, so nothing needs declaring. If either is built later, both become new Data Safety entries and new justifications. |

**Not verifiable without a build:** `expo-image-picker` is **not listed in
`app.json` → `plugins`**, so its config plugin never runs and the permissions
come only from the library's own manifest merge. Run `npx expo prebuild -p
android` and read the generated `android/app/src/main/AndroidManifest.xml`
before submitting. You cannot justify a permission list you have not seen.

---

### C. DATA SAFETY FORM — inventory of what is ACTUALLY collected

Compiled from every write path in `app/(auth)/_lib/`. §14.2 asks specifically
that this reflect reality; the §6a public exposure is flagged separately below
because it changes the *answers*, not just the item list.

**Personal info**

| Data | Where | Shared/public? |
|---|---|---|
| Phone number | Firebase Auth, `users/{uid}.phone` | **Visible to other users** on reveal (§6); **PUBLIC, no login** on `profiles/{slug}.phone` (§6a) |
| WhatsApp number (optional, separate) | `profiles/{slug}.whatsapp` | **PUBLIC, no login** |
| Person's name | `users/{uid}.name`, `listings.sellerName` | Visible to signed-in users. Deliberately **not** on the public profile |
| Business name | `users`, `listings`, `profiles`, `notifications.actorBusinessName` | **PUBLIC** via profiles |
| District (self-selected, not GPS) | `users`, `listings`, `profiles` | **PUBLIC** via profiles |
| Business category | `users`, `listings`, `profiles` | **PUBLIC** via profiles |

**Photos & user content**

| Data | Where | Public? |
|---|---|---|
| Listing photos | `listing-photos/` | Auth-gated |
| Catalog photos | `catalog-photos/` | Auth-gated (§6a keeps catalog private, deliberately) |
| **Portfolio photos** | `profile-photos/` | **PUBLIC READ** in `storage.rules` — required, or an anonymous visitor sees broken images |
| Titles, descriptions, report reasons | `listings`, `reports` | Signed-in users; reports are admin-only |

**App activity & identifiers**

| Data | Where |
|---|---|
| FCM device token | `users/{uid}/devices/{token}` |
| Interests / reveals | `interests`, `listings.interestCount` |
| Views | `listings.viewCount` |
| Daily action counters | `users/{uid}/rateLimits/{yyyymmdd}` |
| In-app notifications | `notifications` |
| Crash logs (uid only, no PII) | Crashlytics |
| Analytics events (post type, category, district) | Firebase Analytics |

> ⚠️ **Firebase Analytics also collects automatically** — app instance ID,
> device model, OS version, coarse location derived from IP, and session data.
> That is collected whether or not you log a single custom event, and it must
> be declared. Our own events are deliberately PII-free (Task 14, D14.6), but
> that does not exempt the SDK's automatic collection.

**The answer that needs the most care:** for phone number, business name,
district, category and portfolio photos the Data Safety answer is **not**
"shared with third parties" — it is that the app makes them **publicly
accessible with no account required**, by design (§6a). Understating that is
the kind of mismatch that gets an app pulled after launch rather than
rejected before it. The ToS section 5 written in Task 10 is the user-facing
half of the same disclosure; the two must agree.

---

### D. CONFIGURATION — smaller, real, cheap to fix

| # | Item | Detail |
|---|---|---|
| D1 | `app.json` → `"name": "w2d"` | This is the **launcher label on the device**. Should be `"Wedding2day"`. `slug` can stay `w2d`. |
| D2 | No `expo-splash-screen` plugin | `assets/splash-icon.png` exists but nothing references it — the app ships the default splash. Cosmetic, not a blocker. |
| D3 | `expo-dev-client` in `dependencies` | Correct for dev builds and inert in production, but it is bundle weight in a release build. Worth confirming the production profile excludes it. |
| D4 | `SUPPORT_WHATSAPP_NUMBER` still carries a `TODO` | `app/(auth)/_lib/support.ts` — flagged in the code itself as "replace before Play Store submission". The support email is also a **personal Gmail**, which becomes the public developer contact. |
| D5 | Content rating + target audience | Questionnaire not started. The app is 18+ business-to-business (ToS §2), so declare adults-only — that also keeps it clear of Families policy. |
| D6 | Version code | `eas.json` uses `appVersionSource: "remote"` with `autoIncrement: true` on production. Correct; no action. |
| D7 | Icons | `icon.png` 1024×1024 ✓, adaptive foreground/background 512×512 ✓, monochrome 432×432 ✓ (themed-icon ready). Only the *store* assets are missing (A6). |
| D8 | No payments, no ads, no third-party SDK sharing | Genuinely simplifies the form — §6/§10 rule out in-app payments, and there is no ad SDK. Worth answering confidently rather than defensively. |

---

### E. SECURITY ITEM WORTH RAISING BEFORE LAUNCH, NOT AFTER

From Task 14: any signed-in account can still walk `listings`, collect
`sellerId`s, and read those `users` documents one at a time to harvest phone
numbers. Enumeration is now blocked and the daily reveal cap is enforced, but
the underlying read is not. This is not a Play *policy* blocker — Play does
not test for it — but it is a data-protection exposure of exactly the data
class §6a already makes the most sensitive part of this app, and it is much
cheaper to fix before there are real users' numbers in the database than
after. Concrete option in Task 14's write-up (a per-pair reveal-grant document
that the `users` read rule checks with `exists()`).

---

### F. Sequencing suggestion for the morning

1. A1 (one line, fixed below) and D1 — trivial, do first.
2. A2 — the reviewer test number. Needs only Firebase Console access and blocks
   nothing else, but is the most commonly fatal item on this list.
3. A3 + A5 together — one hosting decision solves both, and it is the same
   decision blocking §6a's real browser URL (`w2d-public-profile-route` note).
4. A4 — Privacy Policy rewrite, mirroring the ToS section 5 written tonight.
5. A6 + D5 — store assets and the rating questionnaire, both mechanical.
6. B's `WRITE_EXTERNAL_STORAGE` strip — after a prebuild, so you are reading
   the real merged manifest.
---

## FOLLOW-UP (not Task 16) — fixing the defect Task 16 found in Task 11

Task 16 stayed research-only. This is a separate commit, for the same reason
as the Task 15 follow-up: the finding is a defect in tonight's own work, and
shipping push notifications that cannot ever obtain permission would make all
of Task 11 dead on arrival.

**`POST_NOTIFICATIONS` now declared** in `app.json` under
`expo.android.permissions`. Verified with `npx expo config --json` that it
resolves into the config. On targetSdk 35 (which is what
`expo-modules-autolinking` resolves to) Android 13+ shows no notification
prompt at all without it, and neither RNFirebase messaging's own manifest nor
its config plugin declares it — so `messaging().requestPermission()` would
have failed silently on every modern device, with nothing in the logs to say
why. Documented at the top of `push.ts` next to the code that depends on it,
because removing the line disables push without any visible error.

**Also fixed D1 while there:** `expo.name` was `"w2d"`, which is the launcher
label users see on their home screen. Now `"Wedding2day"`; `slug` stays `w2d`
so nothing about the EAS project changes.

Verified: `expo config --json` shows `name: Wedding2day` and the permission
present; `tsc --noEmit` clean; `expo export --platform android` bundles; all
143 tests still pass.
---
---

# FINAL SUMMARY — morning review

Everything below is verified state, not intent. Final run: `tsc --noEmit`
clean, **143 tests passing**, `expo export --platform android` bundles,
`firebase deploy --dry-run` compiles, both repos committed and pushed clean.

## What completed

| Task | Status |
|---|---|
| 9 — Profile / Settings | **Done.** Category model, catalog editor, business-details editor, complete account deletion, phone-change profile sync. |
| 10 — ToS + in-app deletion (RELEASE GATE) | **Done.** ToS rewritten incl. a public-profile disclosure section; real deletion flow with re-auth; suspended accounts can delete; returning-user routing bug fixed. |
| 11 — FCM infrastructure | **Done, minus what Blaze blocks.** Full client plumbing + a working, server-free in-app notification centre. Sending is blocked; the exact split is tabled under Task 11. |
| 12 — Matching, Tier 1 only | **Done.** Plus the taxonomy migration it silently depended on, and the six surviving §7 role gates. **Tier 2 not started**, as instructed. |
| 13 — My Listings, expiry, search | **Done.** Two of its pieces (My Interests, responders list) turned out to have never worked and now do. |
| 14 — Ops hygiene | **Done.** Rules-enforced daily caps, spam throttling, Crashlytics + Analytics. |
| 15 — Legacy audit (research) | **Done.** Full `file:line` list by severity. |
| 16 — Play Store audit (research) | **Done.** Blockers, permissions, Data Safety inventory. |
| §14 item 7 (UI revamp), item 8 (admin app) | **Not started**, as instructed. |

Two follow-up commits fixed defects the research tasks found in tonight's own
work — the broken `w2d-admin` verification scripts, and the missing
`POST_NOTIFICATIONS` declaration. Both are labelled as follow-ups, not as
Tasks 15/16, which stayed research-only.

## The five things most worth your attention

1. **`expressInterest` never worked.** `.exists` is a method in RNFirebase
   v25, so the dedupe check always said "already exists" and **no interest
   document was ever written**. My Interests, the requirement responders list
   and every seller-facing interest count have been reading a permanently
   empty collection. Fixed in Task 9, along with four sibling misuses. Counts
   on pre-tonight listings will read 0 — a real history gap, not a display
   fault.

2. **Account deletion destroyed data and then failed.** The old flow deleted
   all Firestore data *first*, then hit `auth/requires-recent-login` and told
   the user "your account is still active". It also never deleted the
   world-readable `profiles/{slug}` document, which holds a phone number, and
   Storage deletes were denied by a rules bug so no image was ever removed.
   All three fixed (Tasks 9, 10).

3. **Tier 1 matching could never have matched.** `users.category` used the
   29-item list while every picker still wrote the retired 10-item decor list —
   zero shared values. Task 12 migrated the pickers and made the compiler
   enforce it.

4. **The reveal cap was guarding a door with no wall.** `users` was
   *listable* by any signed-in account, and phone numbers live there, so bulk
   harvesting never needed the reveal flow. Enumeration is now blocked
   (Task 14). A narrower gap remains — see "Decisions I'd most like you to
   review" below.

5. **Push would have been dead on arrival.** `POST_NOTIFICATIONS` was declared
   nowhere, so on Android 13+ the permission prompt could never appear. Found
   by the Task 16 audit, fixed in the follow-up.

## Testing

Built from nothing tonight — there was no automated test in the repo.

| Suite | Tests | Covers |
|---|---|---|
| `test:matching` | 15 | Tier 1 ranking, pure |
| `test:expiry` | 20 | expiry windows + free-text search, pure |
| `test:ratelimit` | 6 | caps + the UTC day key that must match the rules |
| `test:rules` | 60 | Firestore security rules, emulator |
| `test:storage` | 5 | Storage rules, emulator |
| `test:flows` | 37 | multi-step client sequences replayed against the real rules |

`npm test` runs all six. Per-task edge cases, empty states and error paths are
tabled under each task above.

**Two real bugs were caught only by the flow suite, not by reading code or by
single-permission tests:** the `< cap` off-by-one that made every daily cap one
lower than advertised, and the seed distribution that produced zero Tier-1
matches for any viewer. That is the argument for keeping that suite.

## Decisions I'd most like you to review

All decisions are logged inline above with reasoning; these are the ones where
a different call is most defensible.

1. **D14.1 — the cap numbers** (25 reveals / 20 posts / 10 reports per day).
   Invented; §6 mandates a cap but fixes no number. One-line change in
   `constants/rateLimits.ts` **and** `firestore.rules`.
2. **D13.2 — expiry windows** (60d supply, `neededBy`+3d, 30d requirements,
   30d renewal). Also invented. Constants in `_lib/expiry.ts`.
3. **D12.5 — category outranks district** in Tier 1. §16 lists both without
   ordering them. My reasoning is in Task 12; the opposite is arguable for a
   trade where delivery distance dominates.
4. **D13.5 — the sold/unavailable toggle only applies to `approved`
   listings.** This works around §3's single `status` field carrying both
   moderation state and availability, without adding a field to a schema
   DECISIONS.md fixes explicitly. If you would rather have a separate
   `availability` field, that is a §3 change.
5. **The residual phone-harvesting gap** (Task 14 §E, Task 16 §E). A signed-in
   account can still walk `listings`, collect `sellerId`s and read those user
   documents one at a time. Closing it means contact details cannot live in a
   broadly-readable document — a redesign of §6's mechanic surface, which I
   deliberately did not do unsupervised. Much cheaper before there are real
   numbers in the database.

## What you must do before this can build or ship

**Before the app runs at all:**

- ⚠️ **A fresh EAS dev-client build is required.** Three new native modules
  landed: `@react-native-firebase/messaging`, `/crashlytics`, `/analytics`
  (§13.1 — "Cannot find native module" is a rebuild, not a code fix).
  **Nothing breaks before then** — every one is behind a guarded
  require-and-probe, so on the current dev client they no-op and the in-app
  notification centre still works.
- **Deploy the rules and the new indexes**: `firebase deploy --only firestore`.
  Tonight added `firestore.indexes.json` with three composite indexes. The
  emulator does not enforce indexes, so without this the notification centre
  and My Listings fail **only in production**, on first use.

**Before Play Store submission** — full detail in Task 16:

1. Reviewer sign-in path (Firebase test phone number + Play "App access").
   Phone-OTP-only apps get rejected without it, and nothing in the repo can fix
   it for you.
2. A hosted Privacy Policy URL, and an account-deletion request URL.
3. Privacy Policy content rewritten (it still describes the deleted role model
   and never mentions the public profile). Parked tonight per instruction.
4. Store assets: 512×512 icon, 1024×500 feature graphic, ≥2 screenshots.
5. `npx expo prebuild -p android`, then strip `WRITE_EXTERNAL_STORAGE` from the
   merged manifest — the app never writes to shared storage.
6. Replace the support WhatsApp number `TODO` and the personal Gmail contact.

## Suggested next steps

1. The EAS rebuild + `firebase deploy --only firestore`, so tonight's work is
   actually live and push can be tested on a device.
2. Review decisions 1–4 above; each is a one-line constant change.
3. Decide on the phone-harvesting gap (decision 5) — it is materially cheaper
   now than post-launch.
4. Task 15 §B: the Privacy Policy and the stale `docs/TERMS_OF_SERVICE.md`,
   which now **contradicts** the shipped terms screen, plus the signup screen's
   "wedding decoration materials" tagline (wrong for 28 of 29 categories).
5. Then §14 item 7 (UI revamp) — items 1 and 5 added the screens it was
   sequenced to wait for, so its precondition is now met.

---
---

# OVERNIGHT_NOTES — unattended run, night of 2026-08-20 → 2026-08-21

Append-only. Previous night's log is above, untouched.

Context read in full before starting: `DECISIONS.md` (755 lines, including the
new §0b and the rewritten §7/§9), `PRODUCT_CONTEXT.md` (480 lines), and the
entire 2026-08-19→20 `OVERNIGHT_NOTES.md` entry (1408 lines).

Tasks in scope: 1–9, in strict order. Phase 1 (1–3) is the role split and must
pass before Phase 2 (4–7), which must pass before Phase 3 (9, the UI revamp).

**Baseline before touching anything:** `npm test` exit 0, 143 tests across six
suites, on commit `3db79bc`. Emulators (Auth 9099 / Firestore 8080 / Storage
9199) confirmed up. Both repos clean on `main`.

---

## TASK 1 — Restore the Vendor/Manufacturer role split (revised)

### The one thing I checked before writing any code

§19.7 and §19.10 both say to cross-check the file against actual code rather
than trusting either. The claim that mattered was §7's: that commit `6f479b3`
removed six role gates and all six need restoring. Verified each site
individually — all six were exactly as described, each carrying a comment
explaining the removal. `PRODUCT_CONTEXT.md`, by contrast, is **stale**: its §0,
§1, §3, §7 and §12 all still describe the role split as dropped. `DECISIONS.md`
§19.9 settles which wins ("newer supersedes older, full stop"), so I built
against `DECISIONS.md` and updated `PRODUCT_CONTEXT.md` to match rather than the
reverse. See D1.9 below.

### 1. Schema — `role` added, additively

- `app/(auth)/_lib/firestore.ts`: new `BusinessRole = 'vendor' | 'manufacturer'`.
  `UserProfileInput.role` **required**; `UserProfile.role` **optional**;
  `getCurrentUserProfile` reads it with no default.
- `listings.sellerRole` — denormalized in `createListing`, in `ListingDoc`, and
  backfilled by `updateBusinessDetails`.
- `profiles.role` — optional on `PublicProfile`, written by
  create/updatePublicProfile, read by `fetchPublicProfile`.
- `SellerContact` gained `role`, `district` and `verified` (§6/§15 list all
  three in the reveal screen's trust context; only `role` was strictly Task 1,
  the other two came free from the same document read).
- The retired `userType` is untouched and still `@deprecated`. §0b is explicit
  that this is a NEW field, not a revival, and every doc comment says so — the
  single most likely future misreading of this work is "oh, userType is back".

**Decision D1.1 — `role` is required at signup but optional on read.** An
account with no role can create *nothing*, because the symmetric gate denies
both directions on an absent field. So the write path must never produce one,
while the read path must handle the ~1 day of accounts that already have none.

### 2. `BUSINESS_ROLE_MAP` — new `constants/businessRoles.ts`

All 29 rows from §9, transcribed verbatim in that section's order, each of the
five "Both" rows marked with a `// §9: Both — default Manufacturer` comment.
Also exports `AMBIGUOUS_ROLE_CATEGORIES` (those five), `roleForCategory`,
`isAmbiguousRoleCategory`, `roleLabel`, `roleDescription`, and `roleMapDrift()`.

**A real bug, found by the test I wrote for the map rather than by reading it:**
`roleForCategory('constructor')` returned `Object.prototype.constructor` — a
truthy *function* — because a plain object literal inherits from
`Object.prototype`, so `map['constructor']`, `map['toString']`,
`map['valueOf']` and `map['hasOwnProperty']` all resolve. A `?? null` does not
catch that. `category` reaches this function from stored Firestore data (legacy
10-item-taxonomy values exist, per D12.2), so a malformed value could have
produced a function where a role belonged, and that value is what gets written
back as `role`. Now validated by VALUE (`=== 'vendor' || === 'manufacturer'`),
and `roleMapDrift()` uses `hasOwnProperty.call` for the same reason.

### 3. ONE combined confirm screen

Studied `useCategoryPrompt.ts` + `pick-category.tsx` first, as instructed. The
loop-avoidance pattern is a module-level listener set: the gate lives in the
root layout and is therefore a *different hook instance* from the screen, so
without an explicit notification the gate re-redirects the moment the screen
navigates away. Both replacements keep that pattern exactly.

- `app/(auth)/_lib/useCategoryPrompt.ts` → **`useBusinessProfileGate.ts`**.
  Fires when EITHER `category` or `role` is absent.
- `app/(auth)/pick-category.tsx` → **`confirm-business.tsx`**. Category list on
  top; the moment one is tapped, a role card appears below it, derived live from
  §9's map, with an explicit tick ("Yes — I'm a Manufacturer.") that must be
  ticked before Continue enables. Changing category re-arms the tick.
- One write, `setCurrentUserCategoryAndRole(category, role)`, replacing
  `setCurrentUserCategory`. The two-argument signature is the enforcement: §3
  rejects two sequential prompts, and this makes a category-without-role write
  impossible from this path.

**Decision D1.2 — renamed the route and the hook rather than extending them
under the old names.** `pick-category` and `useCategoryPrompt` would both have
become lies: the screen now collects two fields and the gate now fires on two
conditions. §19.2's "if it looks wrong but is listed, it's a decision" cuts the
other way here — nothing in `DECISIONS.md` names either file, and a misleading
filename is exactly how a future session "fixes" a combined screen back into a
category-only one. Deleted the old files rather than leaving them as a loaded
gun (D12.1's reasoning).

**Decision D1.3 — the tick is a confirmation, never a picker.** §7 says a
business "does not choose its own role freely; it is determined by category."
So there is no role selector anywhere in the app — not on the confirm screen,
not at signup, not in Business Details. What the user does is agree. For the
five "Both" categories the screen additionally *says so out loud* ("This
category can go either way. We've set the more common one.") — §9 calls those
five a first draft, and a confirmation step that hides which rows are guesses
isn't really asking.

**Decision D1.4 — the gate skips the `p/[slug]` segment.** A signed-in owner
opening their own shared public-profile link was previously yanked into the
migration prompt mid-view. §6a makes that route the one screen non-users see;
interrupting it is the wrong trade. Additive, one segment check.

### 4. `profile-setup.tsx` — same combined pattern

Category picker, then the same derived role card and tick, then district.
Validation rejects a missing role and an unticked confirmation separately, with
distinct copy. `logEvent('signup_completed')` now carries `role` alongside
category and district — still no name, per D14.6.

### 5. `firestore.rules` — the exact diff

New helpers, after `isNotSuspended()`:

```
+   function roleOf(uid) {
+     return get(/databases/$(database)/documents/users/$(uid)).data.get('role', '');
+   }
+   function isVendor() {
+     return isSignedIn() && roleOf(request.auth.uid) == 'vendor';
+   }
+   function isManufacturer() {
+     return isSignedIn() && roleOf(request.auth.uid) == 'manufacturer';
+   }
+   function roleMayCreatePostType(postType) {
+     return postType == 'requirement'
+       ? isVendor()
+       : (postType in ['sell-used', 'sell-new', 'rental'] && isManufacturer());
+   }
```

`match /listings/{listingId}`:

```
    allow create: if isSignedIn()
      && isNotSuspended()
      && request.auth.uid == request.resource.data.sellerId
+     && roleMayCreatePostType(request.resource.data.get('postType', ''))
      && underDailyCap('posts', postCap());
```

`match /users/{userId}/catalogItems/{itemId}` — the combined
`allow create, update` was SPLIT so only create carries the gate:

```
-   allow create, update: if isSignedIn()
-     && request.auth.uid == userId
-     && isNotSuspended();
+   allow create: if isSignedIn()
+     && request.auth.uid == userId
+     && isNotSuspended()
+     && isManufacturer();
+   allow update: if isSignedIn()
+     && request.auth.uid == userId
+     && isNotSuspended();
```

`match /profiles/{slug}`:

```
    function publicProfileFields() {
      return [
-       'ownerId', 'businessName', 'category', 'district',
+       'ownerId', 'businessName', 'category', 'role', 'district',
        'phone', 'whatsapp', 'photos', 'slug',
        'createdAt', 'updatedAt',
      ];
    }
    function isValidProfilePayload() {
      ...
        && request.resource.data.photos.size() <= 20
+       && (
+         !('role' in request.resource.data)
+         || request.resource.data.role in ['vendor', 'manufacturer']
+       );
    }
```

`requiredProfileFields()` is **unchanged** — see D1.5.

**Decision D1.5 — `role` is on the profile allowlist but NOT in
`requiredProfileFields`.** Requiring it would reject every edit to a profile
published before today, including the phone-change sync, which writes `phone`
alone and relies on the merged document satisfying the required list. That would
break precisely the accounts this migration serves. Pinned by two new tests.

**Decision D1.6 — an unknown `postType` is now DENIED.** `roleMayCreatePostType`
falls through to `false` rather than into one role's branch. §4 fixes exactly
four post types, so anything else is a typo or a hand-rolled write and should
not inherit either role's permissions. This is a tightening beyond what was
asked; it costs nothing and closes a gap the old rule had. Tested both roles.

**Decision D1.7 — `catalogItems` update/delete are deliberately NOT role-gated.**
The task said gate create; I gated only create. A Vendor who built a catalog
during the 2026-08-19→20 window when it was open to everyone can still correct
and remove those items, but cannot add more. Freezing existing items unreadably
in place would punish a business for a rule that changed under it, and a stale
uneditable item is worse for the trade than an editable one. Tested.

**Decision D1.8 — `profiles.role` is NOT cross-checked against
`users/{uid}.role`.** It is a display label on the public page; the permission
gates read `users` and never this document. Coupling the two would make a
profile uneditable for anyone mid-migration. It IS constrained to the two
literals, for the same anti-injection reason the notification rules constrain
their strings: this document is read by people with no account, so nothing on
it may be free text.

### 6. The six restored sites — all `role`, none `userType`

| Site | Restored as |
|---|---|
| `(tabs)/_layout.tsx` | Needs tab hidden unless `canSeeNeedsFeed(role)`. Uses `href: null` so the ROUTE stays registered — a deep link then hits `needs.tsx`'s redirect instead of a router error. |
| `(tabs)/needs.tsx` | Redirects non-Manufacturers to Available, and filters the feed to Vendor-posted requirements. Role check runs BEFORE the feed's own loading state, so a Vendor never sees a flash of the tab. |
| `(tabs)/post.tsx` | `intentOptionsForRole(role)`, plus a line naming the role so a Vendor seeing one option knows why. |
| `post/requirement.tsx` | Wrapped in the new `RoleGate` (Vendor-only). |
| `constants/postTypes.ts` | `intentOptionsForRole` / `postTypesForRole` restored, now SYMMETRIC, plus `roleMayPost` as the single source both the picker and the forms read. |
| `profile-setup.tsx` | Covered in step 4 above. |

Category logic in all six left untouched, as instructed.

**Decision D1.9 — I added client-side gates to the FOUR supply forms too, not
just `requirement.tsx`.** The task listed six sites because those are the six
`6f479b3` removed — but that commit was removing an *asymmetric* model, so
there was never a supply-side gate to remove. §0b's symmetry means a Vendor
deep-linking to `/post/sell-new` would otherwise fill in a whole form, upload
photos, and hit `permission-denied` at the write. New shared
`app/post/_components/RoleGate.tsx` explains the rule where the person hits it,
and is used by all five post surfaces (four supply + requirement + catalog).
A screen rather than a silent bounce: a bounce reads as the app losing your work.

**Decision D1.10 — `PRODUCT_CONTEXT.md` updated to match `DECISIONS.md`.** Its
§0, §1, §7 and §12 still argued the role split was correctly dropped. Left
as-is, the next cold session reads a *reasoning* document that contradicts the
*rules* document — the exact failure `PRODUCT_CONTEXT.md` §13 says the process
exists to prevent. Rewritten to carry the restore's reasoning (§0b's
service-vs-supply structure), with the 2026-08-19 argument preserved as history
rather than deleted.

### 7. `matching.ts` — Tier 1, role-aware

Added `canSeeNeedsFeed(role)`, `isVendorPostedRequirement(listing)` and
`filterNeedsFeed(items)`. `matchRank` / `rankListings` / `countMatches` are
**untouched** — §16 says only who the query runs for and what it surfaces
changes, and D12.5 (category outranks district) still stands.

**Decision D1.11 — a requirement with NO `sellerRole` stays VISIBLE.** §3 says
pre-migration listings are not backfilled, so every requirement posted before
today has no role tag. Filtering them out would blank the existing Needs feed on
the first app open after this lands — the same retroactive-erasure trap D13.3
avoided for `expiresAt`, and §3's own "fall back gracefully when absent" points
the same way. Only an explicit `'manufacturer'` is hidden, and the rules make
new ones of those impossible.

### 8. Both seed scripts

`w2d-app/scripts/seed.mjs`: `USER_SEEDS` now carries `{category, role}` per row,
each with a `// §9 row N` comment. **Six** of eight seeds are categorised now
(was four) — three Manufacturers, three Vendors — because requirement posters
must be Vendors and supply posters Manufacturers, and with only two of each a
third of the fixtures could never post anything. Users 7-8 stay bare, exercising
the absent-category AND absent-role paths. Listings draw their seller from the
role-appropriate pool; `sellerRole` denormalized; catalog items seeded only
under Manufacturers.

Verified against the emulator after re-seeding:

```
users with userType (want 0):             0
users with categories[] (want 0):         0
seed users manufacturer / vendor / null:  3 / 3 / 2
listings violating §4's role table:       0
requirements: 6, distinct posters: 3      (all three vendors)
supply listings: 19, distinct posters: 3  (all three manufacturers)
catalog owners: seed-user-1/2/3 only, all manufacturer
viewer seed-user-2 (mfr/Audio and Lighting/Coimbatore) bands:
  both:0 category:1 district:3 none:2     (unchanged from D12.8's figure)
```

**A defect the emulator check caught that reading could not:** the first
verification run showed catalog items owned by seed-users 4, 5 and 6 — all
Vendors. They were **stale documents from the previous seed distribution**, which
cycled all eight users. Every other seed document is idempotent because it
reuses its id, but a *re-owned* document is not: the re-run writes the new owner
and leaves the old copy behind. So the emulator was holding Vendor-owned catalog
items that §4 now forbids, and anyone hand-checking "catalog is
Manufacturer-only" would have found counter-examples the app cannot produce. The
seed now clears `seed-catalog-*` from all eight users before writing. Re-verified
clean.

`w2d-admin/scripts/seed-admin.mjs`: added three **admin-owned** role fixtures
(`admin-seed-manufacturer`, `admin-seed-vendor`, `admin-seed-norole`).

**Decision D1.12 — the admin seed gets its OWN fixture ids rather than merging
`role` onto `seed-user-*`.** That script's existing header explains it
deliberately does not write `category`, because those values belong to
`w2d-app/scripts/seed.mjs` and duplicating them across two repos invites drift
(the same risk audit item C5 flags for the category list). Merging a role onto
`seed-user-1` would do exactly that, and would conflict with the app seed's own
value. Distinct ids give the admin's role filter and role tile data even when
the app seed has not been run in that emulator — the admin repo has to be
verifiable on its own — with nothing to drift against.

### 9. Inverted verify-script assertions

`verify-admin.mjs` — the category-only MIX check now also asserts a role MIX,
that both roles are present, and that no seed user carries `userType`. That last
one is the part that must NOT be inverted: §0b is explicit the restore is not a
`userType` revival, so anything writing it again is a regression. Added a check
that no seeded listing contradicts §4's table. Wrote the full three-model
history into the comment, because this assertion has now been rewritten twice
and the next reader shouldn't have to guess which model it's on.

`verify-rules.mjs` — case 1 **inverted** (`expectAllowed` → `expectDenied`:
a Manufacturer may no longer raise a requirement). New cases 1b, 2b, 12, 12b
cover the supply and catalog directions, which no previous model gated at all.
Fixtures now carry a real `role`, and the "vendor" fixture's category moved from
`Stage Decoration` to `Catering` — §9 maps Stage Decoration to *Manufacturer*,
which was harmless while the label meant nothing and wrong now that it does.

**Decision D1.13 — case 3 (suspension) now suspends the VENDOR, not the
Manufacturer.** It suspended the Manufacturer and retried a requirement create,
which is now denied by role *as well as* suspension — so the case would have
passed while proving nothing about suspension. Each case is now arranged so
exactly one of the three possible denial reasons applies. Same reason case 4
switched to a supply doc.

**A second real defect, found by running rather than reading:** `verify-rules.mjs`
**hung forever on a PASSING run** and exited cleanly on a failing one — the worst
possible arrangement. `main()` had no `process.exit(0)`, and the Firebase JS SDK
holds open gRPC streams plus an Auth refresh timer, so node never exits. The
script's logic finishes in under a second; I only found this by instrumenting it
after `npm run verify` sat for 15 minutes. Added a `shutdown()` that signs out,
calls `terminate(db)` and exits 0. Also renumbered my catalog cases 5/5b → 12/12b:
the public-profile block already owned cases 5-11, and two different "case 5"s in
one output is how a failing case gets attributed to the wrong assertion.

### 10. Rules tests — all six combinations, plus the gaps around them

New group `role gate — all six combinations (§4, §7)` in `tests/rules.test.mjs`,
carrying the same three-model history comment. §14 item 0b's six cases, in order:

| # | Case | Result |
|---|---|---|
| 1 | Vendor creates requirement | ALLOWED |
| 2 | Vendor creates sell listing (all three types) | DENIED |
| 3 | Manufacturer creates every supply postType | ALLOWED |
| 4 | Manufacturer creates requirement | DENIED |
| 5 | Manufacturer creates catalog item | ALLOWED |
| 6 | Vendor creates catalog item | DENIED |

Plus, in the same group: an account with NO role denied all three directions; an
unknown `postType` denied for both roles; a Vendor forging
`sellerRole: 'manufacturer'` on its own payload denied (the gate reads
`users/{uid}`, never the document being written); a Vendor's leftover catalog
item still editable and deletable.

Also new: `tests/roles.test.ts` (`npm run test:roles`, 19 tests) covering the map
itself, intent filtering, and the Needs-tab who/what split. And in
`tests/flows.test.mjs`, a new group replaying both role journeys end to end —
this is where the "denied for the wrong reason" class of bug lives, since a
create can now fail for four different reasons.

**Decision D1.14 — `test:roles` asserts a FLOOR on each role's category count
(≥5 each), not exact counts.** §9 calls individual rows correctable, so pinning
12/17 would turn a legitimate founder correction into a test failure. What must
hold is that neither side collapses — a 29/0 split would silently disable half
the product.

### 11. Role made visible everywhere it belongs

Reveal screen (§6/§15 trust context — "the highest-trust moment in the app"),
requirement responders list, own Profile screen (badge + info row), public
profile page (§6a chip), public profile editor (read-only), Business Details
(read-only, with an explicit warning when a pending category change would move
the role). "My Catalog" row hidden for Vendors; "Add to Catalog" button hidden
for a non-Manufacturer owner.

**Decision D1.15 — no role badge on `ListingCard`.** Every card in Available is
Manufacturer-posted and every card in Needs is Vendor-posted, so the tab already
says it. A badge there would be noise on the densest surface in the app.
`sellerRole` is still stored — the Needs filter and Tier 2 need it.

### Testing pass — Task 1

Build/verify: `npx tsc --noEmit` clean · `npx expo export --platform android`
bundles (4.98 MB) · `firebase deploy --only firestore --dry-run` compiles.

| Suite | Before | After |
|---|---|---|
| `test:matching` | 15 | 15 |
| `test:expiry` | 20 | 20 |
| `test:ratelimit` | 6 | 6 |
| `test:roles` | — | **19 (new)** |
| `test:rules` | 60 | **70** |
| `test:storage` | 5 | 5 |
| `test:flows` | 37 | **40** |
| **total** | **143** | **175** |

Edge cases, empty states and error paths exercised beyond the six combinations:

| Case | Result |
|---|---|
| Account with `category` but NO `role` (the real migration population) | denied supply, requirement AND catalog — all three directions |
| Unknown `postType` (`'auction'`) from a Vendor / from a Manufacturer | denied / denied |
| Vendor forging `sellerRole: 'manufacturer'` on its own listing payload | denied — gate reads `users/{uid}`, not the payload |
| Vendor editing / deleting a catalog item created while the rule was open | allowed / allowed (D1.7) |
| Role gate leaking into `update` — Manufacturer reading a Vendor requirement, Vendor bumping counters on a supply post | neither blocked; §6's reveal mechanic intact |
| Owner still fully controls own post after the gate (status toggle, delete) | allowed |
| Suspended Vendor creating the one thing its role permits | denied — suspension, isolated from role |
| sellerId spoof using a post type the actor's role DOES permit | denied — ownership, isolated from role |
| `profiles.role` as free text (`'Wedding2day Official'`) | denied |
| `profiles.role` = `'manufacturer'` | allowed |
| Pre-2026-08-20 profile (no `role`) edited phone-only | allowed — D1.5's whole point |
| Same profile adding `role` later without a full rewrite | allowed |
| `roleForCategory` with undefined / null / `''` / legacy value / `'constructor'` | null in every case (the last one was a real bug) |
| Needs feed: requirement with no `sellerRole` / `null` / `''` | all stay visible (D1.11) |
| `filterNeedsFeed` on an empty feed, and input mutation | empty, no crash; input untouched |
| Both roles partition `INTENT_OPTIONS` exactly | no gap, no overlap |
| `intentOptionsForRole(null)` | `[]` — the intent screen shows explanatory copy instead of buttons |
| Seed re-run leaving Vendor-owned catalog items | fixed; re-verified clean |
| `verify-admin.mjs` | 5/5 PASS (`roled=6/8 vendor=3 manufacturer=3 legacyUserType=0`) |
| `verify-rules.mjs` | 18/18 PASS, and now actually exits |

Not verifiable in node (no RN runtime): the React screens themselves. Typecheck
plus a clean Metro bundle plus the emulator seed check is the available
verification, same as previous nights.

**Noted, not fixed:** `firestore.rules` still has a dead `capFor()` function
inside the `rateLimits` block — `firebase deploy` warns about it. Pre-existing,
unrelated to role, and I did not want an unnecessary edit in the most
security-sensitive file mid-task. Cleaning it up in Task 4, which is in that
file anyway.

---

## TASK 2 — w2d-admin UI for role

- `src/lib/types.ts`: new `BusinessRole`; `UserDoc.role?`; `ListingDoc.sellerRole?`.
  Also fixed two things the previous night's audit had flagged and left open:
  `UserDoc.userType` is now **optional** (audit C1 — it was required, which
  asserted something false of every post-2026-08-19 account) and `ListingDoc`
  gained **`expiresAt`** (audit C3 — without it an expired post looks approved
  and live in the moderation queue while being invisible to every user).
- `src/lib/categories.ts`: `BUSINESS_ROLE_MAP` + `roleForCategory` + `roleLabel`,
  with the same prototype-chain guard as the mobile copy, and a header saying
  plainly that this is the SECOND deliberate cross-repo duplicate.
- **Users screen**: `Role` column; role filter with four options (Vendor,
  Manufacturer, *No role (needs migration)*, *Role ≠ category default*);
  a "no role" count in the header; per-row badges for a role/category mismatch
  and for §9's five "Both" categories; two explicit `→ Vendor` / `→ Mfr`
  buttons.
- **Dashboard**: new "Roles (§7)" tile — `vendor / manufacturer`, and
  `N with NO role` when any remain.
- **Listings**: `Seller role` row in the moderation detail panel, with a
  `contradicts §4` badge when a post's type and seller role disagree.

**Decision D2.1 — the admin CAN change a role; the mobile app never can.** These
look contradictory and aren't. §7 says a *business* does not choose its own role,
which rules out a picker in the app. §9 says its five "Both" defaults are "a
first draft… correctable" per business — and an uncorrectable first draft isn't a
draft. Ops is the only place both can hold: a human weighs the case, and the
change is auditable. The confirmation dialog spells out that it changes what the
account can post, and when the target contradicts §9 it says so and points at
§17 for logging. It deliberately does **not** touch `category` — rewriting the
public identity (§6a) to fix a permission is the wrong lever, and it would
clobber the denormalized copies on that business's listings without the backfill
the mobile editor performs.

**Decision D2.2 — two explicit role buttons, not one toggle.** A toggle has to
guess a starting point for an account that currently has no role, and that guess
would be a permission.

**Decision D2.3 — the legacy `userType` badge now reads `legacy userType:
vendor`.** It sat unlabelled next to the new `Role` column, where two
same-valued badges would have been read as the same field. The badge stays
because it still marks pre-migration accounts (audit D2), but it now says which
field it is.

### Testing pass — Task 2

`npm run typecheck` clean · `npm run lint` clean (one pre-existing
`AuthContext` fast-refresh warning) · `npm run build` succeeds ·
`npm run seed:admin` writes all three role fixtures · `npm run verify` →
`verify-admin` 5/5 PASS, `verify-rules` 18/18 PASS.

Edge cases traced through the filter logic:

| Case | Behaviour |
|---|---|
| Account with role but no category | `Role ≠ category default` excludes it — nothing to compare against |
| Account with category but no role | excluded from `mismatch`, included in `No role` — the two never double-count |
| Account with a legacy 10-item category | `roleForCategory` returns null → not a mismatch, correctly |
| One of §9's five "Both" categories | `role default` badge, so ops knows the assignment is a guess |
| Listing with no `sellerRole` (pre-2026-08-20) | `— (pre-2026-08-20 post)`, not an error |
| Listing whose type and role contradict §4 | `contradicts §4` badge (only possible for a 2026-08-19→20 post) |
| Empty filter result | existing "No users match" row, colSpan corrected to 10 |

---

## TASK 3 — VERIFY Task 1 + 2 (gate before Phase 2)

### Full suite

`npm test` — **180/180 passing**, exit 0. `tsc --noEmit` clean.
`expo export --platform android` bundles. `firebase deploy --dry-run` compiles
with **no warnings** (see below).

| Suite | Tests |
|---|---|
| `test:matching` | 15 |
| `test:expiry` | 20 |
| `test:ratelimit` | 6 |
| `test:roles` | 19 |
| `test:rules` | **74** |
| `test:storage` | 5 |
| `test:flows` | **41** |
| **total** | **180** |

`w2d-admin`: typecheck / lint / build clean; `verify-admin` 5/5,
`verify-rules` 18/18.

### Trace A — a Vendor attempting every gated action

Traced in code, not through tests. Every row followed from the screen the user
would actually be on to the rule that decides it.

| Action | Path | Result |
|---|---|---|
| Open app with role set | `_layout` → `useBusinessProfileGate` → both fields present | no redirect |
| See the Needs tab | `(tabs)/_layout` → `canSeeNeedsFeed('vendor')` = false → `href: null` | tab button absent |
| Deep-link `/(tabs)/needs` | route still registered → `needs.tsx` → `<Redirect href="/(tabs)/available">` | bounced, not a router error |
| Browse Available | rules `listings` read = `isSignedIn()` | allowed — unchanged |
| Post tab | `intentOptionsForRole('vendor')` = `['requirement']`, plus "You're a Vendor" copy | one option |
| Post a requirement | `RoleGate` passes → `createListing` → `roleMayCreatePostType('requirement')` → `isVendor()` | allowed |
| Deep-link `/post/sell-new` \| `sell-used` \| `rental` \| `catalog` | `RoleGate` → `roleMayPost` false → explanation screen | blocked with a reason, form never renders |
| Profile → My Catalog | row filtered out by `MANUFACTURER_ONLY_ROWS` | absent |
| Deep-link own `/catalog/{uid}` | screen renders; "Add to Catalog" hidden; existing items editable/deletable | matches the rules exactly (D1.7) |
| Reveal a phone on a supply post | `interests` create — **no role gate** | allowed, and must be: this is the core trade flow |
| Answer another Vendor's requirement | `mayRespondToListing` → `isManufacturer()` | **denied — see the finding below** |
| Own requirement's responders list | `interests` query + `users` get | allowed |
| Public profile editor | `role` shown read-only, written from `users` | no picker anywhere |

### Trace B — a Manufacturer attempting every gated action

| Action | Result |
|---|---|
| Needs tab visible, feed filtered to Vendor-posted requirements | yes |
| Post tab | four options: sell-used, sell-new, rental, catalog |
| Post each supply type | allowed |
| Add a catalog item | allowed |
| Deep-link `/post/requirement` | `RoleGate` → explanation screen |
| Answer a Vendor requirement ("I can supply this") | allowed |
| Reveal a phone on another Manufacturer's supply post | allowed (§4/§6 do not restrict this; noted, not gated) |
| Tier 1 ranking on the Needs tab | unchanged — category still outranks district |

### FINDING 1 (fixed here) — §4's second half was not enforced

The Vendor trace turned up a real gap the six-combination tests could not,
because it is not one of the six. §4 states the requirement direction in **two**
halves:

> "requirement creation is Vendor-only again, **and responding to a requirement
> is Manufacturer-only**"

Task 1's brief covered gates on `listings` create and `catalogItems` create.
`interests` create — which is how a response is recorded (§6) — had **no role
gate at all**. So a Vendor reaching a requirement they don't own could tap "I
Can Supply This" and the write would succeed.

Reachability is low but real: the Needs tab is hidden from Vendors, so the paths
are a pasted `/listing/{id}` link, a shared link, or a stale back-stack entry.
Low reachability is not the point, though. **Shipping a documented
"Manufacturer-only" that isn't a permission is precisely the inaccuracy §0's
first correction found in the 2026-08-01 model** — the file said
Manufacturer-only and the reality was a hidden tab and a client redirect. Doing
that again, in the same release that fixes it, would be the wrong call.

**Decision D3.1 — fixed it, in rules and in the client, rather than logging it
for the morning.** It is a stated product rule in the same §4 sentence as the
gate I was asked to build; Task 3's brief says to trace and fix here rather than
carry a known-broken gate into Phase 2. Scope creep was the alternative risk and
I judged it smaller than the alternative.

Rules — new helper plus one added clause:

```
+   function isRequirementListing(listingId) {
+     return exists(/databases/$(database)/documents/listings/$(listingId))
+       && get(/databases/$(database)/documents/listings/$(listingId))
+           .data.get('postType', '') == 'requirement';
+   }
+   function mayRespondToListing(listingId) {
+     return !isRequirementListing(listingId) || isManufacturer();
+   }

    match /interests/{interestId} {
      allow create: if isSignedIn()
        && isNotSuspended()
        && request.auth.uid == request.resource.data.buyerId
+       && mayRespondToListing(request.resource.data.get('listingId', ''))
        && underDailyCap('reveals', revealCap());
```

**Decision D3.2 — `exists()` first, and a missing listing counts as "not a
requirement".** Two reasons. Product: an interest against a listing that isn't
there is meaningless either way, so gating it buys nothing. Practical: the
reveal-cap flow suite deliberately writes 25 interests against listing ids that
do not exist (it is testing the counter, not the listing), and a rule that
demanded `get().data` on an absent document would have failed that suite for a
reason unrelated to what it tests. Supply listings are untouched — a Vendor
revealing a Manufacturer's number is the product's main flow, not something to
gate. Pinned by four new tests, including the nonexistent-listing case
explicitly, so nobody "tightens" it later without seeing why it is loose.

Client half: `listing/[id].tsx` now reads the viewer's role and replaces the "I
Can Supply This" button with an explanation for a non-Manufacturer, rather than
offering a button the rules will refuse.

### FINDING 2 (fixed here) — a freshly-migrated business saw a stale UI

Tracing the migration flow end to end surfaced a genuine UX break that no test
would catch, because it is about component lifetime rather than permissions.

`(tabs)/_layout.tsx` and `(tabs)/post.tsx` each read `role` **once, on mount**.
The tab layout is mounted *before* the confirm screen runs — the gate redirects
out of the tabs and back into them. So a business that had just confirmed
"Manufacturer" would land in an app with **no Needs tab and no post options**,
and stay that way until it force-restarted. The one moment a role changes
mid-session is the migration, which makes it the one moment this matters.

**Decision D3.3 — two different fixes, matching each file's nature.**
`(tabs)/post.tsx` is a screen, so it moved to `useFocusEffect`, the pattern
`(tabs)/profile.tsx` already uses in this repo for exactly this reason ("edits
made on sub-screens show on return"). A layout has no reliable focus event, so
`useBusinessProfileGate` now also exports `subscribeBusinessProfileConfirmed()`
and the tab layout re-reads when the confirm screen reports a successful write.
It reuses the listener set that already existed for loop-avoidance rather than
adding a second mechanism.

This also fixes the smaller case of a category change in Business Details moving
the role while the Post tab sits mounted but unfocused.

### Trace C — the combined confirm screen, existing pre-migration account

Two populations, one screen, no branching. State: `category` present, `role`
absent (an account created 2026-08-19/20), or neither present (an account
predating the category migration).

1. `onAuthStateChanged` → `refresh()` → profile read → `!category || !role` is
   true → `needsConfirm = true`.
2. Redirect effect. While segments are `['(auth)','otp']` the gate stays quiet
   (`AUTH_FLOW_SEGMENTS`); once `otp.tsx` replaces to `(tabs)/available` the
   segments change, the effect re-runs, and the redirect fires. **Verified this
   works in either order** — if the read finishes after the navigation, the
   `ready` change triggers the same effect.
3. `confirm-business` renders with `gestureEnabled: false` — no back gesture, no
   skip, no dismiss. Correct: an account with no role can create nothing, so
   "remind me later" would leave someone in an app they cannot use.
4. Tap a category → role appears immediately from §9's map. For one of the five
   "Both" rows, the extra "this category can go either way" copy appears too.
5. Continue stays disabled until the tick. Changing category re-arms the tick.
6. Continue → `update({category, role})` — rules allow it (owner, and it touches
   neither `status` nor `verified`).
7. `notifyBusinessProfileConfirmed()` fires **before** `router.replace`, so the
   gate's own listener clears `needsConfirm` and cannot bounce the user back.
   This is the loop-avoidance pattern `pick-category` established, kept intact.
8. Tab layout re-reads via the new subscription (Finding 2) → Needs tab appears
   for a Manufacturer.

| Edge case | Behaviour |
|---|---|
| Neither field present | identical flow; the condition is `true \|\| true` |
| Category changed mid-flow | tick re-arms, Continue re-disables, role card updates |
| Write fails (offline) | error shown, stays on screen, no navigation, notify NOT called |
| Suspended account | `refresh()` returns early — suspension lockout wins, the two gates never fight |
| App foregrounded while on the screen | `refresh()` re-runs; the `confirm-business` segment check prevents a redirect loop |
| Signed out | `needsConfirm = false` |
| Profile read throws | fails OPEN (no prompt), matching `useSuspendedLockout`. Safe: rules still deny every create, so the worst case is a user who can browse but not post |
| Owner opens their own public profile link | `p` segment exempted — not yanked out of §6a's one public screen |

### Trace D — a brand-new signup

1. phone → otp → `verifyPhoneOtp`.
2. `otp.tsx`: `getCurrentUserProfile()` returns **null** (no document yet) →
   routes to `profile-setup`. Unchanged, and still correct.
3. Gate: at sign-in the profile was null, so `needsConfirm` is false and the
   gate never fires during signup. `AUTH_FLOW_SEGMENTS` covers it belt-and-braces.
4. `profile-setup` collects name → business name → category → **role card +
   tick** → district. Validation order matches visual order, and the role and
   the tick produce separate messages.
5. `createUserProfile` writes both fields in the one `set()`. A brand-new
   account can therefore never exist in the state the confirm screen repairs.
6. `logEvent('signup_completed', { category, role, district })` — still no name
   (D14.6).

| Edge case | Behaviour |
|---|---|
| Category chosen, tick not given | Continue blocked, "Please confirm you are a Manufacturer." |
| Category changed after ticking | tick cleared, must re-confirm |
| Signup abandoned after OTP, app reopened | profile is null → gate quiet → lands on the phone screen via `index.tsx`. Pre-existing behaviour, unchanged by this work |
| Returning user re-running signup | still routed to the tabs, so `createUserProfile`'s whole-document `set()` cannot orphan `profileSlug` (the Task 10 fix, intact) |

### Also cleaned up while in the rules file

Removed the dead `capFor()` function inside the `rateLimits` block — nothing
called it, and `firebase deploy` had been printing `Unused function: capFor` on
every run. I had flagged it in Task 1 and deliberately deferred it; a standing
warning in the security-rules file is noise that hides a real one. The dry run
is now completely clean.

### PASS / FAIL

**PASS.** Both role gates verified in rules and in the client, in both
directions, plus the third gate (requirement responses) that the trace found
missing and that is now closed. 180/180 tests, both repos build, both verify
scripts green. Two real defects found by tracing rather than by testing, both
fixed here rather than carried forward. Proceeding to Phase 2.

---
---

═══ PHASE 2 ═══

## TASK 4 — Close the phone-number enumeration gap (§6)

The most security-sensitive change of the night, and the one §6, Task 14 §E and
Task 16 §E all flagged as "much cheaper before there are real users' numbers in
the database than after."

### The gap, precisely

`users/{uid}` is readable by any signed-in account (`allow get: if isSignedIn()`)
and it held `phone`. Enumeration was already blocked in the 2026-08-19 run
(`list: isAdmin()`), but a harvester never needed enumeration: read the feed →
collect `sellerId`s off the listings → read those user documents one at a time.
§6's daily reveal cap sat entirely outside that path. It guarded a door with no
wall.

Tracing every read path also turned up **two more uncapped routes nobody had
listed**, both of which had to be closed for the fix to mean anything:

| Route | Status before |
|---|---|
| Walk `listings` → read `users/{uid}` | the known gap |
| **Requirement responders list** | rendered EVERY responder's number on load — no grant, no interest on the poster's side, no cap charge. "Post a requirement, collect the numbers" was free |
| **Catalog header** (`/catalog/{uid}`) | rendered the business's number the moment the screen loaded. §6 says "For **all** trade post and catalog types: tap to reveal" — this screen simply never did |

### The design

Contact details no longer live in a broadly-readable document.

| Piece | What it is |
|---|---|
| `users/{uid}/contact/info` | one document, one field (`phone`). Readable by the owner, an admin, or a grant holder. **Never listable** except by an admin |
| `revealGrants/{subjectId}_{viewerId}` | per-pair grant. Created by the viewer, **cap-gated**, immutable, undeletable |
| `users/{uid}` | keeps its broad `get`. It no longer contains a number, so a broad read is harmless |

The security property, stated as plainly as I can: **there is no path to another
account's phone number that does not go through a `revealGrants` document, and
minting one costs a reveal from today's allowance.** The cap is now on the only
route to a number instead of sitting beside it.

**Decision D4.1 — the grant rule deliberately does NOT also require proof of an
interaction.** An earlier draft required an `interests` document, the way the
notification rules do. I dropped it. Creating an interest is itself cap-bounded,
so an attacker willing to create one gains nothing — the extra clause bought no
additional bound, only two more `get()`s and more surface in the file where
surface costs most. Attribution survives either way: every grant is a document an
admin can read, so a harvesting pattern is still visible, which is the backstop
§15 chose. Written into the rule as a comment so it does not get "hardened" back
in without the reasoning.

**Decision D4.2 — `NO client-side fallback` to `users/{uid}.phone`.** The
tempting shape was: read the contact document, and if it is absent fall back to
the old field so nothing breaks during rollout. I did not do that. A fallback
makes the gap reachable again for every un-migrated account, which is the exact
thing being fixed — a hole that closes "eventually" is a hole. Un-migrated
subjects read as contact-unavailable, with honest copy, until the migration
reaches them. This is only defensible because the app is pre-launch; the note is
in the code.

**Decision D4.3 — grants are per (subject, viewer) PAIR, not per listing.** A
genuine behaviour change, and I checked the direction carefully against the "do
not weaken the cap" constraint. Old model: one charge per *listing* interest, so
25 charges bought *at most* 25 distinct numbers. New model: one charge per
*number*, so 25 charges buy exactly 25. A harvester's bound is unchanged at 25
distinct numbers a day; what changes is that an honest buyer browsing two posts
from the same supplier stops paying twice for a number they already have. The
cap is about harvesting numbers, so counting numbers is the more honest meter.

**Decision D4.4 — the requirement poster now pays per responder.** This is the
one real cost imposed on an existing flow. Reading your own responders list used
to be free and showed every number at once; it is now a "Show contact" tap per
responder, one reveal each. The alternative — minting grants for all responders
on load — would silently spend someone's allowance without asking. And leaving it
free would have left the harvesting route wide open: post a requirement, wait,
collect. §6's mechanic is tap-to-reveal; the list was the one place that ignored
it. Cost to an honest user: a business with more than 25 responders in one day
has to come back tomorrow, which I judged acceptable against the alternative.

**Decision D4.5 — the catalog header now needs a tap too.** §6 already said it
should ("for **all** trade post and catalog types"), so this is the code catching
up with the decision rather than a new restriction.

**Decision D4.6 — one cap charge per reveal, not two.** `expressInterest` mints
the grant inside the same charged action, and the grant's create rule only
*checks* the counter. Charging on both writes would have silently halved every
advertised cap — the mirror image of the `<` vs `<=` bug D14.2 caught. Pinned by
a flow test that replays the client's exact order and then asserts the counter.

**Decision D4.7 — admins get a collection-group read on `contact`.** Ops answers
support calls that arrive *by phone*, so `w2d-admin`'s Users screen has to be
able to search on the number, and doing it one document at a time is an N+1 per
page load. Restricted to `list` and to `isAdmin()`. This is convenience, not
capability: an admin already held `list` on `users` and `get` on every contact
document, so it could read the same data one request at a time today.

**Decision D4.8 — `revealGrants` reads are limited to the two parties (plus
admins), not "any signed-in account".** A grant holds only two uids, so the
tempting call is to leave it open. But grant ids are guessable — uids appear on
every listing — so an open `get` would let anyone probe "has A revealed B's
number", one request at a time. That is business-relationship metadata, and
leaking the reveal graph while fixing the phone leak would be a poor trade.

### The migration — and why it takes two mechanisms

Moving the read side is only half the job. The old field has to actually go, or
the grant system is a lock on a door that is still open. With Blaze blocked
(§8, §13) there is no Cloud Function to do it, and only an owner may write their
own document. So:

1. **`scripts/migrate-contacts.mjs`** (new) — one Admin-SDK pass over every user
   document: copy `phone` into the contact document, delete the field. This is
   what actually closes the gap. Idempotent, batched at 200 (each account costs
   two writes, and a batch caps at 500), and it **verifies afterwards** rather
   than assuming — it re-reads and reports how many documents still carry a
   `phone`, exiting non-zero if any do. `--dry-run` prints what would change
   with numbers **masked to the last four digits**: a migration log is not a
   place to print a full contact list, which is the exact data this change
   protects.
   Guarded in the reverse shape to `seed.mjs`: seeding production would be a
   disaster so that script refuses anything but an emulator, whereas this one is
   *meant* to run against production — but only with an explicit `--production`,
   because it deletes a field from every user document.
2. **`useContactMigration`** (new hook, mounted in the root layout) — the same
   move, owner-side, once per session, silently. Catches stragglers from an app
   build that predates the change. Deliberately invisible: unlike the §3
   confirmation, nothing here needs a human decision — the number is already
   known, it is just in the wrong place.

Verified on the emulator: 14/14 user documents migrated, then a re-run reported
`0 to migrate · 14 already done` and `0 documents still carry a phone field`.
Re-seeding afterwards produced 14 contact documents and 0 stray `users.phone`.

### Everything that had to move with it

| Site | Change |
|---|---|
| `createUserProfile` | writes the contact document; no `phone` on `users/{uid}` |
| `getCurrentUserProfile` | own phone now comes from `auth().currentUser.phoneNumber` FIRST — **zero extra reads**, and it is the authoritative source anyway (§8 keys the whole account on a verified phone) |
| `updateCurrentUserPhone` | writes the contact document, and deletes any stale `users/{uid}.phone` on the way past |
| `getSellerContact` | split. New `getSellerIdentity` (no phone) for screens that show a business *before* a reveal; `revealSellerContact` for the reveal itself |
| `revealContactPhone` | new — mints the grant if absent, then reads. Charges only when there is no grant, so D14.3 ("only a FIRST-time reveal is charged") is preserved exactly |
| `expressInterest` | takes `grantSubjectId`, mints the grant in the same charged action. Only for supply posts — on a requirement the direction reverses (§4) and the poster mints its own later |
| `getInterestedSuppliers` | identity only |
| `deleteCurrentUserFirestoreData` | deletes the contact document. **The most important addition to that list**: it holds the number, so leaving it behind keeps a deleted account's contact details in the database — the same failure the `profiles/{slug}` omission was, and the same thing a Play data-deletion review would find |
| `listing/[id].tsx` | reveal via `revealSellerContact`; per-responder "Show contact" with its own cap handling |
| `catalog/[uid].tsx` | identity on load, "Show contact" to reveal |
| `seed.mjs`, `seed-admin.mjs`, `verify-rules.mjs` | fixtures write contact documents, never `users.phone` |
| `w2d-admin` Users page | reads phones via one collection-group query, contact document winning over any stale `users.phone` |

`revealGrants` are **not** deleted on account deletion — they are undeletable by
rule (that immutability is what stops a grant being dropped and re-minted for
free), and once the user document and its contact document are gone a grant is a
pair of uids with no personal data left in it. Same reasoning D9.2 gives for
`interests`, and documented in the same place.

### The residual limitation, stated rather than implied

Exactly the one D14.2 already documents for every capped write: rules cannot
require two documents to change together, so **a client that skips the counter
increment can still mint grants**. Unclosable without a trusted server (Blaze,
§8/§13) or App Check. There is a test that asserts this explicitly — it mints a
grant without touching the counter and checks the counter is still zero — so the
next reader cannot over-read the guarantee from a passing suite.

What changed is real all the same: the cap is now on the only path to a phone
number instead of beside it, and the two silent uncapped routes are gone.

### Testing pass — Task 4

`npm test` — **209/209**. `tsc --noEmit` clean. `expo export` bundles.
`firebase deploy --dry-run` compiles clean. `w2d-admin` typecheck/lint/build
clean; `verify-admin` 5/5; `verify-rules` **23/23** including five new
grant cases run through a real Auth client.

| Suite | Before Task 4 | After |
|---|---|---|
| `test:rules` | 74 | **95** |
| `test:flows` | 41 | **49** |
| others | 60 | 60 |
| **total** | **175** | **209** |

Edge cases, abuse paths and error paths — all pass:

| Case | Result |
|---|---|
| `users/{uid}` still carrying a `phone` field | asserted absent — the test fails loudly if anyone adds it back |
| Read a contact document with no grant | denied |
| Read it with a grant | allowed |
| Grant used in REVERSE (subject tries to read viewer's number) | denied — grants are directional |
| Grant for one subject used on another | denied |
| Owner / admin / anonymous reading a contact document | yes / yes / no |
| `list` the contact subcollection as owner / as a grant-holding peer / as admin | denied / denied / allowed (D4.7) |
| Collection-group read as admin / ordinary user / anonymous | allowed / denied / denied |
| Non-owner writing a contact document, **even holding a grant** | denied — a grant is read access, never write |
| Contact document with an extra field, missing `phone`, or a non-string phone | all denied |
| Mint a grant with no counter document at all | denied |
| Mint a grant past the cap | denied |
| Mint a grant for a third party | denied |
| Grant id not matching its payload (`{other}_{me}`, or arbitrary) | denied |
| Grant naming yourself | denied |
| Grant naming a nonexistent account | denied |
| Grant with an extra field, or missing `viewerId` | denied |
| Suspended account minting a grant | denied |
| Editing or deleting an existing grant | both denied |
| Enumerating `revealGrants` | denied for everyone but an admin |
| Probing a grant as an unrelated ordinary account | denied (D4.8) |
| Account deletion removing the contact document | allowed, and included in the deletion batch |
| A **suspended** account deleting its own contact document | allowed (§14.2 — deletion must complete) |
| Anyone else deleting your contact document | denied |
| **The whole old harvesting route, replayed step by step** | feed read ✓, sellerIds collected ✓, user docs read ✓, numbers — **nothing** |
| Full reveal sequence in the client's exact order | one charge, one grant, number readable only after |
| Re-opening the same listing | no second charge; the re-write is a no-op by immutability |
| A second listing from the same seller | no second grant needed (D4.3) |
| **25 distinct subjects harvested, then a 26th** | 25 succeed, the 26th cannot be charged and its number stays unreadable |
| Requirement poster reading responders | identities yes, numbers only per-reveal per-charge |
| Catalog visit | catalog items readable, number not, until revealed |
| Deleted account whose grant survives | grant resolves to a nonexistent contact document, no crash |
| Skipping the counter increment | **still mints a grant** — the documented, unclosable limitation, asserted |

---

## TASK 5 — ToS role-model consistency pass

`app/legal/terms.tsx` was rewritten on 2026-08-19 for the *no-role* model, which
means it had scrubbed every trace of a role split. That is now wrong in the
opposite direction, and wrong in the way that matters most for a terms document:
**`role` decides what an account may post, and terms that do not say what an
account may post are not terms.**

Revised sections, all against §0b/§4/§7's actual text:

| Section | Change |
|---|---|
| 1 — What W2D is | New paragraph defining the two roles in the product's own service-vs-supply terms (§0b), stating that role follows from category and decides what you can post |
| 3 — Your account | Four new bullets: role comes from category and is confirmed, not picked; the five ambiguous categories get a default and we say so on screen; changing category can change role and we warn before saving; exactly one role, no dual registration (§10, §12) |
| 4 — Posting | Retitled "what your role allows". Spells out both directions explicitly, states that the limit is deliberate rather than a bug, and says it is **enforced on our servers, not just in the app** |
| 5 — Public profile | `role` added to the list of what becomes publicly visible |
| 6 — Connecting | Rewritten for §4's reversed requirement direction (only Manufacturers answer, only Vendors post), for reveals being per business rather than per post, and for responder numbers not being handed over in bulk |
| 11 — Deleting | `role` added to the profile line; a third retained-records bullet for reveal grants |

**Decision D5.1 — I added `role` to section 5's public-disclosure list, even
though the brief said §6a is "unaffected content-wise".** §6a's own text now
reads: "Scope of what's public: identity, contact, portfolio photos, AND now
role." So the *policy* is unaffected — the page is still open and still not
reveal-gated — but the *set of published fields* grew by one. A disclosure
section that omits a newly published field is precisely the failure D10.1 was
written to prevent, and the disclosure has to match `publicProfileFields()` in
`firestore.rules`, which now lists `role`. Added a doc comment tying the two
together so the next field added in one place is noticed in the other.

**Decision D5.2 — section 6 also absorbed Task 4's changes, which were not
strictly in this task's brief.** Task 4 changed what a reveal *is* (per business,
not per post), added a cost to reading your own responders list, and moved where
contact numbers are stored. Leaving section 6 describing the old mechanic would
have shipped, in the same release, terms that misdescribe the thing they exist to
describe. Three paragraphs, same section, same day's work.

**Decision D5.3 — the "who can do what" framing is stated as a real permission,
not a UI convention.** Section 4 says outright that it is "enforced on our
servers, not just in the app". §0's first accuracy correction found the
2026-08-01 model had documented a permission that was only ever a hidden tab —
now that it IS a permission, saying so is both true and the difference between a
term and a description of a screen.

### Testing pass — Task 5

`tsc --noEmit` clean. `expo export --platform android` bundles (a legal screen is
a data array, so a bundle is the real check that no string broke the file).
No test-suite change: nothing here touches logic, and the full suite was re-run
anyway — 209/209.

Cross-checks performed by reading, side by side:

| Claim in the ToS | Checked against | Result |
|---|---|---|
| "Manufacturers can post sell/rent and keep a catalog" | `intentOptionsForRole` + `roleMayCreatePostType` in the rules | matches |
| "Vendors can post requirements … cannot post items for sale or rent, and do not have a catalog" | same, both directions | matches |
| "Only Manufacturers can answer a requirement" | `mayRespondToListing` (Task 3, Finding 1) | matches |
| "enforced on our servers" | `firestore.rules` — a real create gate, not a redirect | true |
| Section 5's public list | `publicProfileFields()` in `firestore.rules` | exact match, all nine visible fields |
| "Revealing a number is per business, not per post" | `revealGrants/{subjectId}_{viewerId}` — per pair (D4.3) | matches |
| "you choose when to see each responder's number" | per-responder reveal (D4.4) | matches |
| "daily limit … applies to every way of getting one" | grant create is cap-gated on all three paths | matches |
| Section 11's deletion list | `deleteCurrentUserFirestoreData()` line by line | matches, including the new contact document |
| Section 11's retained list | rules: `interests` and `revealGrants` are both undeletable | matches |
| "we will change it" (role correction) | the admin Users screen's role buttons (Task 2, D2.1) | a real path exists |

**Flagged, not changed:** section 14's contact address is still a personal Gmail
(`(removed)`), which becomes the public developer contact. That
is the founder's call — it needs an inbox that exists — and it is already on the
Play Store list as item D4. Carried into Task 6's prep document rather than
silently swapped for a placeholder.

---

## TASK 6 — Privacy Policy rewrite + Play Store content prep

### 1–3. The legal texts — made ungenerateable-apart rather than re-synced

The brief was to rewrite `app/legal/privacy.tsx` and `docs/PRIVACY_POLICY.md`,
and to fix `docs/TERMS_OF_SERVICE.md` where it contradicts the shipped ToS. I did
all three, and then did one more thing, because re-syncing by hand would have
recreated the exact problem in a month.

**Decision D6.1 — the two `docs/*.md` files are now GENERATED from the same
content modules the screens render.** The audit's own framing pointed here:
"two ToS texts disagreeing is worse than one being out of date — whichever a
reviewer finds first is the one that counts" (B5), and `docs/PRIVACY_POLICY.md`
had drifted so far it still named the THREE-value Manufacturer/Decorator/Supplier
model superseded on 2026-08-01, two models before the one shipping (B4). Both are
symptoms of maintaining a legal text in two places by hand. So:

- text moved to `app/legal/_content/terms.ts` and `_content/privacy.ts` (plus a
  tiny `types.ts`, so the renderer can import content without pulling in React
  Native);
- the screens became 20-line renderers;
- `npm run docs:legal` writes both markdown files, with a "GENERATED FILE — do
  not edit by hand" header naming its source and the regenerate command;
- **`npm run test:legal` fails if the checked-in markdown has drifted**, pointing
  at the first differing line and the one command that fixes it.

Drift is now a failing test rather than a silent contradiction in a store
listing. That is the actual fix; the rewrite is just content.

A detail worth recording: the renderer had to become `.ts` rather than `.mjs`.
`sucrase-node`'s require hook resolves *static* imports of extensionless `.ts`
paths but not a runtime `import()`, so the first version failed with
`ERR_MODULE_NOT_FOUND`. Noted in the file so nobody converts it back.

**Privacy Policy content.** Rewritten from nothing, against the actual write
paths in `app/(auth)/_lib/`. It now covers, in order: who you are (including
`role`, and that it is derived rather than self-declared); what you create; the
public profile as its own prominent section mirroring ToS section 5; **what
happens to your phone number inside the app** (new, and necessary after Task 4);
crash reports and analytics; storage; what we don't do; deletion and retention;
your choices; changes; contact.

**Decision D6.2 — the policy discloses Firebase Analytics' AUTOMATIC collection
by name.** App-instance id, device model, OS version, approximate location
derived from IP, session timings. Our own events are deliberately PII-free
(D14.6), and it would have been easy to describe only those and look cleaner. But
the SDK collects that set whether or not we log a single event, the Data Safety
form has to declare it, and a policy that omits it while the form declares it is
the mismatch that gets an app pulled. Pinned by a test.

**Decision D6.3 — the policy says district is NOT a location reading.** Users
pick from 38 fixed options; there is no GPS and no location permission anywhere
in the merged manifest. Saying so plainly is worth a sentence, because "district"
in a data inventory reads like location data to a reviewer, and the honest answer
is better than the ambiguous one. Also pinned by a test.

`docs/TERMS_OF_SERVICE.md` is fixed by construction — it is now generated from
the same module as the shipped screen, so it cannot contradict it.

### 4. `WRITE_EXTERNAL_STORAGE` stripped — and two more found while looking

Added `expo.android.blockedPermissions` to `app.json`, then did what the audit
said to do and **actually read the generated manifest** (`npx expo prebuild -p
android`) rather than trusting the config. That is where it got interesting.

`WRITE_EXTERNAL_STORAGE` now carries `tools:node="remove"` — verified in the
file, not inferred. But the manifest also contained two permissions nobody had
listed, and neither is ours:

| Permission | Where it came from | Action |
|---|---|---|
| `SYSTEM_ALERT_WINDOW` | Expo's prebuild template, which seeds it under a comment reading "OPTIONAL PERMISSIONS, REMOVE WHATEVER YOU DO NOT NEED" | **blocked** |
| `RECORD_AUDIO` | same template | **blocked** |

The app draws no overlay and records no audio. `SYSTEM_ALERT_WINDOW` ("Display
over other apps") is one of the permissions Play scrutinises hardest — shipping
it unused and unjustified in the same submission where we carefully stripped
`WRITE_EXTERNAL_STORAGE` would have been an odd place to stop.

**Decision D6.4 — blocked both, and documented the one risk.** React Native's
own *debug* manifest declares `SYSTEM_ALERT_WINDOW` separately for the dev
overlay, and build-type manifests outrank the main manifest in Android's merger —
so a debug build should keep it while release builds do not, which is exactly
what we want. I cannot verify that here without a Gradle build (no Android SDK in
this environment), so the prep document names the exact one-line revert and says
to sanity-check the dev menu after the next rebuild. Blocking a Play-scrutinised
permission the app never uses was the better risk than leaving it in: a broken
dev menu is discovered instantly and reverted in one line, a rejection costs days.

**One audit claim corrected.** Task 16's permission table listed `CAMERA` as
shipping from `expo-image-picker`. It is **not** in the app manifest — but it *is*
in `expo-image-picker`'s own library manifest, and library manifests merge at
build time, so it does ship. That distinction matters: the app manifest is not
the final manifest, and `tools:node="remove"` working at all is the proof (it is
a merge directive). The prep document states which permissions come from where,
so the Play Console justification is written against reality.

I also deleted the generated `android/` directory afterwards. Leaving it makes
Expo treat the project as bare and changes what `expo start` does — that would
have quietly broken the dev loop. Prebuild also rewrote `package.json`'s
`android`/`ios` scripts to `expo run:*`; reverted, since this is a managed-
workflow project where `android/` is gitignored and EAS does the building.

### 5. Content-rating questionnaire answers

`docs/PLAY_CONTENT_RATING.md` — a submit-ready answer sheet, since I cannot fill
in a Play Console form.

Every factual answer was checked against code rather than assumed, and the
document says which were checked and how. The parts worth flagging:

- **Category: Utility/Productivity, not Social.** Choosing "Social" would pull in
  a much stricter question set that does not describe this app — there is no
  chat, no inbox, no follows, no comments (§10, and §18's social layer is still
  drafted and unbuilt). Picking the wrong category is the most likely way to end
  up answering questions about an app we did not build.
- **The user-generated-content section is answered fully, not minimised.** Users
  exchange personal information (a phone number) — that is the app's core loop,
  not a side effect, and it is declared as Yes. Moderation is declared as
  **manual human review before publication**, which is true and stronger than
  most apps can claim, and I deliberately did **not** claim automated or AI
  filtering, because there is none (§15: "Manual, no committed SLA").
- **On in-app communication**, the honest answer is nuanced: there is no in-app
  chat by design (§6, §10), but users do end up in contact. The document says to
  answer Yes if the form offers no nuance — overstating our controls is the worse
  error.
- **Target audience: 18 and over only**, and explicitly NOT opting into Designed
  for Families. A B2B app that exchanges adults' phone numbers should not be in
  Families policy, and the 18+ answer is what keeps it out.
- Notes that the expected outcome is a low *content* rating alongside an 18+
  *audience*, and that those are not in conflict — and that a rating coming back
  above "Teen" means something has been over-declared.

**Decision D6.5 — I did not change the personal Gmail.** It appears in the ToS,
the Privacy Policy and `support.ts`, and becomes the public developer contact.
Substituting a placeholder would have put a fake address into two legal documents;
substituting a guessed business address would be worse. It needs an inbox that
exists, so it is called out as the one open item the document cannot close, with
all three locations named and the regenerate command.

### Testing pass — Task 6

`npm test` — **226/226** (new `test:legal` suite, 17 tests). `tsc --noEmit`
clean. `expo export --platform android` bundles. `npx expo config --json`
confirms `blockedPermissions` resolves. `npx expo prebuild -p android` confirms
all three `tools:node="remove"` directives land in the manifest.

The new suite is not a formality — it asserts the things a reviewer or a Data
Safety form would care about and that no other test would notice:

| Case | Result |
|---|---|
| Checked-in markdown matches the content modules | pass, and reports the first differing line when it does not |
| Regeneration is byte-deterministic | pass — otherwise the drift check would flake and get muted |
| Both documents describe both roles | pass |
| The ToS states what each role may post, in BOTH directions | pass |
| Neither document mentions `userType` | pass |
| Neither document names the 3-value model (B4) | pass |
| Neither document describes a "self-declared" business type | pass — role is derived (§7) |
| Neither document still calls the app decoration-only (B1) | pass |
| Both mention the 29 categories | pass |
| Both disclose the public profile, incl. no-login, business name, **role**, WhatsApp, forwardable link, and catalog-never | pass — this is B3, the disclosure gap the audit called the most consequential |
| Both describe the daily reveal limit and per-business reveals | pass |
| The policy discloses that reveals are RECORDED and the phone is stored separately | pass (new, Task 4) |
| The policy discloses Crashlytics, Analytics, IP-derived location, device model | pass (D6.2) |
| The policy distinguishes district from a GPS reading | pass (D6.3) |
| No-ads / no-payments / no-selling claims present | pass, and verified against the dependency tree |
| Both describe deletion, retention, and the can't-receive-the-code fallback | pass |
| Retained-records list names all three kinds incl. reveal grants | pass |
| Both state 18+ and give a contact address | pass |
| Headings numbered from 1, no gaps or repeats | pass — a renumbering slip would silently break a cross-reference |
| No empty block, no TODO/TBD/FIXME/lorem placeholder | pass |

---

## TASK 7 — Signup tagline

`app/(auth)/phone.tsx` — the first line a new user ever reads.

**Before:** "Buy & sell wedding decoration materials"
**After:** "Tamil Nadu's wedding trade — source what you need, supply what you
make"

Two things were wrong with the old line, and the audit (item B6) only named the
first:

1. **Decoration-only.** Wrong for 28 of the 29 categories §9 locks. A caterer or
   a photographer reads it and concludes the app is not for them — the cheapest
   possible way to lose a signup.
2. **"Buy & sell" describes only half the mechanic.** The demand side is not a
   secondary feature: `PRODUCT_CONTEXT.md` §3 calls `requirement` "the single
   most important design element in the trade layer — it's what stops the app
   being dead on arrival". A tagline that only mentions supply undersells the
   half that works when the catalog is thin.

**Decision D7.1 — "source what you need, supply what you make" rather than
naming the roles.** It carries both directions of §7's split, and therefore both
halves of the mechanic, without asking someone to learn a role model before they
have an account. "Wedding trade" carries the 29 categories without listing them,
and "Tamil Nadu's" is the region §1 fixes. Kept to one line: this sits above a
phone field on the pre-signup screen, so it is a tagline, not a value
proposition. The reasoning is in a comment beside it, because a one-line string
is exactly the kind of thing a future session "tidies" without knowing what it
had to fit.

Also swept the rest of `app/` and `constants/` for surviving "decoration
materials" / "wedding decoration" copy — nothing left outside the comment I just
wrote and the legal texts, which Task 6 already rewrote.

### Testing pass — Task 7

`tsc --noEmit` clean; `expo export --platform android` bundles; `npm test`
226/226 (a copy change should move nothing, and it moved nothing). The
`test:legal` suite independently asserts that no legal text still calls the app
decoration-only, so the same mistake now has a test behind it in the two places
it did the most damage.

---

## TASK 8 — VERIFY Phase 2 (gate before the UI revamp)

### Full suite

`npm test` — **226/226**, exit 0. `tsc --noEmit` clean.
`expo export --platform android` bundles. `firebase deploy --dry-run` compiles
with no warnings.

| Suite | Tests |
|---|---|
| `test:matching` | 15 |
| `test:expiry` | 20 |
| `test:ratelimit` | 6 |
| `test:roles` | 19 |
| `test:legal` | 17 |
| `test:rules` | 95 |
| `test:storage` | 5 |
| `test:flows` | 49 |
| **total** | **226** |

`w2d-admin`: typecheck clean, lint clean (one pre-existing `AuthContext`
fast-refresh warning, unrelated), build succeeds, `verify-admin` 5/5,
`verify-rules` 22/22.

### Phase 1 regression check

Checked directly rather than inferred from a green suite, because Phase 2 edited
the same rules file and the same screens:

| Phase 1 guarantee | Still there? |
|---|---|
| `roleMayCreatePostType` on `listings` create | yes, line 478 |
| `isManufacturer()` on `catalogItems` create | yes, line 356 |
| Both role helpers present and used | yes, 8 references across the file |
| The six-combination test group | present, all six passing |
| Six restored client sites still role-aware | all six carry role logic |
| Combined confirm screen still the only migration path | yes |
| `test:roles` map/drift guard | 19/19 |

Nothing from Phase 1 regressed.

### FINDING (fixed here) — a flake I introduced in Task 4

`w2d-admin`'s `verify-rules.mjs` **passed the first time and failed on every run
after**. This is exactly what Task 8 is for, and the suite being green did not
catch it — the emulator-clearing rules suite has no equivalent problem, because
it calls `clearFirestore()` before each test.

The cause is a property of Task 4's own design. Reveal grants are **undeletable**
by rule, and that immutability is load-bearing: a grant that could be dropped and
re-minted would be a second phone number for free, and the cap would stop
bounding anything. So a script that mints one cannot clean up after itself.

Two cases depended on the grant *not* existing:

| Case | Problem | Fix |
|---|---|---|
| 13b — "no grant, no number" | read the same subject case 13c grants, so after one run the number was legitimately readable and the assertion was wrong, not the rule | added a third fixture, `rules-ungranted@…`, that exists solely to be a subject nobody ever holds a grant for |
| 13c — "mint own grant" | a re-`setDoc` on an existing grant is an `update`, correctly denied | the mint is now conditional, and the comment states plainly what that costs: on a re-run it verifies the READ but not the CREATE |

**Decision D8.1 — a conditional mint, with the gap named, rather than a
weaker rule or an Admin-SDK bypass.** The tempting fixes were both wrong:
allowing grant `update` would undo the immutability the whole cap bound rests on,
and minting via the Admin SDK would stop exercising the create rule at all. The
create IS verified from scratch on every run, by `w2d-app/tests/rules.test.mjs` —
cap gate, id/payload match, self-grants, nonexistent subjects, extra fields,
suspension — because that suite clears Firestore per test. This script's distinct
value is exercising the same rules through a real Auth client, and the read
assertion does that unconditionally. Both halves of the reasoning are in the file
rather than only in a commit message.

Worth noting this is the **second** run-order flake in that one script; the first
was the bare `users.size === 8` assertion the previous night fixed. It is the
failure mode that gets a verification script quietly ignored, which is why the
fix is documented in place both times.

Verified idempotent afterwards: four consecutive runs, all exit 0, 22 cases PASS.

### PASS / FAIL

**PASS.** 226/226 tests, both repos build, both verify scripts green and now
genuinely repeatable, Phase 1 intact, rules compile clean. One flake found and
fixed. Proceeding to Phase 3.

---
---

═══ PHASE 3 ═══

## TASK 9 — Full UI/UX revamp (§14 item 7)

Two commits: the design system plus the highest-value screens, then the rest.

### The starting point, stated precisely

There WAS a `DESIGN_SYSTEM.md`, written 2026-08-01, and most of its content was
good — the palette, the Facebook/Instagram/WhatsApp pattern mapping, the
cards-versus-rows distinction. What it did not have was anything enforcing it:
`tailwind.config.js` had an **empty `theme.extend`**, so every screen hardcoded
its own hex values, and the file's own opening rule ("never invent a new color")
was followed unevenly.

By last night the codebase held about a dozen near-duplicate greys and browns —
`#201a19` beside `#1d1b1a`, `#534341` beside `#56413e`, `#857371` beside
`#8a716c`, `#d8c2bf` beside `#ddbfb9`. Those are not design decisions, they are
copy-paste history. Alongside them:

| | Before |
|---|---|
| Buttons | 40–56px tall, four pressed states, three disabled treatments |
| Inputs | three heights, two focus treatments (one being none), error text in a different place per screen |
| Bottom-sheet picker | reimplemented five times |
| Status pills (My Listings) | five hand-picked colour pairs, none from the palette |
| `getInitials` | two copies — flagged in `DESIGN_SYSTEM.md` on 2026-08-01, with "promote this the next time either file is touched" |
| Loading | a bare `<ActivityIndicator>` |
| Empty | one grey sentence |
| Error | one grey sentence and an unstyled "Retry" — visually identical to empty |
| Tab bar | active tab signalled by colour alone |

**Decision D9.1 — build the system first, then apply it, in that order.** The
alternative (restyle screens one at a time and extract patterns as they recur) is
how you end up with a system that describes what happened rather than one that
decides it. This is the second attempt at a design system for this app; the
first became a document nobody could enforce, and the difference is that this
one is code.

### 1. Tokens — `tailwind.config.js`

Semantic COLOUR roles rather than shades: `brand`/`brand-tint`/`brand-wash`,
`canvas`/`surface`/`inset`/`sunken`, `ink`/`ink-secondary`/`ink-muted`,
`line`/`line-strong`, `danger`/`success`/`warning` each with a tint, and
`external-whatsapp`. Three ink levels, not six — the drift replaced six shades
doing the work of three.

A TYPOGRAPHY scale (`display` → `micro`) **with line heights baked in**. That is
not tidiness: React Native does not inherit line-height, so a size without one
renders at the platform default, and the legal screens and listing descriptions
were rendering cramped for exactly that reason.

RADII named by what they wrap (`field`/`card`/`sheet`/`pill`), and two spacing
additions (`gutter`, `touch` = 48dp).

**Extended rather than replaced**, deliberately, so `text-sm`/`text-base` kept
working and the migration could go screen by screen without a broken
intermediate state.

**Verified the tokens actually resolve**, rather than assuming: bundled the app
and grepped the compiled output for `#8a5000`, `#ffeeda`, `#e3f3e8`, `#c22d1c`,
`#fff3f0`, `#1da851` — six values that appear nowhere as literals in the source.
All six present. A misspelled Tailwind class produces nothing silently, so this
was worth checking before migrating 25 screens onto it.

### 2. Components — `app/_components/ui/`

`Button` (5 variants) · `Badge`/`BadgeRow` · `Card`/`RowGroup`/`Row`/`InfoRow`/
`SectionLabel`/`Divider`/`Hint` · `Field`/`TextField`/`SelectField`/
`ReadOnlyField`/`Checkbox`/`Chip` · `Screen`/`ScreenHeader`/`ScreenFooter`/
`Sheet`/`SheetOptions` · `Avatar` · `LoadingState`/`EmptyState`/`ErrorState`/
`InlineMessage`/`Skeleton`.

**Decision D9.2 — `ui/colors.ts` exists, and it is the ONE place a raw hex is
allowed.** Three things in this stack cannot read a class: `@expo/vector-icons`
takes a `color` string, `ActivityIndicator` takes `color`, and React
Navigation's `Tabs`/`Stack` options take style objects. Before this file, each of
those carried its own literal — which is how a palette drifts even when a token
system exists. The file says out loud that it mirrors `tailwind.config.js` and
must be changed with it.

**Decision D9.3 — `ReadOnlyField` is a separate component from a disabled
`TextField`.** They look similar and mean opposite things: one is a value being
SHOWN, the other a control you cannot use. That distinction earns its keep on
every screen where a derived value (a role, a verified phone) sits beside
editable fields — rendering it as a greyed-out input invites a tap that does
nothing.

**Decision D9.4 — kept `DESIGN_SYSTEM.md`'s cards-versus-rows rule and encoded
it in the component names.** Bounded items are `Card`s; ongoing state is a
`RowGroup` of `Row`s. It is the Facebook-feed versus WhatsApp-chat-list
distinction, it carries meaning, and standardising on one for both would flatten
a real signal. Now the type system nudges toward the right one.

### 3. Every screen

All 25. Verified by audit, not by memory: every screen file either imports from
`_components/ui` or is a file with nothing to style (`index.tsx` is a one-line
`Redirect`; the four post-type files are `RoleGate` + `PostForm` wrappers;
`terms.tsx`/`privacy.tsx` delegate to `LegalScreen`; the two `_layout` files are
Stack config).

**Zero hardcoded hex values remain anywhere in `app/`, outside `ui/colors.ts`.**

The changes worth naming, in rough order of how much they matter:

**`ListingCard` — the most-seen surface in the app.**
- The photo now LEADS. It was third, under the business header and the title, so
  every card opened with two rows of small text. A feed of physical goods is
  scanned by image.
- Identity moved BELOW the content. On a marketplace you decide whether an item
  is relevant before you care who is selling it; leading with the business meant
  every card opened with the least discriminating information on the screen.
- The match badge is no longer solid green — it was the only saturated green in
  the app and read as a "success" state on a card where nothing had succeeded.
- `₹45000` → `₹45,000`.
- A photo-less listing is a designed panel with an icon, not a grey rectangle
  indistinguishable from an image that failed to load.
- Requirements skip the photo block entirely (§4 — they have no photos), instead
  of rendering an empty box on every card.

**The reveal card (`listing/[id].tsx`) — §6's "highest-trust moment".**
It was a bordered box with a name, a business name, three grey chips and a phone
number in the same weight as everything else. Now: an avatar; §6/§15's full trust
context as proper badges with verification in the success tone; the NUMBER as the
largest thing on the card, because it is the one piece of information the card
exists to deliver; and one line saying where it came from and that W2D is not
part of what happens next (§6 — we connect and step aside, and saying so is more
trustworthy than implying otherwise).
Also: the five facts became a two-up labelled grid instead of dot-separated runs
that broke at arbitrary points on a narrow screen, and the responders list moved
out of the pinned footer, where Task 4's per-responder reveals had made it
unreachable past the second entry.

**The public profile (`p/[slug].tsx`) — the one screen non-users see.**
§6a makes this a link handed to a couple who has never heard of Wedding2day, so
it is doing a website's job for a business that probably has no website. It was a
left-aligned wall of text ending in the literal string "No photos yet." Now: a
centred identity header with an avatar; contact as its own card, because that is
the point of the page; portfolio as a two-up grid so five photos read as a body
of work; a DESIGNED empty portfolio (the businesses least likely to have uploaded
photos are exactly the ones who most need the page to work); and a footer
explaining what Wedding2day is, because a stranger arriving from a forwarded
WhatsApp link had no idea.

**Decision D9.5 — the public profile footer is NOT an install prompt.** The
obvious move is "get the app". §1 and §10 rule out couple-facing use, so
inviting a couple to download a B2B trade app would be inviting them into a
product that is not for them. It explains instead.

**Empty states, everywhere, and there are now more of them than screens.** The
Available feed has three (nothing exists / nothing matches your search / nothing
matches your filters) because they need opposite advice — the old single grey
sentence told a user with an over-narrow filter the same thing it told a user on
a genuinely empty marketplace. Needs has four (the fourth being "no matches
right now", which offers "show all requirements"). My Listings differs by tab.
The catalog differs by whether you own it.

**The confirm screen (§3) — a blocking screen, so treated as one.** Someone is
FORCED through this, on an app they were already using, to answer a question they
did not ask. It gained a SEARCH over the 29 categories (a blocking screen is the
worst place to make someone scroll hunting for "Psychiatrist for Marriage
Counseling"), visible radio buttons so single-select is obvious before the first
tap, and a footer that says what is still needed rather than just greying out
Continue with no explanation.

**Two copy fixes that were really trust fixes.**
- `otp.tsx` said "Secure verification powered by Wedding2day Trust" next to a
  padlock emoji. That product does not exist. Inventing a trust brand is the
  opposite of trustworthy. It now says something true and actually reassuring:
  that verifying confirms the account is yours, and the number is not shown to
  anyone until you choose to share it.
- The suspended screen had Sign out as the filled primary button and Contact
  support as the outlined one — emphasis on leaving rather than on the one action
  that can resolve the situation. Inverted.

**Accessibility, picked up along the way rather than as a separate pass:** the
tab bar now fills its icon when focused (it relied on colour alone, which is the
one signal a colour-blind user does not get); every button is at least 48dp
(several were 40); `Checkbox` makes the box and label one tap target; `hitSlop`
on the small icon buttons; `accessibilityLabel` on every icon-only control.

### What I did NOT do, per the brief's item 5

Logged as suggestions rather than done, because each is a flow change rather
than a visual one:

1. **A photo lightbox on the public profile.** Tapping a portfolio photo does
   nothing. A full-screen viewer is the obvious next step and it is new
   behaviour, not styling.
2. **A date picker for `neededBy`.** It is a free-text `YYYY-MM-DD` field. The
   expiry logic already tolerates a malformed value (D13.2 pins that), but a
   picker would remove the class of mistake entirely. New component, new
   dependency question.
3. **Skeleton loaders on the two feeds.** `Skeleton` exists and is unused. The
   feeds still show a centred spinner. Using it would be a genuine improvement
   and needs a device to tune against — a skeleton that does not match the real
   layout is worse than a spinner.
4. **Draft autosave is still not surfaced.** §15 lists "post forms autosave
   locally on-device" and I could not find an implementation. Worth confirming
   whether it exists at all; if not, it is a §15 gap rather than a UI one.
5. **Pull-to-refresh on the detail screens.** Only the feeds have it.
6. **The five "Both" categories can only be corrected via support** (D2.1). The
   copy says "contact support and we'll change it", which is honest but manual.
   An in-app request would be a flow addition.

### Testing pass — Task 9

`npm test` — **226/226**, unchanged from Task 8. That is the signal the brief
asked for: a visual pass should not move a single test, and it did not.
`tsc --noEmit` clean. `expo export --platform android` bundles at every one of
the six checkpoints I took during the migration.

| Check | Result |
|---|---|
| Full suite before and after | 226 / 226 — identical |
| `tsc --noEmit` | clean |
| `expo export --platform android` | bundles |
| Token classes actually resolve | verified in the compiled bundle, six token-only colour values |
| Hardcoded hex remaining in `app/` | **0**, outside `ui/colors.ts` |
| Screens on the design system | 25/25, audited file by file |
| Data model touched | none |
| Business logic touched | none |
| Navigation structure touched | none |
| Permission gating touched | none |

Things I checked specifically because a "visual" pass is where they get broken
by accident:

| Case | Result |
|---|---|
| Needs-tab role gate (redirect + `href: null`) | intact — the redirect still runs before the feed's loading state |
| `filterNeedsFeed` still applied to the feed | yes |
| `RoleGate` on all five post surfaces | yes |
| Reveal flow: `expressInterest` grant subject, cap handling, rate-limit branch | unchanged; only the presentation of the result moved |
| Per-responder reveal handler and its cap error path | unchanged, relocated out of the footer |
| `mayRespondToRequirement` gate on the respond button | intact |
| Catalog Add button still gated on `role === 'manufacturer'` | yes |
| "My Catalog" row still hidden for Vendors | yes |
| Confirm screen: tick re-arms on category change; `notifyBusinessProfileConfirmed` still fires BEFORE navigation | both intact — the second one is the loop-avoidance the whole gate depends on |
| PostForm validation order and the data written | unchanged |
| Public profile still makes exactly one unauthenticated read | yes — no `auth()` call added |
| Legal content still rendered from the generated modules | yes; `test:legal` 17/17 |

**Not verifiable here:** how any of it actually looks. There is no device or
emulator screenshot in this environment, so every visual claim above is reasoned
from the code rather than seen. That is the main caveat on this task — the
structure, spacing and token usage are verifiable; the aesthetic result needs
your eyes on a build.

---
---

# FINAL SUMMARY — morning review (night of 2026-08-20 → 2026-08-21)

Everything below is verified state, not intent. Final run: `tsc --noEmit` clean,
**226 tests passing**, `expo export --platform android` bundles,
`firebase deploy --dry-run` compiles with no warnings. `w2d-admin`: typecheck,
lint and build clean; `verify-admin` 5/5; `verify-rules` 22/22 and now
repeatable. Both repos committed and pushed clean.

## What completed

| Task | Status |
|---|---|
| 1 — Restore the role split (revised) | **Done.** `role` on `users`/`profiles`, `sellerRole` on `listings`, §9's 29-row map verbatim, symmetric rules gates, ONE combined confirm screen, all six restored sites, Tier 1 revision, both seed scripts, inverted verify assertions, all six rule combinations tested |
| 2 — w2d-admin role UI | **Done.** Role column, four-option filter incl. "no role" and "role ≠ category default", Dashboard tile, moderation-panel role with a "contradicts §4" flag, and an admin-only role correction path |
| 3 — Verify Phase 1 | **PASS.** Two real defects found by tracing and fixed here — see below |
| 4 — Phone-enumeration gap (§6) | **Done.** Contact details behind a per-pair reveal grant; two further uncapped routes found and closed; migration script plus in-app fallback |
| 5 — ToS role-model pass | **Done.** Six sections revised, every factual claim cross-checked against code |
| 6 — Privacy Policy + Play prep | **Done.** Policy rewritten; both `docs/*.md` now GENERATED so they cannot drift again; three dead permissions stripped; content-rating answer sheet written |
| 7 — Signup tagline | **Done.** |
| 8 — Verify Phase 2 | **PASS.** One flake of my own found and fixed |
| 9 — Full UI/UX revamp | **Done.** Real design system; all 25 screens; zero hardcoded colours left |

Tests went from **143 → 226**. Two new suites (`test:roles`, `test:legal`).

## The six things most worth your attention

**1. §4's response half was never a permission — now it is.** §4 says
requirement creation is Vendor-only "**and responding to a requirement is
Manufacturer-only**". Only the first half was ever enforced. A Vendor reaching a
requirement via a pasted `/listing/{id}` link could answer it. Found by tracing
every gated action in Task 3, not by a test — it is not one of the six
combinations §14 item 0b lists. Fixed in rules and in the client. This is the
same class of inaccuracy §0's first correction found in the 2026-08-01 model, and
I was not willing to ship it again in the release that fixes it.

**2. The phone-enumeration gap is closed, and two more like it were closed with
it.** The known route (walk `listings` → read `users/{uid}`) is gone: phone
numbers no longer live in a broadly-readable document. While tracing every read
path I found two uncapped routes nobody had listed — the requirement responders
list handed over every responder's number on load, and the catalog header handed
over one the moment the screen opened, both with no charge against §6's cap.
Both are tap-to-reveal now. **This is only fully closed in production once you
run `npm run migrate:contacts -- --production`.**

**3. Two bugs found by testing rather than by reading.** `roleForCategory
('constructor')` returned `Object.prototype.constructor` — a truthy *function* —
because a plain object literal inherits from `Object.prototype`, and a `?? null`
does not catch that. It is reachable because `category` comes from stored data
and legacy values exist. Separately, the seed left Vendor-owned catalog items
behind when catalog changed owners, so the emulator held data §4 forbids and a
hand-check of "catalog is Manufacturer-only" would have found counter-examples.

**4. A freshly-migrated business saw a stale app.** `(tabs)/_layout.tsx` and
`(tabs)/post.tsx` read `role` once on mount, and the tab layout is mounted
*before* the confirm screen runs. So a business that had just confirmed
"Manufacturer" landed in an app with no Needs tab and no post options, and stayed
that way until it force-restarted. No test would catch this — it is about
component lifetime, not permissions. The migration is the one moment a role
changes mid-session, which makes it the one moment it mattered.

**5. `w2d-admin`'s `verify-rules.mjs` was hanging on a PASSING run.** No
`process.exit(0)`, and the Firebase JS SDK holds open gRPC streams — so a passing
run never returned and a failing one exited cleanly, which is the worst possible
arrangement. The previous night's "all 14 cases PASS" was almost certainly read
from a run that had to be interrupted. It now exits. (And Task 8 then caught a
second, separate flake I had introduced in Task 4.)

**6. The legal texts can no longer drift.** `docs/TERMS_OF_SERVICE.md` had come
to *contradict* the shipped ToS screen, and `docs/PRIVACY_POLICY.md` still named
the three-value business-type model superseded two models earlier. Both are now
generated from the same content modules the screens render, and
`npm run test:legal` fails if the checked-in markdown has drifted. That removes
the class of problem rather than one instance of it — which matters because one
of those files is going to be a public URL in your store listing.

## Decisions I'd most like you to review

All are logged inline above with reasoning. These are the ones where a different
call is most defensible.

1. **D4.3 — reveals are now per BUSINESS, not per listing.** Checked carefully
   against "do not weaken the cap": 25 charges used to buy *at most* 25 distinct
   numbers and now buy exactly 25, so a harvester's bound is unchanged. The
   change is that an honest buyer browsing two posts from one supplier stops
   paying twice. I think this is right, but it is a real semantic change to §6's
   meter.
2. **D4.4 — the requirement poster now pays per responder.** The only real cost
   imposed on an existing flow. Reading your own responders list used to be free
   and showed every number at once; it is now one reveal per responder. It closes
   a genuine harvesting route ("post a requirement, wait, collect"), but a
   business with more than 25 responders in a day now has to come back tomorrow.
3. **D4.1 — the grant rule does NOT require proof of an interaction.** An earlier
   draft did. I dropped it because creating an interest is itself cap-bounded, so
   it added no bound — only more rule surface in the file where surface costs
   most. If you would rather have the belt-and-braces, it is a three-line
   addition and the reasoning is in the rules file.
4. **D2.1 / D9.5 — role corrections are an admin action.** §7 says a business
   does not choose its own role; §9 says its five "Both" defaults are
   correctable. Ops is the only place both hold, so the admin can change a role
   and the app has no picker. The user-facing copy says "contact support and
   we'll change it", which is honest but manual.
5. **D3.1 / D5.2 / D6.4 — three places I went slightly beyond a task's brief**
   (enforcing §4's response half, absorbing Task 4's changes into ToS §6, and
   blocking two dead permissions beyond `WRITE_EXTERNAL_STORAGE`). Each is
   argued at the point it happens. If you disagree with any, they are all
   independently revertible.
6. **Dark mode is structured for but not switched on.** The palette is semantic
   and no screen hardcodes a colour, so it is a token swap. I did not enable it
   because it needs looking at on a device and a half-converted dark mode is
   worse than none.

## What I could NOT verify, and you should

**How any of the UI actually looks.** There is no device or emulator screenshot
in this environment. Every visual claim in Task 9 is reasoned from the code:
structure, spacing, token usage and the absence of hardcoded colours are all
verified, but the aesthetic result is not. This is the single biggest caveat on
the night's work. Please walk the app on a build before judging it.

Also unverifiable here, all for the same reason (no native build / no Android SDK):

- The `SYSTEM_ALERT_WINDOW` block. React Native's *debug* manifest declares it
  separately and build-type manifests outrank the main one, so a debug build
  should keep it while release builds do not — which is what we want. If the dev
  menu misbehaves after the next rebuild, the revert is one line in `app.json`
  and it is documented in `docs/PLAY_CONTENT_RATING.md`.
- Anything requiring a real OTP round-trip.

## What you must do before this can ship

**Before the app runs correctly:**

1. **Run the contact migration.** `npm run migrate:contacts` for the emulator
   (already done here), and `npm run migrate:contacts -- --production` against
   production. **Until that runs, §6's enumeration gap is closed in code but not
   in data.** The script is idempotent, batched, and verifies itself.
2. **Deploy rules and indexes:** `firebase deploy --only firestore`. Tonight's
   rules changes are substantial — the symmetric role gates, the reveal-grant
   model, the contact subcollection — and none of it is live until this runs.
3. A fresh EAS dev-client build is **not** newly required tonight — no native
   module was added. The three from the previous run (messaging, crashlytics,
   analytics) still need the rebuild that was noted then.

**Before Play Store submission** — unchanged from the previous audit except where
noted:

1. **Reviewer sign-in path.** Firebase test phone number + Play Console → App
   access. Still the most commonly fatal item, and still nothing in the repo can
   do it.
2. **A hosted Privacy Policy URL and an account-deletion URL.** The text is now
   ready and generated; it needs somewhere to live. One hosting decision solves
   both, plus §6a's real browser URL.
3. **Store assets**: 512×512 icon, 1024×500 feature graphic, ≥2 screenshots.
4. **Content rating + target audience**: answers are written and ready to submit
   in `docs/PLAY_CONTENT_RATING.md`. You submit; I cannot.
5. **Data Safety form**: inventory unchanged, and the Privacy Policy now matches
   it. Note that phone, business name, district, category, role and portfolio
   photos are **publicly accessible with no account** for any business with a
   public profile — that is "made public", not "shared with third parties", and
   understating it is what gets an app pulled after launch.
6. **The support contact.** Still a personal Gmail in the ToS, the Privacy Policy
   and `support.ts`, and the support WhatsApp number still carries a `TODO`. It
   needs an inbox and a number that exist. When you have them, all three change
   together and `npm run docs:legal` regenerates the two documents.

## Suggested next steps

1. **Look at the app.** Walk every screen on a build. That is the one thing
   tonight's work cannot self-check, and Task 9 is the task where it matters
   most.
2. **Run the two deploys above** (rules, then the contact migration) so the
   security work is actually in effect.
3. **Review decisions 1–4** in the list above. Each is a small, isolated change
   if you want it different.
4. **Correct any of §9's role assignments that are wrong.** The five "Both" rows
   are founder-assigned guesses by your own note; the app now says so on screen,
   and the admin can fix an individual business. If a whole row is wrong, it is
   one line in `constants/businessRoles.ts` — plus the same line in
   `w2d-admin/src/lib/categories.ts`, which is a deliberate cross-repo duplicate
   and flagged as such in both files.
5. **Six UI restructuring ideas** are logged under Task 9 as suggestions rather
   than done, per the brief. The two I would actually pick up: a photo lightbox
   on the public profile, and skeleton loaders on the feeds (the `Skeleton`
   component exists and is unused).
6. **`§15`'s draft autosave** — listed as a decided feature ("post forms autosave
   locally on-device") and I could not find an implementation. Worth confirming
   whether it exists; if not, it is a §15 gap rather than a UI one.
7. **`PRODUCT_CONTEXT.md` and `DECISIONS.md` were both updated tonight** —
   `PRODUCT_CONTEXT.md` still argued the role split was correctly dropped, which
   would have left the reasoning file contradicting the rules file, and
   `DECISIONS.md` §6 still said the enumeration gap was unfixed. Neither edit
   changes a product decision; both are noted at the top of the files. The new
   §17 rows are worth a read.

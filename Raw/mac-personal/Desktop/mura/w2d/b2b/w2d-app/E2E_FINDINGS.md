# W2D — E2E findings, overnight run 2026-09-01

Consolidated from Batches 1–10 of `OVERNIGHT_RUN_PROMPT.md`. One file, per
Batch 11's instruction not to scatter these. Every entry states how it was
verified rather than asserting it — several of the reported premises turned out
to need qualifying, and two findings below were not in the batch list at all.

- **Fixed this run:** 9
- **Found and NOT fixed (out of scope, recorded for the human):** 4
- **NEEDS HUMAN DECISION:** 4 (§6a phone exposure is the flagged one)

`OVERNIGHT_RUN_LOG.md` carries the per-batch narrative and the reasoning behind
each deviation. This file is the defect list.

---

## FIXED

### F1 — `users.role` was self-writable *(CRITICAL)* — Batch 1
`roleOf()` in `firestore.rules` is the input to all six §7 role gates, and
nothing constrained its value: `create` validated no fields, and the
owner-update allowlist named only `status`/`verified`. Any account could write
itself the other role and walk through every gate. The gates were real
permissions; their input was self-service.

**Fix:** a written `role` must equal the role §9 locks to the `category` on the
same document. Not a flat block — see D1 for why one is impossible.
**Verified:** 11 tests in `rules.test.mjs`, both directions plus junk values.

### F2 — the §9 role table had no drift guard across its copies — Batch 1, Batch 9
F1's fix forced a second hand transcription of §9's 29 rows into
`firestore.rules`. Auditing that turned up a **fourth** copy in
`w2d-admin/src/lib/categories.ts`. All copies agreed on the day, but nothing
enforced it — and the rules copy decides what Firestore accepts, while the
admin copy drives the Users screen's role-mismatch filter.

**Fix:** two guards, one per repo, each parsing the other file as text so it
cannot re-derive the answer from the thing it is checking. Both mutation-tested
by injecting a wrong row and confirming failure.

### F3 — `interests` doc id was unbound from its payload — Batch 2
`{listingId}_{buyerId}` is load-bearing: `expressInterest()` dedupes by reading
that exact id, and interests are immutable by rule so one response stays one.
Unbound, the same payload could be rewritten under one fresh id per known
listing — each passing dedupe, each charging another `interestCount` increment
and another responders-list row, replaying one permitted response as many.

**Fix:** mirrors the id-binding `revealGrants` already used.
**Verified:** 4 tests, including the replay exploit run end to end.

### F4 — `postType` was mutable after creation — Batch 3
§7's role gate runs on create only, so this defeated all of it: create the one
type your role allows, then edit it into one it does not. Everything downstream
believes the field — which tab the post lands in, whether answering it is
Manufacturer-only, which form ever validated its shape.

**Fix:** immutable on update, **for admins too** — nothing writes it after
creation (`PostForm.tsx:236` sets it once; no listing-edit screen exists;
`w2d-admin` reads it in three screens and never writes it).

### F5 — `phone` could be re-added to `users/{uid}` — Batch 4
The rules file has carried "DO NOT ADD `phone` BACK TO THIS DOCUMENT … and no
test would fail" since 2026-08-20. That was exactly true: a comment, not a
permission. `users/{uid}` is `get`-open to every signed-in account, so one
phone field turns `listings` back into a phone book (§6).

**Fix:** adding or changing a `phone` is refused; **deleting one is not**, so
the two `FieldValue.delete()` migration calls keep working and an unmigrated
account can still edit its own profile.

### F6 — three tests were asserting F5's hole *(not in the batch list)* — Batch 4
Found by running the suite, not predicted. `flows.test.mjs` had three tests
writing `phone` onto `users/{uid}` and asserting it **succeeded** — a shape the
app stopped producing on 2026-08-20. Rewritten against
`users/{uid}/contact/info`, which is what `updateCurrentUserPhone()` writes.

A fourth, `re-running signup is rejected outright for a verified account`, still
*passed* — but its `assertFails` would now trip on the phone field before
reaching the `verified` check it exists to prove. Field removed so the assertion
still isolates its own subject.

### F7 — the Available feed had no role filter — Batch 5
§5 fixes the Available tab as "Manufacturer-posted only" and nothing enforced it
on the read side. Rows this removes: supply listings posted by Vendors during
the 2026-08-19–20 window when §4's supply gate was not yet a real permission.

**Fix:** `filterAvailableFeed`, the exact mirror of the existing
`filterNeedsFeed`. **Client-side, deliberately** — see D2.

### F8 — `confirm-business.tsx` had no pre-fill and no backfill — Batch 6
Three defects in the mandatory migration screen:
- started at `null` and read nothing, so the larger of its two populations
  (accounts that have a category and only lack a role) had to re-pick from 29
  rows, with a mistake silently rewriting their category;
- wrote only `users/{uid}`, leaving `sellerCategory`/`sellerRole` stale on every
  listing and on the public profile. For 2026-08-19–20 accounts those listings
  have **no `sellerRole` at all**, and this screen is the first moment one can
  be written;
- a failed read left a blank form the user could submit.

**Fix:** pre-fill (answer, not consent — §3 requires the tick either way);
backfill ordered *before* the authoritative write so a partial failure retries
rather than stranding the copies; fail-closed with a retry on the screen.

### F9 — a pre-existing flake in `verify-admin.mjs` *(not in the batch list)* — Batch 9
`A2.1` asserts ≥1 pending listing, but the same script approves and rejects
pending listings further down and never restores one, so a second run against
the same emulator failed — while `A4.1`'s own reset fallback sat forty lines
later and never ran. It bit during this run. Fixed by self-healing; verified
idempotent over three consecutive runs.

---

## FOUND, NOT FIXED (out of scope — Run Rule 6)

### N1 — `updateBusinessDetails()` does not backfill the `profiles` document
The profile editor re-denormalizes onto `listings` but not onto the public
profile, so changing your category leaves the **public** page showing the old
one. This is the same gap Batch 6 just closed on the confirm screen, in the
sibling write path. Batch 6's scope was `confirm-business.tsx`; touching the
editor would have been out of it. `backfillSellerDenormalization()` in
`firestore.ts` is written to be reusable when someone does pick this up.

### N2 — the public profile link is not a browser URL
`buildProfileShareUrl()` returns `w2d://p/${slug}`, which only opens for people
who already have the app — while §6a's whole purpose is a page shareable "to
anyone, including couples, outside the app". The code says so honestly and the
UI does not over-promise, so this is a known limitation, not a defect.

**It is closer to solvable than it was this morning:** Batch 7 configured
Firebase Hosting for the legal pages, so `wedding2day-a99ea.web.app` now exists
as a real origin. Serving `/p/{slug}` from it is a separate piece of work (an
Expo web build, or a small static renderer reading the public `profiles` doc)
and a product call, not a bug fix.

### N3 — reviews and GST verification are documented but not built
§15 describes full reviews and an optional GST field for the verification
badge. Neither exists: no `reviews` collection in `firestore.rules`, no UI
writes a `gstNumber`, and the `profiles` rules actively reject that field. Not a
bug — but it is the same *class* of thing §0's first accuracy correction caught,
so it is recorded rather than left to be rediscovered. `PLAY_DATA_SAFETY_ANSWERS.md`
depends on this staying true.

### N4 — the cap can still be exceeded by a client that skips the increment
Unchanged, unclosable without a trusted server or App Check, already documented
in §6 and asserted explicitly in a test. Listed only so this file is a complete
picture rather than a partial one.

---

## NEEDS HUMAN DECISION

### D1 — §6/§6a: the public profile publishes a phone number, reveal-free *(the flagged one)*
**Not fixed, deliberately — this is Batch 11's explicitly flagged item.**

§6 spent an entire overnight pass closing the phone-enumeration gap: contact
details moved out of `users/{uid}` into a subcollection behind a per-pair reveal
grant, so the number of phone numbers any account can obtain in a day is bounded
by the reveal cap. §6a then publishes the same number on a page that needs **no
login at all** — `profiles` has a public read rule and `app/p/[slug].tsx` is
reachable anonymously.

Both are correct as written. §6a says so in terms: *"Contact info is VISIBLE
DIRECTLY on the public profile — NOT reveal-gated. Deliberate, unchanged. Do not
'harmonize' with §6 without asking."* §12 records reveal-gating the public
profile as considered and rejected.

**What the human is being asked, precisely.** The reveal cap bounds harvesting
through the *trade* layer only. The showcase layer's bound is different: a slug
is not guessable, and `profiles` is `get`-open but never `list`-open (asserted
by tests, and by `verify-rules.mjs` case 9), so there is no enumeration route —
harvesting requires already holding the links. That is a real bound, and it may
well be the right trade for a product whose founder interviews said the dominant
pain is "not enough business". But it is a **product** decision about how much
reach is worth how much exposure, and Run Rule 8 says an agent does not take
those. Three ways it could go — all of them the human's call:
1. **Leave it.** Reach is the point; the slug is the bound. (Current state.)
2. Keep the page public but make the *number* tap-to-reveal for signed-in
   viewers while staying open to anonymous ones — splits the two audiences.
3. Reveal-gate it, which §12 has already rejected once.

**No code was changed for this.** It is listed here so it is a decision on
record rather than an inconsistency someone rediscovers.

### D2 — should the Available-feed role filter become query-level?
Batch 5 asked for a query-level filter and got a client-side one. Firestore
cannot express "equals X *or* field missing" in a `where`, and §3 does not
backfill `sellerRole`, so a query filter would silently empty the feed of its
entire pre-2026-08-20 back catalogue — and because the emulator does not enforce
indexes (§13.4), that breakage would not have shown up locally.

Going query-level needs, in order: a `sellerRole` backfill migration over
existing listings (same shape as `scripts/migrate-contacts.mjs`), then a
composite index deployed with `firebase deploy --only firestore`. Worth doing if
the feed grows; not worth doing blind.

### D3 — should the app hard-block at launch when the profile read fails?
Batch 6 asked `confirm-business.tsx` to fail closed. It now does, at the screen.
The **gate** (`useBusinessProfileGate`, in the root layout) no longer fails open
either — it simply does not decide on a failed read, so a network blip never
clears a known-needed confirmation and never yanks a migrated user into a
blocking screen.

Making the root layout render a hard block instead is a product call about
offline behaviour, and DECISIONS.md says nothing about it. Not taken.

### D4 — `W2D_Test_Plan.xlsx` does not exist
Batch 10's input artifact is missing. A search of the **entire home directory**
to depth 6 for `*test*plan*` / `*Full_Test*` returned exactly one hit — an
unrelated `test_plan_item_validator.yml` inside a VS Code Python extension. It
is in neither repo, in no `docs/` or `docs/archive/`, and referenced nowhere but
`OVERNIGHT_RUN_PROMPT.md` itself. Either it lives somewhere this machine cannot
see (another device, or cloud storage), or it was never created. `TEST_SHEET_RUN.md`
delivers the batch's stated purpose with its own `TL-` numbering, prefixed so it
can never be mistaken for TC-001–TC-048. **If the real sheet exists, it still
needs running.**

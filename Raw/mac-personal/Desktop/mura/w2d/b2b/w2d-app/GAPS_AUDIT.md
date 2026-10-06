# W2D — Gaps Audit (beyond the feature roadmap)

> Everything in `BUILD_PLAN_v1.2.md` is feature work. This is the other
> category: things a "proper" shipped app needs that aren't features —
> infrastructure, hardening, launch mechanics. Found by auditing the actual
> code against standard React Native/Expo launch practice and marketplace-
> app conventions, not guessed.

Created: 2026-08-01

---

## Critical — more urgent than any feature in the roadmap

### 1. No git history — the single biggest risk to the whole project

`git log` shows exactly one commit: the initial scaffold. Every real
feature built since — weeks of work, everything in tonight's overnight
run — exists only on your Mac's disk. No backup, no history, no way to
recover if the machine has a problem. This was already flagged as "biggest
remaining infrastructure risk" back in July and is still exactly true.
**Fix this before anything else, including before the demo** — it's a 10
minute task (`git add -A && git commit` and push to a private GitHub repo)
that removes an existential risk, not a feature.

### 2. Listings feed has no pagination

`fetchListings()` in `firestore.ts` has no `.limit()` — it fetches every
document in the `listings` collection, every time, on every load. Fine at
25 seeded listings. Not fine at 500 real ones — slow to load, and every
load costs a Firestore read per document. Needs cursor-based pagination
(`limit()` + `startAfter()`) before real usage, not before Play Store
submission specifically — this is a cost/performance issue, not a policy
one.

### 3. Terms of Service is a link, not a consent record

Standard marketplace onboarding (confirmed via research) uses a checkbox
at signup — "I agree to the Terms" — not just a tappable link. Tonight's
Phase D.2 makes the link actually work, which is a real improvement, but
there's still no record that a given user *agreed* to anything, just that
the text existed somewhere they could have read. Worth a real checkbox +
a `termsAcceptedAt` timestamp on the user profile before this scales past
friends-and-family testing.

---

## Important — pre-launch hardening, not yet scheduled anywhere

### 4. Zero automated tests
No unit or integration tests anywhere in the project. Given this session
alone found two real regressions from manual review (`@expo/vector-icons`
resolution, the tsc-scoping issue in Task 1.1) that automated checks would
have caught immediately, this is a real ongoing risk as the app grows —
not urgent tonight, but worth planning for once the core feature set
stabilizes.

### 5. No crash reporting or analytics
Already named in `DECISIONS.md` §14 roadmap item 6 ("Ops hygiene") but
worth re-flagging: standard launch practice treats "crash-free performance
confirmed via real crash reporting" as a hard pre-submission requirement,
not a nice-to-have. Currently there's no visibility into what breaks for a
real user after install.

### 6. No image compression before upload
Photos go straight from the camera/gallery to Firebase Storage at full
resolution via `uploadListingPhoto`. Modern phone cameras produce
multi-megabyte images — this means slow uploads on average Indian mobile
data and higher Storage costs. A simple client-side resize/compress step
(e.g. via `expo-image-manipulator`) before upload is standard practice,
not currently done anywhere.

### 7. No deep-link sharing of a listing
Worth calling out specifically because this app is WhatsApp-centric by
design — a Vendor being able to share a direct link to their own listing
*into* a WhatsApp chat (outside the app) is a natural, nearly-free growth
mechanic for exactly this product, and nothing like it exists yet. Not in
the original roadmap at all. Consider adding.

### 8. No account recovery path
If a user loses access to their WhatsApp/phone number (new SIM, lost
phone, etc.), there's currently no documented recovery flow beyond
`DECISIONS.md` §8's phone-number-change feature (Task 10.1, requires the
user to already be signed in to change it) — that doesn't help someone
who's already locked out. Worth a support-mediated recovery path at
minimum (manual, via the support contact already built).

### 9. No Play Store listing assets prepared
Screenshots, feature graphic, short/long description, promo text — none
of this exists yet and none of it is code. Needs to happen before
submission regardless of the closed-testing timeline; can be prepared in
parallel with dev work rather than waiting.

### 10. Rate limiting / spam prevention on posting
Already named in `DECISIONS.md` §14 item 6 — no throttle currently exists
on how often an account can post. Low risk pre-launch with few real users,
becomes real once the app has any traction.

---

## Explicitly not gaps — deliberate, already-confirmed decisions

Listing these so they don't get "rediscovered" and treated as missing
work:

- **Admin dashboard = Firebase Console only.** Standard marketplace advice
  treats a real admin panel as day-one essential — `DECISIONS.md` §11
  already knows this and deliberately deferred it (admin web app,
  sequenced last). Not an oversight, a stated trade-off.
- **No in-app payments/chat.** Core, repeatedly-confirmed product decision
  (`DECISIONS.md` §6, §10) — not a gap.
- **No multi-language support.** English-only is locked (`DECISIONS.md`
  §15) — not a gap, a scope decision.

---

## Suggested priority, once tonight's demo build is done

1. Git backup (today, unrelated to any feature work — do this first)
2. Pagination on `fetchListings` (before any real usage, not launch-gated)
3. Image compression (same — cost/performance, not launch-gated)
4. ToS consent checkbox + timestamp
5. Crash reporting (before Play Store submission)
6. Play Store listing assets (prepare in parallel, not blocking dev)
7. Deep-link sharing (growth feature, worth considering — your call)
8. Account recovery path, rate limiting, automated tests — as the app
   gains real users, not urgent for a demo or early testing

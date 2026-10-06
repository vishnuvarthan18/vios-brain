# W2D — Product Context

> Companion to `DECISIONS.md`.
> `DECISIONS.md` = the rules (what to build, what not to).
> This file = the reasoning behind them (why they are what they are).
>
> If you only need to know what to do, read `DECISIONS.md`.
> Read this when you need to understand *why*, or when considering a change.

Last updated: 2026-08-19 (deep-audit clarification pass — see DECISIONS.md §17)

---

## 0. WHY TODAY'S CHANGES HAPPENED — READ THIS FIRST

`DECISIONS.md` §0 documents what changed today. This section is the *why*
behind the two substantive changes (the accuracy corrections in that same
§0 are just fixing inaccurate documentation — nothing to reason about there,
the code simply didn't match the file).

**The trigger:** development had been paused because the founder suspected
the product's direction was wrong, ahead of any further building. Two
things happened next, in order:

1. **Desk research** (AI-run, external to this repo) into the wedding-decor
   B2B trade space found that the original reveal-phone, no-chat/payment/
   booking mechanic matches most documented reasons B2B marketplaces fail
   to retain users after first contact, and that direct competitors already
   run the same mechanic nationally. It recommended validating with real
   interviews before building further, rather than guessing at a fix.

2. **Founder-led interviews** — actually conducted, with real decorators,
   manufacturers, and suppliers — found something the desk research hadn't
   tested for: **the dominant real pain across all three groups was "not
   enough business,"** not sourcing friction, not difficulty finding
   suppliers, not difficulty comparing them.

**Why "not enough business" changes the product, not just the marketing:**
a tool that only helps a decorator find and compare manufacturers does
nothing for "not enough business" — it routes *existing* demand, it doesn't
create new demand for anyone. The trade layer as originally conceived
(§1–§9 area of `DECISIONS.md`) cannot fix this pain by itself, no matter how
well it's built.

**What followed from that, and why each piece specifically:**

- **The public profile/showcase layer (`DECISIONS.md` §6a) exists because
  it directly answers "not enough business."** It gives every business a
  self-promotion tool — a link they can hand to a bride's family at a
  wedding, or post in their own WhatsApp status — that generates value the
  moment they sign up, independent of whether any other business is using
  the trade layer yet. This also happens to solve the classic cold-start
  problem (the app is worthless to the very first user) without adding
  chat, payments, or booking — none of which are being introduced.
- **Contact info on the public profile is visible directly, not
  reveal-gated (unlike the trade side).** The profile's entire purpose is
  to be shared *outward*, to people who aren't inside the app at all. A
  reveal-tap only makes sense when the app is mediating a connection
  between two of its own users — it actively works against a profile whose
  job is to be handed to an outsider. This is a deliberate difference
  between the two mechanics, not an inconsistency to fix.
- **Category widening to all 29 wedding-trade categories (`DECISIONS.md`
  §9) exists because this repo's own draft (§17–§18, dated 2026-08-04) had
  already independently proposed the same widening, for a related but
  distinct reason: the trade is a supply chain across the whole wedding
  industry, not just within decor (a photographer buying a used lens from a
  wholesaler is the same shape of transaction as a decorator buying a
  backdrop from a manufacturer).** That earlier draft left its own final
  category list unlocked. The founder supplied the actual 29-item list
  directly, and it's now locked verbatim in `DECISIONS.md` §9.
- **The Vendor/Manufacturer role split (§7 as it stood before today) was
  dropped, not adjusted.** That draft had also flagged, as its own open
  item, whether the role split still makes sense once categories widen past
  decor — and the honest answer is no. There's no clean "raises
  requirements vs. fulfills them" split across all 29 categories the way
  Vendor→Manufacturer mapped onto decor sourcing specifically. Forcing an
  awkward mapping (e.g. deciding whether a photographer is a "vendor" or a
  "manufacturer" relative to a caterer) would have been worse than dropping
  the split and using the category field descriptively instead.

**What did NOT change today:** the social layer (likes, comments, follow,
auto-joined category communities) and the 3-tab Feeds/Market/Profile nav
redesign, both proposed in that same 2026-08-04 draft, remain exactly where
they were — parked, drafted, not built, not rejected. They weren't part of
what the interviews or the desk research pointed at, and adopting them today
would have been scope creep riding along with a change that had real
evidence behind it. See `DECISIONS.md` §18.

---

## 1. What the product is, in one line

A two-layer platform for the Tamil Nadu wedding trade industry: a B2B trade
marketplace where any registered business (across 29 wedding-trade
categories) can buy, sell, rent, or resell equipment and materials, plus a
public, no-login, shareable profile page for every business.

### How the scope got here — full history, not just the latest version

**Original v1 (earliest):** a resale marketplace for used/surplus decoration
materials only, connecting manufacturers and decorators.

**Widened (still the trade-layer's foundational insight, preserved from the
original reasoning and still true today):**

> A decorator does not work alone. To execute a single wedding they source
> from many suppliers — lighting, LED wall, furniture, flowers, panthal,
> drapes, pillars. A decorator gives an order to a manufacturer; the
> manufacturer needs lighting for the decor; where does *he* buy it?

The trade is a **supply chain**, not a set of isolated sellers. A
resale-only app captured one thin slice of a web where money already flows
in every direction.

**Two specific weaknesses of resale-only, identified at that stage and
still relevant to why the product is shaped the way it is:**

1. **Low repeat usage.** A decorator resells surplus occasionally. But they
   *source supplies for every single wedding.* The sourcing behaviour is
   recurring; the resale behaviour is not.
2. **Cold-start fragility.** A supply-only marketplace is worthless until
   enough people list. There is no value on day one.

**Role model added, then dropped (2026-08-01 → 2026-08-19):** a
Vendor/Manufacturer split was introduced to gate who could post a
requirement vs. who could fulfill one. It worked for decor-only sourcing.
It was dropped once categories widened past decor, because the split
doesn't map cleanly onto all 29 categories — see §0 above and
`DECISIONS.md` §7.

**Categories widened to all 29 wedding-trade categories, and a public
profile layer added (2026-08-19):** driven by founder-led interview
findings — see §0 above. This is the current state.

---

## 2. Why the trade layer is cross-category, not decor-only

This extends the original supply-chain insight (§1) beyond decor
specifically:

A photographer needs a camera lens upgrade and could buy from a wholesaler
at low cost instead of retail. A caterer has used equipment sitting idle
after a slow season. A decorator has decor items used only 2–3 times and
wants to resell them cheaply rather than let them sit unused. **The trade
is a supply chain across the whole wedding industry, not just within
decoration.** Restricting it to decor materials was itself a narrower slice
of the same underlying insight that motivated the original widening from
pure resale.

This also increases the *frequency* argument from §1: sourcing now happens
across a much larger surface (any of 29 categories, not just decor), which
gives any given business more reasons to open the app.

### Trade-listing eligibility is deliberately NOT restricted by category

Every one of the 29 categories can technically post any `postType`. A
category with little physical-goods trade (e.g. Iyer, Psychiatrist for
Marriage Counseling) will simply see little to no trade activity
organically. Restricting this in code was considered and rejected — the
data will settle the distinction on its own, and building a "trade-eligible
category" system would be unnecessary complexity for something usage
already predicts.

---

## 3. Why `requirement` posts still matter (unchanged reasoning, now open to all categories)

Requirement posts remain the demand-side post type, and the reasoning for
why they matter most is unchanged from the original design:

Most post types are **supply-side** — "I have something, come find me." A
marketplace built only on those is as empty as whatever people bother to
list. `requirement` is **demand-side**, and it works *even when the catalog
is thin*. On day one, with almost nothing listed, a business can still post
"I need 50 pillars in Madurai by next week" and others can respond.

This is still the single most important design element in the trade layer —
it's what stops the app being dead on arrival. **What changed today:** this
now works across all 29 categories, not gated by the Vendor/Manufacturer
role split (see §0, `DECISIONS.md` §7). Do not remove or weaken this post
type when working on the data model.

---

## 4. Why catalog is profile-only (unchanged, confirmed still correct)

Catalog lives in a `catalogItems` subcollection under a business's profile,
not in the main `listings` feed. This was decided in an earlier pass and
confirmed still correct by the 2026-08-19 code audit — nothing about
today's changes affects this.

Two other approaches were tried in reasoning and rejected before landing
here:

- **As a profile-only section with no discoverability** — rejected at the
  time for the opposite reason (nobody browses profiles in a new app with
  no traffic). That tradeoff was later revisited and accepted anyway —
  catalog is reached by visiting a profile, not by browsing the feed. The
  public profile layer added today (§0, §6 below) makes profiles more
  worth visiting than they were before, which strengthens rather than
  undermines this decision.
- **As a checkbox on a sell post** — rejected once it became clear a
  catalog item is a business's *standing product range* (unlimited
  quantity, always available), a fundamentally different thing from a
  one-off sale.

---

## 5. Why the app has two feed tabs, not the drafted three-tab redesign

The current, locked app shape is two tabs (Available, Needs) plus Profile.
A 2026-08-04 draft (`DECISIONS.md` §17–§18) proposed collapsing this into
three tabs (Feeds, Market, Profile) with toggles instead of separate tabs.
**That redesign remains unadopted as of today** — see §0. The reasoning
below is for the current, locked two-tab shape.

The risk of many post types across many categories is obvious: the app
becomes a cluttered noticeboard where nobody knows where to look. The fix
is to organise the interface by **direction**, not by post type or
category:

| Direction | Post types | User intent |
|---|---|---|
| Supply — "I have" | sell-used, sell-new, rental | I'm offering something |
| Demand — "I need" | requirement | I need something |

Which produces: an **Available** tab (all supply posts, filterable by
category and type), a **Needs** tab (requirements only, now open to
everyone regardless of category — see §0), and a **Post button** that asks
intent once, then routes to the right form.

### The evidence behind this (unchanged)

- IndiaMART keeps browse-listings and buyer-requirements as structurally
  separate flows.
- Facebook keeps "for sale" separate from "wanted" — and where the platform
  *doesn't* enforce that separation, group admins have to invent the rules
  manually.
- OLX runs a single-type feed and still drew clutter complaints.

The lesson: when a platform doesn't separate post types structurally, the
clutter burden lands on users and moderators. This logic doesn't change
just because categories widened — if anything, more categories makes the
separation more important, not less.

---

## 6. Why reveal-phone on the trade side, and why the public profile does the opposite

**Trade side (unchanged):** when two businesses connect through a trade
post, the app reveals a phone/WhatsApp number and they take it from there.
No in-app messaging, no inbox, no payments. This was always a deliberate,
stage-appropriate trade-off, not an oversight:

| | Reveal-phone | In-app connection |
|---|---|---|
| Ship speed | Fast — already built | Slow — messaging is a real build |
| Matches user habit | Yes — deals already close on WhatsApp | No — asks users into a new surface |
| Business risk | Deals leak off-platform, no transaction data | Protects data and monetisation |
| Right for which stage | Pre-launch / validation | Post-PMF / scaling |

The known risk — off-platform leakage, a well-documented killer of
two-sided marketplaces — is accepted and deferred. **Restated as of today:**
external desk research independently confirmed this is the most common
reason B2B marketplaces of this exact shape (no payment capture, no
embedded workflow) fail to retain users after first contact. This isn't
being re-litigated or solved today — it's still accepted, but should be
treated as a real, evidenced risk rather than a theoretical one when
evaluating the trade layer's traction later.

**Public profile (new today, `DECISIONS.md` §6a): the opposite choice, on
purpose.** Contact info is visible directly on the public profile, with no
reveal-tap. This is not an inconsistency to fix — the two mechanics do
different jobs:

- The trade-side reveal mechanic mediates a moment of connection *between
  two businesses already inside the system*. Friction there is acceptable
  and even useful (it's the "highest-trust moment in the app," per
  `DECISIONS.md` §6).
- The public profile's entire purpose is to be shared *outward* — to a
  bride's family, to a lead met in person, to anyone outside the app
  entirely. A reveal-tap actively defeats that purpose; the whole point is
  frictionless outward sharing.

Do not "harmonize" these into one pattern without asking — this was decided
with the inconsistency explicitly in view, not missed.

### Requirement responses are reversed (unchanged)

For supply posts: a buyer taps "I'm interested" → the seller's number is
revealed. For requirement posts: another business taps "I can supply this"
→ *their* number goes to the poster. Same mechanic, opposite direction. No
new system needed. This still holds regardless of the role-split removal —
it was never dependent on Vendor/Manufacturer, only on supply-vs-demand
direction.

---

## 7. Why the category field replaces userType entirely, not just extends it

See `DECISIONS.md` §0, §7 for the full reasoning and the change log entry.
The short version: the original `userType` model went through two states —
first a 3-value label that gated nothing (Manufacturer/Decorator/Supplier),
then a 2-value role that gated real permissions (Vendor/Manufacturer, added
2026-08-01). Both were built around a decor-specific mental model of who
buys and who sells.

Once the trade layer widened to all 29 wedding-trade categories, that model
stopped making sense: there's no clean way to say a photographer is a
"vendor" or a "manufacturer" relative to a caterer, the way Decorator→
Manufacturer mapped cleanly onto decor sourcing specifically. Rather than
invent an artificial mapping, the field was replaced entirely with a single
`category` field (the 29-item list), used descriptively — not as a
permission gate — for both the trade layer and the new public profile
layer. One taxonomy serves both purposes; do not build two.

Multi-select for `category` was considered (a business could plausibly span
more than one — a decorator who also does catering, for instance) and
rejected for now, consistent with the very first `userType` decision's
original reasoning: keep it simple, revisit only if real usage shows a
strong need.

---

## 8. Why phone OTP (current) and WhatsApp OTP (planned, not built)

**Current, working, unchanged by anything today:** phone verification via
Firebase's built-in phone auth. Google Sign-In was considered early on and
rejected — phone verification is mandatory regardless of sign-in method,
because the entire connection mechanic (trade-side reveal) depends on a
verified phone/WhatsApp number. Google Sign-In would have added a second
auth surface, native-module rebuild overhead, and recurring SHA-1/token
bugs, while removing zero required steps. Pure cost, no benefit — this
reasoning is unaffected by anything else that changed today.

**Planned but not built:** a later pass (2026-08-01) planned moving from
plain phone OTP to WhatsApp-delivered OTP via a custom Cloudflare Worker +
Meta WhatsApp Cloud API, reasoning that users already live on WhatsApp for
this trade so SMS OTP had no advantage once WhatsApp verification was
possible. **This was written into `DECISIONS.md` as if already implemented,
which was inaccurate — corrected today (§0 above; full detail in
`DECISIONS.md` §8).** The plan itself is preserved and still intended, just
clearly marked as not yet built. It does not block the trade layer or
public profile layer, both of which work fine on the existing, working
phone auth.

---

## 9. Why the public profile layer is genuinely new, not a repackaging of catalog

It would be easy to assume the public profile (§0, §6, `DECISIONS.md` §6a)
is just catalog with better marketing. It isn't, and the distinction
matters for anyone building either:

- **Catalog** (§4) is a business's standing product range — items,
  discoverable by visiting their profile *inside the app*, requires the
  viewer to be a signed-in user, and reveal-gates contact like everything
  else on the trade side.
- **Public profile** is not about products at all — it's business identity
  (name, category, district, photos/portfolio) plus *directly visible*
  contact info, reachable by anyone with the link, **no app account
  required to view it.**

Catalog answers "what does this business sell." The public profile answers
"who is this business and how do I reach them" — and it's built to be
handed to someone who has never opened the app and may never need to.

**Confirmed 2026-08-19: the public profile shows identity, contact, and
portfolio photos ONLY — never catalog or pricing.** This was raised as an
open question during a deep-audit pass and resolved directly: catalog stays
inside the app, behind login, exactly as §4 describes. The two layers stay
cleanly separated by what they show, not just by who can see them.

---

## 10. Why this isn't a B2C pivot, even though it looks adjacent

It would be easy to read "29 categories, public profiles, shareable links"
as quietly building the couple-facing directory that's supposed to be the
long-term end vision. **It is not, and the distinction matters:**

- There is no in-app couple search, browsing, or booking flow. A couple can
  only reach a profile via a link a business chose to share — the app does
  not surface businesses to couples on its own.
- The trade layer (buying/selling equipment) has no couple-facing
  equivalent at all — it's purely business-to-business.

The public profile layer is deliberately shaped so it *could* become seed
data for the couple-facing directory later — real businesses, real
categories, real portfolios, already populated — but that is a future
option being kept open, not a commitment being made now. Building the full
couple-facing experience (search, booking, discovery) remains explicitly
out of scope. See `DECISIONS.md` §1, §10.

---

## 11. Why the collection is still called `listings`

The data model is a single Firestore collection holding all trade post
types, distinguished by a `postType` field. That collection is named
`listings`, which is semantically imperfect since a `requirement` post ("I
need 50 pillars") is not a listing.

A rename to `posts` was considered and **rejected twice now** — first early
on, and reaffirmed today. The rework cost (runtime code, seed script,
security rules, documentation) has outweighed the naming gain both times,
and the collection now serves an even wider set of categories than when
this was first decided, which if anything raises the rework cost further.
**The name is historical. It is not a bug. Do not rename it.**

---

## 12. Scope discipline (updated for today's changes)

Several larger ideas came up during scoping and were deliberately parked
rather than built. Two entries below moved out of "parked" today; the rest
are unchanged.

| Idea | Status | Why parked / resolved |
|---|---|---|
| Full couple-facing wedding directory (search/booking) | Still parked, end vision | Different product, different users, much bigger build. Public profiles now deliberately designed as future seed data — see §9. |
| ~29 wedding service categories | **Resolved 2026-08-19 — adopted for the trade+profile layers.** No longer belongs only to "the directory vision, not the trade platform" — see §0, §2, `DECISIONS.md` §9. |
| Vendor/Manufacturer role split | **Resolved 2026-08-19 — dropped entirely,** replaced by the category field — see §0, §7. |
| In-app rental booking | Rejected | Booking engine — out of scope by design |
| In-app chat / inbox (trade side) | Rejected, risk restated today | Reveal-phone is the mechanic; disintermediation risk explicitly confirmed by external research today, not re-solved — see §6 |
| In-app payments | Rejected | The app connects; deals close off-app |
| Reveal-gating the public profile | Considered today, rejected | Profile is intentionally open-contact — see §6, §9 |
| Restricting trade listings to a "physical goods" category subset | Considered today, rejected | Kept open deliberately — see §2 |
| Social layer (likes/comments/follow/communities), 3-tab nav redesign | Still parked, still draft | Proposed 2026-08-04, not adopted today — see §0, `DECISIONS.md` §18 |

The pattern to preserve, unchanged since the very first version of this
file: **a big vision does not require a big v1.** The trade+profile
platform is still the wedge that earns the right to build the wider
directory later.

---

## 13. How to work on this project

The build process assumes a non-technical founder driving AI tools, with a
separate PM/planning layer (a parallel planning session) and a separate
coding AI agent (Cursor) doing implementation:

- Decisions and planning happen in the planning session; code gets written
  in Cursor.
- Cursor writes code — it does not make architecture or product decisions.
- One feature or screen per prompt; review the diff before accepting;
  reject edits touching unrelated files.
- Every Cursor session starts by reading `DECISIONS.md`.
- Cursor prompts should carry the *why*, not just the *what* — a tool
  without context will "correct" deliberate decisions it mistakes for bugs.

### The failure this process exists to prevent (still true, now twice-demonstrated)

A deliberate data-model change made in one session was once silently
reverted by a later Cursor session, which saw an inconsistency and "fixed"
it — with no way of knowing the inconsistency was intentional. The tool
wasn't wrong; it had no access to the decision.

**Today's reconciliation work is a second instance of the same underlying
risk, at the documentation level instead of the code level:** a separate
planning session had drifted out of sync with this repo's actual
`DECISIONS.md`, to the point of contradicting real, already-shipped
decisions (the role-gating model) while also trusting written claims in
this repo's own file that turned out not to match the actual code
(WhatsApp OTP). Neither side was malicious or careless in isolation — the
drift happened because decisions were being made in more than one place
without a reconciliation step.

**The lesson, stated plainly for future sessions:** this file and
`DECISIONS.md`, kept in the repo, are the only source of truth — not chat
memory, not a synced knowledge project, not any single AI's running
context. When starting work after any gap, or when picking up a plan from
outside this repo, verify it against the actual current `DECISIONS.md` and
the actual current code before treating it as ground truth. If something in
either file seems to contradict the running code, say so explicitly and
ask, rather than trusting either the file or your own assumptions by
default — `DECISIONS.md` §19 makes this an explicit instruction for exactly
this reason.

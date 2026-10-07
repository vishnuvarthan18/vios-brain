# W2D — Product Context

> Companion to `DECISIONS.md`.
> `DECISIONS.md` = the rules (what to build, what not to).
> This file = the reasoning behind them (why they are what they are).
>
> If you only need to know what to do, read `DECISIONS.md`.
> Read this when you need to understand *why*, or when considering a change.

Last updated: 2026-07-21

---

## 1. What the product is, in one line

A B2B trade connection platform for the Tamil Nadu wedding industry — where
decorators, manufacturers and suppliers connect to buy, sell, rent and source
wedding decoration materials.

### How the scope got here

The original v1 was much narrower: a resale marketplace for **used/surplus**
decoration materials only, connecting manufacturers and decorators.

That was reconsidered because of a structural insight about how the trade
actually works:

> A decorator does not work alone. To execute a single wedding they source from
> many suppliers — lighting, LED wall, furniture, flowers, panthal, drapes,
> pillars. A decorator gives an order to a manufacturer; the manufacturer needs
> lighting for the decor; where does *he* buy it?

The trade is a **supply chain**, not a set of isolated sellers. A resale-only
app captured one thin slice of a web where money already flows in every
direction.

**Two specific weaknesses of resale-only:**

1. **Low repeat usage.** A decorator resells surplus occasionally. But they
   *source supplies for every single wedding.* The sourcing behaviour is
   recurring; the resale behaviour is not.
2. **Cold-start fragility.** A supply-only marketplace is worthless until enough
   people list. There is no value on day one.

Widening to the full supply chain addresses both.

---

## 2. Why five post types

The core unit is not a "listing" — it is a **post**, and the transaction type is
an *attribute* of the post rather than a separate product.

| Post type | Direction | Why it exists |
|---|---|---|
| `sell-used` | Supply | The original wedge — offloading surplus stock |
| `sell-new` | Supply | Manufacturers and suppliers sell new product, not just surplus |
| `rental` | Supply | A large share of wedding decor is hired, not bought |
| `catalog` | Supply | A manufacturer's permanent product range — unlimited quantity, always available |
| `requirement` | **Demand** | "I need X" — the demand side |

### Why `requirement` matters most

Four of the five are **supply-side** — "I have something, come find me." A
marketplace built only on those is as empty as whatever people bother to list.

`requirement` is **demand-side**, and it works *even when the catalog is thin*.
On day one, with almost nothing listed, a decorator can still post "I need 50
pillars in Madurai by next week" and suppliers can respond.

This is the single most important design element in the product: it is what
stops the app being dead on arrival.

### Why catalog is its own post type

Catalog was almost built two other ways, both rejected:

- **As a profile-only section** — rejected. Nobody browses profiles in a new app
  with no traffic. A catalog nobody can discover is worthless.
- **As a checkbox on a sell post** — rejected once it became clear a catalog item
  is not surplus stock. It is a manufacturer's *product range*: unlimited
  quantity, permanently available, ordered any time. That is a different thing
  from a one-off sale.

So catalog lives in the feed as its own post type, discoverable alongside
everything else.

### Why rental is a label, not a system

Real rental implies availability dates, duration, returns, double-booking
prevention and rate structures. That is a booking engine, and it was explicitly
out of scope.

The version that got built instead: a post *tagged* "for rent", with the phone
revealed, and dates/deposit/return negotiated on WhatsApp — exactly like every
other connection on the platform.

This matches the product's whole model: **the app connects; the deal closes
off-app.** It costs almost nothing to build and still captures rental supply.

---

## 3. Why two tabs

The risk of five post types is obvious: the app becomes a cluttered noticeboard
where nobody knows where to look.

The fix is to organise the interface by **direction**, not by post type:

| Direction | Post types | User intent |
|---|---|---|
| Supply — "I have" | sell-used, sell-new, rental, catalog | I'm offering something |
| Demand — "I need" | requirement | I need something |

Which produces:

- **Available** tab — all supply posts, filterable by type
- **Needs** tab — requirements only
- **Post button** — asks intent once, then routes to the right form

The user never thinks in "five types." They think: *let me see what's
available* / *let me see who needs stuff* / *let me post something*.

Five types exist in the data model. The interface only ever shows two
directions.

### The evidence behind this

- IndiaMART keeps browse-listings and buyer-requirements as structurally
  separate flows.
- Facebook keeps "for sale" separate from "wanted" — and where the platform
  *doesn't* enforce that separation, group admins have to invent the rules
  manually.
- OLX runs a single-type feed and still drew clutter complaints.

The lesson: when a platform doesn't separate post types structurally, the
clutter burden lands on users and moderators.

### The two feeds feed each other

```
Post a NEED    → matching suppliers get notified → they respond
Post SUPPLY    → surfaces to people with matching needs
```

Category and district are the matching keys.

---

## 4. Why reveal-phone, not in-app chat

When two parties connect, the app reveals a phone/WhatsApp number and they take
it from there. No in-app messaging, no inbox, no payments.

This is a **deliberate, stage-appropriate trade-off**, not an oversight:

| | Reveal-phone | In-app connection |
|---|---|---|
| Ship speed | Fast — already built | Slow — messaging is a real build |
| Matches user habit | Yes — deals already close on WhatsApp | No — asks users into a new surface |
| Business risk | Deals leak off-platform, no transaction data | Protects data and monetisation |
| Right for which stage | Pre-launch / validation | Post-PMF / scaling |

The product is **pre-launch and unvalidated**. The only question that matters
right now is whether decorators and suppliers will use this at all.
Reveal-phone answers that fastest and matches exactly how these users already
behave.

The known risk — off-platform leakage, which is a well-documented killer of
two-sided marketplaces — is **accepted and deferred**. If leakage becomes a
problem, that means people are transacting, which is a good problem. In-app
connection can be added then, funded by traction.

Building the in-app version first would mean spending weeks defending revenue
that doesn't exist yet, on a product that hasn't been tested.

### Requirement responses are reversed

For supply posts: a buyer taps "I'm interested" → the seller's number is
revealed.

For requirement posts: a supplier taps "I can supply this" → the *supplier's*
number goes to the poster.

Same mechanic, opposite direction. No new system needed.

---

## 5. Why userType is a label that gates nothing

The original v1 offered only two types: Manufacturer and Decorator. That was a
blocker — every other supplier in the chain (lighting, LED wall, furniture,
flowers, drapes) had no way to register at all, which would have killed the
supply side.

A wider, business-based classification was considered (splitting suppliers by
what they deal in), but a simpler realisation settled it: **the classification
doesn't gate anything.** Everyone can do everything.

- A manufacturer makes goods *and* sells them, so is also a supplier.
- A decorator buys to execute weddings *and* resells surplus, so is also a
  supplier.
- Everyone is partly a supplier.

The categories overlap by nature and will never split cleanly. Since the field
controls no permissions or filtering, chasing a perfect taxonomy is wasted
effort.

**Result:** three single-select labels — Manufacturer, Decorator, Supplier.
Purely descriptive. Anyone can post any post type and respond to anything.
There is no buyer/seller distinction and no separate home screen per type.

Multi-select was considered (to handle people who are genuinely both) and
rejected as unnecessary complexity for a field that gates nothing.

---

## 6. Why phone OTP only

Google Sign-In was built into the original plan and then dropped.

The reasoning: phone verification is **mandatory regardless of sign-in method**,
because the entire connection mechanic depends on a verified phone/WhatsApp
number. So Google Sign-In added:

- a second auth surface to maintain
- native-module rebuild overhead
- recurring SHA-1 / token bugs

...while removing **zero** required steps for the user. It was pure cost.

---

## 7. Why the collection is still called `listings`

The data model is a single Firestore collection holding all five post types,
distinguished by a `postType` field.

That collection is named `listings` — which is semantically imperfect, since a
requirement ("I need 50 pillars") is not a listing.

A rename to `posts` was started and then **cancelled**. The rename would have
touched runtime code, the seed script, security rules and five documentation
files, and the app already worked correctly with `listings` everywhere. The
rework cost outweighed the naming gain.

**The name is historical. It is not a bug. Do not rename it.**

---

## 8. Scope discipline

Several larger ideas came up during scoping and were deliberately parked rather
than built:

| Idea | Status | Why parked |
|---|---|---|
| Full couple-facing wedding directory | End vision, not now | Different product, different users, much bigger build |
| ~29 wedding service categories (photography, DJ, catering, iyer, etc.) | Parked | Belongs to the directory vision, not the trade platform |
| In-app rental booking | Rejected | Booking engine — out of scope by design |
| In-app chat / inbox | Rejected | Reveal-phone is the mechanic |
| In-app payments | Rejected | The app connects; deals close off-app |

The pattern to preserve: **a big vision does not require a big v1.** The trade
platform is the wedge that earns the right to build the wider directory later.
Every marketplace that won this pattern started narrow and expanded.

---

## 9. How to work on this project

The build process assumes a non-technical founder driving AI tools:

- Decisions and planning happen in conversation; code gets written in Cursor.
- Cursor writes code — it does not make architecture or product decisions.
- One feature or screen per prompt; review the diff before accepting; reject
  edits touching unrelated files.
- Every Cursor session starts by reading `DECISIONS.md`.
- Cursor prompts should carry the *why*, not just the *what* — a tool without
  context will "correct" deliberate decisions it mistakes for bugs.

### The failure this process exists to prevent

A deliberate data-model change made in one session was silently reverted by a
later Cursor session, which saw an inconsistency and "fixed" it — with no way
of knowing the inconsistency was intentional.

The tool wasn't wrong; it had no access to the decision. That is why decisions
live in the repo (`DECISIONS.md`) rather than in any single AI's memory, and why
Cursor session summaries get reviewed for drift.

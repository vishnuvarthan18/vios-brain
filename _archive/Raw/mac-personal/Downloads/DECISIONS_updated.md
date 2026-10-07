# W2D — LOCKED DECISIONS

> **READ THIS FIRST before changing any code.**
> This file is the single source of truth for product and architecture decisions.
> If code contradicts this file, the FILE is right and the code needs fixing.
> If you (an AI assistant) think something here is a bug — it is NOT. It is a decision.
> Do not "fix" anything listed here. Ask the human first.

Last updated: 2026-08-20 (Vendor/Manufacturer role split RESTORED, revised — see §0b, §7)

---

## 0b. WHAT HAPPENED ON 2026-08-20 — READ THIS SECOND (after §0)

**The 2026-08-19 removal of the Vendor/Manufacturer role split (§7 as it
stood 2026-08-19–2026-08-20) is itself superseded, one day later, by
founder correction.**

Founder's reasoning: the 2026-08-19 removal assumed no clean split exists
across all 29 categories — a photographer and a caterer aren't naturally
"vendor" or "manufacturer" to each other, so the model was dropped rather
than force an awkward mapping. That reasoning was **correct that a
per-category-instance split doesn't work, but wrong that no split works at
all.** A cleaner split exists — not "does this specific business raise or
fulfil requirements," but a structural one:

- **Vendor** — a business that provides a wedding-day **service**:
  photography, DJ, catering, bridal makeup, priest (Iyer), musicians
  (Mangala Vathiyam), event hosting (RJ), venues (Mahal or Mandabam),
  counseling, financial services, travel booking. These businesses
  **request** materials/products from Manufacturers.
- **Manufacturer** — a business that **manufactures or supplies physical
  materials/products**: raw decor materials, garlands, furniture, printed
  invitations, ornamental stock, rental equipment. These businesses
  **supply** to Vendors.

This is restored as a real permission-gating role, on top of (not instead
of) the 29-item `category` field — a business has both a `category` (what
it does) and a `role` (Vendor or Manufacturer). Each category has a single
locked role assignment — see the updated table in §9. **Do not treat this
as "just re-add userType" — it is a new field (`role`), reasoned from
service-vs-supply rather than from the old 2026-08-01 model, and it
coexists with `category` rather than replacing it.**

**Gating is fully symmetric (confirmed 2026-08-20), stricter than the
2026-08-01 model:**
- Only **Manufacturers** can post `sell-used` / `sell-new` / `rental`
  listings and maintain a Catalog (§4). A Vendor CANNOT post any of these,
  even if they have inventory to move.
- Only **Vendors** can post `requirement` posts. A Manufacturer CANNOT post
  a requirement. The Needs tab is Manufacturer-facing (§5).
- This is a deliberate, confirmed departure from the 2026-08-01 model,
  which only restricted requirement creation. The new model is symmetric
  because Manufacturers supply / Vendors request is the whole logic of the
  split — allowing crossover in either direction would undermine the
  reasoning that justifies having the split at all.
- Every business picks exactly **one role**, even in the 5 categories
  where the category itself could plausibly go either way (§9) — no
  dual-role registration. **Confirmed 2026-08-20: this tradeoff (those 5
  categories lose access to 4 of 5 post types) is accepted deliberately,
  not an oversight.** The founder-assigned default is a starting guess,
  correctable per-business later if wrong (log any correction in §17).
- Tier 1 matching (§16) now also factors in role, not just category +
  district.

**What does NOT change:** the public profile layer (§6a) — still open,
still not reveal-gated, still shows `category`; now also shows `role`.
The 29-category list itself (§9) is unchanged in content, only gains a
role column. §6 (reveal mechanic), §8 (auth), §10 (out of scope) are
unaffected.

**Status of the corrections/foundation work done under the OLD (no-role)
model on 2026-08-19–20 (Batch 1, Batch 2, and the overnight run):** all of
that work is real, necessary, and NOT wasted — the category migration,
public profile layer, security-rules hardening, account deletion fixes,
matching engine, ops hygiene, and bug fixes (`.exists()` cluster,
`expressInterest`, the phone-enumeration gap) all stand. Commit `6f479b3`
(overnight Task 12) deliberately REMOVED six role gates as part of that
prior model — this decision now REVERSES that specific commit's role-gate
removal (not its category-migration work, which stands). See §14 item 0b
for the full restore-site list, cross-checked against actual code as of
2026-08-20.

---

## 0. WHAT HAPPENED ON 2026-08-19 — READ THIS FIRST

This file went through three changes on 2026-08-19, in order. Documented
here so no one re-derives or re-argues something already settled. **Note:
the second bullet of point 3 below (role split dropped) is ITSELF now
superseded by §0b, one day later — kept here for history, not as current
truth.**

1. **Accuracy correction.** Two claims in the pre-existing file overstated
   what was actually built. Verified directly against code:
   - §7's "Manufacturer-only to see" requirement visibility was **not** a
     real Firestore permission — it was UI-only (tab-hide + client redirect).
   - §8's WhatsApp OTP via Cloudflare Worker was written as if implemented.
     It is **not built** — the app still runs on plain Firebase phone auth.
     Corrected below in §8; the WhatsApp OTP plan is preserved as a plan,
     clearly marked not-yet-built.
2. **Founder-led research + interviews** (separate from this repo, run in a
   parallel planning session) found the dominant real pain across
   decorators, manufacturers, and suppliers interviewed was **"not enough
   business,"** not sourcing friction. Full reasoning in
   `PRODUCT_CONTEXT.md` §0.
3. **Reconciliation with this file's own §17–§18 draft.** This file already
   had an unlocked draft (2026-08-04) proposing category widening and a
   social layer — independently arrived at, before the interview finding
   above. That draft is NOT fully adopted. Specifically:
   - **Category widening: ADOPTED, and now LOCKED** — see §9. The draft's
     own open item ("final category list not yet locked") is resolved.
   - ~~**Vendor/Manufacturer role split for requirements: DROPPED.**~~
     **SUPERSEDED 2026-08-20 — see §0b. The role split is RESTORED, in a
     revised form.**
   - **Public shareable profile (no login required): NEW**, not in the
     original draft at all — see §1, §6a.
   - **Social layer (likes, comments, follow, category communities), 3-tab
     nav redesign (Feeds/Market/Profile): STILL PARKED, still draft.** Not
     adopted in this pass. Revisit later — see §18, unchanged.

If you're an AI assistant picking this file up cold: sections marked
**DRAFT** are not ready to build. Everything else is locked. **Read §0b
before §7 — §7 has been rewritten to match §0b.**

---

## 1. WHAT THIS PRODUCT IS

**Wedding2day (W2D)** — a two-layer platform for the Tamil Nadu wedding trade
industry:

1. **B2B trade layer** — Manufacturers supply materials/products;
   Vendors (service providers) request them. Every registered business has
   both a `category` (one of 29, §9) and a `role` (Vendor or Manufacturer,
   §7). Role gates who can post what — see §7.
2. **Public profile / showcase layer** — every registered business gets a
   free, public, no-login-required, shareable profile page. Shareable via
   link to anyone, including couples, outside the app. See §6a.

- **Current scope = v1** (trade + profile layers). There is no separate
  "v2" tag in this file going forward.
- **NOT couple-facing browsing/booking inside the app.** A couple can view
  a *shared* profile link, but there is no in-app couple search or booking
  flow. The full couple-facing directory remains the long-term end vision
  — explicitly out of scope now (§10).
- Region: Tamil Nadu (38 districts)
- Primary metrics, tracked separately (do not collapse into one number):
  **interests per trade post** (trade-side liquidity) + **profile views/
  shares** (showcase-side reach)

---

## 2. STACK (LOCKED — DO NOT SUGGEST ALTERNATIVES)

| Layer | Choice |
|---|---|
| App | React Native + Expo (Expo Router, managed workflow) |
| Styling | NativeWind |
| Backend | Firebase — Auth, Firestore, Storage, FCM |
| IDE | Cursor Pro |
| Firebase project | `wedding2day-a99ea` |
| Android package | `com.w2d.app` |

**Old stack (FlutterFlow / Supabase / Twilio) is fully abandoned.** Never reference it.

---

## 3. DATA MODEL

### Collection name: `listings` — THIS IS DELIBERATE

**DECISION (2026-07-21, reaffirmed 2026-08-19): The main collection stays
named `listings`.** A rename to `posts` was considered and **rejected**
twice now.

> ⚠️ The `listings` collection holds all non-catalog post types, including
> requirements ("I need X"). The name is historical. **This is not a bug.**

### Collections

| Collection | Purpose |
|---|---|
| `users` | id = uid, createdAt, name, businessName, **category** (single-select from the 29-item list, §9), **role** (`vendor` \| `manufacturer` — NEW 2026-08-20, see §7), district, phone |
| `listings` | posts — `sell-used`, `sell-new`, `rental` (Manufacturer-only), `requirement` (Vendor-only) — see §4, §7 |
| `interests` | listingId, buyerId, createdAt |
| `reports` | user reports on posts |
| `blocks` | blockerId, blockedId, createdAt — see §15 |
| `profiles` | public-readable profile document per business — see §6a. Now also carries `role`. |

### `listings` document fields

createdAt, sellerId, **postType**, title, **category**, condition, price,
quantity, district, description, imageUrls[], status, **deliveryOption**,
**negotiable**, **neededBy** (requirement only), **viewCount**,
**interestCount**, **sellerName**, **sellerBusinessName**,
**sellerCategory**, **sellerRole** (NEW 2026-08-20 — denormalized `role`,
same purpose as `sellerCategory`)

- `status` defaults to `"pending"`; also supports `"sold"` / `"unavailable"` (§15)
- `price` and `quantity` are stored as **numbers**, not strings
- `deliveryOption` — one of `pickup-only` / `delivery-available` / `both`
- `negotiable` — boolean
- `neededBy` — optional date, `requirement` posts only
- `sellerName` / `sellerBusinessName` / `sellerCategory` / `sellerRole` —
  denormalized from the poster's `users` profile at creation time. Not
  backfilled on listings created before the relevant migration — UI must
  fall back gracefully when absent, not crash.
- Storage path: `listing-photos/{sellerId}/` — 5MB max, JPEG/PNG/WebP

**Migration required (role, NEW 2026-08-20):** every account created
between 2026-08-19 and 2026-08-20 has a `category` but no `role`. **The
category-migration prompt (Task 8) and this new role-confirmation prompt
are now COMBINED into one screen (confirmed 2026-08-20)** — a single
screen collects/confirms category AND role together, with role pre-filled
live from whichever category is selected (using the lookup table in §9),
always requiring explicit confirmation before proceeding. Do not build
these as two sequential blocking screens.

---

## 4. POST TYPES — via the `postType` field

Four post types live in the `listings` collection. **Catalog is separate — see below.**

| postType | Direction | Meaning | Who can post (REVISED 2026-08-20) |
|---|---|---|---|
| `sell-used` | Supply | Offload surplus / old stock | **Manufacturer only** |
| `sell-new` | Supply | Sell new product | **Manufacturer only** |
| `rental` | Supply | Item available for hire | **Manufacturer only** |
| `requirement` | **Demand** | "I need X" — others respond | **Vendor only** |

**Gating is symmetric in both directions (confirmed 2026-08-20) — a Vendor
cannot post any supply listing even if they have inventory to sell; a
Manufacturer cannot post a requirement even if they need to source
something.** This is stricter than the 2026-08-01 model, which only
restricted requirement creation. Deliberate — see §0b.

### Rules per type

- **Rental is a LABEL ONLY.** No in-app booking/calendar/availability system.
- **Requirement** = the demand side, Vendor-only to create. Response is
  reversed: a Manufacturer taps "I can supply this" → their phone is
  revealed to the Vendor poster. Poster sees a list of all responding
  Manufacturers (name, business, district, badge) on their own post — §15.
- **Requirement direction — REVISED 2026-08-20 (second reversal):** the
  2026-08-19 change opened requirement creation to any category. That is
  now superseded — **requirement creation is Vendor-only again**, and
  responding to a requirement is Manufacturer-only.

### CATALOG — REVISED 2026-08-20: Manufacturer-exclusive

**Catalog is profile-only** — does not appear in `listings` or the
Available feed (unchanged since 2026-07-25).

**NEW 2026-08-20: only Manufacturers can maintain a Catalog.** Vendors'
public-facing equivalent is the portfolio/photos on their public profile
(§6a). This reverses the 2026-08-19 "available to all categories" rule.

Catalog items live in `users/{uid}/catalogItems/{itemId}` — title,
category, photos, price, description. No quantity limit, no expiry, no
`status` field.

---

## 5. APP SHAPE — TWO TABS + PROFILE (current, locked)

| Tab | Holds |
|---|---|
| **Available** | `sell-used`, `sell-new`, `rental` — Manufacturer-posted only (§4, §7). Filterable by category (§9) and type. |
| **Needs** | `requirement` only, Vendor-posted only. **REVISED 2026-08-20: visible to Manufacturers (who respond), NOT Vendors** — reverses the 2026-08-19 "visible to everyone" rule. A Vendor sees their own posted requirements in My Listings, not a general Needs feed. |
| **Profile (own or viewed)** | Includes a **Catalog** section (Manufacturer-only, §4), and the business's **public profile** content — §6a |
| **Post** (action, not a feed) | Intent picker, now role-filtered: Manufacturers see sell-used/sell-new/rental/catalog; Vendors see requirement only. |

**Why two feed tabs:** deliberate anti-clutter decision, unchanged.
**Do not merge Available and Needs into one feed.**

**Forms:** a separate, minimal form per post type. **Do NOT build one
universal form for all post types.**

---

## 6. CONNECTION MECHANIC — REVEAL PHONE (trade side, unchanged)

For **all** trade post and catalog types: tap to reveal the other party's
phone/WhatsApp, then they connect directly off-app.

- **NO in-app chat. NO in-app inbox. NO in-app payments.**
- Every reveal logs an interest event to the `interests` collection (with
  dedupe) and increments the listing's `interestCount`.
- **Reveal screen must show trust context**: name, business name, district,
  role, verification badge (§15), member-since date, active listing count.
- **Daily reveal cap per account** (§15) to prevent bulk phone-number
  scraping. **Known residual gap as of the 2026-08-19/20 overnight run: a
  signed-in account can still read other users' phone numbers one at a
  time by walking `listings` for `sellerId`s — closing this needs a
  redesign of how contact info is stored (e.g. a per-pair reveal-grant
  document). Flagged, not yet fixed — do not consider §6 fully hardened
  until this is resolved.**
- **Block**: a user can block another account.

---

## 6a. PUBLIC PROFILE / SHOWCASE LAYER (2026-08-19, updated 2026-08-20)

**Why this exists:** founder-led interviews found the dominant pain was
"not enough business." This layer gives every business a self-promotion
tool with value on day one.

- Every registered business gets one public profile document (`profiles`
  collection, §3): business name, category (§9), **role** (NEW
  2026-08-20 — a couple viewing a profile link should know whether they're
  looking at a service provider or a materials supplier), district,
  photos/portfolio, phone/WhatsApp contact, a shareable link/slug.
- **Contact info is VISIBLE DIRECTLY on the public profile — NOT
  reveal-gated.** Deliberate, unchanged. **Do not "harmonize" with §6
  without asking.**
- **Must be reachable without authentication** — public read rules on
  `profiles`, write restricted to the profile's own authenticated owner.
- Public, no-login view is a separate route/screen (`app/p/[slug].tsx`,
  built 2026-08-19 — needs its role display added).
- **Deliberately NOT the couple-facing directory** — see §1.
- **Scope of what's public: identity, contact, portfolio photos, AND now
  role. Catalog is explicitly EXCLUDED** — and since Catalog is now
  Manufacturer-only (§4), this only ever applied to Manufacturers anyway.

---

## 7. ROLE MODEL — VENDOR / MANUFACTURER (RESTORED 2026-08-20, REVISED)

**STATUS: this section replaces the 2026-08-19 "category model replaces
role split" version of §7. See §0b for the full reasoning — read that first.**

**The role split is back, but not the 2026-08-01 model.** The old model
tried to derive vendor-vs-manufacturer per business without a structural
basis. What's restored is different: role is determined by **whether the
business's category is a service or a supply category** (§9's table).

**The model:**

- Every business has a `role`: **`vendor`** or **`manufacturer`**. In
  addition to `category` (§9), never a replacement for it.
- **Each of the 29 categories has ONE locked role** in the table in §9,
  except 5 categories marked "Both," where the role is a single
  founder-assigned default — **a business does not choose its own role
  freely; it is determined by category**, pre-filled and confirmed at
  signup/migration (§3).
- **Vendors REQUEST.** Only Vendors can create `requirement` posts (§4).
  A Vendor does not see a general Needs feed.
- **Manufacturers SUPPLY.** Only Manufacturers can post `sell-used` /
  `sell-new` / `rental` and maintain a Catalog (§4). Manufacturers see the
  Needs tab (§5) and respond to Vendor requirements.
- **Gating is symmetric (confirmed 2026-08-20)** — stricter than
  2026-08-01: neither role can cross into the other's post types at all.
  See §0b for the reasoning.
- **No dual-role registration**, even for the 5 ambiguous categories — one
  role per business, full stop. **Confirmed 2026-08-20: the resulting loss
  of 4/5 post types for those 5 categories is an accepted, deliberate
  tradeoff**, not an oversight.
- **Public profile (§6a)** shows `role` alongside `category`.
- **Firestore rules implication — a real code change:** `firestore.rules`
  needs a `role`-based gate re-added on `listings` create (`requirement`
  requires `role == 'vendor'`; the three supply types require
  `role == 'manufacturer'`), and on `catalogItems` create (Manufacturer
  only). This is new work — do not simply revert to the 2026-08-19 rules
  text; the old 2026-08-01 gate only checked `requirement` against a bare
  `userType` and never touched supply listings or catalog at all.
- **This is a direct reversal of overnight commit `6f479b3` (Task 12)**,
  which deliberately removed six role gates: `(tabs)/_layout.tsx` (Needs
  tab visibility), `(tabs)/needs.tsx` (redirect), `(tabs)/post.tsx` (intent
  picker), `post/requirement.tsx` (redirect), `constants/postTypes.ts`
  (role-based option filtering), `profile-setup.tsx` (role selector
  removed). All six need to be restored, revised to use `role` instead of
  the old `userType`. Task 12's CATEGORY migration work (the 29-item list,
  `BUSINESS_CATEGORIES`, category-based Tier 1 ranking logic) is NOT
  reversed — only the role-gate removal is.

**Existing accounts (created 2026-08-19–20, under the no-role model) need
the combined category+role migration prompt** — see §3.

---

## 8. AUTH — WhatsApp OTP is a PLAN, not yet built

**Current live behavior: plain Firebase phone auth.**
`@react-native-firebase/auth` — `signInWithPhoneNumber` /
`confirmation.confirm(code)`, wired into `app/(auth)/_lib/auth.ts`,
`phone.tsx`, `otp.tsx`. Do not assume WhatsApp OTP is live anywhere.

**The plan below (dated 2026-08-01) remains the intended future direction,
not yet executed:**

Auth is planned to move to **WhatsApp OTP**. No SMS, no email, no Google
Sign-In, once built.

**Planned architecture:**

- Firebase Auth's built-in phone provider does **not** support WhatsApp.
- OTP send/verify via a **Cloudflare Worker** — Blaze plan currently
  blocked by billing bug `OR_BACR2_44` (unresolved since 2026-07-21, §13).
- Flow: app requests OTP → Worker generates code, calls Meta WhatsApp Cloud
  API → app submits code → Worker verifies → Worker mints a Firebase
  custom auth token → app signs in via `signInWithCustomToken`.
- **Blocker:** Meta Business verification + WhatsApp template approval
  needed before any OTP can be sent. **Status as of 2026-08-19: unknown
  whether started — verify before assuming any timeline.**

**Not currently blocking anything else in this file.**

Google Sign-In was **rejected**. **Do not re-add it.**

**Phone number change:** in-app flow (built 2026-08-19/20, §14 item 1) —
re-verify with a new number, re-linked to the same account/UID, preserving
listing and interest history. Not an account-loss event.

---

## 9. THE 29 WEDDING BUSINESS CATEGORIES + ROLE MAP
     (category list LOCKED 2026-08-19, role column ADDED 2026-08-20)

**Fixed list — not freeform text entry.** The **Role** column is new — §7.

**"Both" categories: resolved to a single founder-assigned default below,
NOT user-choice.**

| # | Category | Role |
|---|---|---|
| 1 | Banana Tree | Manufacturer |
| 2 | Green Panthal | Manufacturer |
| 3 | Welcome Entrance | Both — default **Manufacturer** |
| 4 | Welcome Girls | Vendor |
| 5 | Welcome Toys | Manufacturer |
| 6 | Plate Decors | Manufacturer |
| 7 | Stage Decoration | Both — default **Manufacturer** |
| 8 | Photography and Videos | Vendor |
| 9 | DJ | Vendor |
| 10 | Catering | Vendor |
| 11 | Bridal Makeup | Vendor |
| 12 | Maalai | Manufacturer |
| 13 | Iyer | Vendor |
| 14 | Mangala Vathiyam | Vendor |
| 15 | RJ | Vendor |
| 16 | Ice Cream and Beeda | Manufacturer |
| 17 | Return Gift | Manufacturer |
| 18 | Honeymoon Trip | Vendor |
| 19 | Psychiatrist for Marriage Counseling | Vendor |
| 20 | Financial Support | Vendor |
| 21 | Costume Rental Service | Both — default **Manufacturer** |
| 22 | Furniture for Wedding | Manufacturer |
| 23 | Wedding Dress Materials | Manufacturer |
| 24 | Ornamental Rental Service | Manufacturer |
| 25 | New Ornamental Sales | Manufacturer |
| 26 | Mahal or Mandabam | Vendor |
| 27 | Invitation | Manufacturer |
| 28 | Audio and Lighting | Both — default **Manufacturer** |
| 29 | LED Wall | Both — default **Manufacturer** |

**This table is a first draft, accepted directionally by the founder on
2026-08-20 pending real-world validation — treat individual row
assignments as correctable, not permanently locked the way the category
NAMES themselves are. Log any correction in §17 same-session.**

**Conditions** (unchanged): New · Used - Like New · Used - Good · Used - Fair

**Districts** (unchanged): 38 Tamil Nadu districts — hardcoded constant.

**Photos:** max 3 per post/catalog item (unchanged).

---

## 10. EXPLICITLY OUT OF SCOPE — DO NOT BUILD

- Full couple-facing wedding directory, in-app search/booking
- In-app chat / inbox / messaging (trade side — §6)
- In-app rental booking, calendar, or availability
- In-app payments
- Tamil-language UI or search (English-only UI, icon-heavy — §15)
- AI listings
- Dual-role registration for a single business (§7 — one role only, even
  for "Both" categories in §9)
- The social layer (likes, comments, follow, category communities) and the
  3-tab Feeds/Market/Profile nav redesign — **still DRAFT, see §18.**
- Any use of `wedding2day.com`'s codebase/backend

> Note: a dedicated admin app was previously listed here as out of scope.
> That is **superseded** — see §11.

---

## 11. ADMIN APP — OVERRIDE (2026-07-25, unchanged; role field added 2026-08-20)

Previous decision "no admin app in v1, Firebase Console only" is
superseded. An admin web app is planned — sequenced **last** (§14).

A lightweight admin web dashboard (`w2d-admin` — Vite/React, same Firebase
project) already exists. It was migrated off `userType` onto `category` on
2026-08-19/20 — it now needs a **second** migration to add the `role`
field/filter alongside `category`, per §7. Do not re-add `userType`-based
logic; add `role` freshly, following the same pattern used for `category`.

---

## 12. REJECTED / SUPERSEDED IDEAS (do not re-propose without asking)

| Idea | Status |
|---|---|
| Rename `listings` → `posts` | Rejected (twice) |
| Multi-select `category`/`userType`/`role` | Rejected |
| Catalog as a checkbox on a sell post | Rejected — chose its own type |
| Catalog in the feed as its own post type | Superseded 2026-07-25 — profile-only, see §4 |
| Google Sign-In | Rejected |
| In-app rental booking | Rejected — rental is a label only |
| In-app chat/inbox (trade side) | Rejected — reveal phone instead, see §6 |
| No admin app in v1 | Superseded 2026-07-25 — see §11 |
| Tamil UI translation | Rejected for v1 |
| 3-value userType (Manufacturer/Decorator/Supplier) | Superseded 2026-08-01, then removed entirely 2026-08-19 |
| Vendor/Manufacturer role split (2026-08-01 model, per-business free assignment, asymmetric gating) | Superseded 2026-08-19, then **RESTORED 2026-08-20 in revised, symmetric, category-derived form (see §7)** — the 2026-08-01 free-assignment/asymmetric model specifically stays rejected |
| Category model fully replacing role (2026-08-19 version of §7) | **Superseded 2026-08-20** — see §0b, §7 |
| Requirement/supply-listing open to all categories/roles (2026-08-19 rule) | **Superseded 2026-08-20** — requirement is Vendor-only, supply listings are Manufacturer-only again |
| Catalog open to all categories (2026-08-19 rule) | **Superseded 2026-08-20** — Manufacturer-only again |
| Phone OTP via SMS (Firebase Auth phone provider) | Plan to supersede via WhatsApp OTP exists but NOT YET BUILT — see §8 |
| Reveal-gating the public profile | Considered 2026-08-19, rejected |
| Dual-role registration for "Both" categories | Considered 2026-08-20, rejected — one role per business |
| Asymmetric role gating (only requirement-creation restricted) | Considered 2026-08-20, rejected — symmetric gating confirmed, see §0b |
| Two sequential migration prompts (category, then role) | Considered 2026-08-20, rejected — combined into one screen, see §3 |

---

## 13. KNOWN INFRASTRUCTURE GOTCHAS

1. **Any new native module requires a fresh EAS dev-client build.**
2. **Emulator connections must be initialised exactly once**, at module
   level in `app/_layout.tsx` inside the `__DEV__` block.
3. **Emulator ports:** Auth 9099 · Firestore 8080 · Storage 9199 · UI 4000.
   Host `192.168.31.16` for physical device testing.
4. `firestore.rules` and `storage.rules` live at project root, referenced
   in `firebase.json`. Composite indexes in `firestore.indexes.json` — the
   emulator does NOT enforce indexes; deploy with
   `firebase deploy --only firestore` after any index-affecting change.
5. **Canonical remotes:**
   `https://github.com/vishnuvarthan18/wedding2day-app` and
   `https://github.com/vishnuvarthan18/w2d-admin` (both private).
6. Dev seeding: `scripts/seed.mjs` (w2d-app) and
   `w2d-admin/scripts/seed-admin.mjs` — both need `role` added to
   fixtures, per §7, as part of §14 item 0b. Note `verify-admin.mjs` and
   `verify-rules.mjs` currently assert on the ABSENCE of role gates (from
   the overnight run) — these assertions need inverting.
7. **`.exists` is a METHOD in `@react-native-firebase` v25, not a
   property.** `if (!doc.exists)` always evaluates false. Fixed across most
   of the codebase 2026-08-19/20 — see `EXISTS-IS-A-METHOD` comment block
   in `app/(auth)/_lib/firestore.ts`. Watch for this pattern in new code.
8. A fresh EAS dev-client build was required after the overnight run added
   `@react-native-firebase/messaging`, `/crashlytics`, `/analytics` —
   completed 2026-08-20.
9. `expo start --dev-client` (Metro) must be running for a dev-client
   build to load any JS.
10. Firebase emulator Auth does NOT send real SMS — read the OTP from the
    Emulator UI (`http://127.0.0.1:4000/auth`) during local dev.

---

## 14. REMAINING V1 ROADMAP (in build order)

Everything below is **v1**. Items marked **(RELEASE GATE)** must ship
before Play Store submission.

**0. Foundation work — CATEGORY (done 2026-08-19/20):**
   - Migrated `users`/`listings`/seed data from `userType` to the 29-item
     `category` field.
   - Built the `profiles` collection + public security rules + public
     profile route (§6a).
   - Updated `w2d-admin` for the category migration.

**0b. Foundation work — ROLE (NEW, sequenced next, 2026-08-20):**
   - Add `role` field to `users`/`listings`/`profiles` schemas (§3, §7).
   - Add `sellerRole` to seed fixtures in both `seed.mjs` and
     `seed-admin.mjs`, using the §9 lookup table.
   - Re-add the requirement-create gate to `firestore.rules`, revised to
     be `role`-based (symmetric — both directions gated, §0b) — also add
     Manufacturer-only gates on supply-listing creation and catalog
     creation.
   - Build ONE combined "confirm your category & role" migration screen
     (not two sequential prompts, §3) for every account created under the
     no-role model — pre-fill role live from category selection using the
     §9 lookup table, always require explicit confirmation.
   - Update `profile-setup.tsx` (new-signup flow) to collect category +
     role together for new signups going forward, same combined pattern.
   - **Restore, revised to use `role` instead of `userType`, the six sites
     overnight commit `6f479b3` removed:** `(tabs)/_layout.tsx` (Needs tab
     visibility), `(tabs)/needs.tsx` (redirect), `(tabs)/post.tsx` (intent
     picker), `post/requirement.tsx` (redirect), `constants/postTypes.ts`
     (role-based filtering), `profile-setup.tsx` (already covered above).
   - Update Tier 1 matching (§16) — the Needs-tab query now only runs for
     Manufacturer viewers and only surfaces Vendor-posted requirements;
     the existing category/district ranking logic itself is unchanged.
   - Update `w2d-admin` for the role field (§11) — second migration on top
     of the category one. Invert `verify-admin.mjs` / `verify-rules.mjs`
     assertions that currently check for the ABSENCE of role gates.
   - Write/extend the rules test suite: Vendor-creates-requirement
     (allowed), Vendor-creates-sell-listing (denied),
     Manufacturer-creates-sell-listing (allowed),
     Manufacturer-creates-requirement (denied),
     Manufacturer-creates-catalog-item (allowed),
     Vendor-creates-catalog-item (denied).

   This precedes everything below — do not build new feature work assuming
   the no-role model is still current.

1. **Profile / Settings screen** — DONE 2026-08-19/20. Catalog editor needs
   verification that it's now gated Manufacturer-only per §4's revision.
2. **Terms of Service + in-app account deletion (RELEASE GATE)** — DONE
   2026-08-19/20. ToS needs a follow-up pass to reflect the role model
   (§7) once item 0b lands.
   - **Privacy Policy content + Play Store Data Safety form (RELEASE GATE)**
     — still NOT done. Must reflect §6a's public contact-info exposure AND
     the role field once item 0b lands.
3. **Push notifications (FCM)** — infrastructure DONE 2026-08-19/20;
   sending blocked on Blaze (§8/§13).
4. **Matching / notification rules engine** — Tier 1 needs the item 0b
   revision pass (role-aware viewer eligibility). Tier 2 still gated on Blaze.
5. **My Listings + listing expiry + free-text search** — DONE 2026-08-19/20.
6. **Ops hygiene** — DONE 2026-08-19/20. **Known gap:** the phone-
   enumeration issue in §6 — flagged, not yet fixed.
7. **Full UI revamp** — deliberately last; should wait until item 0b (role)
   is fully landed so screens aren't revised twice.
8. **Admin web app** — own project, sequenced last (§11).

**RELEASE GATE** = blocks Play Store submission.

---

## 15. ADDITIONAL PRODUCT DECISIONS (2026-07-25, unchanged unless noted)

| Area | Decision |
|---|---|
| Business verification | **Soft.** Optional GST/business-name field on profile; shown as a badge if filled. |
| Ratings/reviews | **Full reviews** — star rating + written review, available after a phone reveal. |
| Report handling | **Manual, no committed SLA.** |
| Support channel | **WhatsApp/email**, shown in Profile/Settings. |
| Post-approval feedback | **Push notification** the moment a pending post is approved/rejected. |
| Mark as sold | **Status toggle** in My Listings — Manufacturer-only in practice now (§4, §7). |
| Reveal-screen trust context | Name, business name, district, role, verification badge, member-since date, active listing count — §6. |
| My Interests | Saved list of every listing the user has revealed a phone for. |
| Requirement responders | List view on the poster's own requirement post — per responding Manufacturer (§7). |
| Cold-start onboarding | 3-screen intro, role-aware: Manufacturers see Available/post-to-sell framing, Vendors see Needs/post-a-requirement framing. Should also introduce the public profile (§6a). |
| UI language | **English only, icon-heavy.** |
| Draft autosave | Post forms autosave locally on-device. |
| Delivery/pickup | Tag on every supply post: `pickup-only` / `delivery-available` / `both`. |
| Price flexibility | `negotiable` boolean checkbox. |
| Requirement urgency | Optional `neededBy` date field. |
| Reveal-scraping protection | Daily cap on phone reveals per account (§6 — residual gap flagged). |
| Block | User-initiated block. |
| Seller-facing stats | `viewCount` and `interestCount` shown to the owner. |
| In-app notification center | Bell/list in-app — built 2026-08-19/20, server-free. |
| Location permission | Planned for "near me" sorting — not yet built. |
| Contacts permission | Planned for "invite friends" — not yet built. |

---

## 16. MATCHING ENGINE — TIER PLAN
     (2026-08-01, revised 2026-08-19, revised again 2026-08-20)

**Tier 1 — built 2026-08-19/20 for the no-role model; NEEDS A REVISION
PASS under item 0b:**

- Query on the Needs tab, now Manufacturer-facing only (§5, §7) — a Vendor
  does not see this tab at all.
- Ranking logic (category outranks district — decision D12.5 from the
  overnight build) does **not** need to change — only WHO the query runs
  for and WHAT it surfaces changes: Manufacturer viewers only,
  Vendor-posted requirements only.

**Tier 2 — later, gated on Blaze:**

- Cloud Function trigger on new `requirement` doc → query matching
  Manufacturers by district + category → FCM push.

---

## 17. SCOPE CHANGE LOG

> Every scope change gets a row here, same session it's decided.

| Version tag | Date | Change | Reason |
|---|---|---|---|
| v1.2 (draft) | 2026-08-04 | Categories widened beyond decor to all wedding vendor trades | Same supply-chain logic as the original pivot |
| **(locked)** | **2026-08-19** | **29-item category list LOCKED verbatim** | Founder-supplied list — see §9 |
| **(superseded 2026-08-20)** | **2026-08-19** | ~~Vendor/Manufacturer role split DROPPED~~ | See next row |
| **(locked)** | **2026-08-20** | **Vendor/Manufacturer role split RESTORED, revised** — category-derived role (§9's table), symmetric gating (confirmed, stricter than 2026-08-01), one role per business even for the 5 "Both" categories (confirmed), combined category+role migration screen (confirmed). Reverses overnight commit `6f479b3`'s role-gate removal specifically, not its category work. | Founder correction: a structural service-vs-supply split works even though a per-category-instance split doesn't. See §0b. |
| **(locked)** | **2026-08-19** | Public, no-login, shareable profile layer ADDED | Founder-led interviews — "not enough business." See §6a |
| v1.2 (draft) | 2026-08-04 | Full social layer, category communities, catalog-in-profile-grid, 3-tab nav proposed | Still DRAFT, not adopted, see §18 |

**Note on §9's role table:** individual "Both"-category role defaults were
accepted directionally, not validated against real vendor/manufacturer
interviews, on 2026-08-20. If corrected later, log the correction here.

---

## 18. SOCIAL LAYER & SIMPLIFIED NAV — DRAFT (2026-08-04, UNCHANGED, STILL NOT ADOPTED)

**Status: still draft.** Not built, not locked, not rejected — parked.

### Nav shape (draft, would supersede §5's two-tab shape IF adopted)

| Tab | Contains |
|---|---|
| **Feeds** (default landing) | Toggle: **All** / **[District] [Category]**. |
| **Market** | Toggle: **Supply** / **Needs** |
| **Profile** | Single post grid. Follow + Reveal-phone on other users' profiles. |

### New mechanics not built

- Follow, likes + comments, category communities, 3-option post intent
  picker redesign.

### Open items

- [x] ~~Final category list~~ — RESOLVED 2026-08-19, see §9.
- [x] ~~Whether userType still cleanly maps once categories widen~~ —
      RESOLVED, revised twice: dropped 2026-08-19, restored-revised
      2026-08-20. See §0b, §7.
- [ ] Terminology pass, if this draft is adopted later
- [ ] Follow/like/comment data model — still not designed
- [ ] Moderation model for comments/social content — still undefined
- [ ] If this draft is ever adopted, it needs its own pass to incorporate
      the role model (§7) into "Market" tab's Supply/Needs toggle

---

## 19. HOW TO WORK ON THIS PROJECT (for AI assistants)

1. **Read this file before changing anything. Read §0b before §7.**
2. If something in the code looks wrong but is listed here — it is a
   DECISION, not a bug. Do not "fix" it. Ask first.
3. Change one feature or screen per prompt.
4. Never touch files outside the stated scope of the request.
5. Never make architecture or product decisions autonomously — surface
   options and ask.
6. If a request contradicts this file, say so instead of silently complying.
7. **If this file's claims and the actual code disagree, say so explicitly
   and ask before proceeding — do not silently trust either one.**
8. **Do not reference or pull code/patterns from `wedding2day.com`.**
9. **This file has reversed itself once already (§0 → §0b, role split
   dropped then restored within 24 hours).** This is not evidence the file
   is unreliable — it's evidence the founder is actively correcting course
   with real reasoning each time. Treat the CURRENT text as authoritative
   regardless of how recently it changed. Newer supersedes older, full
   stop — §0/§0b exist specifically to make that traceable.
10. **If you find a mismatch between what this file claims and what a
    prior AI agent's own audit/notes say (e.g. `OVERNIGHT_NOTES.md`),
    cross-check both against actual running code before acting** — this
    exact situation (agent correctly refusing to guess a 29-row table
    rather than inventing one) happened on 2026-08-20 and is the reason
    this file now includes the full table directly rather than describing
    it in prose.

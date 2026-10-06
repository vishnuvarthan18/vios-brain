# Venue sourcing plan — Tamil Nadu

How to build the call list that feeds `venue_prospects`.

---

## Pick one city first

"Tamil Nadu" is not a launch market. Chennai, Coimbatore, Madurai, Trichy and Salem have
different venues, different price points, and different customers. Trying to cover the
state at once means twenty venues spread across five cities — which looks empty
everywhere.

**Launch where your services team already operates.** That is the deciding factor, not
market size. Your differentiator is that you can run the wedding, and that only works
where your decor, catering and photography people can physically show up. A Chennai
listing you can't service is worse than no listing.

If your team works across several cities, the ranking I'd use:

**Chennai** — biggest market, highest prices (₹50k for smaller halls up to ₹5 lakh+ for
premium), but every competitor is already there. You'd be the eleventh platform.

**Coimbatore or Madurai** — smaller, but far less saturated. Easier to become the
obvious local option, and 15–20 venues covers a meaningful share of the city rather
than a rounding error. For a first market, being visible in a small pond beats being
invisible in a large one.

Decide this before collecting anything. Everything below assumes one city.

---

## Where to find venues

**Google Maps.** Search "kalyana mandapam", "thirumana mandapam", "marriage hall",
"wedding hall" plus each area name in your city. This is the best coverage of any
source. Note: Google's terms prohibit storing their business data in your own database
— read what's on screen and type the details into your sheet yourself, don't build a
scraper against their API for this.

**JustDial and Sulekha.** Both have dense kalyana mandapam listings by city and area,
with phone numbers displayed. JustDial also shows enquiry counts on some listings, which
tells you which venues are actively marketing — those owners already believe in online
leads and are easier to convert.

**Competitor directories** — Mandap.com, VenueLook, WedMeGood, WeddingWire, wikiwed.
Good for finding venues and confirming capacity and rough pricing. **Take contact
details only.** Do not copy photos or descriptions; those belong to whoever produced
them, and you'll be replacing them with your own photos anyway.

**Drive around.** Underrated. Mandapams are physical, sign-posted, and often on main
roads. An afternoon in the right neighbourhoods gets you venues that aren't listed
anywhere — and those owners have heard from nobody, which makes them the warmest calls
you'll make.

**Your own network.** Your services team has worked weddings. They know which halls are
good, who runs them, and which owners are reasonable. Start here — a warm introduction
converts several times better than a cold call, and it costs nothing.

---

## How much to collect

Aim for **60–80 prospects** to sign 15–20 venues.

That implies a conversion rate around 25%, which is realistic for cold outreach where
you're offering something free. If you're converting much better than that, you're
probably not calling enough marginal venues; much worse, and the pitch needs work before
you burn through the list.

Two days of collection gets you there. Don't over-engineer this — a spreadsheet and a
few focused afternoons beats building a scraper.

---

## What to record

Only what you need to make the call and remember what happened. Extra fields you never
fill in are worse than useless.

| Field | Why |
| --- | --- |
| `name` | Venue name |
| `city` | Launch city |
| `phone` | The number you'll actually call |
| `email` | Often missing. Fine. |
| `source` | Where you found it — tells you which source converts |
| `source_url` | Link back, for checking details later |
| `notes` | Capacity, area, rough price, who owns it, anything useful |
| `outreach_status` | `new` on import; updated as you work the list |

The CSV template at `data/venue-prospects-template.csv` matches the `venue_prospects`
table exactly, so it imports through W-029 without transformation.

---

## Working the list

Statuses move in one direction: `new` → `contacted` → `interested` → `onboarded`, or
→ `rejected` at any point.

**Log every call, especially the nos.** Write down *why* they said no. "Already on
Mandap" is a different problem from "doesn't trust a new company" is different from
"season is busy, call back in two months." The first needs a better pitch, the second
needs a reference, the third just needs a calendar reminder.

**A no is often a not-yet.** Venues that decline during peak season are frequently
receptive when bookings are thin. Set a follow-up date rather than marking them dead.

**Onboarded means you have photos, rates, capacity, and a named person who will answer
booking requests.** A venue that agreed on the phone but hasn't given you those is still
`interested`, not `onboarded`. Being strict here is what stops you launching with
listings nobody responds to.

---

## What good looks like

You are ready to open the marketplace publicly when:

- 15–20 venues in one city are `onboarded`
- Every one has your own photos, real capacity, and real pricing
- Every one has a named contact who has agreed to respond within a day
- You have personally tested a booking request end to end with at least three of them

That last one is the important one. Send a real request, see if a human replies. A venue
that doesn't answer your test won't answer a customer.

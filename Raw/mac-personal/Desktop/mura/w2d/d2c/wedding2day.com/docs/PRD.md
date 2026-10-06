# wedding2day.com — Product Requirements

Owner: Vishnu · Status: approved for build · Last updated: 27 July 2026

---

## 1. What we're building

Two independent product lines on one site.

**Line A — Mandap marketplace.** A discovery and booking platform for third-party
wedding venues. Customers browse mandaps, see real availability, and send a booking
request. The venue manager confirms or rejects it. Money settles offline between
customer and venue.

**Line B — Wedding packages.** wedding2day's own in-house team already delivers
end-to-end wedding services — photography, decor, catering, makeup, mehendi, and the
rest. These are sold as packages directly on the site. A customer submits an inquiry
and our team follows up.

**These do not bundle.** A mandap booking and a package inquiry are separate objects
with separate funnels. A customer can do either, or both, but the product never forces
one through the other. This is a deliberate decision — bundling would make us dependent
on venue availability to sell services we fully control.

---

## 2. Why this can win

The Indian market leaders are pure directories. Mandap.com, India's largest with 50k+
venues, is inquiry-driven: you browse, click "View Contact" or "Check Availability", an
advisor calls you, and everything after that is offline. VenueLook runs the same play.
Neither transacts online, and neither owns supply.

We have supply. An in-house services team is something a directory cannot replicate,
and it means we make money on Line B without waiting for marketplace liquidity on Line
A. Line B can launch and earn while Line A is still onboarding venues.

The trade-off: we're competing against platforms with 50k listings and years of SEO. We
will not win on catalog size at launch. We win on depth in one city and on being the
only option that can also run the wedding.

---

## 3. Users

**Customer.** Getting married, or a family member arranging it. Mobile-first, price-
sensitive, comparing 3–8 venues. Wants to know: is my date free, does it fit my guest
count, what does it cost, is this place real.

**Mandap manager.** Owns or runs a venue. Often not technical. Will not log in daily.
Needs the booking inbox to be trivially simple and to get an SMS or WhatsApp nudge when
a request lands — otherwise requests rot and the marketplace dies.

**Admin.** wedding2day staff. Approves managers, moderates listings and reviews, works
the venue prospect pipeline, and handles package inquiries.

---

## 4. The supply problem, and how we solve it

This is the single biggest risk to Line A, so it gets its own section.

We are seeding the venue catalog from scraped and manually-sourced data. **Scraped
records must never be published directly.** They live in a private `venue_prospects`
table that only admins can see.

The reason is product, not just legal. If a customer requests a date at a venue that
never agreed to be listed, nobody replies. That customer is gone, and so is their
word-of-mouth. A directory of unresponsive venues is worse than a small directory of
responsive ones.

So the pipeline is:

1. Scraped or sourced lead lands in `venue_prospects`.
2. Sales contacts the venue and onboards them.
3. Venue supplies its own photos, copy, pricing, and calendar.
4. Listing is created in `mandaps` and published.

Only step 4 is publicly visible. Photos and descriptions must come from the venue, not
from a competitor's site — that copy and imagery belongs to someone.

Get a lawyer to review the scraping itself before running it at scale. I'm not one, and
the rules around automated collection vary.

**Launch gate:** do not open Line A publicly until at least 15–20 verified, responsive
venues are live in the launch city. Below that, the catalog looks empty and search
returns nothing useful. Line B has no such gate and can launch first.

---

## 5. Scope

### In scope for v1

**Line A — marketplace**

- Public venue browse with search and filters: city, guest capacity, price range, date availability, amenities
- Venue detail: gallery, description, capacity, pricing, venue-defined tiers, amenities, availability calendar, reviews
- Customer accounts: sign up, log in, view own booking requests
- Booking request: pick a date, guest count, contact number, notes; submit
- Manager portal: approved managers manage their listings, calendar, and respond to requests
- Manager confirm/reject, with the calendar updating automatically on confirm
- Reviews: only customers with a completed booking can review; admin moderates before publishing
- Admin: approve managers, moderate listings and reviews, work the prospect pipeline, see all bookings

**Line B — packages**

- Public package browse and detail pages showing what's included
- Package inquiry form, open to guests without an account
- Admin inbox for inquiries with status tracking (new → contacted → quoted → won/lost)
- Admin CRUD for service categories, services, and packages

**Cross-cutting**

- Mobile-first responsive across every page
- Email notification to manager on new request; to customer on confirm or reject
- Basic SEO: per-venue and per-package metadata, sitemap

### Explicitly out of scope for v1

- **Online payments.** No gateway, no deposits, no escrow. Competitors don't do it and it triples the build. Revisit once booking volume justifies it.
- **Monetization.** Free for both sides at launch, by decision. Revenue model is deferred — see open questions.
- **In-platform messaging.** Customers and managers exchange phone numbers.
- **Native mobile apps.** Responsive web only.
- **Multi-city expansion tooling.** Launch one city, hardcode the list, generalize later.
- **Vendor marketplace for third-party service providers.** Line B is our own team only.

---

## 6. Key journeys

**Customer books a venue.** Lands on home or a city page → browses or filters →
opens a venue → checks the calendar for their date → signs up or logs in → submits a
request with date, guest count, and phone → sees it as pending in their dashboard →
gets an email when the manager responds → talks to the venue directly to settle money.

**Manager responds.** Gets an email that a request landed → logs in → sees the request
with date, guest count, and customer contact → confirms or rejects with a reason →
on confirm the date is marked booked on the public calendar automatically.

**Customer inquires about a package.** Lands on packages → compares tiers → opens one
and sees exactly what's included → submits name, phone, date, guest count → our team
picks it up from the admin inbox.

**Manager onboards.** Signs up as a manager → account sits unapproved → admin verifies
they actually run the venue → approves → manager creates a listing → listing goes to
pending review → admin publishes.

**Customer reviews.** Booking passes its event date and is marked completed → customer
gets an email asking for a review → submits rating and text → admin moderates → it
appears on the venue page.

---

## 7. Success measures

Line A is healthy when **manager response rate stays above 80% within 48 hours**. That
single number predicts whether the marketplace works; track it from day one and cut
venues that ignore requests.

Also worth tracking: requests per published venue per month, request-to-confirm rate,
share of customers who submit a second request (a proxy for the first venue ghosting
them), and package inquiry-to-won rate on Line B.

Vanity metrics to ignore: total listings, total signups.

---

## 8. Risks

**Managers don't respond.** Highest-likelihood failure. Mitigate with email plus
WhatsApp or SMS nudges, a visible response-rate badge on listings, and admin
intervention on stale requests. If a venue repeatedly ignores requests, unpublish it.

**Stale availability.** A manager who forgets to block a date creates a confirmed
booking that isn't real. Mitigate by making the calendar the manager's primary screen
and confirming dates by phone before the customer relies on them.

**Empty catalog at launch.** Covered by the launch gate in section 4.

**Line B cannibalizing focus.** Line B earns sooner and is easier to build. The risk is
Line A never getting enough attention to reach liquidity. Accept this consciously:
Line B funds the runway, but Line A needs a dedicated onboarding push.

**Scraping exposure.** Covered in section 4. Prospects stay private; published content
comes from venues.

---

## 9. Open questions

- **Revenue model.** Deferred by decision. The three realistic options are commission per confirmed booking, a listing or subscription fee from venues, or a lead fee per inquiry. Commission is hardest to enforce when money settles offline — we can't see whether a booking happened. Worth deciding before we have enough venues to have leverage.
- **Launch city.** Not yet chosen. Determines the whole seeding effort.
- **WhatsApp for notifications.** Email alone will likely underperform with venue managers. WhatsApp Business API is the obvious channel but adds cost and approval time.
- **Who owns manager onboarding.** A person has to make these calls. Product can't fix an unstaffed sales motion.

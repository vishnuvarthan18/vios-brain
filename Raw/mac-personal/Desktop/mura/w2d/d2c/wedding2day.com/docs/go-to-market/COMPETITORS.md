# Competitor flows — how the top 3 actually work

Researched 28 July 2026 by reading live pages, not marketing copy. Where a claim comes
from a company's own blurb rather than something observable on the page, it says so.

---

## The headline finding

**None of the three shows real availability.** Every one of them advertises it. Not one
delivers it.

Mandap.com puts "Price, Availability" in the `<title>` of every page, says "real-time
availability" in the venue description, and the page itself has **no calendar, no date
picker, and no availability data of any kind**. "Check availability" is a button that
opens a lead form.

This is the most useful thing in this document. The category leader — owned by
Matrimony.com, with 100+ cities and years of head start — concluded that maintaining
venue calendars is not worth it, and instead sells the *word* "availability" for SEO
while routing every user into a phone call.

Two ways to read that. Either accurate availability is genuinely uneconomic at scale, or
it's an unclaimed position because everyone chose the cheap version. Both readings point
the same direction for you: **don't build a self-serve calendar hoping to out-feature
them.** If you go after availability, win it with your ops team on 15 venues in one city,
where a human can actually keep it true — not with software across 900.

---

## 1. Mandap.com — your direct competitor

Matrimony.com Group. Same group as Wedding Bazaar and WedAssist. Mandapam-specific,
strong in Tamil Nadu, ~860 Chennai kalyana mandapam listings.

**Flow:** SEO landing (`/chennai/kalyana-mandapams-in-ambattur`) → listing cards →
venue detail → "Check availability" form or "View Phone Number" → phone call → offline.

**What the listing card carries:** photo carousel, "Mandap Exclusive" badge, rating,
name, area + Google Maps link, headline price (`₹1,90,000 per day`), capacity, car
parking, AC, room count. Two CTAs: *Check availability* and *View Phone Number*.

**The phone number is gated.** Classic JustDial mechanic — the click is the lead. That's
the whole conversion event.

**Detail page structure**, which doubles as their onboarding questionnaire:

- Capacity split two ways — 500 seating / 1,200 floating
- Car parking (15) and bike parking (100) as separate counts
- Rooms, AC rooms, electricity back-up, Wi-Fi, bridal room
- **Policies** — outside decorators not allowed, outside DJ allowed, outside food allowed, outside alcohol not allowed
- **Payment policy** — 50% on booking, 50% on date, non-refundable
- Allowed cuisine — Veg
- Price *type* — "Time Based Rent" (mandapams are per-day, not per-plate)
- Trust numbers — established 2017, 9 years in business, 350+ weddings
- Photos filed into 8 named albums: Venue Overview, Parking Area, Stage & Mandap, Food & Catering, Seating Arrangements, Banquet Hall Decorated, Accessibility, Rooms & Suites

**Monetisation:** tiered listings ("Mandap Exclusive"), plus a *WedAssist* upsell block
on every venue page — "assisted wedding planning, a dedicated expert, we help you book at
the best price." The category leader's margin business is selling planning services on top
of listings. **That is Line B.** They are doing your model with someone else's venues.

**Weaknesses visible on the page:**

- Many listing cards render `listingfallback.png` — no photos at all
- The flagship Ambattur venue has 1 review, rated 4.6. Review coverage across Chennai is close to nil
- Venue descriptions are LLM-generated and generic ("Step into the world of celebrations…"), often contradicting the structured data on the same page — one section says outside decorators are not allowed, the prose says you have "the flexibility to bring in your own decorators"

---

## 2. WedMeGood — the national directory

Biggest wedding platform in India. Venues are one category among photographers, makeup,
decor.

**Flow:** city/category page → filter by price, capacity, reviews → venue profile →
enquiry form → vendor calls back. Cards lead with per-plate pricing (veg/non-veg) and
room count, which is a hotel-and-banquet frame rather than a mandapam one.

**Genie** is their paid concierge: you state requirements, a human shortlists and
negotiates with venues for you. Note what this is — the admission that discovery alone
doesn't close a wedding booking. Same conclusion Mandap reached with WedAssist.

**Revenue:** vendor subscriptions and lead fees. The vendor is the customer, which is why
listing quality varies so much.

---

## 3. Weddingz.in (OYO) — the inventory model

The one structurally closest to you, and the one worth studying hardest.

**They don't aggregate — they manage.** Weddingz signs exclusive or semi-exclusive
control of 1,000+ banquets across 30+ cities and sells the venue *plus* decor, catering,
photography and makeup as one managed package with a dedicated planner. They give venue
owners a **Banquet Management System** — a mobile tool holding enquiries, bookings and the
event calendar, with no app install.

Two things to take from this:

**They solved availability by owning the relationship, not by asking nicely.** Calendar
accuracy came bundled with a system the venue actually needed to run its business. The
calendar was a side effect of being useful, never the pitch.

**Their revenue is the services attached to the venue, not the venue.** A hall booking is
low-margin and once-per-customer. Decor, catering and photography on top of it is where
the money is. You already have that team — which means the marketplace's job is to
generate serviced weddings, not to earn listing fees.

---

## Side by side

| | Mandap.com | WedMeGood | Weddingz.in |
| --- | --- | --- | --- |
| Model | Directory + lead gen | Directory + concierge | Managed inventory |
| Availability shown | No — SEO claim only | No | Partial, via venue-side BMS |
| Primary CTA | Gated phone number | Enquiry form | Planner assigned |
| Booking completes | Offline, by phone | Offline, by phone | On-platform |
| Pricing shown | Per day, headline | Per plate, veg/non-veg | Package quote |
| Who pays them | Venue (listing tiers) | Vendor (subscription) | Commission + service margin |
| Services attached | WedAssist upsell | Genie upsell | In-house, core |
| Traffic engine | Area-level SEO pages | Brand + content | Paid + retail stores |

---

## What this means for wedding2day

**Everyone converges on the same endpoint: a human on the phone selling services.**
Mandap has WedAssist, WedMeGood has Genie, Weddingz has planners. Discovery is the
acquisition cost; services are the business. You have the services team already, so you're
starting where they each had to buy their way to.

**Inquiry-first was the right call** — it matches what every competitor actually does,
regardless of what their SEO title tags claim.

**Copy Mandap's data model, not their execution.** Their field set — capacity two ways,
parking split car/bike, explicit outside-vendor policies, payment split, price *type* — is
years of learning what venue owners get asked. Worth diffing against `mandaps` and
`mandap_amenities` in the schema. Their 8-album photo taxonomy is a ready-made shot list
for your venue visits.

**Their weakness is the gap.** Placeholder images, one review per venue, AI descriptions
that contradict the venue's own policy table. Fifteen venues with real photographs, real
policies and real reviews beats 860 stubs on the only axis a bride cares about.

**Don't fight on SEO breadth.** Mandap.com has area-level pages for every neighbourhood
in Chennai and a footer link farm across 100 cities. You will not out-page them. Win one
city on depth and on being the platform that also shows up and runs the wedding.

---

## Sources

- [Mandap.com — Chennai kalyana mandapams](https://www.mandap.com/chennai/kalyana-mandapams)
- [Mandap.com — Rajeshwari Navaraj Mahal, Ambattur (venue detail page read in full)](https://www.mandap.com/chennai/rajeshwari-navaraj-mahal-in-ambattur)
- [WedMeGood — Chennai wedding venues](https://www.wedmegood.com/vendors/chennai/wedding-venues)
- [WedMeGood — Genie concierge](https://www.wedmegood.com/genie)
- [VenueLook — list your venue](https://www.venuelook.com/venues/list-your-venue)
- [VenueLook — about](https://www.venuelook.com/aboutus)
- [OYO blog — Weddingz.in Wz Prime and the banquet management system](https://www.oyorooms.com/officialoyoblog/2021/04/06/oyos-weddingz-in-launches-wz-prime-an-online-platform-to-help-wedding-banquets-across-india-recover-business)
- [Knowlarity — Weddingz.in lead generation via virtual numbers](https://www.knowlarity.com/customer-stories/oyo-weddingz-is-using-knowlarity-virtual-number-solution)
- [YourStory — VenueLook](https://yourstory.com/2016/06/venuelook)

# The plan — in simple words

Last updated 28 July 2026. Supersedes the sequencing in `go-to-market/LAUNCH-SEQUENCE.md`.

---

## The business in one paragraph

We make money from one thing: **running weddings**. Decor, catering, photography, makeup.
Everything else on the site exists to find families who are about to have a wedding.
Mandapam listings are free and earn us nothing directly — they are how we find those
families. A family booking a hall is guaranteed to need everything else, and they tell us
their date, their guest count and their budget while they do it. The hall is the hook.
The wedding is the sale.

---

## The four parts

The work splits into software and ops. Software is nearly finished and is not the risk.
Ops is everything that is left, and it splits three ways.

**Software** — what Cursor builds.
**Ops / Supply** — get mandapams and vendors signed up.
**Ops / Demand** — get families to the site.
**Ops / Delivery** — turn a lead into a paid wedding.

Supply, demand and delivery run **at the same time**, not one after another. Waiting for
supply before starting demand wastes months.

---

## 1. Software

Nearly done. 97 tests passing, build clean. What is left:

**Needed before anything goes live**

- W-018 image upload, so venue and package photos can be stored
- W-031 deploy to Cloudflare, point the domain
- W-033 rate limit the inquiry form, since it will be public
- W-036 move deploys and migrations to CI
- W-029 venue prospect import, so the call list loads from CSV

**Needed as venues come on**

- W-017 listing create and edit
- W-020 admin listing review
- W-019 manager calendar — built, but our ops person is the first user, not the venue owner

**Changes from what was planned**

- **Do not show a public availability calendar yet.** W-009 and W-010 assume the calendar is accurate. It won't be at first. An empty date shown as "available" that turns out to be booked loses the family permanently. Show known booked dates as unavailable and everything else as "availability on request."
- **Add a path from a venue booking to a services offer.** Right now the two lines never touch. That path is the entire business model. Mandap.com puts a planning offer on every venue page; we have a better one because we actually run weddings.
- Add `availability_checked_at` so we can see which venue calendars have gone stale.

Nothing here is hard. None of it is the reason we're not live.

---

## 2. Ops / Supply — mandapams and vendors

Two different jobs that both live on this side.

### Mandapams

The hall listings. Third-party, we don't own them.

**The loop:**

1. Scrape and collect the list — name, area, phone, capacity. Google Maps, JustDial, Sulekha, driving around, and the services team's own contacts.
2. Load it into `venue_prospects`. **This stays private. Nothing scraped ever gets published.**
3. Call every one.
4. Visit anyone interested. Take our own photographs.
5. Collect the full details on that visit — see the intake spec.
6. Publish only what the owner handed us.

**Roughly 60–80 calls gets 15–20 signed venues.** That ratio is the planning number.

**A venue is only "done" when it has:** our own photos, real capacity, real per-day rate,
the outside-vendor policies, and **a named person who has agreed to answer within a day.**
Agreeing on the phone is not done.

### Vendors

We cannot put our own team in every city in Tamil Nadu. So in each city we tie up with
local decorators, caterers, photographers and makeup artists.

**How it works:** the family buys from wedding2day. We coordinate. The local vendor
delivers. We keep the margin. The vendor is not listed on the site and the customer does
not deal with them directly — this is a fulfilment network, not a second marketplace.

Our in-house team stays the standard and covers our home city.

**The real risk here is quality.** Our brand is on the wedding, not theirs. One bad
decorator in Salem damages Chennai. So vendors get signed the same way venues do —
visited, checked, and dropped fast if they fail once.

**Signing venues and signing vendors happen on the same trip.** A mandapam owner gets
asked for decorator and caterer recommendations constantly. Getting onto that list is
worth more than the listing itself.

---

## 3. Ops / Demand — where families come from

This is the part no earlier document covered, and it is the biggest open risk.

**We cannot win SEO against Mandap.com.** They already own every neighbourhood page in
Chennai — `kalyana-mandapams-in-ayappakkam`, `-in-padi`, `-in-surapet` — with
Matrimony.com's domain authority behind them. Venue-listing SEO is a twelve-month bet
against an incumbent. Not the first move.

Three channels, in the order they produce money:

**Instagram and the team's referral network.** We have photographs of real weddings we
actually ran. Mandap has placeholder images and one review per venue. This converts in
weeks and costs nothing. Start here.

**Muhurtham content.** Tamil families pick the date first and the hall second. A proper
muhurtham calendar for the coming year, with a cost guide attached, reaches people months
before they start looking for a venue — and nobody's AI-written venue page competes with
it.

**Google Ads on high-intent terms**, once we know what a lead is worth.

---

## 4. Ops / Delivery — lead to paid wedding

Where every rupee is made, and currently undefined.

**Must be decided before we go live:**

- Who answers an inquiry, by name
- How fast — an inquiry sitting two days is a lost wedding
- What gets quoted, and who is allowed to discount
- Who runs the wedding on the day
- Who collects the money, and on what terms

The admin inbox at `/admin/inquiries` already tracks this pipeline. It only works if a
named person opens it every day.

---

## Order of work

**Now** — packages go live. Real prices, real photos of our own weddings, W-018 / W-031 /
W-033 / W-036 shipped. Line B needs nobody's permission and can earn while everything else
is being built. Instagram and referrals start the same week.

**Same time** — start calling mandapams and signing vendors in the first city.

**Then** — open the marketplace in that city once 15–20 venues are live and responsive.
Test it by sending real booking requests to at least three of them. If a human doesn't
reply to us, they won't reply to a customer.

**Then** — repeat the loop in the next city.

**On covering all of Tamil Nadu:** that's the target, and it's the right one given the
vendor tie-up model. But **do not wait for the whole state before going live.** Five
cities is 300–400 calls and months of work. Launch each city as it fills. Same end state,
revenue starts far sooner.

---

## What's decided

- Tamil Nadu, all of it, city by city
- Venue listings are free, permanently — we make money on services, not listings
- Inquiry-first. No instant booking
- Scraped data stays private. We publish only what an owner gave us
- Vendors are a fulfilment network, not a public listing
- Packages launch before the marketplace

## What's still open

- Which city first
- Who answers inquiries, and how fast
- Whether venues get a referral cut when we service a wedding at their hall
- What a lead is worth, which decides the ad budget

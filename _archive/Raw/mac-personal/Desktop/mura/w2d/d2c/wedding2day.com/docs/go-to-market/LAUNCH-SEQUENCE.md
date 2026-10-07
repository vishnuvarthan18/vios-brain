# Launch sequence — packages first

The marketplace cannot launch until venues exist, and venues take weeks of phone calls.
The packages business has no such dependency: the team is real, the services are real,
and the code is done. So it goes first.

This gets Line B taking real enquiries while Line A fills up in parallel.

---

## Why this order

Line B needs nothing from anyone outside your company. You already deliver decor,
catering and photography. Putting that online is a matter of writing down what you
actually sell and deploying code that already works.

Line A needs fifteen to twenty venue owners to say yes. That is the longest lead time
in the whole project and it has not started.

Running them sequentially wastes the weeks of calling. Running them in parallel means
Line B is earning while Line A is still being built.

There is a second reason. Enquiries from Line B tell you which cities, budgets and dates
people actually want — which tells you which venues to prioritise signing. Launching
packages first makes the venue work better targeted.

---

## Phase 1 — Get Line B live

**Replace the invented packages with your real ones.**

The three packages in the database are placeholders I made up. Sit down with whoever
prices your work and write the real thing: what's in each tier, what it costs, what
guest count it suits. If you don't sell in tiers today, this is the moment to decide
what your two or three standard offerings are.

This is business work, not code. It's also the single highest-value hour in this phase —
the packages page is your sales pitch, and invented numbers on it are worse than nothing.

**Get real photographs of your own work.**

Package images are currently placeholders. You have done weddings; you have photographs
of them. Pick the best fifteen or twenty. Get permission from the families before
publishing anything recognisable.

This is what makes the page credible. A wedding services page with stock imagery
convinces nobody.

**Ship the remaining infrastructure tickets.**

- W-018 — image upload to R2, so photos can actually be stored
- W-035 — admin CRUD for package images
- W-031 — deploy to Cloudflare Pages, point the domain
- W-036 — move production migrations and deploys to CI, then log out of `wrangler` locally
- W-033 — rate limit the inquiry form, since it will be publicly reachable

W-033 stops mattering in theory and starts mattering in practice the moment the form is
on the open internet.

**Decide who answers enquiries, and how fast.**

An enquiry form nobody watches is worse than no form — the customer concludes you're not
serious. Name a person. Agree a response time. Put both in writing.

The admin inbox at `/admin/inquiries` already tracks the pipeline. It only works if
someone opens it.

---

## Phase 2 — Venue onboarding, running in parallel

Start this the same week as Phase 1. Do not wait for the deploy.

- Pick the launch city (see `SOURCING.md` — go where your services team already works)
- Collect 60–80 prospects, import via W-029
- Work the list using `OUTREACH-KIT.md`
- Visit anyone who shows interest; take your own photographs
- Target 15–20 fully onboarded venues

Meanwhile the remaining marketplace code lands: W-017 listing CRUD, W-019 manager
calendar, W-020 admin listing review. Those are what let venue owners manage themselves
instead of you doing it for them — necessary before you pass about ten venues.

---

## Phase 3 — Open the marketplace

Only when the launch gate is met: 15–20 verified venues, each with real photos, real
pricing, and a named person who has agreed to respond within a day.

Before opening publicly, send a real booking request to at least three venues and see
whether a human replies. If they don't answer you, they won't answer a customer.

Then W-026 to W-028 — reviews — which only become meaningful once real bookings have
completed.

---

## What to measure

**Line B:** enquiries per week, and enquiry-to-won rate. If enquiries come but nothing
converts, the problem is pricing or follow-up speed, not the website.

**Line A:** manager response rate within 48 hours. This single number decides whether
the marketplace works. Track it from the first real booking. Any venue that repeatedly
ignores requests should be unpublished — one unresponsive venue costs you more customers
than it brings.

Ignore total listings and total signups. They feel like progress and predict nothing.

---

## The decision still outstanding

**Revenue model.** Currently free for everyone, deliberately.

Free is the right call while you have no traffic — it's what makes the outreach pitch
work. But decide before you have enough venues to have leverage, because introducing a
fee to a hundred established venues is far harder than starting with one.

The three realistic options, and the honest problem with each:

**Commission per confirmed booking** — aligns you with venue success, but with money
settling offline you cannot verify a booking happened. Venues will under-report. This
only works if you eventually handle payment.

**Listing or subscription fee** — predictable and simple to collect, but you're charging
for something the venue already gets free on ten other platforms. Hard to justify until
you're demonstrably sending bookings.

**Services attach rate** — charge nothing for the venue listing, and make your money on
decor, catering and photography attached to those bookings. This is the one that fits
what you actually are. The marketplace becomes customer acquisition for the services
business rather than a business itself.

That third option is worth serious thought. It reframes the whole product: you're not
competing with Mandap.com for listing revenue, you're using venue discovery to feed a
services company that already has margin. It also explains why you'd give venues a
referral cut — they become a sales channel, not a customer.

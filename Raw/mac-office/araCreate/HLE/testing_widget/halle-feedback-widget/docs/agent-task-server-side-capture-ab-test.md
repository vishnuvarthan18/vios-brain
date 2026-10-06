# Agent task — server-side screenshot: compare it against today's method

**This is a quality-and-speed comparison, not a load test.** We are not
checking whether the server can handle lots of people at once — only
checking which method gives a correct, proper-looking screenshot,
quickly. Keep everything simple accordingly.

**Ground rules, non-negotiable:**

- **No server upgrade.** Build and run this on the current server as is
  — a shared box, already running other things, app already capped
  around 1GB memory.
- **Server handles one screenshot at a time.** No queue system, no
  handling-many-at-once infrastructure. We are not simulating real
  traffic here — keep it as simple as a single request in, single
  screenshot out.
- **Today's method (client draws the picture) keeps working exactly as
  it does now.** The new method is added alongside it for comparison,
  not instead of it.
- **Do not commit or push without asking Vishnu**, as always.

Background, read first:
- `docs/research-how-the-industry-solved-this.md` §1 and "Route 2".
- `docs/report-screenshot-speed-500ms.md` (or the project summary,
  `claude/report-screenshot-speed-500ms-summary.md`) — Route 1's result:
  1,580ms as committed, 152ms only by weakening font accuracy.

## Why this task exists

Every mobile-capable competitor in this category (Marker.io, Ybug,
Usersnap, Userback) draws the picture on their own server, not in the
tester's browser. It may be both faster and better for font accuracy
than Route 1. This task exists to find out, directly: for the same
click, does the server-drawn picture actually look right, and is it
actually faster — on our real server, without expanding it.

## What to build

1. **For each test click during this testing phase, capture the
   screenshot both ways** — today's method, and the new server method —
   from the same click, same page state. This gives a direct,
   apples-to-apples comparison instead of comparing different clicks
   taken at different times.
2. **Save both pictures somewhere Vishnu can view side by side**
   (alongside the report, or in a simple log), together with how long
   each one took to produce.
3. **Server side stays simple.** Build one renderer that handles one
   screenshot request at a time. It:
   - Takes the lightweight page description (HTML and CSS, already
     privacy-masked) sent from the tester's browser.
   - Fetches the images and fonts it needs directly from the public
     Webflow site itself (works with no special access, since the site
     is public).
   - Draws the picture and sends it back.
   - No concurrency handling, no queue for bursts — not needed for this
     comparison test.
4. **If the server method fails, that's fine — just log it as a
   failure.** The tester's actual report still goes out using today's
   method's picture, so nothing is ever blocked
   (`agent-rules.md §1.11`).
5. **Privacy masking still runs first**, before anything leaves the
   tester's browser.
6. **Marker pen draws on top of whichever picture is actually used**
   for the real report — unchanged.

## Explicitly not in scope

- No load testing, no concurrency handling, no traffic splitting across
  real users.
- No server upgrade or new hosting.
- No change to the marker pen, review screen, or the 16-screens work.
- Do not remove or change today's method — it stays as the working
  default; the server method is only for this comparison.
- Do not pick a winner. Report the side-by-side results and let Vishnu
  decide.

## Report back

- For a handful of real test clicks on the real Contact page: both
  pictures shown side by side, and the time each one took.
- Any case where the server-drawn picture looked wrong or broken.
- A plain recommendation: which one looks more correct/proper, and which
  is faster.

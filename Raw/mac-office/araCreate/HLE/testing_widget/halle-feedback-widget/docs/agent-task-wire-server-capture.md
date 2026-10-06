# Agent task — connect the widget to the new server-side capture, carefully

## Where things stand

`docs/agent-task-install-browser-libs-on-prod.md` is done — the production
server can now launch a real headless browser and take real screenshots.
`docs/agent-task-hybrid-server-capture-feasibility.md` already proved the
core idea works: take the exact clone `capture.ts` already builds (already
proven pixel-correct) and photograph it with a real browser instead of the
buggy SVG trick, instead of anything client-side.

This task wires that up for real, carefully, with safety limits — this is
a shared, memory-tight server (RAM dipped to 239MB free just from
installing packages, no swap at all), so this is not just "call a new
endpoint," it needs real protection against overloading the box.

## What to build

1. **A new internal endpoint** on the existing `halle-feedback` app (don't
   stand up a separate service unless there's a good reason not to) that:
   accepts the clone's HTML (the same thing that currently goes to
   `modern-screenshot` in the browser), renders it with the now-working
   headless browser, and returns the image.
2. **The widget** (`capture.ts`) sends the clone to this endpoint instead
   of rendering it in-browser, and uses the image that comes back.
3. **Hard safety limits on the new endpoint, non-negotiable given the
   server's tight memory:**
   - A concurrency limit — at most 1-2 real browser renders happening at
     once, everything else queued or rejected. Do not let unlimited
     simultaneous requests each spin up a browser render.
   - A reasonable max size on the incoming HTML payload.
   - A timeout on the render itself (a stuck render must not hang forever
     and slowly exhaust memory).
   - Basic protection against this becoming an open, abusable endpoint —
     it should only be reachable by the widget's own capture flow, not
     something a random visitor to the internet could hit repeatedly to
     load up the server. Use whatever's simplest and consistent with how
     the rest of this app is already secured (check existing auth/token
     patterns in the codebase before inventing a new one).
4. **Fail gracefully, matching this project's existing rule** (never block
   a report on a failed screenshot, `agent-rules.md §1.11`): if the server
   render fails, times out, or the server is unreachable for any reason,
   **fall back to today's client-side method** rather than losing the
   screenshot entirely or blocking the report. This fallback path already
   basically exists (it's just what `capture.ts` already does) — the new
   server path is additive, not a replacement that removes the old one.

## Test before calling this done

1. Test the whole flow end to end locally first.
2. Measure and report the real total time a tester would experience (not
   just the render step) — compare honestly against the ~360-480ms
   estimate from the earlier feasibility test.
3. **Watch server memory during a real render**, not just during the
   library install — confirm it doesn't come dangerously close to
   exhausting the 239MB-free floor already observed. If it does, the
   concurrency limit needs to be tighter, not shipped as-is.
4. Confirm the specific bugs this was meant to fix (the shifted navy card,
   the growing text box on `/contact`) are actually gone when captured
   through the real, deployed flow — not just in an isolated test.
5. Confirm the fallback actually works: deliberately make the server
   unreachable or slow in a test and confirm the widget still produces a
   (less accurate but present) screenshot instead of failing silently with
   nothing.

## Commit, don't push, don't deploy without being told

Commit as its own clear change. **Do not push, and do not deploy this to
the point where it affects real widget traffic**, until Vishnu has seen
the numbers and explicitly says to go live. This is a bigger, riskier
change than anything shipped today — treat it accordingly.

## Report back

- Confirmation the end-to-end flow works, with real numbers (total time,
  memory used during a render, memory used with the concurrency limit
  maxed out).
- Confirmation the fallback path works when the server side fails.
- Confirmation the original bugs are actually fixed through the real flow.
- Anything about the safety limits you had to make a judgment call on.

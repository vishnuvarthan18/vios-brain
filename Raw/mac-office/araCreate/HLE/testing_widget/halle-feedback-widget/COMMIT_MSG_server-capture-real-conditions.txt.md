---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_server-capture-real-conditions.txt
---

test: re-test server capture under real conditions

Answers docs/agent-task-server-capture-real-conditions-test.md. The first
result was measured on loopback against a warm, idle renderer, so it was a
best case for the server method rather than a fair one. This closes that gap
as far as it can be closed without a decision from Vishnu.

The renderer ran on the production box and worked. Chromium did not: the box
is missing nine shared libraries, and supplying them means 19 new system
packages including dbus-user-session, at-spi2-core and dconf-service —
session and desktop infrastructure on a VPS shared with JupyterHub, four
Python services and another Node app. The ground rule is that nothing
permanent is added to that box, so the test stopped there and reported rather
than installing them. All 15 server attempts failed at browser launch; none
was a rendering fault.

Today's method was measured under the same real conditions: 444/453/546ms
across the three clicks, consistent with the first test's 393/409/527ms.

The network question is answered even without server timings. Round-trip time
to the box is ~260-350ms, against the 4ms the first report measured on
loopback, so real conditions can only widen a gap the first report already
called a floor.

The app was never at risk and never restarted: NRestarts=0, peak 251MB of its
1024MB cap, box never below 986MB available, sampled every second across 323
samples. Worth recording separately: the box has no swap at all, so there is
no cushion before the OOM killer. The box was left clean — 677MB test
directory removed, no shared Playwright cache, no stray processes, port free,
and apt was never invoked.

One real bug fixed on the spot. The renderer was binding 0.0.0.0 rather than
loopback, so on the production box it was briefly listening on the public
interface while accepting JSON payloads it renders as HTML in a headless
browser. Only an upstream provider firewall was dropping the traffic, and a
firewall we did not configure is not a security posture. RENDER_HOST now
defaults to 127.0.0.1 and the startup line names the interface. No rendering
or comparison logic changed.

Also added, with the comparison logic itself left alone as the task asks:

- The network/render split, measured per pass rather than subtracted from two
  medians, from the renderer's own x-render-ms header.
- A memory gate that exits non-zero and blocks staging, plus a watcher
  sampling every second — a before/after pair would step over the spike that
  is the actual risk.
- Staging and teardown. Chromium installs into TEST_ROOT via
  PLAYWRIGHT_BROWSERS_PATH, never a shared cache and never apt, so teardown
  is one delete; teardown then checks the shared caches anyway.
- make ab-capture-real and make ab-tunnel, writing to docs/ab-capture-real so
  the first report's evidence stays intact. A localhost renderer URL is now
  labelled as loopback or as a tunnel, since the numbers differ enormously
  and the URL cannot distinguish them.

Two teardown bugs found by running it: pkill -f matched the SSH session
carrying the teardown and killed the connection mid-run, twice. The kill now
filters out this script, its parents and the session leader. Self-tested.

Today's method remains untouched and remains the only source of a real
report's picture. All 51 widget acceptance tests pass; lint and tsc clean.

Decided: keep today's method. Route 2 is not adopted, the real-conditions
testing stops here, and nothing is to be installed on production. The two
reports and the runbook record that outcome so the question is not reopened
by someone reading them later. The harness is kept because it stays useful if
the tester's device ever becomes the constraint — the one result the first
report said would justify revisiting this.

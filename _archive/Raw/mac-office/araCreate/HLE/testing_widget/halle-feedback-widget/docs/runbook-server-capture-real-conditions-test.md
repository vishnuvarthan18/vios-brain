# Runbook — the real-conditions render test

> **CLOSED, 10 September 2026 — Vishnu decided to keep today's method.**
> Do not run the steps below, and **do not install anything on production**.
> Kept for the record, and because the harness itself stays useful if the
> tester's device ever becomes the constraint (see the first report's
> "one result that would justify revisiting this").

Answers `docs/agent-task-server-capture-real-conditions-test.md`. Everything
below is prepared and verified; the **run itself is blocked on SSH access to
the production box** (see "Blocked on" at the end).

The comparison logic is unchanged, exactly as the task asks — "only where the
renderer lives and what URL the harness points at". What was added is the
memory gate, the network/render timing split, and the teardown.

## The order, and why it is this order

| Step | Where | Command |
| --- | --- | --- |
| 1 | the box | `scripts/ab/preflight-memory.sh` |
| 2 | the box | `scripts/ab/stage-server-test.sh` |
| 3 | the box, 2nd shell | `scripts/ab/watch-memory.sh` |
| 4 | your machine | `AB_SSH_HOST=you@212.227.213.174 make ab-tunnel` |
| 5 | your machine, 2nd shell | `AB_TUNNEL_OK=1 make ab-capture-real` |
| 6 | the box | `scripts/ab/teardown-server-test.sh` |

**Step 1 is not optional and is not advisory.** The ground rule is to check
free memory "before running anything", and to stop rather than push through if
the renderer would risk the app. So the gate is a script that exits non-zero,
and `stage-server-test.sh` runs it again itself and refuses to install
anything if it fails. A non-zero exit means *report it*, not *retry smaller*.

The gate is set at **600MB** available: ~415MB measured last time
(renderer ~95MB + Chromium ~320MB) plus headroom, because Chromium's peak is
spikier than its steady-state RSS.

## How the three "real conditions" are met

1. **Renderer on the actual production box** — `stage-server-test.sh` puts it
   there, temporarily.
2. **The browser is somewhere else, over the real internet** — the test runs
   on your own machine against the real Contact page, and its ~410KB payload
   travels to Germany and back. The task explicitly allows this:
   "Running the test from a developer's own machine, pointed at the real
   server's address instead of `localhost`, satisfies this."
3. **The server is not artificially idle** — nothing is stopped. The app keeps
   serving throughout, and `watch-memory.sh` records what else was competing.

### Why a tunnel, and why that still counts as a real network trip

The renderer binds **127.0.0.1 only**, and is reached through
`ssh -L 4599:127.0.0.1:4599`. Nothing is exposed publicly and no firewall rule
is opened — which is what "reachable only for this test, not exposed
permanently or publicly beyond what's needed to run it once" asks for.

The payload still crosses the real internet: the tunnel is a TCP connection to
Germany, so the ~410KB upload and the image's return trip pay real latency and
real bandwidth. SSH adds encryption overhead the real widget would not have
(it would use HTTPS, which has its own), so **the network figure is a close
estimate, not an exact production number** — worth stating in the report
rather than implying a precision the setup does not have.

## The number the re-test exists to produce

The harness now measures the split **per pass**, not by subtracting two
medians:

- `render_ms` — the renderer's own time, measured on the server, returned in
  its `x-render-ms` header.
- `network_ms` — the browser's full round trip minus `render_ms`. That is
  everything that is *not* rendering: TCP, the payload's upload, the image's
  trip back. Clamped at zero, because the two clocks are on different machines
  and a few ms of skew must not read as negative travel.

On loopback this was **~2–4ms and invisible**. Verified locally after the
change (1 pass, loopback): `render 1326ms / network 2ms`, and the pixel
figures reproduced the first report exactly (7.71% / 4.85% / 0.61%) — so the
instrumentation was added without disturbing the comparison.

The summary also prints the renderer's URL and warns when it is loopback, so a
pasted result can never be mistaken for the real-conditions run. Output goes
to `docs/ab-capture-real/`, leaving the first report's evidence intact.

## Leaving the box clean

`teardown-server-test.sh` removes only what the test added. Two choices in
`stage-server-test.sh` are what make that possible:

- **Everything under one directory.** Chromium is installed into `TEST_ROOT`
  via `PLAYWRIGHT_BROWSERS_PATH`, not into `~/.cache/ms-playwright` and not
  into `/usr`. Teardown is one recursive delete, and teardown *also checks the
  shared cache locations* and reports anything present rather than assuming.
- **No apt, and no `--with-deps`.** That flag installs system libraries and is
  exactly the "nothing permanent added" violation. If Chromium then will not
  launch for a missing shared library, **stop and report it** — do not reach
  for apt. That is a real possibility on this box and the honest outcome is a
  report, not a workaround.
- **No systemd unit.** A unit would survive a reboot. This is a plain
  background process with a pidfile.

Teardown also prints the app's `ActiveEnterTimestamp`, which should predate the
test — if the app restarted during the run, that goes in the report.

## Status — run 10 September 2026

Ran. Results in `docs/report-server-capture-real-conditions-test.md`.

SSH access works (`(secret removed)`, key `~/.ssh/halle_agent`). The
memory gate passed at 1,078MB available, the renderer started on the box and
answered `/health`, and the app stayed safe throughout (peak 251MB of 1,024MB,
`NRestarts=0`).

**It stopped at Chromium.** The box is missing nine shared libraries
(`libnspr4.so` first), and the caveat two sections above turned out to be the
real outcome: supplying them means **19 new system packages**, including
`dbus-user-session`, `at-spi2-core` and `dconf-service` — session and desktop
infrastructure on a box shared with JupyterHub and other tenants. That is a
decision for Vishnu, not a workaround to apply, so the test reported instead
of installing them.

The box was left clean and `apt` was never invoked. Re-running steps 1-6 will
reach the same `libnspr4.so` error until that question is decided; the report
sets out the three options.

Also fixed during the run: the renderer was binding `0.0.0.0` instead of
loopback. `RENDER_HOST` now defaults to `127.0.0.1`.

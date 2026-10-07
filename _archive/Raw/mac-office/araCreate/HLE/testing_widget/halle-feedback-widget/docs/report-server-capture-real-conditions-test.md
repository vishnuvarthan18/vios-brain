# Report — server-side capture, re-tested under real conditions

10 September 2026. Answers
`docs/agent-task-server-capture-real-conditions-test.md`.

> **DECIDED, 10 September 2026 — Vishnu: keep today's method.** The
> real-conditions testing stops here and **nothing is to be installed on
> production**. The three options at the end of "Why the server method could
> not run" are closed: option 3 was taken. Route 2 is not adopted, and this
> report is kept as the evidence behind that decision rather than as an open
> question. Nothing further is planned on this task.

**Headline: the test could not be completed, because Chromium will not run on
the production box without installing 19 system packages — and that decision
is yours, not mine.** The renderer ran there fine; the browser it needs did
not. Today's method was measured under the same real conditions and came in at
**444–546ms**, consistent with the first test.

The app was never at risk: it never restarted, and peaked at **251MB of its
1,024MB cap**. The box was left clean — nothing installed, nothing running,
`apt` never invoked.

## Results

| Click (scroll) | Today's method | Server method | Pixels differing |
| --- | --- | --- | --- |
| top-of-page (0) | **444ms** | could not run | — |
| form-and-address (400) | **453ms** | could not run | — |
| page-bottom (1,131) | **546ms** | could not run | — |

Medians of 5 passes. Client failures: **0 of 15**. Server failures:
**15 of 15**, all the same launch error, none of them a rendering fault.

Today's method here (444/453/546ms) against the first test's
(393/409/527ms): slightly slower, same shape. The difference is this run's
browser being a laptop on a home connection rather than the same idle machine
as the renderer — the client method fetches the page's own images and fonts,
so its network matters too. `top-of-page` pass 1 was 2,257ms and the rest
424–449ms; that first pass is cold-cache and the median discards it.

## Why the server method could not run

Chromium exits immediately with:

```
chrome-headless-shell: error while loading shared libraries:
libnspr4.so: cannot open shared object file: No such file or directory
```

Nine libraries are missing from the box:

```
libXdamage.so.1  libasound.so.2  libatk-1.0.so.0  libatk-bridge-2.0.so.0
libatspi.so.0    libnspr4.so     libnss3.so       libnssutil3.so
libxkbcommon.so.0
```

The renderer itself was **fine** — it started, bound its port, answered
`/health`, accepted all 15 payloads, and returned a clean `500` with a logged
reason each time rather than crashing or hanging. Route 2's server-side code
is not what failed here; the box simply has no browser to give it.

### What installing them would cost

`apt-get install -s` (simulation only — nothing was installed):

```
0 upgraded, 19 newly installed, 0 to remove and 8 not upgraded.
```

The nine libraries pull in 19 packages, and they are not all leaf libraries:

- **`dbus-user-session`** — changes session handling for every user on the box.
- **`at-spi2-core`** — an accessibility bus daemon, system-wide.
- **`dconf-service`, `dconf-gsettings-backend`, `gsettings-desktop-schemas`** —
  desktop configuration infrastructure.
- plus `alsa-*`, `xkb-data`, `libnss3`, `libnspr4` and the rest.

This is desktop and session infrastructure on a **production VPS shared with
JupyterHub, four Python services and another Node app**. The ground rule is
"nothing permanent added to the production box... do not leave a Chromium
install sitting on the production server", and `agent-rules.md §4` requires
asking before adding a dependency.

Nineteen system packages including a session bus is not something to
install-then-remove quietly and hope nothing else on the box notices —
`dbus-user-session` in particular affects other tenants, and removing it later
is not reliably a no-op. **So I stopped and am reporting instead of pushing
through**, which is what the ground rule asks for in the memory case and
applies just as well here.

**Superseded by the decision above — option 3 was taken, and none of the
following is to be actioned.** Recorded as the reasoning that was available
at the time:

1. **Install the 19 packages temporarily**, measure, then remove them. Honest
   risk: `dbus-user-session` and `at-spi2-core` touch the shared box, and
   uninstalling is not guaranteed clean.
2. **Measure on an identical throwaway VPS instead** — same provider, same
   Debian 12, nothing shared. Costs a euro or two and answers the rendering
   question without touching production. It does not answer "how does the
   *real* box behave under its *real* load", but nothing else about it is
   different.
3. **Leave it.** The first test already showed today's method 2.4× faster in
   the server method's own best case, and the network figure below shows real
   conditions can only widen that.

## Network travel versus rendering

The task asks how much of the server method's time is genuinely network. The
server method produced no timings, so this cannot be reported as measured —
but the trip itself was measured, and it settles the question anyway.

**Round-trip time from the test machine to the box: ~260–350ms, typically
~310ms** (five TCP handshakes; ICMP is blocked, so this is a real connection
setup, not a ping).

That is the floor for a *single* round trip, before the ~410KB payload's
upload time or any rendering. The first report measured the network portion at
**4ms on loopback**; here the same portion starts at **~310ms** and rises with
upload time on an asymmetric connection.

Set against the first report's rendering figure of ~1,027ms — which was on an
idle machine, and this box is not idle — the server method's real-conditions
total would land **well above 1,300ms**, against today's **444–546ms**
measured on the same connection. The gap the first report called a floor is
confirmed to be one.

Two honest caveats: the ~310ms is my connection to Germany, not a
representative tester's; and the tunnel adds SSH encryption the real widget
would not pay (it would pay HTTPS instead). Neither changes the conclusion's
direction.

## Memory, and the app's safety

The gate ran first, before anything was installed, per the ground rule:

```
MemAvailable: 1078MB   SwapFree: 0MB
Route 2 needs: ~415MB measured, gate set at 600MB
RESULT: OK to proceed.
```

**Note `SwapFree: 0MB` — this box has no swap at all.** There is no cushion
between memory pressure and the OOM killer, which is worth knowing
independently of this test.

Sampled every second throughout (323 samples,
`docs/ab-capture-real/memwatch-server.log`):

| | Worst observed |
| --- | --- |
| App's own cgroup | **251MB** of its 1,024MB cap |
| Box MemAvailable | never below **986MB** |
| Renderer process | 110MB |
| Chromium | 0MB — never launched |

**The app never restarted:** `NRestarts=0` and `ActiveEnterTimestamp=Thu
2026-09-10 01:16:15 UTC`, the same before and after, so it was untouched
throughout. Nothing was stopped to make room — the app, JupyterHub and the
Python services all kept running, as "not artificially idle" requires.

The ~415MB Chromium cost was never actually incurred, so this run does **not**
confirm the box has room for the server method under load. That question is
still open and depends on option 1 or 2 above.

## The box was left clean

Verified after teardown:

| Check | Result |
| --- | --- |
| `/tmp/halle-ab-render-test` (677MB) | removed |
| `/root/.cache/ms-playwright` | absent |
| `/usr/local/ms-playwright`, `/home/*/.cache/ms-playwright` | absent |
| Stray Chromium profile dirs in `/tmp` | none |
| Renderer / Chromium processes | none |
| Port 4599 | free |
| `/var/log/apt/history.log` | **nothing from this test** — last entries are unattended-upgrades, 9 Sept |
| App | `active`, `NRestarts=0`, unchanged start time |
| Disk | back to 11G / 103G free |

Chromium was installed into the test directory via
`PLAYWRIGHT_BROWSERS_PATH`, never a shared cache, so removal was one delete.
`--with-deps` was deliberately not used — that flag is what would have run
apt.

## One thing fixed on the spot, and it matters

**The renderer was binding `0.0.0.0`, not loopback.** `server.listen(PORT)`
with no host argument binds every interface, so for a few minutes the renderer
was listening on the production box's public interface. It accepts a JSON
payload and renders arbitrary HTML/CSS in a headless browser, so it must never
be publicly reachable.

It was not in fact reachable from outside — an upstream provider firewall was
dropping the traffic (there are no local `iptables`/`ufw` rules). But "a
firewall we did not configure happens to save us" is not a security posture,
and this would have been a live exposure on any box without that firewall.

Fixed in `src/render/renderer.mjs`: the host is now `RENDER_HOST`, defaulting
to `127.0.0.1`, and the startup line prints the interface it bound. Verified
on the box (`LISTEN 127.0.0.1:4599`) and locally. This changes no rendering or
comparison logic — loopback is what `make ab-render` already used in practice.

## What was added to the harness

Comparison logic unchanged, as the task asks — "only where the renderer lives
and what URL the harness points at".

| Piece | Where |
| --- | --- |
| Network/render split, measured per pass | `scripts/ab/capture-ab.mjs` |
| Memory gate — exits non-zero, blocks staging | `scripts/ab/preflight-memory.sh` |
| Memory watcher, 1s samples | `scripts/ab/watch-memory.sh` |
| Temporary staging on the box | `scripts/ab/stage-server-test.sh` |
| Teardown, with shared-cache checks | `scripts/ab/teardown-server-test.sh` |
| `make ab-capture-real`, `make ab-tunnel` | `Makefile` |
| Runbook | `docs/runbook-server-capture-real-conditions-test.md` |
| This run's evidence | `docs/ab-capture-real/` |

`network_ms` is measured per pass, not subtracted from two medians:
`render_ms` comes from the renderer's own `x-render-ms` header, and
`network_ms` is the browser's round trip minus that. Clamped at zero, since
the two clocks are on different machines.

A localhost URL is now labelled by `AB_VIA_TUNNEL` — it is either true
loopback (the first test's best case) or a tunnel to the box (real
conditions), the numbers differ enormously, and the URL alone cannot tell them
apart.

Two teardown bugs found by running it: `pkill -f "$TEST_ROOT"` and
`pkill -f renderer.mjs` both matched the SSH session carrying the teardown and
killed the connection mid-run. Now the pids are resolved and this script, its
parents and the session leader are filtered out first. Self-tested.

## Ground rules, each one honoured

- **No server upgrade.** Nothing upgraded, nothing reconfigured.
- **Nothing permanent added.** Nothing was installed at all — the sticking
  point above is precisely the refusal to install. Verified against apt's log.
- **Memory watched like a hawk.** Gate before anything ran, 1s sampling
  throughout, app peaked at 251MB of 1,024MB, never restarted.
- **One screenshot at a time.** Unchanged; no concurrency work.
- **Today's method untouched.** `capture_screenshot()` unmodified and still
  the only source of a real report's picture. All 51 widget acceptance tests
  pass; lint and `tsc --noEmit` clean.
- **Nothing committed or pushed.** Message drafted to
  `COMMIT_MSG_server-capture-real-conditions.txt`, awaiting your instruction.
- **No winner picked.** The numbers are above; the decision is yours.

## Reproducing

```
# on the box
scripts/ab/preflight-memory.sh          # the gate — stop if it fails
scripts/ab/stage-server-test.sh         # temporary renderer
scripts/ab/watch-memory.sh              # second shell

# on your machine
(secret removed) make ab-tunnel
(secret removed) (secret removed) make ab-capture-real

# on the box, afterwards
scripts/ab/teardown-server-test.sh
```

Step 2 will fail at the first render with the `libnspr4.so` error until the
system-package question above is decided.

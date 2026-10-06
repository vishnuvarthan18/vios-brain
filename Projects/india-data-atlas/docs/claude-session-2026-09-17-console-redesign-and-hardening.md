# Session 2026-09-17 — console redesign, test suite, production hardening

_Continues `session-2026-09-16-pause-close-and-ops-console.md`. That one covers closing the observation pause, D-71, and building the console. This one covers making it good enough to trust._

Repo: `~/india-data-platform/india-ops-console` (move to `~/india-ops-console`). 74 files, 5 commits. **Still not deployed.**

## 1. Interface rebuilt around operator flow (commit `8b12deb`)

The first version was a correct site with no flow: thirteen pages, a row of links, nothing telling you where to start. Rebuilt the shell and every page.

**Organising rule: colour means something.** Grey is the whole interface; red, amber and green appear only where they carry a fact about platform health. If everything is coloured, nothing is a signal — and spotting what is wrong is the entire job of the tool.

What changed:

- **Overview** opens with one sentence on whether anything is wrong, then a ranked "needs attention" list merging job alerts, silently-empty runs and stale sources into one answer to "what do I do now" rather than three competing tables. Every row carries its own action (inspect log / run job / open engine). No row is a dead end — that gap was found during testing and fixed.
- **Privileged actions confirm first**, naming what will happen and why it is safe (harvests are idempotent). Buttons report they are working, so nobody double-fires a harvest.
- **Filters became removable chips** with a result count and a real empty state offering a way out. A record carries its filter, so "back to results" returns to your own search.
- **Job page auto-refreshes only while a run is in flight**, so an idle page is not reloading itself in a forgotten tab all week.
- **Key rotation** moved into a dialog that states the value is never logged, stored or shown again.
- Sidebar grouped by intent (watch / explore / manage) with live counts, breadcrumbs, Cmd+K command palette, g-then-key shortcuts, theme toggle defaulting to system.
- Everything degrades: with JavaScript off it is still a complete server-rendered site. No framework, no CDN, no build step.

### Contrast audit — 121 failures found, both themes now zero

Audited 14 pages in both themes **including hover states**. Light mode had 121 failures; dark had 1. Both are now 0.

Two were real design faults, not oversights:

- Secondary text (`--ink-3`) sat at **4.24:1** on the table-header surface. Darkened to `#6b6961`, which clears AA on every light background it lands on.
- Hovering an alert row *deepened* its red tint until labels fell to **3.91:1** — the exact state you are in while reading it. Hover now **lightens** the row instead.

Also verified: skip link, landmarks, scoped table headers, `aria-current`, labelled controls, visible focus, no horizontal scroll at 390px.

## 2. Test suite (commit `672c6ac`)

**184 tests, ~9 seconds.** `./scripts/run-tests.sh` starts a disposable PostGIS container, applies the real schema (needs `CORE_REPO` pointing at india-data-core), seeds a fixture, runs pytest, tears down.

**85 tests need no database at all** — including the whole control-service suite. The highest-consequence code should be the easiest thing to check.

| File | Covers |
|---|---|
| `test_control_service.py` | 24 hostile unit names, rotation refusals, log forging, allowlist re-read |
| `test_precision.py` | publish_precision on detail page, CSV and map |
| `test_auth.py` | every endpoint signed-out, traversal, cookie flags |
| `test_pages.py` | every page + filter combination, empty states, filter round-trip |
| `test_alerting.py` | notification fires on transitions only, survives dead webhook |
| `test_queries.py` | the D-70 staleness fix, pinned by a fleet fixture |
| `test_migrations.py` | idempotency, and that the role cannot write engine data |
| `test_rate_limit.py` | lockout behaviour, bounded tracking table |
| `test_errors_and_sessions.py` | error pages leak nothing, epoch revocation |

`tests/README.md` states what is **not** covered: nothing has run against production, no browser tests, map JavaScript untested, no load testing.

### Four bugs found writing it — three were in the tests

Worth remembering as a pattern: a new test suite tests the tests first.

1. **Real app bug.** The session epoch was stamped from a module-level settings snapshot taken at import, while its validator read settings fresh. Production restarts would have hidden this indefinitely. Both now read the same source.
2. The withheld-coordinates leak test matched an inline SVG icon's path data (`m10.2 10.2 3.3 3.3`) and cried wolf. It now reads the page's visible text. **A leak test that false-alarms gets deleted, which is worse than not having one.**
3. Migration tests ran as the console's own restricted role, which correctly cannot `ALTER TABLE`. They run as superuser now; the restriction working is tested separately.
4. A test asserting proxy headers are ignored matched the docstring explaining why they are ignored. It now parses the function body via `ast`.

## 3. Production-readiness gaps closed

Asked directly "is this production ready" — answered no, listed four gaps, then closed three.

**Login rate limiting.** Five failures locks that client out for five minutes. A locked-out client **cannot get in even with the correct password** — otherwise the lockout only delays an attacker until the moment it matters. A correct sign-in clears the count. Counted per client address with a hard ceiling on tracked addresses. No proxy header is trusted: the service binds to localhost, so an `X-Forwarded-For` would hand an attacker unlimited fresh identities.

**Control service, second hardening pass — found a genuine privilege escalation.** A rotation value containing a newline would have written a second `KEY=value` line into the secrets file, including for `POSTGRES_PASSWORD`, which the service explicitly refuses to rotate. That is escalation, not a formatting bug. Line breaks now refused, with a test. Also added request/path size limits and control-character rejection *before* the allowlist comparison, so a refused unit name cannot forge lines in the journal of the service that refused it.

**Error page** that never echoes the underlying message (an error can carry a connection string). JSON endpoints still get JSON.

**Session revocation** via `SESSION_EPOCH` — signs everyone out without changing the signing secret.

## 4. Performance, measured rather than assumed

Loaded **20,153 entities / 120,153 facts** — roughly live scale — and timed it:

| Page | Time |
|---|---|
| Overview | 0.061s |
| Data browser | 0.047s |
| Filtered data | 0.050s |
| Name search | 0.053s |
| Engines | 0.025s |
| Map GeoJSON | 0.026s |
| CSV export (20k rows) | 0.173s |

The aggregate subqueries over `entity_fact` were the suspected weak point. They are not. **Performance is not a concern at this scale.**

## 5. Still open

- **Deploy and verify against the live database.** The remaining blocker. Three sessions have now built and tested against throwaway schemas.
- **Base image unpinned.** `./scripts/pin-base-image.sh` resolves the tag to a digest; needs Docker and a registry route. This session had neither and would not commit a digest it could not verify.
- **No external review of `control/control_service.py`** — ~300 lines, runs as root, reviewed only by its author. If one file gets reviewed by someone else, that is it.
- **No browser-level automated tests.** Contrast/keyboard checks were manual (Playwright) and are not in the suite.

## Environment notes

- **Background processes in the cloud container die between Bash calls** unless started with `setsid --fork`. Cost several retries; `nohup ... &` alone is not enough.
- `pkill` in a compound command can kill the calling shell (exit 144). Kill and restart in separate calls.
- The cloud container **cannot reach cdnjs or Docker Hub's registry** (egress policy) but **npm and pip work**. Leaflet was obtained via `npm pack leaflet@1.9.4`; the base-image digest could not be resolved at all.
- PostGIS installs via `apt-get install postgresql-16-postgis-3`, which is what made testing against the real schema possible.
- Deletion in mounted folders needs `device_request_delete_permission` — hit this when a stale `.git/HEAD.lock` blocked a commit.

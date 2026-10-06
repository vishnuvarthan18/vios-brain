# Ops Control Panel (Admin Web App) — Plan

_Updated 2026-09-17. **All nine sections built, interface rebuilt around operator flow, 184 committed tests, production-hardened.** Committed as `india-ops-console` (74 files, 5 commits). **Not deployed — that is the only remaining blocker.** The original plan text is preserved below the status section._

## Status — 2026-09-17

**Repo:** `~/india-data-platform/india-ops-console` on the Mac. Move it out first:

    mv ~/india-data-platform/india-ops-console ~/india-ops-console

**Not deployed.** Three build sessions in a row had no route to the VPS. Full deploy steps for both phases are in the repo's README.

### Section status

| Plan section | State |
|---|---|
| 1. Overview / home | Built — status banner, ranked "needs attention" list with inline actions, stat tiles, 30-day sparkline, engine table, recent runs |
| 2. Engine health | Built — card per engine, detail page with coverage meters, per-source table (licence, tier, last success, key needed) |
| 3. Data browser + map | Built — filter chips, full record view, Leaflet map with coverage table, CSV export |
| 4. Run health + control | Built — job table split urgent/healthy, trigger a run with confirmation, tail the journal, audit log, auto-refresh while running |
| 5. Alerts & notifications | Built — alert episodes with history, webhook + SMTP on transitions only |
| 6. Server & infrastructure | Built — disk breakdown, memory, load, container states, backup age, DB size, largest tables |
| 7. API keys & credentials | Built — registry with pending/registered status, rotation dialog, orphan-source detection |
| 8. Decisions & history | Built — renders each repo's DECISIONS.md with search. **Deploy history still not built.** |
| 9. Open items / roadmap | Built — interactive checklist, seeded from `open-items-next-session.md` |

### Production readiness — honest status

| Gap | State |
|---|---|
| No repeatable verification | **Closed** — 184 tests, ~9s, `./scripts/run-tests.sh`; 85 need no database |
| No login rate limiting | **Closed** — 5 failures locks that client out 5 min |
| Control service only self-reviewed | **Partly** — second hardening pass done; no external review |
| Base image unpinned | **Open** — `./scripts/pin-base-image.sh`; needs Docker + registry route |
| Bare JSON error pages | **Closed** — real error page, never echoes the underlying message |
| No session revocation | **Closed** — `SESSION_EPOCH` |
| Never run against production | **Open — the remaining blocker** |

### Performance, measured at live scale

At 20,153 entities / 120,153 facts: overview 0.061s, data browser 0.047s, filtered 0.050s, name search 0.053s, engines 0.025s, map GeoJSON 0.026s, 20k-row CSV export 0.173s. The aggregate subqueries over `entity_fact` were the suspected weak point and are not a concern.

### D-70 is fixed (migration 0019)

`heartbeat.silence_after` defaulted to 36 hours for **every** job. Of the 16 live jobs, 8 are monthly and 4 weekly — two thirds of the fleet permanently flagged stale between healthy runs. An alert list that is always red is one nobody reads.

The fix adds a declared `cadence` per job (reusing the existing `schedule_tier` enum), derives the tolerated silence from it, keeps `silence_after_override` for exceptions, and stops treating "registered but not yet due" as a failure — precisely the confusion the September observation pause ran into.

On a fleet fixture: **6 alerts before, 2 after**, both survivors genuine. Now pinned by `tests/test_queries.py`, which fails loudly if anyone reinstates a flat window.

### Safety model as built

Four controls, none relying on the web code being bug-free:

1. **`ops_console` DB role cannot write engine data.** SELECT on all of `public`, write access only inside the `ops_console` schema. Tested: `DELETE FROM entity_fact` returns `permission denied`.
2. **No shell, no Docker socket in the web app.** Host metrics come from a root-owned collector on its own timer writing a JSON file the console reads read-only.
3. **Actions go through an allowlist the console cannot edit.** `ops-control` runs on the host, checking every unit name against root-owned files outside the container. Three verbs plus a single-line secrets write; every subprocess call is an argument list with `shell=False`. 24 hostile unit names tested, all refused.
4. **Not internet-facing.** Both services bind `127.0.0.1` (D-2 reasoning), reached over an SSH tunnel.

Plus: Argon2id login with rate limiting, signed session cookie (`httponly`, `samesite=lax`), 12-hour expiry, epoch-based revocation.

`keyvars.allow` deliberately excludes `POSTGRES_PASSWORD`, `MINIO_ROOT_PASSWORD`, `SESSION_SECRET` and `CONTROL_TOKEN`.

### Security finding, 2026-09-17

**A genuine privilege escalation, found and fixed during the hardening pass.** A rotation value containing a newline would have written a second `KEY=value` line into the secrets file — including for a variable the service explicitly refuses to rotate. Line breaks are now refused outright, with a test. Also added: request/path size limits, and control characters refused *before* the allowlist comparison, so a refused unit name cannot forge lines in the journal of the service that refused it.

### Three design decisions worth remembering

**Colour means something.** Grey is the whole interface; red, amber and green appear only where they carry a fact about platform health. If everything is coloured, nothing is a signal — and spotting what is wrong is the entire job of this tool.

**publish_precision is honoured on the way out, not only in.** The `entity_publish_precision_guard` trigger already refuses to *store* sacred groves, traditional knowledge and FRA claims at full precision. The browser applies the matching read-side rule: restricted entities plot at a district representative point, CSV exports drop their coordinates, `withhold` never exports. Stricter than an internal tool needs — but an export is a file that leaves the platform the moment someone forwards it. Verified: 8 test groves collapse to 3 district points with empty CSV latitudes.

**Alerts are episodes, not a live list.** A view forgets a problem the moment it clears. `ops_console.alert_event` records each with `opened_at`/`resolved_at`. Notifications fire on transitions only, enforced by a partial unique index rather than by the notifier remembering. A dead webhook still records history and exits 0 — an alerting system that becomes a failed unit because a webhook is down is the same noise loop D-70 was.

### Accessibility — audited, not assumed

14 pages, both themes, including hover states. Light mode had **121 contrast failures**; both themes now measure **zero**. Two were real design faults: secondary text at 4.24:1 on the table-header surface, and hovering an alert row deepening its tint until labels fell to 3.91:1 — hover now lightens instead. Skip link, landmarks, scoped headers, `aria-current`, labelled controls, visible focus, no horizontal scroll at 390px. Works with JavaScript off.

### Known gaps carried forward

- `ops_console.host_metric` exists for trend history but nothing writes to it.
- Container memory in the snapshot is always null (`docker stats` costs ~1s per container).
- No deploy history (second half of section 8).
- No browser-level automated tests — contrast and keyboard checks were manual.
- Heartbeat `cadence` values in 0019 are hardcoded from the 2026-09-16 timer list. A new job needs a row, or it falls back to the legacy default and shows as "undeclared".

---

## Goal

One web app, one login, that is the entire operating interface for the platform. Anything that currently needs SSH/terminal/code should have a button or form here instead: checking health, browsing collected data, triggering runs, managing keys, viewing logs, reading decisions/history, and getting alerted when something breaks.

## Sections

### 1. Overview / home
- Top-line numbers: total entities, total facts, engines live vs pending, open alerts count, server status at a glance
- Recent activity feed (last N harvest runs, last N decisions logged, last deploys)

### 2. Engine health
- Card per engine (8 total): status, entity/fact counts, growth sparkline, last run per source
- Detail page: full list of its data sources, each source's history, per-source entity counts, license info

### 3. Data browser
- Browse and search entities — filter by engine, state/district, source, publish precision
- Individual record view (all facts, sources, confidence, license) without querying Postgres by hand
- Map view for anything with geometry — spot gaps visually
- Export a filtered view to CSV

### 4. Run health + control
- Full job table: cadence, last run, status, per-job correct staleness logic (fixes D-70)
- Trigger a manual run for any harvester directly from the UI
- View/tail logs in the browser
- Retry a failed job

### 5. Alerts & notifications
- Central alert list: stale sources, failed runs, thresholds
- Push/email hook when something goes red
- Alert history

### 6. Server & infrastructure
- Uptime, disk usage with component breakdown, RAM/CPU, Postgres size, largest tables, backup status

### 7. API keys & credentials
- Every key, which engine/source needs it, registered vs pending
- Rotate from the UI rather than SSH + manual edit
- Rotation reminders where the source enforces one

### 8. Decisions & history log
- Render `DECISIONS.md` from each repo, searchable
- Deploy history: what was deployed when, from which session, to which engine

### 9. Open items / roadmap tracker
- The standing "needs the user" list as an interactive checklist

## Design decisions (as built)

- **Where it lives:** its own repo and container, joining the existing `core-infra` Docker network. Reads the shared Postgres through a restricted role rather than the core API, because the console's aggregate queries are not things the write-oriented core API should grow endpoints for.
- **Auth:** single admin, Argon2id hash, rate-limited, signed session cookie, 12-hour expiry, epoch revocation.
- **Action safety:** a separate host service with a root-owned allowlist, never a shell in the web app. Retry is the same action as run, since harvests are idempotent.
- **Tech:** FastAPI + Jinja server-rendered HTML, one hand-written stylesheet, vendored Leaflet, no framework, no CDN, no build step — the app must render with no outbound network.
- **Hosting:** on the VPS, ports bound to 127.0.0.1, reached by SSH tunnel.

## What's next for this app

1. **Deploy both phases and verify against the live database.** The only real blocker.
2. Pin the base image (`./scripts/pin-base-image.sh`).
3. Get `control/control_service.py` reviewed by someone other than its author.
4. Wire the metrics collector to write `host_metric` rows for real trend graphs.
5. Deploy history — the remaining half of section 8.

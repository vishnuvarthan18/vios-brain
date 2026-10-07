<!--
SPDX-License-Identifier: LicenseRef-Proprietary
Copyright (C) 2026, B. Halle
Author: Vishnu araCreate <vishnu@aracreate.group>
Description: This file contains an index of the production deployment files
-->

# DEPLOY

Production deployment configuration for the backend (database, API, admin
dashboard). The tester-facing pages are hosted separately on Webflow
(`halle-dev.webflow.io`) and are not covered here.

**Start with [runbook.md](runbook.md).** It is the only file meant to be run
commands from; the rest are copied into place at the point the runbook says so.

| File | What it is | Goes where |
| --- | --- | --- |
| `runbook.md` | Step-by-step manual deployment, written to be followed literally | stays in the repo |
| `setup-database.sql` | Creates this project's own Postgres role + database. Touches no other database | run once via `sudo -u postgres psql` |
| `env.production.example` | Every environment variable the app needs, with placeholders | copied to `src/web/.env`, `chmod 600` |
| `halle-feedback.service` | systemd unit — non-root user, restart on crash and boot | `/etc/systemd/system/` |
| `apache-halle-feedback-optional.conf` | Reverse-proxy vhost template. **Not needed yet** — see below | not installed for now |
| `halle-feedback-backup.sh` | Nightly `pg_dump` of this project's database, keeping the most recent 14 | `/opt/halle-feedback/backup.sh` |
| `halle-feedback-backup.service` + `.timer` | Runs that script at 03:20 nightly | `/etc/systemd/system/` |
| `halle-feedback-retention.service` + `.timer` | Runs the screenshot retention sweep weekly, Sundays 04:10 | `/etc/systemd/system/` |
| `halle-feedback-journald.conf` | Caps journal size so logs cannot grow without limit. **Read its caveat — the cap is server-wide** | `/etc/systemd/journald.conf.d/halle-feedback.conf` |
| `halle-feedback-logrotate.conf` | Rotates the optional Apache vhost's two log files. Only needed with that vhost | `/etc/logrotate.d/halle-feedback` |

## Decisions baked into these files

- **Port 3000.** Free on this server (80/443 are Apache's existing sites,
  8000/8001 are another Node app and JupyterHub). Nothing in this repo
  reserves 3000 for anything but local dev, so there is no conflict.
- **Runs as the `halle-feedback` system user**, never root. Root is used only
  to create that account and install the service unit.
- **Reached directly at `http://<server-ip>:3000`.** Apache is not in front of
  the app, so no existing vhost is touched.
- **Working directory is `src/web`**, not the repo root — `lib/widget-asset.ts`
  resolves the widget bundle relative to the process cwd, and the CLI scripts
  load a literal `./.env`.
- **Screenshots live in `/var/lib/halle-feedback/storage`**, outside the git
  checkout, so a redeploy can never delete tester data.

## Why the Apache config is optional

There is no domain yet. A name-based vhost needs a hostname to match on, and
matching on the bare IP would mean claiming the default vhost that currently
answers for the other sites on this server. So the app binds its own port and
Apache is left completely alone. The template becomes useful on the later
domain + HTTPS task.

## Two things to know before deploying

1. **Admin login does not work over plain `http://`.** The session cookie is
   `Secure` in production mode, and browsers won't send it unencrypted. The API
   and widget are unaffected. runbook.md step 9 covers the SSH-tunnel
   workaround; HTTPS is the real fix.
2. **The widget's API origin is baked in at build time**, not read at runtime.
   Rebuild the widget with `WIDGET_API_ORIGIN=...` whenever that address
   changes, or reports go nowhere silently.

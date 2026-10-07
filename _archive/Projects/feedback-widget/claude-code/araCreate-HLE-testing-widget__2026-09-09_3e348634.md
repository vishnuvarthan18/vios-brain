**Vishnu** (2026-09-09T06:31): TASK: Prepare production deployment config for the real hosting server. Do not commit or push. Do not run anything on the server itself — this is prep work only, for Vishnu to review and run by hand.

SERVER FACTS (already checked by hand, do not re-check):
- Debian 12, 4 CPU cores, 3.8 GB RAM (often close to full — be memory-conscious)
- Node.js v20.20.2 already installed
- Postgres v18.3 already installed and already running on port 5432 (localhost only) — there are OTHER databases already on this instance for other projects. Never touch, list, or drop anything except a new database you create for this project.
- Ports 80 and 443 are already used by Apache, serving other unrelated sites on this same server. Do not bind directly to 80/443.
- Ports 8000 and 8001 are already used by another Node app and Jupyterhub. Do not use these.
- Plenty of disk space (105 GB free), not a concern.
- This is a SHARED server with other unrelated projects already running on it. Every config you write must be scoped only to this project — no changes to Apache's existing sites, no touching other databases, no killing other processes.

DECISIONS ALREADY MADE (do not re-ask):
- Backend address: plain IP + port for now (e.g. http://212.227.213.174:3000). No domain/DNS setup needed yet — that's a later task, not this one.
- This server is the PERMANENT home for this project, not a stopgap. Build the deployment to last: a real systemd service that restarts on crash and on reboot.
- The app must run under a NEW, dedicated, non-root Linux user account, not root. Root is only used once to create that account and to add the Apache forwarding rule.
- The widget's front-end pages are separately hosted on Webflow staging (halle-dev.webflow.io) — that is NOT part of this task. This task is only the backend: database, API, and admin panel.

WHAT TO PRODUCE (all under a new `deploy/` folder in the repo — do not touch any other file):
1. Pick one currently-free port for this app to listen on (e.g. 3000 — confirm it's not already reserved anywhere in this repo's own docs first).
2. Instructions/commands for creating the new non-root Linux user on the server (e.g. `feedback-app`), with no login shell needed beyond running this service.
3. An Apache reverse-proxy config snippet (a new vhost file, not edits to any existing vhost) that forwards requests on the server's IP at that port — since there's no domain yet, this may just mean the app is reached directly at IP:port and Apache is not required to front it yet. Note this clearly rather than guessing.
4. A systemd service unit file that runs the app as that new non-root user, restarts on failure and on boot.
5. A SQL script (or migration note) to create a new, separate Postgres database and a new database user for this project only — with a random generated password placeholder, not a real one. Must not touch any existing database.
6. A `.env.production.example` file listing every environment variable the app needs for production (DB connection string using the new database, port, any API keys already used in dev), with placeholder values, not real secrets.
7. A plain step-by-step deployment runbook, written for a non-technical person to follow manually on the server terminal — this is the ONLY file meant for Vishnu to actually run commands from. Cover: creating the new user, uploading/cloning the code, installing dependencies, running migrations against the new database, starting the systemd service, and how to check it's working at the IP:port address.

Stop after writing these files — do not commit, do not push, do not attempt to SSH anywhere. List the files you created when done, and flag anything you were unsure about instead of guessing.

**Claude** (2026-09-09T06:31): I'll start by understanding the repo — what the app actually is, how it runs, what env vars it uses, and whether any port is already reserved in its docs.

**Claude** (2026-09-09T06:31): _[tool: Bash]_

**Claude** (2026-09-09T06:31): It's a Next.js app. Let me read the package manifest, README, and docs to learn how it builds/runs and what's configured.

**Claude** (2026-09-09T06:31): _[tool: Bash]_

**Claude** (2026-09-09T06:31): It's an npm workspace with `src/web` (the Next.js app) and `src/widget`. Let me look at the web workspace and hunt for port references across the repo's docs, per your instruction to confirm the port isn't already reserved.

**Claude** (2026-09-09T06:31): _[tool: Bash]_

**Claude** (2026-09-09T06:32): The env example is the key file for requirement 6. Let me read it, plus check for a real `.env` (dev secrets I must not copy into the example) and search all docs for port reservations.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Interesting — the local `.env` still carries stale S3 keys while `.env.example` documents local-disk storage instead. I need to verify which the code actually reads before writing the production example, rather than trusting either file. Let me grep the source for env usage and port references simultaneously.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Confirmed: no S3 in the code — those keys in `.env` are dead, and storage is local disk. Note `WIDGET_CDN_URL` and `REPORTS_RATE_LIMIT_PER_HOUR` appear in `.env.example` but not in `process.env` usage. Let me check whether they're read indirectly, and search the whole repo for port reservations.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Important find: `REPORTS_RATE_LIMIT_PER_HOUR` is a **hardcoded constant** in [rate-limit.ts:27](halle-feedback-widget/src/web/lib/db/rate-limit.ts#L27), not read from env — so putting it in `.env.production` would be misleading. `WIDGET_CDN_URL` appears unused entirely. Let me confirm and check the port question across docs.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Ports 3000 (app) and 5432 (Postgres) plus 4319/4320. Let me see exactly where 3000 and 4319/4320 are used, to pick a port that collides with nothing in this repo's own conventions.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Port 3000 is only ever the *dev* address (and 4319/4320 are Playwright test fixtures) — nothing reserves 3000 for production, so it's free to use given your server facts. One critical find: [build.mjs:25](halle-feedback-widget/src/widget/scripts/build.mjs#L25) bakes an API origin into the widget bundle at **build time**.

That matches the "API-origin pattern" convention I have in memory. Let me confirm the details, since it affects the runbook's build step.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Confirmed the build-time origin bake-in. Now let me check the Makefile (how the app builds/starts), the migration mechanism, and how the widget bundle is served in production.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Now the migration script and how the widget bundle is served — the `/v1.js` route suggests the app serves it itself, which matters for whether the widget needs a separate CDN in production.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Key detail: the migration runner loads `.env` from the **web workspace cwd** (`config({ path: '.env' })`), and the app serves `/v1.js` itself — so no separate CDN is needed. Let me verify how the app reads env at runtime (does Next load `.env.production`?) and check the storage dir default.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): Critical: the app must run with **cwd = `src/web`** (that comment says Next sets cwd there, and the widget dist is resolved relative to it). That directly determines `WorkingDirectory` in the systemd unit. Let me verify how env is loaded at runtime and check `STORAGE_DIR` resolution.

**Claude** (2026-09-09T06:32): _[tool: Bash]_

**Claude** (2026-09-09T06:32): `STORAGE_DIR` resolves **relative to cwd** — so a relative value would put uploads inside the git checkout. For production I'll use an absolute path outside the checkout so screenshots survive redeploys. Let me check the setup-env script and the existing docs' deployment/env guidance.

**Claude** (2026-09-09T06:33): _[tool: Bash]_

**Claude** (2026-09-09T06:33): No existing deployment config — this is genuinely new, and no port is reserved for production. One thing to check: `build-plan.md:86` says the widget needs object storage/CDN because "Webflow's Assets panel rejects `.js`". Let me confirm that's satisfied by the app serving `/v1.js`, and check the session/cookie security since we're on plain HTTP.

**Claude** (2026-09-09T06:33): _[tool: Bash]_

**Claude** (2026-09-09T06:33): This is a significant finding I must flag rather than paper over: [actions.ts:72](halle-feedback-widget/src/web/app/login/actions.ts#L72) sets `secure: true` when `NODE_ENV=production`. A `Secure` cookie is **not sent over plain HTTP**, so admin login over `http://IP:3000` would fail to persist a session. Let me confirm how the cookie is read back and whether HTTPS is truly required.

**Claude** (2026-09-09T06:33): _[tool: Bash]_

**Claude** (2026-09-09T06:33): Confirmed — the admin login will silently fail over plain HTTP with `NODE_ENV=production`. I'll flag this prominently and give a safe, non-code-changing workaround in the runbook. Let me check the drizzle config and migrations, then confirm the DB user needs schema-creation rights.

**Claude** (2026-09-09T06:33): _[tool: Bash]_

**Claude** (2026-09-09T06:33): Migrations need `gen_random_uuid()` and `CREATE TRIGGER`/`CREATE FUNCTION` — so the new DB user must own the database. `gen_random_uuid()` is built-in in Postgres 13+, so no extension needed on PG 18. Let me verify no `CREATE EXTENSION` is required and check the user-create script (needed for the first admin login).

**Claude** (2026-09-09T06:33): _[tool: Bash]_

**Claude** (2026-09-09T06:33): I have everything I need. Two more checks: whether `next start` respects a `PORT` env var / hostname binding, and the memory-consciousness angle for `next build` on a 3.8 GB box.

**Claude** (2026-09-09T06:33): _[tool: Bash]_

**Claude** (2026-09-09T06:33): Important gotcha: `.gitignore` has `.env.*` with only `!.env.example` un-ignored — so a file named `.env.production.example` **would be git-ignored** and never reach the server via `git clone`. I'll name the file to avoid that trap and note it. Let me confirm.

**Claude** (2026-09-09T06:33): _[tool: Bash]_

**Claude** (2026-09-09T06:33): Confirmed. I'll ship `deploy/env.production.example` (tracked) so it actually reaches the server, and note the rename in the runbook. Now writing the files.

**Claude** (2026-09-09T06:34): _[tool: Bash]_

**Claude** (2026-09-09T06:34): Now the SQL script for the dedicated database and user, scoped so it cannot touch the other projects' databases.

**Claude** (2026-09-09T06:34): _[tool: Bash]_

**Claude** (2026-09-09T06:34): Now the systemd unit. The critical details: cwd must be `src/web`, run as the non-root user, restart on failure and boot, plus memory limits for the 3.8 GB box.

**Claude** (2026-09-09T06:34): _[tool: Bash]_

**Claude** (2026-09-09T06:35): Now the Apache vhost. Per your instruction, I'll be explicit that Apache is **not required yet** rather than guessing a domain.

**Claude** (2026-09-09T06:35): _[tool: Bash]_

**Claude** (2026-09-09T06:35): Now the runbook — the one file you'll actually run commands from, written for a non-technical reader.

**Claude** (2026-09-09T06:36): _[tool: Bash]_

**Claude** (2026-09-09T06:36): Let me add a short README to orient anyone opening `deploy/`, then verify everything.

**Claude** (2026-09-09T06:37): _[tool: Bash]_

**Claude** (2026-09-09T06:37): Now let me verify my work: check nothing outside `deploy/` was touched, no real secrets leaked in, and the files are git-visible.

**Claude** (2026-09-09T06:37): _[tool: Bash]_

**Claude** (2026-09-09T06:37): All 5 `deploy/` files are git-visible. The other modified/untracked entries are pre-existing uncommitted work from earlier sessions — **not mine**. Let me confirm that by checking timestamps.

**Claude** (2026-09-09T06:37): _[tool: Bash]_

**Claude** (2026-09-09T06:37): Confirmed — those three were modified at 11:27, before this session started (my files are 12:04–12:07). I touched nothing outside `deploy/`. Now let me validate the content of what I wrote: no real secrets, and correct placeholders.

**Claude** (2026-09-09T06:37): _[tool: Bash]_

**Claude** (2026-09-09T06:37): Two flagged "leaks" need checking — they're likely harmless coincidences (a dev URL and the number `60`), not secrets. Let me verify.

**Claude** (2026-09-09T06:37): _[tool: Bash]_

**Claude** (2026-09-09T06:37): Both flags were false positives — `http://localhost:3000` (the tunnel URL in step 9) and the number `60`. No real secret is present. Now let me validate the systemd unit and SQL syntactically, since I can't run them on the server.

**Claude** (2026-09-09T06:37): _[tool: Bash]_

**Claude** (2026-09-09T06:37): One systemd subtlety worth verifying: `StartLimitIntervalSec`/`StartLimitBurst` belong in `[Unit]`, not `[Service]`, on modern systemd. Let me confirm and fix.

**Claude** (2026-09-09T06:37): _[tool: Bash]_

**Claude** (2026-09-09T06:37): Real bug: these are in `[Service]` but systemd (v230+, so certainly Debian 12) only honours them in `[Unit]`. There they'd be silently ignored, so a crash-looping app would restart forever and compete for RAM — exactly what I wrote them to prevent. Moving them.

**Claude** (2026-09-09T06:38): _[tool: Bash]_

**Claude** (2026-09-09T06:38): Fixed. Now let me check two more things I shouldn't assume: whether `npm run start` works with `ProtectSystem=strict` (npm may need a writable cache), and whether the `.env` file format is compatible with systemd's `EnvironmentFile` parser.

**Claude** (2026-09-09T06:38): _[tool: Bash]_

**Claude** (2026-09-09T06:38): The env file is systemd-compatible and dotenv-compatible. One remaining risk I should not leave unverified: `ProtectSystem=strict` makes the whole filesystem read-only except `ReadWritePaths`, but npm writes a cache to `$HOME/.npm` and Next.js may write into `.next`. Let me check.

**Claude** (2026-09-09T06:38): _[tool: Bash]_

**Claude** (2026-09-09T06:38): `npm` writes a cache/logs to `$HOME/.npm`, and Next.js can write into `.next/cache` at runtime — both under the read-only `/opt/halle-feedback` with `ProtectSystem=strict`. That would cause a hard-to-diagnose start failure. Let me harden the unit against that.

**Claude** (2026-09-09T06:38): _[tool: Bash]_

**Claude** (2026-09-09T06:38): `ProtectHome=true` would hide `/home` but `/opt/halle-feedback` is the home dir here, so it's unaffected. Let me do a final review of the complete unit file and confirm the docs stay consistent with the fixes.

**Claude** (2026-09-09T06:38): _[tool: Bash]_

**Claude** (2026-09-09T06:38): One inconsistency to fix: a stale comment says the app writes "ONLY to its screenshot store", which the new `ReadWritePaths` contradicts. Also the runbook references "step 8"/"step 9" for the unit while the actual steps are 10 and 7.

**Claude** (2026-09-09T06:38): _[tool: Bash]_

**Claude** (2026-09-09T06:38): That's my own edit reflected back. All step cross-references now resolve correctly (5, 7, 9, 10, 2 all exist). One last check — `ProtectHome=true` vs. the home directory being `/opt/halle-feedback`:

**Claude** (2026-09-09T06:38): _[tool: Bash]_

**Claude** (2026-09-09T06:39): Done. Nothing committed, nothing pushed, no SSH attempted, and no file outside `deploy/` touched.

## Files created

| File | Purpose |
|---|---|
| [deploy/RUNBOOK.md](halle-feedback-widget/deploy/RUNBOOK.md) | **The only file to run commands from.** 12 steps, plain language, with an "Expected:" result after each command |
| [deploy/setup-database.sql](halle-feedback-widget/deploy/setup-database.sql) | Creates one role + one database, both named `halle_feedback` |
| [deploy/env.production.example](halle-feedback-widget/deploy/env.production.example) | Every env var, placeholders only |
| [deploy/halle-feedback.service](halle-feedback-widget/deploy/halle-feedback.service) | systemd unit — non-root, restart on crash + boot |
| [deploy/apache-halle-feedback.conf.OPTIONAL](halle-feedback-widget/deploy/apache-halle-feedback.conf.OPTIONAL) | Reverse-proxy template, **not to be installed yet** |
| [deploy/readme.md](halle-feedback-widget/deploy/readme.md) | Index of the above |

**Port 3000** — grepped the whole repo: it appears only as the local dev address and in test fixtures (4319/4320 are Playwright's). Nothing reserves it for production, and it avoids 80/443/8000/8001. Step 1 of the runbook still makes you verify with `ss -ltnp` before proceeding, with 3100 as the fallback.

**Apache: not needed yet, as you suspected.** I did not guess a domain. A name-based vhost needs a hostname to match on, and matching the bare IP would mean claiming the default vhost that currently answers for your other sites — the one line we must not cross. So the app binds 3000 itself and Apache is untouched. The template is there for the later domain task.

## Things you should know before running this

1. **Admin login will not work over `http://IP:3000`.** [actions.ts:72](halle-feedback-widget/src/web/app/login/actions.ts#L72) sets the session cookie `secure: NODE_ENV === 'production'`, and browsers won't send a Secure cookie over plain HTTP — so you'd enter the right password and just bounce back to the login page with no error. The API and widget are unaffected; reports still save. I did **not** change that line, since weakening session security for the real site is your call. Step 9 gives an SSH-tunnel workaround (`ssh -L 3000:127.0.0.1:3000`, then use `http://localhost:3000/app`) which browsers treat as secure. HTTPS is the real fix.

2. **The env file must be named `.env`, not `.env.production`.** Next.js would read the latter, but `db-migrate.mts`, `user-create.mts` and `drizzle.config.ts` all call dotenv with a literal `'.env'` — so migrations would fail with "DATABASE_URL is not set" while the app looked fine. Relatedly, I named the repo file `env.production.example` **without** a leading dot because `.gitignore` ignores `.env.*` and only un-ignores `.env.example` — a dotted name would have been silently git-ignored and never reached the server via `git clone`. Verified with `git check-ignore`.

3. **The widget's API origin is baked in at build time**, not read at runtime ([build.mjs:25](halle-feedback-widget/src/widget/scripts/build.mjs#L25)). If it's wrong the widget silently posts nowhere. The runbook passes `WIDGET_API_ORIGIN=` explicitly on the build, and repeats it in the update procedure.

4. **`WorkingDirectory` must be `src/web`** — [widget-asset.ts](halle-feedback-widget/src/web/lib/widget-asset.ts) resolves the widget bundle from `process.cwd()/../widget/dist`, so `/v1.js` 500s from anywhere else.

5. Two bugs I found and fixed in my own drafts while checking them: `StartLimitIntervalSec`/`StartLimitBurst` were in `[Service]`, where systemd ignores them silently (moved to `[Unit]`); and `ProtectSystem=strict` alone would have blocked npm's cache and Next's `.next/cache` writes, causing an EROFS failure that reads like an unrelated Node error.

## Unsure / assumed — worth a look

- **The IP `212.227.213.174`** came from your example. It appears in `APP_URL` and the runbook — confirm it's really this server's public IP.
- **The git remote URL** is a `<THE-REPOSITORY-URL>` placeholder; I couldn't infer it (the parent dir isn't a repo and I didn't query remotes). If it's private you'll need a deploy key at step 3.
- **`/usr/bin/npm`** is assumed for `ExecStart`. Step 10 has you run `which npm` to confirm.
- **Locale `en_US.UTF-8`** in the SQL — if the server's Postgres was initialised with a different locale, `CREATE DATABASE` will error. Easy fix if it does; I didn't want to guess by dropping the setting.
- **Firewall** — I don't know whether port 3000 is open externally. Step 11 tests it from your Mac and tells you to report back rather than change rules yourself.
- **Not set up, by design:** DB backups, log rotation, and a timer for `make retention` (the 90-day screenshot sweep exists as a command but nothing calls it). Listed at the end of the runbook as known gaps.
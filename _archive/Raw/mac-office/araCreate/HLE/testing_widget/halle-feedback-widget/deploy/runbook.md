<!--
SPDX-License-Identifier: LicenseRef-Proprietary
Copyright (C) 2026, B. Halle
Author: Vishnu araCreate <vishnu@aracreate.group>
Description: This file contains the manual, step-by-step production deployment runbook
-->

# Deployment runbook — B. Halle feedback widget backend

> **⚠ STALE SERVER TARGET — read this before anything else.** Every
> `feedback.arametrics.app` / `212.227.213.174` below names the **old server**,
> retired 30 September 2026. **The real production server is documented in
> [`../docs/server-migration-plan.md`](../docs/server-migration-plan.md)** —
> host `217.160.93.75`, the app runs inside its `webapp` container
> (`10.10.0.20:3000`), reached through the `inbox webapp` helper, not a plain
> SSH login. The real public address is **`https://apps.b-halle.de`**.
> This file is kept as a reference for how a from-scratch setup works, not as
> a target to deploy to. **Before running anything here, or anywhere in this
> `deploy/` folder, confirm which server you are actually on** — do not trust
> a domain or IP in any file without checking it against the migration plan
> first.

**This is the only file in `deploy/` you run commands from.** The other files
are things you copy into place; this file tells you when.

Read this once from top to bottom before typing anything.

## What you are setting up

The backend of the feedback widget: the database, the API the widget talks to,
and the admin dashboard you log in to. The tester-facing pages stay on Webflow
(`halle-dev.webflow.io`) and are not part of this.

When you are done, the dashboard is at **http://feedback.arametrics.app:3000/app** and
it starts itself again after a crash or a reboot.

## Ground rules

This server also runs other, unrelated projects. Every command below is scoped
to this project only.

- **Never** run `sudo systemctl restart apache2`, `postgresql`, or anything
  you were not told to. Other people's sites are on this machine.
- Only two steps use `sudo` (creating the user, installing the service).
  Everything else runs as the new project user.
- If a command prints an error, **stop** and send me the output. Don't
  improvise or re-run it with changes — there is nothing here that a second
  attempt fixes on its own.
- Copy commands one line at a time. Where you see `feedback.arametrics.app`, check it
  really is this server's IP before using it.

## Before you start — three things to have ready

1. SSH access to the server with a `sudo`-capable account.
2. The git repository URL for `halle-feedback-widget` (and a way to
   authenticate to it — a deploy key or token, if it's private).
3. Ten to twenty minutes. The dependency install and build take a while.

---

## Step 1 — Log in and confirm the port is free

```bash
ssh your-user@feedback.arametrics.app
```

Check that nothing is already listening on port 3000:

```bash
sudo ss -ltnp | grep ':3000' || echo "PORT 3000 IS FREE - good, continue"
```

**Expected:** `PORT 3000 IS FREE - good, continue`

If instead it prints a line showing a process, port 3000 is taken. Stop and
tell me — we pick another port (3100 is the fallback) and I will update three
files. Do not kill whatever is using it.

While you're here, confirm Node is the version we expect:

```bash
node --version
```

**Expected:** `v20.20.2` (or any `v20.x` / newer). If it says `command not
found`, stop and tell me.

## Step 2 — Create the dedicated non-root user

The app gets its own Linux account with no password and no interactive login,
so nothing that reaches the app is ever running as root.

```bash
sudo useradd --system --create-home \
     --home-dir /opt/halle-feedback \
     --shell /usr/sbin/nologin \
     halle-feedback
```

What the options mean, briefly: `--system` makes a service account (not a
person), `--shell /usr/sbin/nologin` means nobody can log in as it, and
`--home-dir` is where the code will live.

Create the folder for screenshot storage, owned by that user:

```bash
sudo mkdir -p /var/lib/halle-feedback/storage
sudo chown -R halle-feedback:halle-feedback /var/lib/halle-feedback
sudo chmod 750 /var/lib/halle-feedback
```

Check it worked:

```bash
id halle-feedback && ls -ld /opt/halle-feedback /var/lib/halle-feedback/storage
```

**Expected:** a `uid=... halle-feedback` line, then two directory lines both
showing `halle-feedback` as the owner.

## Step 3 — Get the code onto the server

Everything from here until step 8 runs **as the project user**, not as you.
This command opens a shell as that user:

```bash
sudo -u halle-feedback -H bash
```

Your prompt changes. You are now the `halle-feedback` user. Clone the code:

```bash
cd /opt/halle-feedback
git clone <THE-REPOSITORY-URL> app
cd app
```

If the repository is private, git will ask for credentials here. If you don't
have them to hand, stop at this step and tell me — the alternative is that I
send you a `.tar.gz` to upload instead.

Confirm you're in the right place:

```bash
pwd && ls
```

**Expected:** `/opt/halle-feedback/app`, and a listing that includes
`Makefile`, `package.json`, `src` and `deploy`.

## Step 4 — Generate the two secrets you'll need

Generate them now and paste them somewhere safe (a password manager) — you
need each one twice, and the database password cannot be recovered later.

Database password:

(secret removed)
openssl rand -base64 24 | tr -d '/+=' | cut -c1-24
```

Session secret:

(secret removed)
openssl rand -hex 32
```

Copy both outputs. Below they're called **[DB-PASSWORD]** and
**[SESSION-SECRET]**.

## Step 5 — Create this project's own database

This creates one new database and one new database user, both named
`halle_feedback`. It does not read, list or alter any other database on this
Postgres server.

First put your generated password into the script. Still as the project user:

```bash
cd /opt/halle-feedback/app
cp deploy/setup-database.sql /tmp/setup-database-filled.sql
nano /tmp/setup-database-filled.sql
```

In the editor, find the line:

```
       PASSWORD 'REPLACE_WITH_GENERATED_PASSWORD'
```

Replace `REPLACE_WITH_GENERATED_PASSWORD` with your **[DB-PASSWORD]**, keeping
the single quotes. Save and exit: `Ctrl+O`, `Enter`, `Ctrl+X`.

Now leave the project-user shell for a moment, because only an admin can create
a database:

```bash
exit
```

Run the script:

```bash
sudo -u postgres psql -v ON_ERROR_STOP=1 -f /tmp/setup-database-filled.sql
```

**Expected:** `CREATE ROLE`, `CREATE DATABASE`, `COMMENT`, a line about
connecting, then some `REVOKE`/`ALTER`/`GRANT` lines. No `ERROR:` anywhere.

If you see `ERROR: role "halle_feedback" already exists`, stop and tell me —
something was set up before and I need to look before we overwrite it.

Delete the filled-in copy, since it contains the real password:

(secret removed)
shred -u /tmp/setup-database-filled.sql
```

Now check the app's database user can actually log in:

```bash
PGPASSWORD='[DB-PASSWORD]' psql -h 127.0.0.1 -U halle_feedback -d halle_feedback -c '\conninfo'
```

**Expected:** a line saying you are connected to database `halle_feedback` as
user `halle_feedback`. If it says authentication failed, the password in the
SQL file didn't match what you're typing here — redo step 5.

## Step 6 — Write the configuration file

Back in as the project user:

```bash
sudo -u halle-feedback -H bash
cd /opt/halle-feedback/app
cp deploy/env.production.example src/web/.env
chmod 600 src/web/.env
nano src/web/.env
```

Change exactly two values:

| Find this line | Change it to |
| --- | --- |
| `DATABASE_URL=postgresql://(secret removed)(secret removed):5432/halle_feedback` | the same line with **[DB-PASSWORD]** in place of `REPLACE_WITH_GENERATED_PASSWORD` |
| `(secret removed)` | `SESSION_SECRET=` followed by your **[SESSION-SECRET]** |

Also check `APP_URL=http://feedback.arametrics.app:3000` shows this server's real IP.

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`), then confirm no placeholders are
left:

```bash
grep -c 'REPLACE_WITH' src/web/.env
```

**Expected:** `0`. If it prints `1` or more, go back and finish editing.

Note: this file must be called `.env` — not `.env.production`. The database
and user-creation commands read a file with that exact name and will fail
otherwise.

## Step 7 — Install dependencies and build

This server has 3.8 GB of RAM and is often close to full, so the build is run
by hand here (never automatically) and with a memory cap.

```bash
cd /opt/halle-feedback/app
npm install
```

This takes several minutes and prints a lot. A few `warn` lines are normal;
`ERR!` is not.

Build the widget bundle, telling it where the API lives:

```bash
WIDGET_API_ORIGIN=http://feedback.arametrics.app:3000 \
  npm run build --workspace halle-feedback-widget-embed
```

**This address is baked permanently into the widget file.** If it's wrong, the
widget on the Webflow page silently sends reports nowhere. When a real domain
arrives later, this build must be re-run with the new address.

Now build the web app:

```bash
NODE_OPTIONS=--max-old-space-size=1536 npm run build --workspace halle-feedback-web
```

**Expected:** ends with a route table and no `Failed to compile`.

If it dies with `Killed` or an out-of-memory error, the server ran out of RAM.
Tell me — the fix is to retry when the machine is quieter, or add temporary
swap. Don't lower the number.

## Step 8 — Set up the database tables and your login

Create the tables:

```bash
cd /opt/halle-feedback/app
make db-migrate
```

**Expected:** `Migrations applied.`

Create your admin login. Change the email and name to suit:

```bash
make user-create EMAIL=vishnu@aracreate.group NAME="Vishnu"
```

The script then prompts `Password:` — type the password you want for this
login (at least 8 characters) and press Enter. It is asked for interactively
so it never lands in your shell history.

**Expected:** `Created vishnu@aracreate.group (...)`.

## Step 9 — Important: read this before starting the service

There is one real problem with running on a plain `http://` address that you
need to know about, because it will look like a broken login otherwise.

The dashboard's login cookie is marked "Secure" whenever the app runs in
production mode. Browsers refuse to send a Secure cookie over an unencrypted
`http://` address. So on `http://feedback.arametrics.app:3000` you can enter the
correct password and simply be bounced back to the login page, with no error
message explaining why.

**The API and the widget are unaffected** — reports from testers will save
correctly. This only affects logging in to the admin dashboard.

You have two options. Pick one and tell me which:

- **Option A (recommended, no code change):** access the dashboard through an
  SSH tunnel from your own laptop, so the browser sees `localhost`, which
  browsers treat as secure. On your Mac, in a new terminal:

  ```bash
  ssh -L 3000:127.0.0.1:3000 your-user@feedback.arametrics.app
  ```

  Leave that running and open **http://localhost:3000/app** in your browser.
  Login works normally. The public API stays reachable at the server's IP for
  the widget.

- **Option B:** get a domain and HTTPS set up (that's the later task, using
  `deploy/apache-halle-feedback-optional.conf`). This is the proper fix and
  makes the dashboard usable directly in a browser.

I deliberately have **not** changed the cookie setting to make plain HTTP
work — that would weaken session security on the real site later, and it's
your call, not mine.

## Step 10 — Install and start the service

Leave the project-user shell:

```bash
exit
```

Install the service file:

```bash
sudo cp /opt/halle-feedback/app/deploy/halle-feedback.service \
        /etc/systemd/system/halle-feedback.service
```

Check that `npm` is where the service file expects:

```bash
which npm
```

**Expected:** `/usr/bin/npm`. If it prints anything else, tell me the path —
one line in the service file needs correcting before it will start.

Load it, enable it (this is what makes it come back after a reboot), and start
it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable halle-feedback
sudo systemctl start halle-feedback
```

`enable` only affects this new service. It changes nothing about Apache,
Postgres, or anything else already running.

## Step 11 — Check it's working

Is the service running?

```bash
sudo systemctl status halle-feedback --no-pager
```

**Expected:** `Active: active (running)` in green. Press `q` to exit if it
pauses.

Is it answering on the port?

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:3000/login
```

**Expected:** `200`.

Is the widget file being served?

```bash
curl -s http://127.0.0.1:3000/v1.js | head -c 120; echo
```

**Expected:** a line starting `/* Halle Feedback Widget v0.0.1 ...`. If you get
an error instead, the widget build in step 7 didn't produce its output — tell
me.

Is it reachable from outside? Run this **on your own Mac**, not the server:

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://feedback.arametrics.app:3000/login
```

**Expected:** `200`. If it hangs or returns `000`, a firewall is blocking port
3000 — tell me and don't change firewall rules yourself.

Finally, open the dashboard — via the SSH tunnel from step 9, Option A:

**http://localhost:3000/app**

Log in with the email and generated password from step 8.

## Step 12 — Confirm it survives a restart

This is the whole point of using systemd, so it's worth proving rather than
assuming.

Simulate a crash:

```bash
sudo systemctl kill -s SIGKILL halle-feedback
sleep 8
sudo systemctl status halle-feedback --no-pager | head -5
```

**Expected:** `Active: active (running)` again, because it restarted itself
within about 5 seconds.

Only if you can afford a reboot of this shared server — **check with the other
projects' owners first, this takes every site on the machine down briefly:**

```bash
sudo reboot
```

Wait a minute, reconnect, and run the status check again. Expected: running,
with no intervention.

---

## Step 13 — Backups, log limits and the retention sweep

Four scheduled/limit artifacts, all in `deploy/`. Install them together —
they are what stops the database being unrecoverable, the disk filling up,
and screenshots being kept forever.

Everything here is scoped to this project by name and runs as the
`halle-feedback` account. None of it touches another site on this server.

### 13a — Nightly database backup

Dumps `halle_feedback` at 03:20 every night and keeps the most recent 14
dumps in `/var/lib/halle-feedback/backups`.

```bash
# The script itself, owned by the service account.
sudo cp /opt/halle-feedback/app/deploy/halle-feedback-backup.sh \
        /opt/halle-feedback/backup.sh
sudo chown halle-feedback:halle-feedback /opt/halle-feedback/backup.sh
sudo chmod 750 /opt/halle-feedback/backup.sh

# The timer and the service it runs.
cd /opt/halle-feedback/app/deploy
sudo cp halle-feedback-backup.service halle-feedback-backup.timer \
        /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now halle-feedback-backup.timer
```

Prove it works **now**, rather than finding out at 03:20:

```bash
sudo systemctl start halle-feedback-backup.service
sudo journalctl -u halle-feedback-backup -n 20 --no-pager
sudo ls -lh /var/lib/halle-feedback/backups
```

You want a `backup: wrote ...` line and one `.sql.gz` file of a plausible
size. A dump under 1 KiB is refused by the script on purpose — that is a
failed dump, not a small database.

**Restoring** (the reason any of this exists). Into a throwaway database
first, never straight over the live one:

```bash
sudo -u postgres createdb halle_restore_check
zcat /var/lib/halle-feedback/backups/halle_feedback-<stamp>.sql.gz \
  | sudo -u postgres psql -q halle_restore_check
sudo -u postgres psql -c '\dt' halle_restore_check   # tables should be listed
sudo -u postgres dropdb halle_restore_check
```

### 13b — Weekly screenshot retention sweep

Runs `make retention`'s underlying command on Sundays at 04:10, deleting
stored pictures past the retention period. The period itself is
`RETENTION_DAYS` in `src/web/lib/retention.ts` (**180 days**) — not
configured in the timer, so the schedule and the policy cannot drift apart.

```bash
cd /opt/halle-feedback/app/deploy
sudo cp halle-feedback-retention.service halle-feedback-retention.timer \
        /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now halle-feedback-retention.timer

# Run one sweep now to confirm it works.
sudo systemctl start halle-feedback-retention.service
sudo journalctl -u halle-feedback-retention -n 20 --no-pager
```

You want a `Retention: swept N expired screenshot(s)` line. `N` being 0 is a
correct result on a young install — nothing is 180 days old yet.

The sweep deletes **files only** and never writes to the `reports` table, so
it re-finds every already-swept report on each run. That is by design, and
why this is weekly rather than nightly.

### 13c — Journal size limit

The app writes all output to the journal, so the journal is what needs
bounding.

```bash
sudo mkdir -p /etc/systemd/journald.conf.d
sudo cp /opt/halle-feedback/app/deploy/halle-feedback-journald.conf \
        /etc/systemd/journald.conf.d/halle-feedback.conf
sudo systemctl restart systemd-journald
journalctl --disk-usage
```

**Read the caveat in the file before installing it.** journald has no
per-unit limit, so these caps apply to the whole server's journal, not just
this project's share. The values are deliberately a generous ceiling
(500M, 3 months) so that capping unbounded growth does not silently shorten
the history other projects keep. If another project needs more, raise
`SystemMaxUse` rather than leaving the journal uncapped.

### 13d — Apache log rotation

**Only if you installed the optional Apache vhost.** The two log files that
vhost creates are named per-project, so the distribution's own apache2
logrotate rules do not necessarily cover them.

```bash
sudo cp /opt/halle-feedback/app/deploy/halle-feedback-logrotate.conf \
        /etc/logrotate.d/halle-feedback
# Dry run first — this prints what it WOULD do and changes nothing.
sudo logrotate --debug /etc/logrotate.d/halle-feedback
```

### Checking the timers afterwards

```bash
systemctl list-timers 'halle-feedback-*' --all
```

Both timers should be listed with a sensible NEXT time. If a timer is absent
it was never enabled; if NEXT is blank, the unit failed to parse — check
`sudo systemctl status <timer name>`.

## Everyday commands

Run these as your normal sudo account.

| What you want | Command |
| --- | --- |
| Is it running? | `sudo systemctl status halle-feedback --no-pager` |
| See the logs live | `sudo journalctl -u halle-feedback -f` |
| See today's errors | `sudo journalctl -u halle-feedback --since today -p err` |
| Restart it | `sudo systemctl restart halle-feedback` |
| Stop it | `sudo systemctl stop halle-feedback` |

## Deploying a code update later

```bash
sudo -u halle-feedback -H bash
cd /opt/halle-feedback/app
git pull
npm install
WIDGET_API_ORIGIN=http://feedback.arametrics.app:3000 npm run build --workspace halle-feedback-widget-embed
NODE_OPTIONS=--max-old-space-size=1536 npm run build --workspace halle-feedback-web
make db-migrate
exit
sudo systemctl restart halle-feedback
```

Screenshots live in `/var/lib/halle-feedback/storage`, outside the code
folder, so a `git pull` never touches tester data.

## If something goes wrong

**Service won't start.** Read the actual reason:

```bash
sudo journalctl -u halle-feedback -n 50 --no-pager
```

Common causes: a typo in `src/web/.env`, the wrong `npm` path in the service
file, or the build in step 7 not having completed.

**`DATABASE_URL is not set`.** The config file is in the wrong place or has the
wrong name. It must be exactly `/opt/halle-feedback/app/src/web/.env`.

**Login bounces back to the login page.** That's the Secure-cookie issue in
step 9. Use the SSH tunnel.

**`502` or `500` on `/v1.js`.** The widget bundle is missing. Re-run the widget
build in step 7.

**Don't** fix things by running the app as root, or by editing files under
`/etc/apache2/sites-*`. Send me the log output instead.

## What is deliberately not set up yet

These are known gaps, not oversights — each is a later task:

- **Backups stay on this server.** Step 13a keeps 14 nightly dumps in
  `/var/lib/halle-feedback/backups`, which covers an accidental deletion or a
  bad migration — but not the loss of the server itself. Copying dumps
  somewhere off this machine is not set up.
- **Nothing alerts you when a scheduled job fails.** A failed backup or sweep
  is recorded in the journal and nowhere else; no one is emailed. Check
  `systemctl list-timers 'halle-feedback-*'` occasionally, or read
  `sudo journalctl -u halle-feedback-backup --since '2 days ago'`.
- **No restore has been rehearsed on this server.** The restore commands in
  step 13a have been run against a dump from this schema, but not as a full
  drill against production data on this machine.

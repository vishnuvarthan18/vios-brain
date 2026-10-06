**Vishnu** (2026-09-09T09:03): <ide_selection>The user selected the lines 1 to 511 from /Users/vishnuvarthanvenkatapathy/araCreate/HLE/testing_widget/halle-feedback-widget/deploy/RUNBOOK.md:
<!--
SPDX-License-Identifier: LicenseRef-Proprietary
Copyright (C) 2026, B. Halle
Author: Vishnu araCreate <vishnu@aracreate.group>
Description: This file contains the manual, step-by-step production deployment runbook
-->

# Deployment runbook — B. Halle feedback widget backend

**This is the only file in `deploy/` you run commands from.** The other files
are things you copy into place; this file tells you when.

Read this once from top to bottom before typing anything.

## What you are setting up

The backend of the feedback widget: the database, the API the widget talks to,
and the admin dashboard you log in to. The tester-facing pages stay on Webflow
(`halle-dev.webflow.io`) and are not part of this.

When you are done, the dashboard is at **http://212.227.213.174:3000/app** and
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
- Copy commands one line at a time. Where you see `212.227.213.174`, check it
  really is this server's IP before using it.

## Before you start — three things to have ready

1. SSH access to the server with a `sudo`-capable account.
2. The git repository URL for `halle-feedback-widget` (and a way to
   authenticate to it — a deploy key or token, if it's private).
3. Ten to twenty minutes. The dependency install and build take a while.

---

## Step 1 — Log in and confirm the port is free

```bash
ssh your-user@212.227.213.174
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

Also check `APP_URL=http://212.227.213.174:3000` shows this server's real IP.

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
npm ci --omit=dev --ignore-scripts || npm install
```

This takes several minutes and prints a lot. A few `warn` lines are normal;
`ERR!` is not.

Build the widget bundle, telling it where the API lives:

```bash
WIDGET_API_ORIGIN=http://212.227.213.174:3000 \
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

Create your admin login. Change the email and name; keep `ROLE=staff`:

```bash
make user-create EMAIL=vishnu@aracreate.group NAME="Vishnu" ROLE=staff
```

**Expected:** `Created staff vishnu@aracreate.group (...)` and a generated
password printed once. **Copy that password now** — it is not shown again.

## Step 9 — Important: read this before starting the service

There is one real problem with running on a plain `http://` address that you
need to know about, because it will look like a broken login otherwise.

The dashboard's login cookie is marked "Secure" whenever the app runs in
production mode. Browsers refuse to send a Secure cookie over an unencrypted
`http://` address. So on `http://212.227.213.174:3000` you can enter the
correct password and simply be bounced back to the login page, with no error
message explaining why.

**The API and the widget are unaffected** — reports from testers will save
correctly. This only affects logging in to the admin dashboard.

You have two options. Pick one and tell me which:

- **Option A (recommended, no code change):** access the dashboard through an
  SSH tunnel from your own laptop, so the browser sees `localhost`, which
  browsers treat as secure. On your Mac, in a new terminal:

  ```bash
  ssh -L 3000:127.0.0.1:3000 your-user@212.227.213.174
  ```

  Leave that running and open **http://localhost:3000/app** in your browser.
  Login works normally. The public API stays reachable at the server's IP for
  the widget.

- **Option B:** get a domain and HTTPS set up (that's the later task, using
  `deploy/apache-halle-feedback.conf.OPTIONAL`). This is the proper fix and
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
curl -s -o /dev/null -w '%{http_code}\n' http://212.227.213.174:3000/login
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
npm ci --omit=dev --ignore-scripts || npm install
WIDGET_API_ORIGIN=http://212.227.213.174:3000 npm run build --workspace halle-feedback-widget-embed
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

- **No domain and no HTTPS.** Plain IP for now, as decided.
- **No automated backups** of the `halle_feedback` database.
- **No log rotation limit** beyond the system journal's default.
- **No scheduled `make retention`** run to delete screenshots past the 90-day
  window. It exists as a command but nothing calls it on a timer yet.


This may or may not be related to the current task.</ide_selection>
BUG: scripts/user-create.mts fails with "SyntaxError: The requested module '../lib/db/schema.ts' does not provide an export named 'USER_ROLES'".

Context: the admin panel was simplified earlier (docs/admin-v2-spec.md) to remove all roles — one plain login only, no role types. The `users` table in src/web/lib/db/schema.ts was updated to match (no `role` column exists anymore). But scripts/user-create.mts was never updated — it still imports `USER_ROLES` and `UserRole` from the schema file (which no longer export those), and the `make user-create` command still accepts a ROLE= argument.

FIX NEEDED: Update scripts/user-create.mts to stop using USER_ROLES/UserRole entirely — remove any role-related logic, since there is no role column in the users table anymore. Check the Makefile's `user-create` target too and remove the ROLE= parameter/docs there if present. Do not add a role column back — the one-login, no-roles decision is final and on record in docs/admin-v2-spec.md.

Scope: fix only this bug. Do not touch anything else. When done, tell me exactly what changed, and confirm `make user-create EMAIL=... NAME=...` (no ROLE=) works end to end with a real local test.

**Claude** (2026-09-09T09:03): I'll start by looking at the actual files involved.

**Claude** (2026-09-09T09:03): _[tool: Bash]_

**Claude** (2026-09-09T09:03): _[tool: Bash]_

**Claude** (2026-09-09T09:03): The file isn't where the bug report says. Let me find it.

**Claude** (2026-09-09T09:03): _[tool: Bash]_

**Claude** (2026-09-09T09:03): _[tool: Bash]_

**Claude** (2026-09-09T09:03): The path is `src/web/scripts/user-create.mts`. Note `db-demo.mts` also references `UserRole` — I'll check whether it's in scope.

**Claude** (2026-09-09T09:03): _[tool: Bash]_

**Claude** (2026-09-09T09:03): _[tool: Bash]_

**Claude** (2026-09-09T09:03): Confirmed: `users` has no `role` column, and the schema exports neither `USER_ROLES` nor `UserRole`. Let me check the Makefile target and the spec.

**Claude** (2026-09-09T09:03): _[tool: Bash]_

**Claude** (2026-09-09T09:03): _[tool: Bash]_

**Claude** (2026-09-09T09:03): The spec confirms roles are gone. Now applying the fix to both files.

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:04): Script is clean. Now the Makefile.

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:04): Both target files are done. The RUNBOOK (step 8, which you have open) still documents `ROLE=staff` — that's the same bug surfacing in docs. I'll note it rather than edit, since you scoped this to the script and Makefile. Now let me set up a real local test.

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:04): Postgres is installed locally. Let me check if a server is running and what the `.env` points at.

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:04): Postgres is up and the dev DB is local (no password). Let me check the dev database state before running anything against it.

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:04): The live dev DB matches the schema — no `role` column — and an org exists. It has 3 real users, so I'll create a clearly-marked test user and delete it afterwards rather than touching existing data.

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:04): End-to-end success — no import error, no ROLE needed. Let me verify the row landed correctly, confirm the password actually authenticates, and check the usage error path.

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:04): Both failed as expected, but npm swallowed the messages. Let me confirm the actual error text and verify the password authenticates against the real login path.

**Claude** (2026-09-09T09:04): _[tool: Bash]_

**Claude** (2026-09-09T09:05): _[tool: Bash]_

**Claude** (2026-09-09T09:05): Usage message is correct and role-free, and a stray `ROLE=` is harmlessly ignored. Let me verify the hash actually authenticates.

**Claude** (2026-09-09T09:05): _[tool: Bash]_

**Claude** (2026-09-09T09:05): Module resolution failed from the scratchpad (deps resolve relative to the workspace). I'll run the verification from inside the web workspace instead.

**Claude** (2026-09-09T09:05): _[tool: Bash]_

**Claude** (2026-09-09T09:05): The created account is fully usable: correct password verifies, wrong one rejects, and no `role` key exists on the row. Now cleaning up the test user and running typecheck/lint.

**Claude** (2026-09-09T09:05): _[tool: Bash]_

**Claude** (2026-09-09T09:05): _[tool: Bash]_

**Claude** (2026-09-09T09:05): Test user deleted (back to the original 3), temp file gone, and typecheck passes clean with zero errors.

Both files are fixed and the fix is verified end to end against the real local database.

## What changed

**[src/web/scripts/user-create.mts](src/web/scripts/user-create.mts)** — note the actual path is `src/web/scripts/`, not `scripts/` as the bug report said:

- [Line 15](src/web/scripts/user-create.mts#L15) — import is now `{ organisations, users }`; dropped `USER_ROLES` and `type UserRole`. This was the crash.
- [Lines 19-29](src/web/scripts/user-create.mts#L19-L29) — docblock no longer says "taking email, name and role"; usage line is now `make user-create EMAIL=... NAME=...`, with a pointer to the admin-v2-spec decision so this doesn't get re-added.
- [Lines 41-47](src/web/scripts/user-create.mts#L41-L47) — removed the `role` env read, dropped `role` from the required-args check, and deleted the `USER_ROLES.includes(role)` validation. Usage error is now `Usage: make user-create EMAIL=<email> NAME=<name>`.
- [Line 91](src/web/scripts/user-create.mts#L91) — insert no longer passes `role`.
- [Line 95](src/web/scripts/user-create.mts#L95) — output is `Created <email> (<id>).` instead of `Created <role> <email> (...)`.

**[Makefile](Makefile)**:
- [Line 35](Makefile#L35) — help text now reads `(EMAIL=... NAME=...)`.
- [Line 110](Makefile#L110) — target no longer forwards `ROLE="$(ROLE)"`.

No role column was added, and nothing else was touched.

## Test results

Ran against the real local dev DB (`halle_feedback_dev`), which I first confirmed has no `role` column:

- `make user-create EMAIL=claude-fixtest@example.invalid NAME="Claude Fix Test"` → `Created claude-fixtest@example.invalid (e395c74e-…)`. No import error.
- Row verified in Postgres: correct email/name, org linked, hash present, `disabled_at` null.
- Password check through the app's own `verify_password`: correct password → `true`, wrong password → `false`. Row keys are `id, org_id, email, name, password_hash, disabled_at, created_at` — no `role`.
- Missing `NAME` → `Usage: make user-create EMAIL=<email> NAME=<name>`.
- A leftover `ROLE=staff` on the command line is now silently ignored, so old muscle memory won't break.
- `tsc --noEmit` on the web workspace: clean, zero errors.

Cleanup: the test user was deleted (`DELETE 1`, table back to its original 3 users) and the temporary verification script removed. Your 3 pre-existing users were never touched.

## Two things to flag, not fixed (outside your scope)

1. **[deploy/RUNBOOK.md:297-300](deploy/RUNBOOK.md#L297-L300)** — the file you have open. Step 8 still says *"keep `ROLE=staff`"* and shows `make user-create EMAIL=… NAME="Vishnu" ROLE=staff`, plus expected output `Created staff vishnu@…`. That command still works now, but the instruction and expected output are stale. Worth a one-line fix before anyone follows the runbook on the server.

2. **[src/web/scripts/db-demo.mts:12](src/web/scripts/db-demo.mts#L12)** — has the identical latent bug: it imports `type UserRole` from the schema and builds `DEMO_USERS` with a `role` field. It'll fail the same way when run. `tsc` didn't flag it, so it's likely outside the web tsconfig's include. Say the word and I'll fix it the same way.

**Vishnu** (2026-09-09T09:06): Yes, please fix both:
1. Update deploy/RUNBOOK.md step 8 to remove ROLE=staff and match the new command (make user-create EMAIL=... NAME=...) and its expected output.
2. Apply the same fix to src/web/scripts/db-demo.mts as you did to user-create.mts.

**Claude** (2026-09-09T09:06): I'll fix both. Starting by reading the current state of each.

**Claude** (2026-09-09T09:06): _[tool: Bash]_

**Claude** (2026-09-09T09:06): _[tool: Bash]_

**Claude** (2026-09-09T09:06): `db-demo.mts` needs a judgment call the previous file didn't: its three demo users exist *only* to demo the three roles. With roles gone they'd be three identical logins. Let me check what consumes them before deciding.

**Claude** (2026-09-09T09:06): _[tool: Bash]_

**Claude** (2026-09-09T09:06): `scripts/demo.sh` hardcodes all three emails in its output, so collapsing to one login would mean editing that too. That's a scope question worth asking rather than guessing.

**Claude** (2026-09-09T09:06): _[tool: AskUserQuestion]_

**Claude** (2026-09-09T09:07): Keeping all three and dropping only the role. Applying both fixes.

**Claude** (2026-09-09T09:07): _[tool: Bash]_

**Claude** (2026-09-09T09:07): I also corrected a second error in that step: it claimed a password is "generated" and "printed once", but the script actually prompts for one interactively. Now `db-demo.mts`.

**Claude** (2026-09-09T09:07): _[tool: Bash]_

**Claude** (2026-09-09T09:07): Both edits applied. Now testing `db-demo` for real. Let me check the dev DB state first, since it writes to it.

**Claude** (2026-09-09T09:07): _[tool: Bash]_

**Claude** (2026-09-09T09:07): The 3 existing users *are* the demo users, so this is a safe re-run — it should hit the idempotent "already exists" path. Let me confirm the old code actually crashed, then that the new code works.

**Claude** (2026-09-09T09:07): _[tool: Bash]_

**Claude** (2026-09-09T09:07): That's an important correction to what I told you earlier: with `type UserRole` re-added, the script ran fine rather than crashing. That's because `--experimental-strip-types` erases `type`-only imports at parse time, so they never hit the module resolver. Let me verify that's the real explanation.

**Claude** (2026-09-09T09:08): _[tool: Bash]_

**Claude** (2026-09-09T09:08): That confirms it precisely: the crash came from `USER_ROLES` being a **value** import (case B), not from `type UserRole` (case A, erased at parse time). So `db-demo.mts` was never going to crash — I overstated that in my last message.

It still had a genuine bug, though. Let me verify that.

**Claude** (2026-09-09T09:08): _[tool: Bash]_

**Claude** (2026-09-09T09:08): Drizzle silently drops the unknown `role` key rather than erroring. So the old `db-demo.mts` would have kept working — the bug was real but cosmetic (dead type, misleading `+ staff login` output), not a crash. Let me clean up and run the actual script.

**Claude** (2026-09-09T09:08): _[tool: Bash]_

**Claude** (2026-09-09T09:08): Clean. Now the real run of the fixed `db-demo`, on the idempotent path.

**Claude** (2026-09-09T09:08): _[tool: Bash]_

**Claude** (2026-09-09T09:08): Idempotent path works and all four `DEMO_*` lines that `demo.sh` parses are intact. I also need to exercise the **insert** path, which the existing users skipped.

**Claude** (2026-09-09T09:08): _[tool: Bash]_

**Claude** (2026-09-09T09:08): The insert branch works — `+ login developer@demo.test`, row recreated correctly, and the DB is back to its original 3 demo users. Final verification.

**Claude** (2026-09-09T09:08): _[tool: Bash]_

**Claude** (2026-09-09T09:08): _[tool: Bash]_

**Claude** (2026-09-09T09:09): That "REFUSING" is the guard working correctly — it detects all three demo accounts still hold the shared demo password, which confirms the hashes are valid and verifiable. Typecheck is clean and no `ROLE=`/`UserRole` references remain.

Both fixes are done and verified against the real local database.

## 1. [deploy/RUNBOOK.md](deploy/RUNBOOK.md) — step 8

[Lines 297-308](deploy/RUNBOOK.md#L297-L308): dropped `ROLE=staff` from both the instruction and the command, and changed the expected output from `Created staff vishnu@aracreate.group (...)` to `Created vishnu@aracreate.group (...)`.

While in there I found a **second error in the same paragraph**, unrelated to roles: it said a password is *"generated"* and *"printed once — copy that password now"*. The script has never done that; it prompts `Password:` interactively. Anyone following the runbook would have waited for a password that never appeared. Replaced with an accurate description of the prompt and the 8-character minimum.

## 2. [src/web/scripts/db-demo.mts](src/web/scripts/db-demo.mts)

Per your choice, kept all three demo logins and removed only the role:
- [Line 12](src/web/scripts/db-demo.mts#L12) — dropped `type UserRole` from the import.
- [Lines 29-36](src/web/scripts/db-demo.mts#L29-L36) — `DEMO_USERS` is now `Array<{ email; name }>`, with a comment recording why three interchangeable logins remain (demo.sh prints all three) and pointing at the spec.
- [Lines 80, 89](src/web/scripts/db-demo.mts#L80-L89) — log lines are `+ login <email>` / `= login <email>` instead of `+ staff login <email>`.
- [Line 83](src/web/scripts/db-demo.mts#L83) — insert no longer passes `role`.

`scripts/demo.sh` needed no changes, as expected.

## Correction to what I told you last message

I said db-demo.mts "has the identical latent bug: it'll fail the same way when run." **That was wrong, and I verified why.** I tested both import forms in isolation:

- `import { USER_ROLES }` — a **value** import — crashes with `ERR_MODULE_NOT_FOUND`. That was the real user-create bug.
- `import { type UserRole }` — a **type-only** import — runs fine. Node's `--experimental-strip-types` erases it at parse time, so it never reaches the module resolver.

I also checked the `role` field in the insert: Drizzle silently drops keys that aren't in the table schema, so that wouldn't have thrown either. I confirmed this by running an insert with `role` against the live table — it succeeded.

So db-demo.mts was **not** broken and `make demo` was not failing. The bug was real but cosmetic: a dead type reference that would break at the next typecheck of that file, and misleading `+ staff login` output implying roles that no longer exist. Worth fixing, but it was not the crash I implied.

## Test results

- `db:demo` idempotent path: `= login` for all three, tester found, all four `DEMO_*` lines that demo.sh parses intact.
- Insert path (deleted `developer@demo.test` to force it): `+ login developer@demo.test`, row recreated with correct email/name/hash and no role.
- `check:demo-password` correctly detects all three accounts on the shared demo password — proving the hashes verify.
- `tsc --noEmit` on the web workspace: clean.
- DB restored to its original 3 users; all temp files removed.
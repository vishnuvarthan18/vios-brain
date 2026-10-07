# Deploying

The server is `aca-htz-vcet` at `89.167.82.144`, reached as `ssh hetzner`.
It runs Debian 13 with Node 20, PostgreSQL 17 and Caddy already installed.
The site is `https://vcet.aracreate.academy`.

## Once

The repository is private and the server holds no GitHub credentials, so the
code is copied from the machine that has the repo rather than cloned.

From your Mac:

```sh
rsync -az --delete \
  --exclude node_modules --exclude .git --exclude .env \
  --exclude uploads --exclude .archives --exclude logs \
  ~/araCreate/bootcamp-dashboard/ hetzner:/tmp/bootcamp-src/

ssh hetzner 'cd /tmp/bootcamp-src && bash scripts/setup-server.sh'
```

The excludes are the same every time this file uses rsync. They are explained
under *Why the excludes matter*; do not trim them, even here where the target
is empty — this is the line people copy.

It prints the URL and the staff password when it finishes. **Write the staff
password down** — it exists only in `/opt/bootcamp-dashboard/.env`.

DNS has to point at the box before this runs. Caddy proves the domain by being
asked for it, so a name that does not resolve gets no certificate.
`vcet.aracreate.academy` is already live.

## Afterwards

A deploy is six steps and they are all necessary. Step 5 has been needed on
every deploy that ran a migration; skipping it — or leaving its restart until
later — takes every route down.

**The exclude list below is not optional and not a suggestion.** Both rsync
lines carry `--delete`, and the second one writes into the live directory. See
*Why the excludes matter* below for what happens without them.

**1. Dump the database first.** It is the only thing to restore from, and a
deploy that goes wrong is exactly when you will want it:

```sh
ssh hetzner 'sudo -u postgres pg_dump bootcamp | gzip > /tmp/predeploy.sql.gz \
  && sudo mv /tmp/predeploy.sql.gz \
       /opt/bootcamp-dashboard/.archives/PREDEPLOY-bootcamp-$(date +%F-%H%M%S).sql.gz \
  && sudo gunzip -t /opt/bootcamp-dashboard/.archives/PREDEPLOY-*.sql.gz && echo OK'
```

**2. Build the payload from the COMMIT, never from the working tree.**

`git archive` writes exactly what is in a commit and nothing else. rsync reads
the tree, so it also sends whatever is merely sitting on disk — half-finished
edits, another agent's untracked migration, a scratch file. On 18 Sep a second
agent's uncommitted `src/server.js` edit reached production this way, because
rsync reads the tree, not the commit.

Name the commit explicitly. `HEAD` is fine when you have just merged and
checked; a SHA is better when anyone else is working in the same clone:

```sh
COMMIT=$(git rev-parse HEAD)          # or the exact SHA you mean to ship
rm -rf /tmp/deploy-payload && mkdir -p /tmp/deploy-payload
git archive "$COMMIT" | tar -x -C /tmp/deploy-payload
echo "shipping $COMMIT"
```

Nothing that is not committed can get out of this step. That is the point: it
is not a tidiness rule, it is what makes the thing you tested and the thing you
ship the same thing.

**2b. The built front end is NOT in the commit.** `src/public/v3/` is
gitignored — it is a build output — so `git archive` leaves it out, and step 4's
`--delete` would then wipe it from the server. That is a blank page for every
user.

Build it and copy it into the payload before sending:

```sh
(cd web && npm run build)
cp -R src/public/v3 /tmp/deploy-payload/src/public/
ls /tmp/deploy-payload/src/public/v3/index.html    # must exist before step 3
```

Found on 23 Sep, when the payload went up without it. The dry run in step 4 is
what catches this: if it lists `deleting src/public/v3/...`, stop and do this.

**3. Send the payload up**, into a staging directory first:

```sh
rsync -az --delete \
  --exclude node_modules --exclude .git --exclude .env \
  --exclude uploads --exclude .archives --exclude logs \
  /tmp/deploy-payload/ hetzner:/tmp/bootcamp-src/
```

The excludes still matter. `git archive` has no `.env` or `uploads` in it, but
the **destination** does on a repeat deploy, and `--delete` is what would
remove them.

**4. Copy it into place and install.** Add `--dry-run` first and read what it
says it will delete — it should say nothing:

```sh
ssh hetzner 'sudo rsync -a --delete \
  --exclude .env --exclude uploads --exclude .archives --exclude logs \
  --exclude node_modules --exclude .npm \
  /tmp/bootcamp-src/ /opt/bootcamp-dashboard/ \
  && sudo chown -R bootcamp:bootcamp /opt/bootcamp-dashboard \
  && bash /opt/bootcamp-dashboard/scripts/update.sh'
```

**5. If you ran any migration, fix the ownership and restart — in that order,
back to back, as one step.** See *Running migrations* below. This is a step,
not a footnote, and the restart is part of it:

```sh
# ... the ownership DO block from *Running migrations* ...
ssh hetzner 'sudo systemctl restart bootcamp'
```

**Do not leave a gap between the ownership fix and the restart.** On 18 Sep the
ownership work finished and the restart came **2m 40s later**, run by hand. For
those 2m 40s the old process was still serving from handles it no longer had
rights to, and every student request failed with `EACCES` — 54 of them across
`/api/profile`, `/api/my-team`, `/api/quiz/open`, `index.html` and the rest,
plus one photo upload lost for good. The restart is what ends the outage, so it
belongs in the same step and the same breath as the chown, not in step 5.

Note that `update.sh` restarts the service itself as its last line. That
restart happens *before* any migration runs, so it does not cover this — step 4
needs its own restart after the ownership fix.

**5b. devDependencies are not on the server.** `update.sh` installs with
`npm ci --omit=dev`, so anything in `devDependencies` is absent. On 23 Sep
`src/routes/certificates.js` required `playwright` at the top of the file; the
module could not resolve, `src/server.js` could not load, and the service would
not start — the whole product down, for every user, over one import in one
route.

Require such things INSIDE the function that needs them, never at module load.
A feature that needs a dev-only package should degrade to a clear error on its
own routes, not take the server with it.

If a feature genuinely needs one in production, install it explicitly and say so
here:

```sh
ssh hetzner 'cd /opt/bootcamp-dashboard && sudo -u bootcamp env HOME=/tmp \
  npm install playwright --no-save && sudo -u bootcamp env HOME=/tmp \
  npx playwright install --with-deps chromium'
```

Certificates need this one: they render a PDF in headless Chromium.

**6. Check it came back**, before you walk away:

```sh
ssh hetzner 'systemctl is-active bootcamp'
curl -s -o /dev/null -w '%{http_code}\n' https://vcet.aracreate.academy/
```

## Running migrations

**1. Run them one at a time**, in the order in `src/db/migrations/readme.md`,
not alphabetical order. Two of them fail out of turn. Check each before
starting the next — a migration that half-applied is worth knowing about
before the next one builds on it.

Pipe the file in rather than passing a path. The app directory is owned by
`bootcamp` and `postgres` cannot always read into it, which fails as
`Permission denied` before a single statement runs:

```sh
ssh hetzner "sudo cat /opt/bootcamp-dashboard/src/db/migrations/<NAME>.sql \
  | sudo -u postgres psql -v ON_ERROR_STOP=1 -d bootcamp"
```

**2. Reassign ownership, every single time, immediately afterwards.**

This is a step of the deploy, not a tidy-up. It was needed on both deploys on
18 Sep. Run it even if only one migration ran, and even if that migration
"only" added a column — it is cheap when unnecessary and it is an outage when
skipped.

`psql` runs as `postgres`, so everything a migration creates — tables, views,
sequences, functions — ends up owned by `postgres`. The app connects as
`bootcamp` and gets `permission denied for view v_leaderboard` on the first
page a student opens. It is a 500 on every route and it looks like a
catastrophe; it is one command:

```sh
ssh hetzner "sudo -u postgres psql -d bootcamp <<'SQL'
DO \$\$
DECLARE r RECORD;
BEGIN
  FOR r IN SELECT tablename FROM pg_tables WHERE schemaname='public' AND tableowner<>'bootcamp' LOOP
    EXECUTE format('ALTER TABLE public.%I OWNER TO bootcamp', r.tablename);
  END LOOP;
  FOR r IN SELECT viewname FROM pg_views WHERE schemaname='public' AND viewowner<>'bootcamp' LOOP
    EXECUTE format('ALTER VIEW public.%I OWNER TO bootcamp', r.viewname);
  END LOOP;
  FOR r IN SELECT c.relname FROM pg_class c JOIN pg_namespace n ON n.oid=c.relnamespace
            WHERE n.nspname='public' AND c.relkind='S' AND pg_get_userbyid(c.relowner)<>'bootcamp' LOOP
    EXECUTE format('ALTER SEQUENCE public.%I OWNER TO bootcamp', r.relname);
  END LOOP;
  FOR r IN SELECT p.proname, pg_get_function_identity_arguments(p.oid) args
             FROM pg_proc p JOIN pg_namespace n ON n.oid=p.pronamespace
            WHERE n.nspname='public' AND pg_get_userbyid(p.proowner)<>'bootcamp' LOOP
    EXECUTE format('ALTER FUNCTION public.%I(%s) OWNER TO bootcamp', r.proname, r.args);
  END LOOP;
END \$\$;
SQL"
```

**3. Confirm nothing is left.** This must print nothing at all. If it prints a
name, the app cannot read that object and step 2 did not finish:

```sh
ssh hetzner "sudo -u postgres psql -d bootcamp -tA \
  -c \"SELECT tablename FROM pg_tables WHERE schemaname='public' AND tableowner<>'bootcamp'\" \
  -c \"SELECT viewname FROM pg_views WHERE schemaname='public' AND viewowner<>'bootcamp'\""
```

**4. Restart now, not later.**

```sh
ssh hetzner 'sudo systemctl restart bootcamp'
```

The running process is still holding what it picked up before the migration.
Until it restarts, the ownership fix above has not reached the app — the
database is correct and students are still getting errors. Every minute
between step 2 and this one is a minute of outage. See step 4 under
*Afterwards*.

## The Google private key is quoted, and that matters

`GOOGLE_PRIVATE_KEY` in `.env` is wrapped in double quotes, and the key inside
carries its newlines written as `\n` rather than as real line breaks. Both are
deliberate: a `.env` line cannot hold a real newline, and `drive.js` puts the
newlines back itself.

The quotes are stripped by systemd, which reads `.env` as an `EnvironmentFile`.
That is the only reason it works. Load the same file any other way — `source`
it in a shell, `export` it by hand, read it with a bare dotenv — and the quotes
arrive as part of the value. OpenSSL then cannot parse the key and every Drive
call dies with:

```
error:1E08010C:DECODER routines::unsupported
```

which says nothing whatsoever about quotes. It is the same error a key with its
newlines mangled gives, so it is easy to spend a while fixing the wrong thing.

If you are reading `.env` from a script, strip a matching pair of leading and
trailing quotes before using the value:

```js
if ((v.startsWith('"') && v.endsWith('"'))
 || (v.startsWith("'") && v.endsWith("'"))) v = v.slice(1, -1);
```

To tell the two cases apart, check whether the key still signs — this prints
`true` on a healthy box and never prints any part of the key:

```sh
ssh hetzner 'cd /opt/bootcamp-dashboard && sudo -u bootcamp node -e "
  const fs = require(\"fs\"), crypto = require(\"crypto\");
  const line = fs.readFileSync(\".env\", \"utf8\").split(\"\\n\")
    .find(l => l.startsWith(\"GOOGLE_PRIVATE_KEY=\"));
  let v = line.slice(19);
  if (/^[\"\x27]/.test(v)) v = v.slice(1, -1);
  try { crypto.createSign(\"RSA-SHA256\").update(\"x\")
    .sign(v.replace(/\\\\n/g, \"\\n\")); console.log(true); }
  catch (e) { console.log(false, e.message); }"'
```

## Why the excludes matter

The commands under *Afterwards* already carry the right flags. This is why they
are there, so nobody trims them back.

`--delete` makes the destination match the source. The staging directory has no
`uploads/` and no `.archives/` — they are excluded on the way up — so without
the same excludes on the way down, rsync sees two directories the source does
not have and removes them.

Measured on the 18 Sep deploy, the older single-exclude command would have
deleted **109 upload files and 3 archives**:

| | |
| --- | --- |
| `uploads/resumes/` | the students' CVs, handed in once, **not** in `pg_dump` |
| `uploads/photos/` | profile headshots, same |
| `.archives/` | **including the PREDEPLOY dump taken minutes earlier** |
| `logs/` | written by the app at runtime, and in no commit |

The `.archives/` row is the one to sit with. A deploy that destroyed the uploads would
also have destroyed the only thing to restore them from, in the same command.

`logs/` is the same category as `uploads/` and `.archives/`: the app writes it
at runtime, so it exists on the server and in no commit. Anything `git archive`
cannot produce will look to `--delete` like a file the source does not have.
A deploy must never delete runtime data — that is the whole rule, and `logs/`
belongs under it.

`node_modules` and `.npm` are excluded for a smaller reason: deleting them makes
the next `npm ci` mandatory rather than merely expected, and a failed install
then leaves the app unable to start. Keeping them means step 3 is additive.

### If a dry run offers to delete something not on that list

It will happen, because the list is a record of what has turned up so far
rather than a prediction. On 18 Sep a dry run offered to remove
`.worktrees/side/` — 102 files of another lane's checkout, shipped by an
earlier rsync-the-tree deploy — and a stray `.DS_Store`.

Read what it names before you let it run. The question to ask of each path is
not "is this in the commit" but **"did the app write this, or did a person's
machine put it there?"**

| | |
| --- | --- |
| The app wrote it at runtime | **exclude it.** `uploads/`, `logs/`, `.archives/` |
| An earlier bad deploy left it | **let `--delete` take it**, once you have looked |
| A person's editor or OS made it | **let it go**, e.g. `.DS_Store` |

Nothing on the second and third rows is worth keeping, but nothing on them is
worth deleting blind either: a directory whose name you do not recognise is a
reason to stop and look, not a reason to add another `--exclude`. The excludes
exist to protect runtime data, not to silence a dry run.

Day to day:

```sh
ssh hetzner 'sudo journalctl -u bootcamp -f'      # watch the logs
ssh hetzner 'sudo systemctl restart bootcamp'      # restart
ssh hetzner 'sudo -u postgres psql -d bootcamp'    # open the database
```

## What the setup does

| | |
| --- | --- |
| Packages | installs only what is missing — this box already has all three |
| Database | `bootcamp`, owned by a `bootcamp` role with a generated password |
| Data | schema and both rosters, **only if the database has no students** |
| Secrets | a fresh `STAFF_PASSWORD` and `SESSION_SECRET`, written to `.env` at mode 600 |
| Service | systemd, restarts on crash and on boot |
| HTTPS | Caddy, certificate renewed automatically |
| Firewall | ufw: port 22 by number, plus 80 and 443 |
| Backups | nightly `pg_dump` to `/var/backups/bootcamp`, 14 days kept |

It is safe to re-run. It never overwrites an existing `.env`, and it never
reloads a database that already has students.

## What the box actually has

Measured on 18 Sep 2026 during an incident check. None of it was written down
before, which is the reason it is here — these are facts to know in advance,
not things to discover at the point you need them.

| | |
| --- | --- |
| CPU | **2 cores** |
| RAM | **3.8 GB** total — 588 MB in use, ~3.2 GB available at 32 students signed in |
| Disk | **38 GB**, 2.4 GB used, **34 GB free** (7%) |
| Swap | **none configured — 0 bytes** |
| Load average | 0.02 / 0.02 / 0.00 under that load |

**No swap is fine at current usage and is not a fault to go and fix.** The app
peaks around 60 MB and the box is using about 15% of its memory with the whole
cohort on it. It matters because there is no cushion: if something does run the
box out of memory, the kernel goes straight to killing a process rather than
slowing down first. So if the app is ever OOM-killed, absence of swap is the
explanation, and the fix is to find what grew — not to assume the box is
undersized.

To re-measure:

```sh
ssh hetzner 'nproc; free -m; df -h /; cat /proc/loadavg'
```

## Access logging

**There is none, and that is a real gap.** The Caddyfile is three lines with no
`log` directive, `/var/log/caddy/` is empty, and the app logs no status codes
or timings. During the 18 Sep check it was impossible to answer three ordinary
questions from the logs: p50, p95, and 5xx grouped by route. Latency had to be
measured by probing the live service, which only shows what is true right now
and says nothing about what students hit an hour ago.

**Do this at the next deploy, not on its own.** It restarts Caddy, which drops
in-flight connections — harmless in a quiet window, needless disruption while
students are working.

Change `/etc/caddy/Caddyfile` to:

```
vcet.aracreate.academy {
	encode zstd gzip
	log {
		output file /var/log/caddy/vcet-access.log {
			roll_size 10MiB
			roll_keep 10
		}
		format json
	}
	reverse_proxy 127.0.0.1:3000
}
```

Then:

```sh
ssh hetzner 'sudo caddy validate --config /etc/caddy/Caddyfile \
  && sudo systemctl reload caddy'
```

`caddy validate` first — a Caddyfile that does not parse takes the site off the
internet, certificate and all. `reload` rather than `restart` picks up the
change without dropping connections.

The log is JSON, so reading it needs `jq`, which **this box does not have**.
Install it in the same quiet window:

```sh
ssh hetzner 'sudo apt-get install -y jq'
```

Afterwards, p50/p95 and status-by-route come out of the log directly. `duration`
is in seconds and `status` is the response code:

```sh
# p50 / p95 for a route
ssh hetzner "sudo jq -r 'select(.request.uri|startswith(\"/api/profile\"))|.duration' \
  /var/log/caddy/vcet-access.log | sort -n | \
  awk '{a[NR]=\$1} END {printf \"p50=%.3fs p95=%.3fs n=%d\\n\", a[int(NR*.5)], a[int(NR*.95)], NR}'"

# 5xx grouped by route and status
ssh hetzner "sudo jq -r 'select(.status>=500)|\"\\(.status) \\(.request.uri)\"' \
  /var/log/caddy/vcet-access.log | sort | uniq -c | sort -rn"
```

Log volume is not a concern: 10 MiB × 10 rolled files is bounded, against 34 GB
free.

## The one thing not to do

`load-eee.sql` and `load-ece.sql` each begin by deleting their department's
students, which cascades to every daily post, attendance mark and quiz answer
those students have. They are setup scripts, not start commands. Never run them
by hand on a live server.

# Ops findings — 19 Sep 2026 (Day 2)

Found while preparing a local dump for the dev agent. Recorded so none of it is
lost.

---

## 1. Nightly backups had never run — FIXED

**Symptom.** `/var/backups/bootcamp` held two files, both dated 17 Sep, both
manual pre-deploy dumps. No nightly file had ever appeared, despite the handover
stating a `pg_dump` runs nightly at 01:00.

**Not the cause.** The cron job existed and ran correctly every night:

```
Sep 19 01:00:01 CRON[153618]: (postgres) CMD (pg_dump bootcamp | gzip > /var/backups/bootcamp/$(date +%F).sql.gz)
Sep 19 01:00:01 CRON[153616]: (CRON) info (No MTA installed, discarding output)
```

**The cause.** `/var/backups/bootcamp` was `drwxr-xr-x root root`. The cron runs
as `postgres`, which could read the directory but not write to it. The redirect
failed every night. With no MTA installed the error was discarded and nothing
ever surfaced it.

**The fix.**

```sh
sudo chown postgres:postgres /var/backups/bootcamp
sudo chmod 750 /var/backups/bootcamp
```

Verified the same day: a manual run as `postgres` produced `2026-09-19.sql.gz`,
80K, exit 0.

**Consequence while it was broken.** Between 17 and 19 September the only backup
of the live database predated the entire v2 deployment. Two days of a live
bootcamp existed in exactly one place.

**Still to do, in a quiet window:**

- Replace the cron one-liner with a script that writes to `.tmp`, runs
  `gzip -t`, and only then renames — so a truncated dump can never look like a
  good backup.
- Send its output to `/var/log/bootcamp-backup.log` instead of nowhere.
- Check that log the following morning. A backup job is not working until a log
  line says it worked.

**The pattern.** Same shape as the eight v2 bugs: the new thing was written
correctly and the layer underneath it silently refused. Add "a scheduled job
that has never produced a file" to the v3 gate list.

---

## 2. What cron actually runs

**The server runs on UTC, not IST.** The "1 AM nightly backup" fires at **06:30
Indian time**. Do not look for it earlier.

`/etc/cron.d/bootcamp-backup`:

| UTC | IST | As | What |
| --- | --- | --- | --- |
| 01:00 | **06:30** | postgres | `pg_dump bootcamp \| gzip > /var/backups/bootcamp/<date>.sql.gz` |
| 01:30 | **07:00** | root | delete `*.sql.gz` older than 14 days |

Debian's own jobs, unrelated to the app: `/etc/cron.hourly` at :17 past the
hour, `/etc/cron.daily` at 06:25 UTC, `e2scrub_all` at 03:10 UTC,
`dpkg-db-backup.timer` at 00:00 UTC.

There is **no root crontab and no postgres crontab**. Everything lives in
`/etc/cron.d`.

---

## 3. Morning check, every day

After 07:00 IST:

```sh
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
```

- A file for today must exist.
- Around **80K or larger** — it grows as the bootcamp fills up.
- **Missing** means the fix did not hold. **19K** means it dumped the wrong
  thing. Either way, stop and investigate before the day starts.

Then the normal day: open today's activities per venue · load quiz questions ·
re-run the CV migration for stragglers.

---

## 4. Findings from the dump's row counts

Taken on Day 2. Each needs checking; none is diagnosed yet.

| Table | Rows | What it suggests |
| --- | --- | --- |
| `daily_posts` | **7** | Two days in, 209 students. The nine-day arc depends on these. Either not released, or nobody has been told |
| `task_submission_orphans` | **50** | Against 56 real `task_submissions`. Nearly one in two. Possibly real student work not being counted |
| `submissions` | **0** | 159 projects exist |
| `scores` | **0** | No project scored at all. If the leaderboard reads `scores`, it is ranking on nothing |
| `assessment_attempts` | **154** | Exactly the ECE cohort. Confirms EEE's 55 have still not had the pre-assessment |
| `assessment_questions` | **4** | The whole assessment is four questions |
| `student_profiles` | **208** | One student of 209 has no profile row |
| `quiz_questions` | **0** | Known. Still blocks every quiz |
| `attendance` | **419** | Across two days for 209 students — near-complete. This part is working well |
| `students` / `teams` | **209 / 53** | Matches production |

**Most urgent: `submissions` and `scores` both at zero.** If projects genuinely
have no hand-ins and no scores two days in, that is the core of the bootcamp not
functioning — and no chase list would reveal it, because it is a schema
question, not a student one.

---

## 5. Two findings raised by the dev agent, not yet acted on

- `src/db/migrations/readme.md` lists 9 migrations; the directory holds 16. The
  six from 18–19 Sep have no documented run order, and it is not known which are
  applied to production.
- `v_student_progress` still exposes `has_photo` and `has_education` after both
  were removed from the profile-completion weights in
  `src/routes/profile-completion.js`. Anything reading that view to decide
  whether a profile is complete is wrong today.

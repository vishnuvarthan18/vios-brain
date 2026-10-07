# The CVs on the server, and their copies on Drive

The handed-in resumes live in two places: `/opt/bootcamp-dashboard/uploads/`
`resumes/` on the server, and the Shared Drive `ac-vcet`, one folder per team.
`scripts/migrate-cvs.js` copies the first to the second. It has **no `--delete`
flag** and never removes anything; deleting the server's copies is a separate,
later job.

This file exists for that later job.

**The one rule that job runs on: a file with no verified Drive copy does not
get deleted.** See [The rule](#for-the-deletion-job-next-week) below.

## For the deletion job next week

> ### The rule
>
> **A file with no verified Drive copy does not get deleted.**
>
> Not because this document says it was copied. Not because an earlier report
> said so. Not because a database row carries a `drive_url`. Verified means
> *you asked Drive, at the moment you were about to delete, and the file was
> there with the same byte size*.
>
> Everything else on this page is how to satisfy that rule. If the two ever
> disagree, the rule wins and the file stays.

These are 88+ students' own CVs, handed in once. They are not in `pg_dump`, and
the server copy is the only other one. A file deleted here without a verified
copy is gone.

**Re-verify against Drive at the time you run it. Do not trust the numbers
below.** They are a snapshot from 18 Sep, taken while students were still
uploading, and they were already stale by the end of that afternoon. They are
here so you can see the *shape* of the problem, not so you can act on them.

As of 18 Sep, late afternoon:

| | |
| --- | --- |
| Resumes on disk | **135** |
| Copied to Drive and verified | **129** |
| Known orphans, no database row | **4** (listed below) |
| Still rising? | **yes** — 83 the night before, 131 by 04:40 |

Those numbers do not reconcile to zero, and they are not supposed to: the gap
is the test-team CV that is deliberately not on Drive, files uploaded after the
last run, and the four orphans. Every one of those categories will have changed
by the time anyone deletes anything.

So the job's first act is to rebuild the picture, not to read this table:

1. **Re-run the migration first** (`--commit`), so anything uploaded since is on
   Drive. It skips what is already copied, so this is cheap.
2. **Re-run the orphan check** below; the list will be longer than four.
3. **For every file you are about to delete, confirm its Drive copy exists and
   its byte size matches** — at that moment, against Drive, not against a
   record of a previous check. A row carrying a `drive_url` is not proof the
   file is still there; somebody can delete things in the Drive UI.
4. Only then delete, and delete by walking database rows, never the directory.

Step 3 is the rule at the top of this section, in the one place it has to be
obeyed. A file that fails it is skipped and reported, not deleted and
mentioned. If that leaves files on disk at the end of the job, that is a
correct outcome, not an unfinished one.

## Read this before deleting anything from uploads/resumes/

The deletion job will work from what is on disk. Disk and database do not
match exactly, and the difference is not an error — it is students
re-uploading. **A file on disk is not proof that a row points at it.**

Delete by walking the database rows, not by walking the directory. For each
row with a `resume_vN_drive_url`, delete the file its `resume_vN_url` names.
Anything left over is one of the orphans below, and those are a separate
decision.

## Orphans: on disk, referenced by no row

Recorded 18 Sep 2026, after the first full migration.

| File | Size | Last written | Why it is stranded |
| --- | --- | --- | --- |
| `v1-99.pdf`   | 122,683 bytes   | 2026-09-17 08:07 | PONARASI V (732925ECR120) re-uploaded as `.docx` |
| `v1-100.pdf`  | 1,273,629 bytes | 2026-09-18 03:29 | NIKITHA V (732925ECR115) re-uploaded as `.docx` |
| `v1-103.docx` | 37,987 bytes    | 2026-09-17 15:56 | NARMATHA S (732925ECR111) re-uploaded as `.pdf` |
| `v1-158.docx` | 10,208 bytes    | 2026-09-17 14:31 | RUBESH R (732925ECR132) re-uploaded as `.pdf` |

Every one of these is a **superseded earlier upload**. In each case the
student handed in a replacement in a different format, the row moved to the
new extension, and the old file was left behind — the upload path writes
`v1-<id>.<ext>`, so a change of format writes a new filename rather than
overwriting.

The current CV of all four students is copied to Drive and verified. Nothing
here is anybody's only copy.

**None of these were copied to Drive**, deliberately: the migration works from
database rows, and no row points at them.

So when the deletion job runs, these four will be on disk with no Drive copy
and no row referencing them. That is expected. Deleting them is almost
certainly right — but make it a decision somebody takes, not something that
happens because a script could not find a row. If in doubt, copy them to a
`0-superseded/` folder on Drive first; they total 1.4 MB.

Re-run the orphan check before deleting, because the list will have grown —
students kept re-uploading all through Day 1:

```sh
ssh hetzner 'sudo -u postgres psql -d bootcamp -tA -c \
  "SELECT regexp_replace(resume_v1_url, '"'"'^/uploads/resumes/'"'"', '"'"''"'"')
     FROM student_profiles WHERE resume_v1_url LIKE '"'"'/uploads/resumes/%'"'"'
   UNION
   SELECT regexp_replace(resume_v2_url, '"'"'^/uploads/resumes/'"'"', '"'"''"'"')
     FROM student_profiles WHERE resume_v2_url LIKE '"'"'/uploads/resumes/%'"'"'" \
  | sort > /tmp/referenced.txt
  sudo ls -1 /opt/bootcamp-dashboard/uploads/resumes | sort > /tmp/ondisk.txt
  comm -13 /tmp/referenced.txt /tmp/ondisk.txt'
```

The same command the other way round (`comm -23`) lists rows whose file is
missing from disk. On 18 Sep that was empty, and it should stay empty: a row
pointing at a file that is not there means a CV was lost, not superseded.

## The count keeps rising

The migration is safe and cheap to re-run: a row that already carries a
`resume_vN_drive_url` is **skipped without a byte being fetched** — no ssh, no
Drive call, no upload. Re-running when there is nothing to do costs a single
database query. That is what makes the numbers below unremarkable rather than
alarming, and it is why the checklist below says to run it again three times.

| When | v1 CVs handed in |
| --- | --- |
| 17 Sep, evening | 83 |
| 18 Sep, 03:45 | 114 |
| 18 Sep, 04:10 | 127 |
| 18 Sep, 04:40 | 131 |
| 18 Sep, late afternoon | 135 on disk, 129 on Drive |

Students were uploading *during* the migration itself. Three separate passes
were needed on 18 Sep just to catch up with a moving target, so treat one
clean run as "current as of a minute ago" rather than as finished.

## Re-run checklist

You do not need to have been there for any of this. Each run is the same two
commands, and a run with nothing to do is harmless.

**Before any run**, check where you are pointed. Running this against a local
database reads the wrong rows and reports confident nonsense:

```sh
node -e "require('/path/to/repo/src/db').check().then(r=>console.log(r))"
```

It must print the production database. If it prints your laptop's, stop.

### 1. End of Day 1, once the room has emptied

The point of waiting is that the number stops moving. Anyone who uploaded in
the last ten minutes of the session is included.

```sh
node scripts/migrate-cvs.js              # dry run: read the numbers
node scripts/migrate-cvs.js --commit     # then copy
```

Expect the dry run to report a handful outstanding. If it reports zero, you are
done — that is a valid outcome, not a sign something is broken.

### 2. Morning of Day 2

For anyone who uploaded overnight or on the way in. Same two commands. This is
usually a very short run.

### 3. Day 8, when the new CVs arrive

Day 8 is when students build a second resume. The script does **both versions
by default** and needs no extra flag — `--version v2` only narrows it if you
want v2 alone.

```sh
node scripts/migrate-cvs.js              # will now list v1 and v2
node scripts/migrate-cvs.js --commit
```

v1 rows copied on Day 1 are skipped untouched. Only the new v2 files move.

### After every run

Two things to confirm, both quick:

```sh
# Nothing left outstanding. Empty output is the goal.
ssh hetzner "sudo -u postgres psql -d bootcamp -c \"
  SELECT s.roll_no, s.name FROM student_profiles p
    JOIN students s ON s.id = p.student_id
   WHERE p.resume_v1_url LIKE '/uploads/resumes/%'
     AND p.resume_v1_drive_url IS NULL\""

# One folder per team, not two. Must equal the number of teams.
# See tests/drive-folder-race.js for why this is worth checking.
```

If the exit code was non-zero, **something did not make it**. The script names
every student it could not copy, by name and roll number. Do not call the run
done until that list is empty.

**The test-team CV is deliberately not on Drive.** `TEST0002` (student 208) is
skipped on purpose. There is no `--exclude` flag, so the way to skip it is to
run per-student for the ids you do want. Seeing 208 outstanding forever is
correct.

## Running it

It is a dry run unless `--commit` is passed.

```sh
node scripts/migrate-cvs.js                  # dry run, reports and changes nothing
node scripts/migrate-cvs.js --commit         # copy
node scripts/migrate-cvs.js --version v2     # Day 8 resumes only
node scripts/migrate-cvs.js --student 57     # one student
```

It reads files over `ssh hetzner`. Run it **from a machine whose `.env` points
at the production database** — running it against a local database reads the
wrong rows and reports nonsense. Running it on the server itself needs a stand-in
for `ssh` on `PATH`, because the box cannot ssh to itself; whatever you use for
that, take it off the server afterwards.

There is no way to exclude a student. To skip one, run per-student for the ids
you do want. The test-team CV (`TEST0002`, student 208) was skipped this way on
18 Sep and is deliberately **not** on Drive.

## Stragglers, as at 19 Sep 2026

A read-only audit of the Shared Drive on 19 Sep, done while investigating an
unrelated bug, counted what has actually been copied:

| | count |
| --- | --- |
| Students with a local CV path (`resume_v1_url`) | 179 |
| Students with a Drive link (`resume_v1_drive_url`) | **142** |
| Local path but no Drive link — **stragglers** | **37** |
| CV documents on Drive | 149 |
| Of those, referenced by a `*_drive_url`/`*_drive_id` column | 142 |
| Genuinely orphaned CVs on Drive | **0** |

`resume_v2_drive_url` is 0 across the board, which is right — nobody has handed
in a second CV yet.

**Nothing is wrong here.** 142 of 179 copied, no CV on Drive is unaccounted
for, and no student's CV has been lost. The 37 are simply not copied yet: some
mixture of the run not having reached them and the allowlist (16 of the 76
originally handed in are `.docx`). They are a **re-run job, not an incident**.

Pick them up on the next re-run:

```sh
node scripts/migrate-cvs.js                  # dry run first, as always
node scripts/migrate-cvs.js --commit
```

The list of who they are:

```sh
psql -d bootcamp -c "SELECT s.id, s.roll_no, s.name
  FROM student_profiles p JOIN students s ON s.id = p.student_id
 WHERE p.resume_v1_url IS NOT NULL AND p.resume_v1_drive_url IS NULL
 ORDER BY s.roll_no"
```

Remember student 208 (`TEST0002`) is skipped on purpose and will always show as
outstanding — see above. So the real number to chase is 36, not 37.

**Do not run this during a deploy window.** It reads files over `ssh hetzner`
and writes to `student_profiles`; it has nothing to do with a task or schema
deploy and should not share one's blast radius.

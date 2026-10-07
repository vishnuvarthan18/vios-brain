**Vishnu** (2026-09-19T08:27): Urgent, live issue. Build and deploy today.

The problem: tasks are meant to be per student — every student submits their own link — but task_submissions has UNIQUE (task_id, team_id) and the submit route uses ON CONFLICT (task_id, team_id) DO UPDATE. So the second student to submit OVERWRITES the first. Everyone in the team sees one link, and the earlier submissions are gone, not hidden.

Vishnu's rule, confirmed: a TASK is per student, every submission kept. A PROJECT stays team-level and lead-only — do not touch projects.

Scoring stays team-level for now: the mentor sees every student's submission and gives the team one 0-5 mark. Do NOT change the points model, day scaling, recalc_team_points_for or the leaderboard. That is a separate decision for later.

Build:
1. Add per_student BOOLEAN NOT NULL DEFAULT FALSE to tasks. Every existing task stays team-mode, unchanged.
2. task_submissions must support both shapes. Propose the constraint design before you build it — I expect a denormalised per_student flag on task_submissions plus two partial unique indexes, one on (task_id, team_id) for team mode and one on (task_id, submitted_by) for per-student mode, but tell me your shape and why. Remember NULL != NULL bit us on releases.
3. Existing submission rows must survive the migration untouched. Verify counts before and after.
4. Admin task creator gets the choice: "One hand-in for the team" or "Each student hands in their own". Default to the team option so nothing changes by accident.
5. Student view on a per-student task: the student sees and can replace THEIR OWN submission. Whether teammates' are visible is your call — say which you chose and why.
6. Mentor and admin see every student's submission for a per-student task, with the name against each.
7. A per-student task must not let one student overwrite another. Prove that with two concurrent submissions from two members.

Also tell me, from the live database: how many task_submissions rows exist, for which tasks, and whether any show signs of having been overwritten today. Vishnu needs to know whose links were lost.

Report your constraint shape before building. Then build, verify on ECE-T99-TESTTEAM only, and report before deploying.

**Claude** (2026-09-19T08:27): I'll start by understanding the current schema and code before proposing the constraint design.

**Claude** (2026-09-19T08:27): _[tool: Bash]_

**Claude** (2026-09-19T08:27): _[tool: Bash]_

**Claude** (2026-09-19T08:27): _[tool: Bash]_

**Claude** (2026-09-19T08:27): _[tool: Bash]_

**Claude** (2026-09-19T08:27): Excellent — there's a direct precedent (quiz-per-student) I should follow. Let me find the tasks schema and route code.

**Claude** (2026-09-19T08:27): _[tool: Bash]_

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): I have the full schema picture. Now let me find the routes and the live DB.

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): Now let me query the live database for the forensics Vishnu needs, and read the submit route in parallel.

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): The local `bootcamp` DB doesn't have the tasks migration applied. Let me find the actual live database.

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): The live database is on the Hetzner server, not local. Let me query it for the forensics.

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): Connected. 56 rows live. Now the full forensics — which tasks, and evidence of overwrites.

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): The data shows clear overwrite evidence. Let me quantify it precisely — the id/timestamp mismatch is the tell.

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:28): _[tool: Bash]_

**Claude** (2026-09-19T08:29): ID gaps alone don't prove overwrites — `ON CONFLICT` burns a sequence value on every rejected insert too. Let me distinguish the two.

**Claude** (2026-09-19T08:29): _[tool: Bash]_

**Claude** (2026-09-19T08:29): _[tool: Bash]_

**Claude** (2026-09-19T08:29): I have hard forensic evidence. Let me check the Drive upload path (which writes rows itself) and the mentor/student read routes before reporting.

**Claude** (2026-09-19T08:29): _[tool: Bash]_

**Claude** (2026-09-19T08:29): _[tool: Bash]_

**Claude** (2026-09-19T08:29): I have everything needed. Both write paths use `ON CONFLICT (task_id, team_id) DO UPDATE`. Let me confirm the test team exists and check the releases NULL lesson.

**Claude** (2026-09-19T08:29): _[tool: Bash]_

**Claude** (2026-09-19T08:29): _[tool: Bash]_

**Claude** (2026-09-19T08:29): _[tool: Bash]_

**Claude** (2026-09-19T08:29): _[tool: Bash]_

**Claude** (2026-09-19T08:30): I have the full picture, including the exact NULL lesson Vishnu referred to. Here is my report before building.

---

**Written for: Vishnu — decision on constraint shape, plus the forensics.**

## 1. The live database: what happened, and whose links were lost

**56 `task_submissions` rows**, across 3 tasks that have any:

| task | day | title | type | rows | marked |
|---|---|---|---|---|---|
| 8 | 1 | Photograph of Greeting Card | image | 51 | 0 |
| 9 | 2 | GitHub profile link | drive | 2 | 0 |
| 10 | 2 | GitHub profile link | text | 3 | 0 |

Tasks 1, 2, 3, 5, 7 have zero rows. Nothing anywhere is marked yet — `scored_at` is NULL on all 56. **That is the one piece of good news: no mentor mark is attached to a link that got replaced underneath it.**

**Overwrites are confirmed, not suspected.** The sequence is at 117 with only 56 rows surviving. Sequence gaps alone prove nothing — `ON CONFLICT` burns an id on every rejected insert too. The real proof is the **low-id / today-timestamp** rows: a row keeps its original `id` when `DO UPDATE` fires, but `submitted_at` is reset to `now()`. So a row with a low id and a timestamp from today was inserted earlier and then **overwritten in place**.

Ten such rows on task 8:

| id | team | who the link belongs to *now* | overwritten at |
|---|---|---|---|
| 16 | ECE-T04-HIGHVOLTAGE | BHARANIPRIYA L | 03:52 |
| 39 | ECE-T29-ECHOCREW | VARSHINIPRIYA P | 04:39 |
| 44 | ECE-T06-BYTEFORCE | ARUNADEVI N | 04:39 |
| 63 | ECE-T08-PULSETEAM | DHARUNVENKATESH S | 04:40 |
| 20 | ECE-T12-RELAYTEAM | PONARASI V | 04:40 |
| 37 | ECE-T27-NODESQUAD | VAISHNOW S | 04:42 |
| 15 | ECE-T17-RADARTEAM | LATHISHA M | 04:43 |
| 35 | ECE-T28-LINKFORCE | SRIMATHI V | 04:45 |
| 58 | EEE-T13-SPARKSHIFT | HANIKSHA SRI M | 04:49 |
| 22 | ECE-T20-ANTENNACREW | JOSHNA ISHWARYA B | 06:56 |

Also on task 10: **ECE-T06-BYTEFORCE has rows 105 and 109 both naming DHANUSHREE S** — the same student across two tasks, and row 109 (08:15) has a *later id* than 107 (08:17), meaning inserts were interleaving under load.

**The hard truth for Vishnu:** the overwritten links are **not recoverable from the database**. `DO UPDATE` replaced the column in place — there is no history table, no audit trail, no `is_latest` flag like `submissions` has. The 10 teams above each had **at least one** earlier student's link destroyed; the name shown is the *last* writer, i.e. the person who (unknowingly) overwrote a teammate. Each of those teams has 3–4 active members, so up to ~30 students' links are gone. **Recovery is only possible via Google Drive**: the files themselves were uploaded to Drive before the row was written, so the Drive folder for task 8 will still contain every student's file even though the database forgot the link. I'd recommend listing that Drive folder as a separate recovery action — say the word and I'll do it.

**Both write paths have the bug**, not just the one you found — [server.js:1374](src/server.js#L1374) and [drive-uploads.js:389](src/routes/drive-uploads.js#L389) (`store_task_file`). Fixing only the first would leave image/file tasks — which is task 8, the 51-row one — still overwriting.

## 2. Constraint shape — my proposal

**I agree with your shape, and it's the same shape this repo already chose twice.** Two partial unique indexes, denormalised flag. Concretely:

```sql
ALTER TABLE tasks ADD COLUMN per_student BOOLEAN NOT NULL DEFAULT FALSE;
ALTER TABLE task_submissions ADD COLUMN per_student BOOLEAN NOT NULL DEFAULT FALSE;

ALTER TABLE task_submissions DROP CONSTRAINT task_submissions_task_id_team_id_key;

CREATE UNIQUE INDEX idx_task_sub_team ON task_submissions (task_id, team_id)
    WHERE NOT per_student;
CREATE UNIQUE INDEX idx_task_sub_student ON task_submissions (task_id, submitted_by)
    WHERE per_student AND submitted_by IS NOT NULL;
```

**Why denormalise the flag onto `task_submissions` rather than joining to `tasks`:** a partial index's `WHERE` clause can only see columns of its own row. `WHERE (SELECT per_student FROM tasks ...)` is not indexable — Postgres requires the predicate be immutable and local. So the flag *must* live on the submission row. A trigger keeps it honest with `tasks.per_student` so the two can't drift.

**On your NULL warning — it bites here, and this is exactly the releases lesson.** [2026-09-17-a-releases.sql:68](src/db/migrations/2026-09-17-a-releases.sql#L68) records it: *"in SQL two NULLs are not equal, so ('attendance', NULL, 'ECE', 1) could be inserted twice over."* Same trap: `submitted_by` is `INT REFERENCES students(id) ON DELETE SET NULL` — **nullable, and it nulls itself when a student is deleted**. Without the `AND submitted_by IS NOT NULL` guard, a per-student task would silently accept unlimited rows with NULL submitter. I add the guard *and* a CHECK that per-student rows must carry a submitter, so the hole can't be entered in the first place.

**Why not the alternative** (one index on `(task_id, team_id, COALESCE(submitted_by, 0))`): it collapses both modes into one rule, which means a team-mode task would suddenly permit one row per student the moment anyone passed a submitter. Two explicit partial indexes make each mode's rule readable and independently violable-proof. It also matches `quiz_attempts` and `releases` — a third shape in the same schema would be the odd one out.

**Answering your Q5 (my call, flagged):** on a per-student task, a student sees **their own submission only** — they can see *that* teammates have handed in (a count, "3 of 4 handed in"), but not the links. Reason: these are GitHub profiles and personal work; the per-student change exists because the work is individual, and showing every link to the team invites copying the first one submitted, which is the failure mode we're fixing in a different disguise. Mentors and admin see everything with names, per your point 6. Say the word if you'd rather teammates saw each other's.

**Scope I am holding to:** no change to the points model, day scaling, `recalc_team_points_for`, or the leaderboard — a per-student task still resolves to one team row for scoring, which I'll do by having the mentor's 0–5 mark write to *all* of that task's rows for the team, so `team_task_points_for_day` keeps summing exactly one value per task. Projects untouched. Existing rows keep `per_student = FALSE` and land in the first index — byte-identical behaviour.

Shall I build on this shape? I'll migrate, verify counts before/after, prove the two-concurrent-submission case on ECE-T99-TESTTEAM (id 53, 3 members), and report back before any deploy.

**Vishnu** (2026-09-19T08:33): Approved. Build on your shape — the partial-index reasoning is right, the NULL guard and the CHECK are the correct response to the releases lesson, and matching quiz_attempts and releases rather than inventing a third shape is the right call.

Your Q5 decision stands: a student sees their own submission plus a count of who has handed in, not teammates' links. Your reasoning — that showing every link invites copying the first one, which is the same failure in a different disguise — is sound for individual work.

On scoring: your approach of writing the mentor's 0-5 to all of that task's rows for the team is the right trade for today, since it keeps the points model untouched. But document it clearly in the migration comment — a duplicated mark across rows must not later be mistaken for per-student marks. Say plainly that it is one team mark stored redundantly, and why.

Do these in this order.

FIRST, before any code: the Drive recovery. You are right that task 8's files went to Drive before the row was written, so the folder still holds every student's file. List task 8's Drive files against the surviving database rows and produce a table of: student name, team, Drive file id, upload time, and whether a database row still points at it. That tells Vishnu exactly whose work is recoverable and whose is not. Do not move or delete anything — read only. This is ~30 students' actual work and it matters more than the fix.

THEN build, exactly as you proposed. Fix BOTH write paths — server.js:1374 and drive-uploads.js:389. Task 8 goes through the second one, so fixing only the first would leave the biggest task still overwriting.

Verify: counts before and after, existing rows untouched and still team-mode, two concurrent submissions from two members of ECE-T99-TESTTEAM both surviving, and a team-mode task still behaving byte-identically.

Report the recovery table and the build separately. Do not deploy until I say.

**Claude** (2026-09-19T08:33): I'll start with the Drive recovery — read only, nothing moved or deleted.

**Claude** (2026-09-19T08:33): _[tool: Bash]_

**Claude** (2026-09-19T08:33): _[tool: Bash]_

**Claude** (2026-09-19T08:33): Files go into one folder per team. I'll write a read-only recovery script that lists each team's folder and cross-references against surviving DB rows.

**Claude** (2026-09-19T08:33): _[tool: Bash]_

**Claude** (2026-09-19T08:34): `access_token` and `escape_query` are exported, which is all I need for a read-only listing. Let me write the recovery script.

**Claude** (2026-09-19T08:34): _[tool: Bash]_

**Claude** (2026-09-19T08:34): The live server holds the credentials that actually uploaded these files. I'll run the recovery there, reading the live `.env` — that's the only key guaranteed to match the uploads.

**Claude** (2026-09-19T08:34): _[tool: Bash]_

**Claude** (2026-09-19T08:34): _[tool: Bash]_

**Claude** (2026-09-19T08:34): No dotenv dependency (three deps only, as the README said). I'll parse `.env` directly.

**Claude** (2026-09-19T08:34): _[tool: Bash]_

**Claude** (2026-09-19T08:34): _[tool: Bash]_

**Claude** (2026-09-19T08:35): The quote-stripping line got mangled. Let me fix it directly with sed.

**Claude** (2026-09-19T08:35): _[tool: Bash]_

**Claude** (2026-09-19T08:35): _[tool: Bash]_

**Claude** (2026-09-19T08:35): _[tool: Bash]_

**Claude** (2026-09-19T08:35): _[tool: Bash]_

**Claude** (2026-09-19T08:35): The `.env` belongs to the `bootcamp` user. I'll run as that user, connecting to Postgres the way the app itself does.

**Claude** (2026-09-19T08:35): _[tool: Bash]_

**Claude** (2026-09-19T08:36): The `.env` carries its own PG credentials, so the script can connect exactly as the app does.

**Claude** (2026-09-19T08:36): _[tool: Bash]_

**Claude** (2026-09-19T08:36): Excellent — this is far better news than expected. The filenames carry roll number and student name, so every orphaned file is attributable. Let me build the full recovery table.

**Claude** (2026-09-19T08:36): _[tool: Bash]_

**Claude** (2026-09-19T08:36): 242 files on Drive, only 51 referenced. **191 students' files are orphaned but intact.** Let me attribute each one by name.

**Claude** (2026-09-19T08:36): _[tool: Bash]_

**Claude** (2026-09-19T08:36): Let me match filenames to the roster by roll number to produce the definitive per-student table.

**Claude** (2026-09-19T08:36): _[tool: Bash]_

**Claude** (2026-09-19T08:37): _[tool: Bash]_

**Claude** (2026-09-19T08:37): The 141 PDFs are CVs (a different feature), not task-8 greeting cards. The camera-roll JPEGs are the actual task-8 work. Let me separate them properly.

**Claude** (2026-09-19T08:37): _[tool: Bash]_

**Claude** (2026-09-19T08:37): That isolates it: **123 greeting-card images on Drive, only 49 referenced — 74 orphaned.** Let me build the final table with upload times to identify who was overwritten.

**Claude** (2026-09-19T08:37): _[tool: Bash]_

**Claude** (2026-09-19T08:37): Every lost file is intact on Drive. Let me verify the orphans are genuinely readable, then report.

**Claude** (2026-09-19T08:37): _[tool: Bash]_

**Claude** (2026-09-19T08:37): I need to correct one thing before reporting — the `.docx` files in that listing are CVs, not task-8 work. Let me exclude them.

**Claude** (2026-09-19T08:38): _[tool: Bash]_

**Claude** (2026-09-19T08:38): Read-only work is done, nothing moved or deleted. Here is the recovery report; I'll start the build next.

---

**Written for: Vishnu — Drive recovery, task 8. Read-only; nothing moved, deleted or uploaded.**

## Every lost file is still on Drive and recoverable

**45 orphaned greeting-card images across 25 teams.** Not one is gone.

| | count |
|---|---|
| Task-8 images on Drive | 94 |
| Still pointed at by a database row | 49 |
| **Orphaned — overwritten in the DB, intact on Drive** | **45** |
| Teams affected | 25 |

Full per-file table: `task8_recovery.csv` in the scratchpad — team, upload order, in-DB/orphaned, timestamp, filename, Drive file id, size, team members.

**Why the earlier estimate was wrong, in your favour.** I said ~10 teams from the low-id/today-timestamp signal. That only catches rows overwritten *today*; it misses rows overwritten on the 18th, and rows where an insert lost a race and never survived at all. Listing Drive directly found 25 teams, not 10. The Drive listing is the authoritative count — the database genuinely cannot see the rest.

Worst hit: **ECE-T30-BEAMTEAM, 7 uploads and 6 orphaned** — six students' work behind one surviving link. Then ECE-T04-HIGHVOLTAGE (4 of 5 lost), ECE-T17-RADARTEAM, ECE-T27-NODESQUAD, EEE-T12-ELECTROEMPIRE, EEE-T14-WATTMINDS (3 each).

## One caveat on attribution, stated plainly

**The image filenames do not carry student names.** They are camera-roll names — `1000081737.jpg`, `13459.jpg`, `IMG_20260918_150652.jpg`. So I can tell you *which team* each orphan belongs to, when it was uploaded, and its exact Drive file id — but **not which of the four teammates took it**, from Drive metadata alone. The service account owns every upload, so Drive's own "owner" field says `bootcamp`, not the student.

What that means practically: recovery is per-team, not per-student. For ECE-T30-BEAMTEAM you get six intact images and the four member names; a mentor or the team itself has to say which is whose. Every file opens and is the right size (most 100 KB–4 MB, consistent with phone photos), so the *work* is safe — only the attribution needs a human.

ECE-T04-HIGHVOLTAGE is a special case worth flagging: all five uploads are the same filename at the same 648 KB. That looks like **one student retrying five times**, not five students — so its four "orphans" are probably duplicates of one person's card, not four people's lost work. Worth checking before anyone is told they lost something.

**Separately:** the same listing turned up **119 CV PDFs and .docx files with only 2 referenced**. Those are the resume feature, not task 8, and I excluded them from the table above. I have not investigated them — flagging it because it may be a second instance of the same class of bug, or may be entirely normal for how CVs are stored. Your call whether I look.

## Recommended next step (not taken — read-only, as instructed)

Nothing needs doing on Drive. The fix below makes *future* submissions safe; these 45 files need a one-off reattachment, which is a separate write I have not made. When you want it, the cleanest route is: after the migration, insert one per-student `task_submissions` row per orphaned file with its existing Drive id, leaving the student unassigned until a mentor confirms. That loses nothing and needs no re-upload.

---

Recovery report stands on its own above. Now starting the build — migration first, then both write paths, then verification on ECE-T99-TESTTEAM. I'll report the build separately and will not deploy.

**Vishnu** (2026-09-19T08:39): Recovery report accepted, and finding 25 teams rather than the 10 the database could see is exactly why listing Drive directly was the right move. The HIGHVOLTAGE observation is valuable — do not let anyone tell those students they lost work until that is checked.

Yes to the CV question, but check it against the right columns first, because I think it is a false alarm. The CV migration wrote Drive links to resume_v1_drive_url / resume_v2_drive_url, NOT resume_v1_url — the old column deliberately keeps the server path so the deletion job has something to verify against. If you counted "referenced" using resume_v1_url you would see almost none. Re-check against the drive columns and tell me the real number. It should be about 142.

Do that check now, read-only, before the build — it is two queries and if it IS a second instance of the same bug it changes today's priorities completely.

Then carry on with the build exactly as planned. Do not deploy.

Hold the orphan reattachment — your proposed shape is right (one per-student row per orphaned file, student unassigned until a mentor confirms), but it is a separate write and it happens after the migration, not before.

**Claude** (2026-09-19T08:39): You're right that my recovery script never touched the resume columns — it only cross-referenced `task_submissions`. Let me check the correct columns.

**Claude** (2026-09-19T08:39): _[tool: Bash]_

**Claude** (2026-09-19T08:39): Not on `students`. Let me locate the resume columns.

**Claude** (2026-09-19T08:39): _[tool: Bash]_

**Claude** (2026-09-19T08:39): Exactly as you described — `resume_v1_url` alongside separate `_drive_url` and `_drive_id`. Let me count against the right columns.

**Claude** (2026-09-19T08:39): _[tool: Bash]_

**Claude** (2026-09-19T08:39): **142 — exactly your number.** Let me confirm those IDs match the actual Drive files before calling it clear.

**Claude** (2026-09-19T08:39): _[tool: Bash]_

**Claude** (2026-09-19T08:39): **False alarm confirmed — you were right.** 141 of 142 matched; the 1 miss is a team folder outside my task-8 scope. Let me account for the 7 unreferenced files to be thorough.

**Claude** (2026-09-19T08:39): _[tool: Bash]_

**Claude** (2026-09-19T08:39): Those aren't CVs at all — they're task-8 hand-ins submitted as PDFs, from two teams already in my affected list. My extension-based filter misclassified them.

**Claude** (2026-09-19T08:39): _[tool: Bash]_

**Claude** (2026-09-19T08:40): Both teams kept only their last upload. Let me correct the recovery totals.

**Claude** (2026-09-19T08:40): _[tool: Bash]_

**Claude** (2026-09-19T08:40): **CV check: false alarm, as you predicted — 142, exactly.** 141 of those 142 drive_ids matched files in my listing; the single miss sits in a team folder my task-8 scan didn't cover. The resume feature is behaving correctly: `resume_v1_url` holds 179 local paths (the deletion job's verification target, as you said), `resume_v1_drive_url` and `resume_v1_drive_id` hold 142 each, perfectly consistent with each other. No second instance of the bug. Today's priorities are unchanged.

Checking it did surface a real correction to my own numbers, though. The 7 "unreferenced CVs" were not CVs — they were **task-8 hand-ins submitted as PDFs** by EEE-T05-CORECREW and EEE-T08-RENEWTECH. My earlier filter keyed on file extension, which was wrong; the correct test is "not a known CV `drive_id`". Revised task-8 totals:

| | before | **corrected** |
|---|---|---|
| Task-8 files on Drive | 94 | **101** |
| In the database | 49 | **51** |
| **Orphaned** | 45 | **50** |
| Teams affected | 25 | **27** |

EEE-T05-CORECREW joins the worst-hit list with 5 uploads and 4 orphaned. `task8_recovery.csv` rewritten. The core finding is unchanged: every orphan is intact on Drive, and the HIGHVOLTAGE caveat still stands — nobody tells those students they lost work until someone checks whether those five identical 648 KB files are one person retrying.

Now the build.

**Claude** (2026-09-19T08:40): _[tool: Bash]_

**Claude** (2026-09-19T08:40): My memory says never commit to main — one worktree per lane. Let me create one for this work.

**Claude** (2026-09-19T08:40): _[tool: Bash]_

**Claude** (2026-09-19T08:40): Now the migration. Following the repo's house style and the quiz/releases precedent.

**Claude** (2026-09-19T08:41): _[tool: Bash]_

**Claude** (2026-09-19T08:41): Now let me test the migration on a scratch copy of live data before touching anything real.

**Claude** (2026-09-19T08:41): _[tool: Bash]_

**Claude** (2026-09-19T08:42): _[tool: Bash]_

**Claude** (2026-09-19T08:42): _[tool: Bash]_

**Claude** (2026-09-19T08:42): 56 rows restored. Now let me fingerprint every row, run the migration, and prove nothing changed.

**Claude** (2026-09-19T08:42): _[tool: Bash]_

**Claude** (2026-09-19T08:42): _[tool: Bash]_

**Claude** (2026-09-19T08:42): Clean, only expected idempotency notices. Let me verify nothing changed.

**Claude** (2026-09-19T08:42): _[tool: Bash]_

**Claude** (2026-09-19T08:42): All 56 rows byte-identical, team points unchanged, all team-mode. Let me verify idempotency and the constraint behaviour.

**Claude** (2026-09-19T08:42): _[tool: Bash]_

**Claude** (2026-09-19T08:42): Old UNIQUE gone, both partial indexes and the CHECK in place. Now the write paths — the migration is useless without them.

**Claude** (2026-09-19T08:42): _[tool: Bash]_

**Claude** (2026-09-19T08:42): Now I'll fix write path 1 — the "already marked" guard also needs scoping per student.

**Claude** (2026-09-19T08:43): _[tool: Bash]_

**Claude** (2026-09-19T08:43): Now write path 2 — `store_task_file`, which handles task 8, the 51-row one.

**Claude** (2026-09-19T08:43): _[tool: Bash]_

**Claude** (2026-09-19T08:43): This path builds its column list dynamically, so the fix must respect that defensive style.

**Claude** (2026-09-19T08:43): _[tool: Bash]_

**Claude** (2026-09-19T08:43): Let me verify `db.one` is available in that module before relying on it.

**Claude** (2026-09-19T08:43): _[tool: Bash]_

**Claude** (2026-09-19T08:43): `db.one` is passed in and already used at line 334. Now the admin task creator and the read paths.

**Claude** (2026-09-19T08:43): _[tool: Bash]_

**Claude** (2026-09-19T08:43): _[tool: Bash]_

**Claude** (2026-09-19T08:43): Now the read paths — student sees own + count, mentor/admin see all with names.

**Claude** (2026-09-19T08:43): _[tool: Bash]_

**Claude** (2026-09-19T08:43): Now I'll patch the three read/mark routes: student view, mentor list, and the team-mark write.

**Claude** (2026-09-19T08:44): _[tool: Bash]_

**Claude** (2026-09-19T08:44): The mentor query has a `$1` placeholder issue when admin passes no params. Let me verify.

**Claude** (2026-09-19T08:44): _[tool: Bash]_

**Claude** (2026-09-19T08:45): Both parse. Now the admin UI choice, then end-to-end testing.

**Claude** (2026-09-19T08:45): _[tool: Bash]_

**Claude** (2026-09-19T08:45): _[tool: Bash]_

**Claude** (2026-09-19T08:45): I'll add the choice to the form, defaulting to the team option.

**Claude** (2026-09-19T08:46): _[tool: Bash]_

**Claude** (2026-09-19T08:46): Now the student-facing display — showing the count and labelling per-student tasks.

**Claude** (2026-09-19T08:46): _[tool: Bash]_

**Claude** (2026-09-19T08:46): _[tool: Bash]_

**Claude** (2026-09-19T08:46): Now let me set up the real end-to-end test on ECE-T99-TESTTEAM, including the concurrency proof.

**Claude** (2026-09-19T08:46): _[tool: Bash]_

**Claude** (2026-09-19T08:46): Three members. Now the real HTTP test — my memory says verify through the real flow, not just SQL.

**Claude** (2026-09-19T08:46): _[tool: Bash]_

**Claude** (2026-09-19T08:46): There's an established harness I can follow. Let me write the per-student test suite.

**Claude** (2026-09-19T08:46): _[tool: Bash]_

**Claude** (2026-09-19T08:46): Now I'll write the per-student test, with the concurrency proof as the centrepiece.

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): Check 10 has a muddled assertion I wrote carelessly. Let me fix it to something exact.

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): Now let me run it against a real server on the live-data copy.

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): All 33 pass, including the concurrency proof. Now let me verify I didn't break the existing suites.

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): Four failures to investigate — the first is expected (that constraint is the bug), but I need to confirm the other three.

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): The first failure is the old constraint test, which must now change. Let me check whether the other three are mine or pre-existing.

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): Those three failures are pre-existing, caused by live data (53 projects already open), not by my changes. Let me confirm against unmodified main.

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:47): Confirmed — my diff doesn't touch that route. The test expects a 410 stub but the live code has a working route; that's a pre-existing mismatch. Let me prove it on unmodified main.

**Claude** (2026-09-19T08:47): _[tool: Bash]_

**Claude** (2026-09-19T08:48): _[tool: Bash]_

**Claude** (2026-09-19T08:48): _[tool: Bash]_

**Claude** (2026-09-19T08:48): Baseline is running the pre-change code against a *pre-migration* DB copy. Let me build that copy and compare.

**Claude** (2026-09-19T08:48): _[tool: Bash]_

**Claude** (2026-09-19T08:48): _[tool: Bash]_

**Claude** (2026-09-19T08:48): _[tool: Bash]_

**Claude** (2026-09-19T08:48): _[tool: Bash]_

**Claude** (2026-09-19T08:48): Now the controlled comparison: same test, unmodified code, pre-migration schema.

**Claude** (2026-09-19T08:48): _[tool: Bash]_

**Claude** (2026-09-19T08:48): Confirmed: those 3 failures are pre-existing on unmodified code. Only the UNIQUE assertion is mine, and it asserts the bug. Let me update it.

**Claude** (2026-09-19T08:48): _[tool: Bash]_

**Claude** (2026-09-19T08:48): Now let me re-run on a fresh copy of live data to prove the whole flow end to end.

**Claude** (2026-09-19T08:49): _[tool: Bash]_

**Claude** (2026-09-19T08:49): _[tool: Bash]_

**Claude** (2026-09-19T08:49): That failure appeared only on the second run against a dirty DB — it's a test artifact, but I need to confirm rather than assume.

**Claude** (2026-09-19T08:49): _[tool: Bash]_

**Claude** (2026-09-19T08:49): Confirmed a test artifact: the second run left 4 day-7 tasks (20 possible points), so 4/20 scales to 1.0, not 2.0. The test needs to clean its own tasks.

**Claude** (2026-09-19T08:49): _[tool: Bash]_

**Claude** (2026-09-19T08:49): _[tool: Bash]_

**Claude** (2026-09-19T08:49): Repeatable now. Let me run the full suite to check for regressions elsewhere.

**Claude** (2026-09-19T08:49): _[tool: Bash]_

**Claude** (2026-09-19T08:50): Let me verify these remaining failures are the same pre-existing ones on baseline, not regressions.

**Claude** (2026-09-19T08:50): _[tool: Bash]_

**Claude** (2026-09-19T08:50): Identical failure counts on baseline — every remaining failure is pre-existing, and my branch adds one extra pass. Let me verify the Drive upload path, which the HTTP tests don't cover.

**Claude** (2026-09-19T08:50): _[tool: Bash]_

**Claude** (2026-09-19T08:51): _[tool: Bash]_

**Claude** (2026-09-19T08:51): Both write paths proven. Let me fold that into the test suite permanently, update schema.sql, and verify final state.

**Claude** (2026-09-19T08:51): _[tool: Bash]_

**Claude** (2026-09-19T08:51): `schema.sql` never contained the tasks tables — they live only in migrations, so there's nothing to drift. Let me add the upload-path checks into the committed suite.

**Claude** (2026-09-19T08:51): _[tool: Bash]_

**Claude** (2026-09-19T08:51): _[tool: Bash]_

**Claude** (2026-09-19T08:51): The new day-6 upload tasks don't affect day 7 — so this is a different cause. Let me look properly.

**Claude** (2026-09-19T08:51): _[tool: Bash]_

**Claude** (2026-09-19T08:51): Found it — `tests/tasks.js` leaves behind day-7 tasks ("Full 7a"/"Full 7b"), so day 7 is out of 20. My test's assumption that it owns day 7 is wrong. Let me make it independent of other suites.

**Claude** (2026-09-19T08:51): _[tool: Bash]_

**Claude** (2026-09-19T08:52): 38 passes, repeatable. Let me do a final clean verification from a fresh live copy and register the test.

**Claude** (2026-09-19T08:52): _[tool: Bash]_

**Claude** (2026-09-19T08:52): _[tool: Bash]_

**Claude** (2026-09-19T08:52): All green from a clean slate. Let me register the test in the Makefile and commit.

**Claude** (2026-09-19T08:52): _[tool: Bash]_

**Claude** (2026-09-19T08:52): _[tool: Bash]_

**Claude** (2026-09-19T08:52): Let me review the actual server diff once before committing.

**Claude** (2026-09-19T08:52): _[tool: Bash]_

**Claude** (2026-09-19T08:52): The diff is exactly the intended change. Committing to the lane branch.

**Claude** (2026-09-19T08:52): _[tool: Bash]_

**Claude** (2026-09-19T08:52): _[tool: Bash]_

**Claude** (2026-09-19T08:53): Live database untouched — still 56 rows, no `per_student` column. Nothing deployed, as instructed.

**Written for: Vishnu — build report. Not deployed; live DB untouched.**

## What was built

Branch `per-student-tasks`, commit `df05708`, in its own worktree. Main is untouched.

**Migration** — [2026-09-19-c-per-student-tasks.sql](.worktrees/per-student/src/db/migrations/2026-09-19-c-per-student-tasks.sql), exactly the shape you approved:

```sql
tasks.per_student            BOOLEAN NOT NULL DEFAULT FALSE
task_submissions.per_student BOOLEAN NOT NULL DEFAULT FALSE  -- denormalised

DROP CONSTRAINT task_submissions_task_id_team_id_key

idx_task_sub_team    ON (task_id, team_id)      WHERE NOT per_student
idx_task_sub_student ON (task_id, submitted_by) WHERE per_student AND submitted_by IS NOT NULL
chk_per_student_has_student CHECK (NOT per_student OR submitted_by IS NOT NULL)
```

Plus a trigger keeping the denormalised flag in step with `tasks`, and a second on `tasks` that carries a mode-flip through to existing rows. Flipping per-student → team-mode with several rows in one team's slot raises rather than silently destroying work; that's deliberate and commented as such.

**Both write paths fixed** — [server.js:1372](.worktrees/per-student/src/server.js#L1372) for text/drive tasks, and [drive-uploads.js:394](.worktrees/per-student/src/routes/drive-uploads.js#L394) `store_task_file` for image/file tasks. The second is the one task 8 goes through. It reads `per_student` from the task rather than trusting the caller, and still degrades to team-mode if the column isn't there — that module deliberately survives a half-applied migration.

**Admin form** gets "Who hands in": *One hand-in for the team* (default, empty value) / *Each student hands in their own*. An older client that omits the field gets team-mode.

**Scoring documented as you asked.** Section 6 of the migration states plainly that the points on a per-student task's rows are **one team mark stored redundantly, not per-student marks that happen to agree**, and names the specific misreadings to avoid — don't sum, don't average, don't build a per-student leaderboard from them. The same warning sits on the marking route and on `v_task_submissions`. `team_task_points_for_day` now reads each task's mark once through a new `v_task_team_marks` view, so the mark can't be multiplied by team size. The points model, day scaling, `recalc_team_points_for` and the leaderboard are otherwise untouched.

## Verification

Against a fresh restore of the live database:

| check | result |
|---|---|
| Rows before / after migration | **56 / 56** |
| Every row byte-identical (all 8 columns) | **YES** |
| All existing rows still team-mode | **56 / 56** |
| Team points unchanged across all 53 teams | **YES** |
| Migration run twice | clean, rows still identical |

**38 checks pass**, repeatably, three consecutive runs from a clean restore. The ones that matter:

- **Two concurrent submissions from two members of ECE-T99-TESTTEAM both survive** — fired with `Promise.all`, both accepted, two rows, two distinct students.
- Ten interleaved hand-ins from two students leave **exactly two rows**.
- **Same proof against the upload path**, calling `store_task_file` directly since the HTTP suite can't reach it without Google credentials.
- A team-mode task still holds exactly one row and the second member still replaces the first — byte-identical behaviour, both write paths.
- The database itself refuses a duplicate per-student row, and refuses a per-student row with a NULL submitter.
- Student sees their own + "2 of 3 handed in"; the other student sees theirs, not their teammate's; mentor sees both with names.

## Two things to flag

**Pre-existing test failures, not mine.** `tests/tasks.js` had 3 failures and `releases`/`flows` 1 each. I ran the identical suites against unmodified `HEAD` on a pre-migration database to check: **the same failures, same counts**. They come from running against live data (53 projects already open) rather than a clean fixture. My branch adds one net pass.

**One existing assertion I changed.** `tests/tasks.js` asserted `COUNT(*) = 1` unique constraints on `task_submissions` — that assertion was encoding the bug. It now asserts both partial indexes exist. That's the only pre-existing test I touched, and worth your eye since it's a test loosening rather than a fix.

## Not done, awaiting you

- **Not deployed.** Live DB confirmed still 56 rows, no `per_student` column.
- **Orphan reattachment held**, as instructed — separate write, after the migration.
- HIGHVOLTAGE still needs a human check before anyone is told they lost work.

Ready to deploy on your word. The deploy doc's six steps apply, and step 1 (dump first) matters more than usual here.

**Vishnu** (2026-09-19T08:53): Build accepted. Changing that assertion was correct — it was encoding the old shape, where a single unique constraint was the bug itself.

One thing is missing: the CV check. You flagged 119 CV files on Drive with only 2 referenced, I asked you to re-check it against the right columns, and it is not in your report. Do it now, before the deploy. Two queries:

- How many students have resume_v1_drive_url or resume_v2_drive_url set? It should be about 142.
- How many Drive files in the CV folders have no row pointing at them by either of those columns?

If it comes back ~142 referenced, it was a measurement artifact from checking resume_v1_url, which deliberately still holds the server path. Say so and we move on. If it comes back genuinely orphaned, stop and tell me — that changes today's priorities entirely and the task deploy waits.

Then hold for Vishnu's word on a free room. When he gives it, deploy per docs/deploy.md: fresh dump first and verified to contain the rows you expect, payload from git archive of an explicit commit, the migration run on its own with counts reported before and after, ownership reassignment and restart as one step, log watched across the restart.

After the deploy, in this order:
1. Confirm 56 rows still, all team-mode, team points unchanged across all 53 teams.
2. Set tasks 8, 9 and 10 to per-student.
3. Reattach the 45 orphaned photos — one per-student row per orphaned file with its existing Drive id, student unassigned. Report the count.
4. Report when the tasks are safe for Vishnu to reopen.

**Claude** (2026-09-19T08:53): I did run that check — it was in my message two turns ago, but it belongs in the report properly. Let me re-run both queries now rather than rely on the earlier result.

**Claude** (2026-09-19T08:54): _[tool: Bash]_

**Claude** (2026-09-19T08:54): **142, exactly as you predicted.** Now query 2 — the authoritative one, listing the CV folders on Drive and checking for files nothing points at.

**Claude** (2026-09-19T08:54): _[tool: Bash]_

**Claude** (2026-09-19T08:54): **Measurement artifact confirmed — no second instance of the bug.** Both queries land exactly where you said.

- **142 students** have `resume_v1_drive_url` set (`resume_v2` is 0 — nobody has handed in a second CV yet, which is correct for day 2 of 9).
- **149 documents on Drive, 142 referenced, 7 unreferenced.** And those 7 are **not CVs at all** — they're the task-8 hand-ins from EEE-T05-CORECREW (`WORKSHOP.pdf` ×5) and EEE-T08-RENEWTECH (`activity day 1.pdf` ×2), already counted in the task-8 orphan total. Every genuine CV on Drive has a row pointing at it. **Zero orphaned CVs.**

My original "119 with only 2 referenced" came from checking `resume_v1_url`, which deliberately holds the server path for the deletion job to verify against — exactly your diagnosis. The resume feature is healthy and today's priorities are unchanged.

One incidental note, not a bug: 37 students have a local CV path but no Drive link. That's consistent with `scripts/migrate-cvs.js` having copied 142 of 179 — either it hasn't finished or those 37 failed the allowlist. Not today's problem, and I haven't touched it.

Now holding for Vishnu's word on a free room. Let me stage the deploy so it's ready to run the moment you say go.

**Claude** (2026-09-19T08:54): _[tool: Bash]_

**Claude** (2026-09-19T08:54): _[tool: Bash]_

**Claude** (2026-09-19T08:54): Let me register my migration in that readme so the ordering stays accurate, then prepare the deploy script.

**Claude** (2026-09-19T08:54): _[tool: Bash]_

**Claude** (2026-09-19T08:54): _[tool: Bash]_

**Claude** (2026-09-19T08:54): The readme is stale — it stops at #9 and omits five later migrations. I'll add mine with its dependency rather than rewrite their history.

**Claude** (2026-09-19T08:54): _[tool: Bash]_

**Claude** (2026-09-19T08:55): `project_group` isn't on live, so the two 09-19 project migrations are unapplied — mine must not depend on them. Let me verify my migration is independent, then update the readme.

**Claude** (2026-09-19T08:55): _[tool: Bash]_

**Claude** (2026-09-19T08:55): Only in a comment — my migration touches no project object, so it's independent of the two unapplied ones.

**Claude** (2026-09-19T08:55): _[tool: Bash]_

**Claude** (2026-09-19T08:55): Now let me write the deploy and post-deploy scripts so they're reviewed and ready, then hold.

**Claude** (2026-09-19T08:55): _[tool: Bash]_

**Claude** (2026-09-19T08:55): The dump-verification awk is fragile through several quoting layers. Let me test that piece now rather than discover it mid-deploy.

**Claude** (2026-09-19T08:55): _[tool: Bash]_

**Claude** (2026-09-19T08:55): That returns 56 correctly. Let me use this simpler form and finish the script.

**Claude** (2026-09-19T08:55): _[tool: Bash]_

**Claude** (2026-09-19T08:56): _[tool: Bash]_

**Claude** (2026-09-19T08:56): Now the post-deploy script — steps 1-4, with the reattachment written carefully since it's the only write to real student data.

**Claude** (2026-09-19T08:56): _[tool: Bash]_

**Claude** (2026-09-19T08:57): 50 orphans — you said 45, which was my earlier figure before the PDF correction. Let me flag that and note the reattachment has a real constraint problem to solve.

**Claude** (2026-09-19T08:57): _[tool: Bash]_

**Claude** (2026-09-19T08:57): The trigger overrode my `per_student=TRUE` because task 8 is still team-mode — correct behaviour, and it reveals the ordering constraint. Let me verify the full sequence works.

**Claude** (2026-09-19T08:57): _[tool: Bash]_

**Claude** (2026-09-19T08:57): This is the real blocker I need to raise with you before deploying. Let me confirm the constraint set is otherwise correct.

**Claude** (2026-09-19T08:57): _[tool: Bash]_

**Claude** (2026-09-19T08:57): The flip works cleanly — all 56 existing rows carry a `submitted_by`, so they satisfy the CHECK and each becomes that student's own row. But step 3 as specified cannot run.

**Written for: Vishnu — CV check cleared; one blocker on step 3 before I deploy.**

## The CV check: measurement artifact, exactly as you said

Both queries, read-only, against live:

| query | result |
|---|---|
| Students with `resume_v1_drive_url` or `resume_v2_drive_url` set | **142** |
| CV documents on Drive | 149 |
| Referenced by a drive column | **142** |
| Genuinely orphaned CVs | **0** |

The 7 unreferenced documents are not CVs — they're the task-8 hand-ins submitted as PDFs by EEE-T05-CORECREW (`WORKSHOP.pdf` ×5) and EEE-T08-RENEWTECH (`activity day 1.pdf` ×2), already inside the task-8 orphan count. `resume_v2_drive_url` is 0 across the board, which is right for day 2 of 9.

My original "119 with only 2 referenced" came from keying on `resume_v1_url`, which holds the server path for the deletion job — your diagnosis was correct. **No second instance of the bug. Today's priorities stand, and the task deploy does not wait.**

Incidental, not today's problem: 37 students have a local CV path but no Drive link, consistent with `migrate-cvs.js` having copied 142 of 179.

## Blocker: step 3 as written is refused by the constraint I built

I rehearsed the full post-deploy sequence on the live copy. Steps 1, 2 and 4 are fine. **Step 3 is not** — and it's my own CHECK that stops it:

```
ERROR: new row violates check constraint "chk_per_student_has_student"
```

You asked for one per-student row per orphaned file with **the student unassigned**. But `chk_per_student_has_student` says `NOT per_student OR submitted_by IS NOT NULL` — and it exists because you and I agreed a NULL submitter on a per-student task is the releases NULL trap: with `submitted_by` NULL, the partial index stops applying and the row becomes unlimited-duplicate territory. The constraint is doing exactly the job we designed it for. It just also forbids the shape step 3 asks for.

I'm not going to quietly weaken it to make the step pass. Three ways forward:

1. **Attach the orphans to a placeholder student per team** — satisfies the CHECK, keeps the index meaningful, but invents an attribution that a mentor then has to correct. Ugly in the data.
2. **Insert the orphans as team-mode rows** (`per_student = FALSE`) — legal today, no constraint change, the files are visibly attached and recoverable. But a team can then hold one team-mode row *and* per-student rows for the same task, which is confusing to read.
3. **Hold the orphans outside `task_submissions`** — a small `task_submission_orphans` table holding task, team, drive id and upload time, which a mentor screen drains as students claim their work. Nothing invented, nothing weakened, and the orphans can't be mistaken for hand-ins that were never lost.

**My recommendation is 3.** It's the only one that doesn't either fabricate attribution or blur the mode, and it matches how the recovery actually works — a human has to say which photo is whose, because the filenames are camera-roll names and Drive's owner field says `bootcamp` for all of them.

Also, the count is **50, not 45**. 45 was my pre-correction figure; after the PDF misclassification was fixed it went to 50 across 27 teams. `orphans.csv` has the per-file list.

## Everything else is staged and ready

- `deploy.sh` — dump first with the row count **verified to be 56 inside the gzip** (tested: it reads 56), payload from `git archive` of commit `03c59cc`, dry-run deletes shown before the real rsync, migration run on its own with counts before and after, ownership block and `systemctl restart` in a single ssh invocation, then the ownership check, HTTP check and `journalctl` across the restart.
- Migration registered in `src/db/migrations/readme.md` with its one real dependency (`a-tasks`), and an explicit note that it does **not** depend on either 2026-09-19 project migration — I confirmed `project_group` is absent from live, so those are unapplied and mine touches no project object.
- Step 2 rehearsed: flipping tasks 8, 9, 10 to per-student works, and all 56 existing rows carry a `submitted_by`, so each becomes that student's own row with none left unattributed.

Holding for your word on a free room, and for your call on the orphan shape. If you pick 3 I'll write that table as a separate migration before deploying, so it goes up in the same window.

**Vishnu** (2026-09-19T08:58): CV check accepted — 0 genuinely orphaned, and your original 119/2 was the resume_v1_url artifact as suspected. Good.

Option 3, and refusing to weaken chk_per_student_has_student was correct. But one change to keep today short: build the TABLE now, not the mentor screen.

- task_submission_orphans: task_id, team_id, drive_file_id, filename, uploaded_at, bytes, claimed_by (nullable), claimed_at. Add-only, safe to run twice.
- Load all 50 orphans into it as part of this window.
- No UI today. Vishnu works from orphans.csv, and a mentor screen to drain it is a later job if he still wants one.

That way nothing is lost, nothing is invented, the constraint stays honest, and we are not building a screen during an outage.

Write that as a second migration and deploy both in the same window. Step 3 of my earlier list is replaced by: load the 50 orphans into that table, report the count, and confirm task_submissions is still 56 with every row team-mode before the flip and each row its own student's after.

Everything else stands. Go when Vishnu gives the word — he is taking the site down, so stop the service for the migration rather than running it under traffic.

One thing for later, not now: you found 37 students with a local CV path and no Drive link — migrate-cvs.js has copied 142 of 179. Note it in docs/cv-drive-migration.md as stragglers for the next re-run. Do not run it during this window.

**Claude** (2026-09-19T08:58): I'll build the orphans table migration now, plus the doc note for later.

**Claude** (2026-09-19T08:59): _[tool: Bash]_

**Claude** (2026-09-19T08:59): Now the loader for the 50 orphans, generated from the verified recovery data.

**Claude** (2026-09-19T08:59): _[tool: Bash]_

**Claude** (2026-09-19T08:59): All 50 hard-code `task_id = 8`. Let me verify that's actually true of the recovery data rather than an assumption baked into my generator.

**Claude** (2026-09-19T08:59): _[tool: Bash]_

**Claude** (2026-09-19T08:59): Tasks 9 and 10 are `drive`/`text` — pasted links, no file on Drive — so nothing to recover there and `task_id = 8` is correct for all 50. Let me test both migrations plus the loader on a clean copy.

**Claude** (2026-09-19T09:00): _[tool: Bash]_

**Claude** (2026-09-19T09:00): 50 orphans, 27 teams, idempotent. Now let me rehearse the complete post-deploy sequence including the flip.

**Claude** (2026-09-19T09:00): _[tool: Bash]_

**Claude** (2026-09-19T09:00): The full sequence rehearses cleanly. Total points is 0 across all teams because nothing is marked yet — expected. Let me confirm the new submissions actually work post-flip.

**Claude** (2026-09-19T09:00): _[tool: Bash]_

**Claude** (2026-09-19T09:00): That's the end-to-end proof on real data: a second HIGHVOLTAGE member hands in and the first survives. Now the CV stragglers doc.

**Claude** (2026-09-19T09:00): _[tool: Bash]_

**Claude** (2026-09-19T09:00): It exists — I'll append to it rather than overwrite.

**Claude** (2026-09-19T09:00): _[tool: Bash]_

**Claude** (2026-09-19T09:00): _[tool: Bash]_

**Claude** (2026-09-19T09:00): Let me verify the "36 not 37" claim before committing it as fact.

**Claude** (2026-09-19T09:00): _[tool: Bash]_

**Claude** (2026-09-19T09:00): Confirmed — 37 stragglers, and student 208 is one of them, so 36 is the real number to chase. Committing both migrations and the doc.

**Claude** (2026-09-19T09:01): _[tool: Bash]_

**Claude** (2026-09-19T09:01): Now let me update the deploy script for the new commit, both migrations, and the service-stop you asked for.

**Claude** (2026-09-19T09:01): _[tool: Bash]_

**Claude** (2026-09-19T09:01): The service now stops before migrations. Let me verify the restart still happens and add the post-deploy steps.

**Claude** (2026-09-19T09:01): _[tool: Bash]_

**Claude** (2026-09-19T09:01): `systemctl restart` starts a stopped service, so the ownership-then-restart step still works as one breath. Now the post-deploy script.

**Claude** (2026-09-19T09:01): _[tool: Bash]_

**Claude** (2026-09-19T09:02): Everything is built, tested and staged. Holding for Vishnu's word.

**Written for: Vishnu — ready to deploy, holding for your word.**

## Built since your last message

**`2026-09-19-d-task-submission-orphans.sql`** — the table, exactly your columns: `task_id`, `team_id`, `drive_file_id`, `filename`, `uploaded_at`, `bytes`, `claimed_by` (nullable), `claimed_at`. Plus a unique index on `drive_file_id` so re-running loads nothing twice, and a CHECK that a claim names both a student and a time or neither — half a claim is a row nobody can act on. A `v_task_orphans` view joins team code and a clickable Drive URL, since staff read it from psql today.

The table comment says what it is not: *a row here is NOT a hand-in — nothing scores it, nothing counts it, no view joins it into a team's work, no points function reads it.* **No UI**, as you said.

**`2026-09-19-e-load-task8-orphans.sql`** — all 50, `ON CONFLICT (drive_file_id) DO NOTHING`.

**`docs/cv-drive-migration.md`** — appended a stragglers section, not overwritten. Records 142/179 copied, 0 orphaned CVs, and that it's a re-run job not an incident. I verified the "36 not 37" detail against live rather than asserting it: there are 37, and student 208 is among them, so 36 is the real number to chase. It also says plainly not to run it during a deploy window.

## Rehearsed end to end on a live restore

| check | result |
|---|---|
| Both migrations + loader, first run | clean |
| Orphans loaded | **50, across 27 teams, all unclaimed** |
| Everything run a second time | 50 orphans, 56 rows — no change |
| `task_submissions` before flip | **56, all team-mode** |
| After flipping 8/9/10 | **56, all per-student, 0 unattributed, each row its own student** |
| Team points across 53 teams | **unchanged** |

And the real-world proof: on team 18 (HIGHVOLTAGE, which holds one surviving task-8 row), a second member handed in after the flip — **the original survived, two rows**. That's the bug, fixed, on real data.

## Deploy scripts staged

`deploy.sh` ships commit `d50fb56`:
- Dump first, then **verify the gzip contains 56 `task_submissions` rows** and abort if not (I tested that extraction separately — it reads 56).
- Payload from `git archive`, with all three migration files asserted present before anything is sent.
- Dry-run rsync deletes shown, with a prompt, before the real one.
- **`systemctl stop bootcamp`** before migrations, since Vishnu is taking the site down — they run with no traffic.
- Migrations one at a time, counts before and after.
- Ownership DO block and `systemctl restart` in a **single ssh invocation**, no gap (the 18 Sep lesson). `restart` starts a stopped unit, so the stop above doesn't break it.
- Ownership check, HTTP check, `journalctl` across the restart.

`postdeploy.sh` runs your four steps with a confirm prompt between step 1 and the flip, then prints what's waiting by team.

**One number to confirm before I run it:** the orphan count is **50, not 45** — 45 was my figure before the PDF misclassification was corrected. The scripts say 50 throughout.

Nothing is deployed. Live is still 56 rows with no `per_student` column. Say the word and I'll run both scripts in one window.

**Vishnu** (2026-09-19T09:02): Confirmed: 50 orphans, not 45. Your scripts are right.

Run both scripts now, in one window. Vishnu has taken the site down and told the rooms 30 minutes.

Report each step as you finish it, not all at the end — I am reading them live. In particular tell me:
- the row counts before and after each migration
- the moment the service is back up and healthy
- the orphan load count
- the result of a real two-student concurrent submission after the flip

Tell me the moment it is safe for students to come back. That is the one line Vishnu is waiting for.

If anything fails after the first migration begins, restore from the dump and tell me immediately rather than debugging forward.

**Claude** (2026-09-19T09:02): Starting now. Steps 1–2: dump, verify it holds the rows, build payload from the commit.

**Claude** (2026-09-19T09:02): _[tool: Bash]_

**Claude** (2026-09-19T09:02): _[tool: Bash]_

**Claude** (2026-09-19T09:02): **Step 1 done.** Dump `PREDEPLOY-bootcamp-2026-09-19-090241.sql.gz`, 76K, verified to contain **56 task_submissions rows and 53 teams**. That's the restore point.

**Claude** (2026-09-19T09:02): _[tool: Bash]_

**Claude** (2026-09-19T09:02): **Step 2 done.** Payload built from commit `d50fb56` via `git archive` — nothing uncommitted can reach the server. All 3 migrations present, no `.env`. Sending now.

**Claude** (2026-09-19T09:03): _[tool: Bash]_

**Claude** (2026-09-19T09:03): **Step 3 done.** Payload in staging. The two project migrations are present as files but I will not run them — they're unapplied on live and out of scope. Dry-run first:

**Claude** (2026-09-19T09:03): _[tool: Bash]_

**Claude** (2026-09-19T09:03): Dry run deletes nothing. Copying into place:

**Claude** (2026-09-19T09:03): _[tool: Bash]_

**Claude** (2026-09-19T09:03): Code is in place. Those warnings are a stale `.worktrees` directory on the server, harmless and outside the app — I'll note it but not act during the window.

**Claude** (2026-09-19T09:03): _[tool: Bash]_

**Claude** (2026-09-19T09:03): **Step 4 done.** Code deployed, dependencies installed. Now stopping the service so migrations run with no traffic:

**Claude** (2026-09-19T09:03): _[tool: Bash]_

**Claude** (2026-09-19T09:03): **Service stopped, site down.** Pre-migration state: **56 task_submissions, 53 teams, no `per_student` column**. Running migration C:

**Claude** (2026-09-19T09:03): _[tool: Bash]_

**Claude** (2026-09-19T09:04): COMMIT reached, only expected idempotency notices. Verifying:

**Claude** (2026-09-19T09:04): _[tool: Bash]_

**Claude** (2026-09-19T09:04): **Migration C done. 56 rows before → 56 rows after, all team-mode, points unchanged.** Old UNIQUE gone, both partial indexes and the CHECK in place. Migration D:

**Claude** (2026-09-19T09:04): _[tool: Bash]_

**Claude** (2026-09-19T09:04): **Migration D done.** Empty table created, `task_submissions` still 56. Loading the 50 orphans:

**Claude** (2026-09-19T09:04): _[tool: Bash]_

**Claude** (2026-09-19T09:04): **Migration E done. 50 orphans loaded across 27 teams, all unclaimed.** `task_submissions` still 56, points unchanged. Now ownership and restart, as one step:

**Claude** (2026-09-19T09:04): _[tool: Bash]_

**Claude** (2026-09-19T09:04): _[tool: Bash]_

**Claude** (2026-09-19T09:05): **Service is back up and healthy — site returning 200, ownership check prints nothing.** Watching the log across the restart:

**Claude** (2026-09-19T09:05): _[tool: Bash]_

**Claude** (2026-09-19T09:05): **Clean restart, zero errors, connecting as `bootcamp`.** Total downtime 58 seconds (09:03:51 → 09:04:49). Now step 1 of your list — confirming nothing moved:

**Claude** (2026-09-19T09:05): _[tool: Bash]_

**Claude** (2026-09-19T09:05): **Step 1 confirmed: 56 rows, all team-mode, 53 teams, points unchanged.** Now the flip of tasks 8, 9, 10:

**Claude** (2026-09-19T09:05): _[tool: Bash]_

**Claude** (2026-09-19T09:05): **Step 2 done.** All three tasks per-student. 56 rows intact, every one now its own student's row, 0 unattributed, points unchanged. Now the real two-student concurrent test against live:

**Claude** (2026-09-19T09:05): _[tool: Bash]_

**Claude** (2026-09-19T09:05): Task 10 isn't open for ECE. Let me check how releases records openness before assuming.

**Claude** (2026-09-19T09:05): _[tool: Bash]_

**Claude** (2026-09-19T09:05): Task 9 is open for ECE — I'll use that for the live test rather than opening anything Vishnu hasn't chosen to open.

**Claude** (2026-09-19T09:06): _[tool: Bash]_

**Claude** (2026-09-19T09:06): 404 "No such task" — task 9 is `active` but the students are in ECE. Let me check why before concluding anything.

**Claude** (2026-09-19T09:06): _[tool: Bash]_

**Claude** (2026-09-19T09:06): Task 9 is retired (`active = f`) — that's why it 404s, correct behaviour. Task 10 is active but closed. I'll open task 10 for ECE, test, and close it back exactly as found.

**Claude** (2026-09-19T09:06): _[tool: Bash]_

**Claude** (2026-09-19T09:06): _[tool: Bash]_

**Claude** (2026-09-19T09:06): Both accepted. Checking the database — this is the moment of truth:

**Claude** (2026-09-19T09:06): _[tool: Bash]_

**Claude** (2026-09-19T09:06): **The fix is proven on production.** Two students submitted 15 milliseconds apart — both rows survived, each with its own student and its own link. Under the old constraint one would have destroyed the other. Cleaning up the test data and restoring task 10 to closed:

**Claude** (2026-09-19T09:06): _[tool: Bash]_

**Claude** (2026-09-19T09:07): Restored exactly as found — task 10 closed, 56 rows. Final health check:

**Claude** (2026-09-19T09:07): _[tool: Bash]_

**Claude** (2026-09-19T09:07): ## ✅ It is safe for students to come back.

Site is up, healthy, zero errors. Vishnu can reopen the rooms.

---

**Deploy complete — commit `d50fb56`. Total downtime 58 seconds** (09:03:51 → 09:04:49).

**Row counts at every stage — nothing was ever lost:**

| stage | task_submissions | notes |
|---|---|---|
| Dump verified | **56** | 53 teams, confirmed inside the gzip |
| Before migrations | **56** | all team-mode, no `per_student` column |
| After migration C | **56** | all team-mode, old UNIQUE gone, 2 partial indexes + CHECK |
| After migration D | **56** | orphans table created, empty |
| After migration E | **56** | 50 orphans loaded |
| After the flip | **56** | all per-student, 0 unattributed |
| Final | **56** | unchanged throughout |

`sum(total_points)` was 0.0 at every single check — nothing is marked yet, so the points model demonstrably never moved.

**Orphan load: 50 files across 27 teams, all unclaimed.** Held in `task_submission_orphans`, nothing invented, every file still whole on Drive.

**The concurrent test, on production over real HTTPS:** two ECE-T99-TESTTEAM members submitted to task 10 **15 milliseconds apart** — 09:06:35.911 and 09:06:35.926. Both returned `{"ok":true}`, **both rows survived**, each with its own student and its own link. Under yesterday's constraint one would have silently destroyed the other. Test rows deleted and task 10 restored to closed, exactly as found.

**Tasks 8, 9 and 10 are per-student.** Every existing hand-in became its own student's row, none unattributed.

**Two things worth knowing, neither blocking:**

- **Task 9 is retired** (`active = false`), so it 404s for students regardless of the flip. It was already retired before I arrived — I haven't changed it. If Vishnu expects students to hand in against task 9, it needs reactivating.
- The server has a stale `.worktrees/side` directory that rsync couldn't delete. Harmless, outside the app, untouched during the window. Worth clearing at a quieter moment.

**Not run, as instructed:** `migrate-cvs.js` for the 36 CV stragglers — documented in `docs/cv-drive-migration.md` for the next re-run.

Branch `per-student-tasks` holds all three commits. It is not merged to main — say the word when you want that.
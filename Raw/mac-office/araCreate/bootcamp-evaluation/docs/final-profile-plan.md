# Collecting the final resume

Decided with Vishnu, 24–25 September 2026.

**The dashboard collects. The comparison is done outside it.**

That is the whole scope. An earlier version of this plan had a comparison
screen, an evaluation button, OCR and text extraction; Vishnu, 25 Sep:
"no need comparison tool... just an upload place is enough, we will do
analysis outside." What is left is the upload, the switch that opens it, and
the door the data leaves by.

---

## 1. What is collected

The **after resume** — a second CV, next to the one the student arrived with.

Nothing else. The five profile questions are *not* re-asked, so no column is
added and `student_profiles.goal` keeps the single answer it has held since
Day 1.

## 2. Who opens it

**An admin, per venue, whenever they choose.**

Today `web/src/pages/Profile.jsx` decides with

```js
const showV2 = d.is_last_day || Boolean(p.resume_v2_url)
               || d.today === d.total_days - 1
```

and `is_last_day` is `cal.day >= cal.total` — a date. This is the one
activity in the product that opens itself on a calendar. Every other one
waits for a person to open it on the release board, per department.

The calendar is the wrong instrument here: EEE and ECE finish at different
times, and a day that slips leaves the box shut while students are waiting,
or open when nobody is.

`2026-09-25-a-final-resume-release.sql` adds `final_resume` as a release
type — whole cohort, so `item_id IS NULL`, the same shape as `attendance`
and `assessment`. **No column is added**: `resume_v2_url`, `resume_v2_at`
and the three `resume_v2_drive_*` columns have existed since the schema was
written, and the upload route already accepts `which=v2` with no gate of its
own. The only thing missing was a switch, so the only thing added is a
switch.

It scores nothing and has no deadline. Nothing in `v_team_day_points_v3`
reads `final_resume`, so it cannot move a team's total.

## 3. What the files are

Measured against Drive on 24 Sep, not assumed:

| | Count |
| --- | --- |
| PDF | 164 |
| DOCX | 32 |
| Images | **0, and impossible** |

The upload route sniffs magic bytes and accepts only `%PDF` and a DOCX zip.
A `.jpg` renamed `.pdf` does not get through, because the bytes decide and
not the filename.

**But a photograph inside a PDF does.** 69 of the 164 PDFs contain no fonts
at all — they are pictures of a CV, exported or "saved as PDF" from a phone.
`pdftotext` returns one character from them.

That does not matter for collecting, and it does not matter for showing: a
picture-PDF displays perfectly. It matters only to whoever does the analysis
outside, and it is why this is written down: **about a third of the CVs
cannot be read as text without OCR.** Tesseract recovered 20 of 20 in
testing, at 2.8 seconds each, if that is ever wanted.

## 4. Getting the data out

`scripts/export-cvs.js`.

```sh
node scripts/export-cvs.js                # the CSV, instant
node scripts/export-cvs.js --files        # and the PDFs, a few minutes
```

- `cv-export/resumes.csv` — **one row per active student**, including the
  ones who handed in nothing. A spreadsheet that quietly omits them answers
  "how did the cohort do" against the wrong denominator, and the people
  missing from it are exactly the ones worth ringing.
- `cv-export/files/<TEAM>/<roll> <NAME> - before.pdf` — and `- after.pdf`.

It reads. It never writes to the database or to Drive, and never deletes.

`scripts/export-resumes.sh` does **not** do this job any more: it selects
`resume_v1_url LIKE '/uploads/resumes/%'`, and since `2026-09-21-b` every
one of those rows is a Drive link, so it reports that nothing was handed in.

## 5. The Drive links are public

Tested signed out, in a browser with no Google cookies: a student's CV
renders. The files are shared open, which is why the CSV of links is enough
on its own for outside analysis.

The same fact is worth a separate decision: those links are readable by
anyone on the internet who has one — full name, home address, personal
email. Tightening Drive sharing to the organisation would close that, and
would still work for signed-in staff, but would break the links for anyone
outside the workspace.

## 6. Order of work

1. **The migration** — written and tested against a copy of the 25 Sep data.
   Applied twice, rolled back, re-applied: no errors, nothing else changed.
   Deploy before opening the release.
2. **The student side** — swap the calendar condition in `Profile.jsx` for
   the release, so the upload box appears when an admin opens it.
3. **The admin side** — one row on the Open tab, a control per venue.
4. **The export** — `scripts/export-cvs.js`, already written.

## 7. Still to chase

**10 students have no before resume** (196 of 206 on 25 Sep). They cannot be
part of any before/after analysis, so this is a phone-call job before the
end rather than a reporting problem afterwards:

```sql
SELECT team_code, roll_no, name, phone
  FROM v_resume_handin WHERE NOT has_before ORDER BY team_code, name;
```

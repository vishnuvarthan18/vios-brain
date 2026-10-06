# Screens — Track 2, the survey

Playwright, 390px, deviceScaleFactor 2, against a real dump with 209 students
and 1,184 answers. Captured 19 Sep 2026. Regenerate with the script named at
the bottom.

**This is Vishnu's checkpoint and the only one before the build is finished.**
The question these answer is not "does it work" — 241 green checks answer that
— but "does it read correctly to a tired student on a phone".

| File | What it shows |
| --- | --- |
| `01-student-my-work.png` | The survey card, first in Today and tagged **Required**. The other items are still there and still usable — Required means it matters, not that the rest is barred |
| `02-survey-before-first-tap.png` | **The one that matters most.** See below |
| `03-survey-after-first-tap.png` | The same screen once one button is tapped: both buttons for that question disabled, the answer echoed back |
| `04-admin-open-tab.png` | The Open tab, both survey rows, a control per venue. The final round is labelled *end of bootcamp only* |
| `05-admin-surveys.png` | The Surveys screen: the list, and the proof report below it |
| `06-admin-loader.png` | Paste-many, with the parsed list shown **before** saving |
| `07-daily-results.png` | Per question, by venue, three states with denominators |
| `08-names-behind-a-count.png` | The students behind a number, with phones |
| `09-proof-report.png` | Before, after, and the gain — per question and per venue |

## 02 — before the first tap

The tap is final and cannot be undone. The logic is proved; what a screenshot
can show and a test cannot is whether the student is told **before** they
touch it.

Measured, not judged by eye:

- the sentence *"Your first answer is saved and cannot be changed"* sits at
  **y=197–234** in an 844px viewport — fully visible without scrolling
- the first question's Yes/No buttons end at **y=499**, also on screen
- so the warning is between the page title and the first button. A student
  cannot reach a button without passing it.

It is bold, and it is the first thing inside the card.

## What to look at

- **02**: is the warning strong enough? It is the only protection against a
  mis-tap, and a mis-tap is permanent.
- **09**: the proof report is what a college is shown. Three numbers on every
  line — yes, no, **not asked** — because a student who joined late did not
  answer *no* to Day 3, they were never asked. Two separate after-figures
  appear on purpose: *after, same people* is what the gain is computed from,
  and *after, everyone* is a different base and says so.

## Regenerating

```sh
./scripts/load-local-dump.sh ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz bootcamp_shots
psql -d bootcamp_shots -f src/db/migrations/2026-09-19-c-surveys.sql
PORT=3099 PGDATABASE=bootcamp_shots node src/server.js &
node tests/screens.js
```

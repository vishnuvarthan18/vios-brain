# Daily survey — full specification

Answers the question the agent marked BLOCKED. Track 2 is now unblocked.

---

## 1. What it is for

**Proof of learning gain.** Before a topic is taught, how many students already
know it. At the end of the bootcamp, how many know it.

That pair — *before teaching* and *after the whole bootcamp* — is the evidence
shown to the college.

It is **not** paired with the quiz. The quiz measures a single day; the survey
measures the nine-day journey.

---

## 2. The two rounds

| Round | When | Questions |
| --- | --- | --- |
| **Daily** | Before each day's teaching, every day | That day's questions. Different each day, written the evening before |
| **Final** | Once, at the end of the bootcamp | **Every question from all nine days, re-asked** |

Both are opened **by hand, per venue**, by the admin, on the Open tab. Nothing
opens itself. The final round is opened by the admin like any other activity.

**This is what makes the report free.** The final round re-asks the *same
question rows*, so before and after are matched by `survey_question_id`. There
is no text matching, no topic tagging, and no stable-key prefix — the agent's
concern about rewording does not arise, because a question is never re-typed.

---

## 3. Rules

- **Yes / No only.**
- **Every student answers individually.** Zero points, no timer, no right answer.
- For the student it is **just another item in "My work"** — first in the list,
  tagged **Required**. It does **not** lock the other items.
- Loaded by **paste-many, one question per line**, like the quiz loader.
- **Each answer saves the moment it is tapped.** No submit-everything button.

---

## 4. Schema

```
surveys          (id, day, round, title, created_at)
                 round: 'daily' | 'final'.  day is NULL for the final round

survey_questions (id, survey_id, position, text)
                 daily rounds only — the final round asks no new questions

survey_answers   (id, survey_question_id, student_id, round, answer bool, answered_at)
                 UNIQUE (survey_question_id, student_id, round)
```

The `round` column on the answer is what carries before and after. One question
row, two answers per student at most. Pairing is a join on
`survey_question_id`, nothing more.

The final round's form is **every `survey_questions` row for the program**,
ordered by day then position. It creates no question rows of its own.

Release rows go in the existing `releases` table, one per survey per venue,
through `isOpenFor`. A survey with zero questions cannot be opened, the same way
a quiz below five questions cannot.

---

## 5. The proof report

All four views are wanted.

### 5.1 Three states, always. This is the rule the rest depends on.

Every proof view shows **three numbers: yes, no, and not asked.** Never two.

A student who joined on Day 5 did not answer *no* to Day 3's question. They
were never asked. Folding that into a denominator inflates every gain, and the
inflation is invisible — the number still looks like a number. This is the
single easiest way for the proof to become a lie, so it is structural rather
than a convention anyone has to remember:

- **Three numbers, always.** yes · no · not asked. A view that shows two is
  wrong, even when *not asked* is zero.
- **Never a bare percentage.** Every percentage carries its denominator beside
  it — "87% (179 of 206)", never "87%".
- **"Not asked" is visible in the UI.** Never hidden, never zero-filled, never
  merged into *no*. On a day when it is zero it still says zero.
- **A test fails the build** if a proof endpoint returns only yes and a total.
  See `tests/survey.js`.

The three states are distinguishable in the data because `survey_answers`
holds a row only when a student actually answered: `answer` is NOT NULL, so
*not asked* is the absence of a row, exactly as a missing attendance row means
nobody took the register. Counting is therefore
`COUNT(*) FILTER (WHERE answer)` and `COUNT(*) FILTER (WHERE NOT answer)`,
with *not asked* as the roster minus the two — never `total - yes`.

### 5.2 The four views

**Per question — the main one.**

> Multimeter — Day 3: **12 knew, 190 did not, 4 not asked** (206)
> → End: **190 know, 12 do not, 4 not asked** (206) · **+86%**

One line per question, ordered by gain. This is the evidence for the college.

**One headline number.**

> Across 40 topics: **18% before (1,432 of 8,240) → 87% after (7,169 of 8,240)**

One sentence for management, with its denominator.

**Per student**, on their own profile page — their personal before and after
across every question they answered. Their own growth, not the cohort's. A
question they were never asked is shown as such, not as a gap.

**Split by venue** — EEE and ECE side by side on every view above, so it is
visible whether one room learned more than the other. Each venue carries its
own three numbers.

Every count links to the named list behind it, with phone numbers and CSV
export, like every other number in the product. `personal_email` is empty for
all 209 students, so those lists are phone-only and say so on screen.

---

## 6. Daily results, during the bootcamp

Separate from the proof report, and needed every day:

- Per question, Yes and No counts, split by venue.
- Every count links to the named list of students behind it.
- Who has not answered today's survey at all — that list belongs in the chase
  set.

---

## 7. What this does not do

- It does not compare against the quiz. Different purpose, different timing.
- It does not carry points, ever.
- It does not block a student from their other work.

---

## 8. Decisions that were likely to block the agent — all answered

Settled 19 Sep so no item in Track 2 has to stop and ask.

| Question | Answer |
| --- | --- |
| In the final round, does the student see what they answered on the day? | **No. Hidden.** Seeing "you said No on Day 3" makes people answer Yes to look good, and the proof stops meaning anything. They answer fresh |
| Can a student change an answer after tapping? | **No. First tap is locked.** Say so on screen *before* the first question — "your first answer is saved, you cannot change it" — so a mis-tap is not a surprise. Disable both buttons the moment one is tapped |
| Who can see the proof report? | **Admin only.** Not mentors, not students. It is evidence for the college and the numbers get checked before anyone else sees them |
| Where does the survey loader live in the admin? | **Beside the quiz loader**, following the same pattern and the same words. The nav is being rebuilt in v3; do not invent a new place for it now |
| What is a survey called? | Auto-titled **"Day N survey"** for daily rounds, **"Final survey"** for the last one. No title field to fill in |
| A student who joined late and missed Day 3's daily round | Still gets every question in the final round. Their Day 3 "before" is simply absent, and the report shows it as *no before answer*, never as a No |
| A question answered by nobody | Shown in the report with a zero and a note, never hidden. A silently dropped row is how a proof becomes a lie |
| Does the daily round close itself when teaching starts? | **No. Nothing closes itself.** The admin closes it, like everything else |

**The rule behind the first three:** this survey exists to be shown to a college
as evidence. Anything that makes the after-number easier to inflate — showing the
old answer, letting answers be edited late, publishing the figure before it is
checked — is worth more as integrity than as convenience.

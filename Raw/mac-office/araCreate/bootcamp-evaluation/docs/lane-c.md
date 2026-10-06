# Lane C — surveys, assessment, projects

**Read `docs/lane-rules.md` first. It is short and every rule in it exists to
stop your branch conflicting with the other three.**

Your branch: `lane-c`. Your report: `docs/lanes/lane-c.md`.

| Order | Screen | Old function | Section |
| --- | --- | --- | --- |
| 1 | **Surveys** | `page_survey_admin` + `survey_loader` + `survey_results` + `draw_proof` | 6.5 |
| 2 | **Assessment** | `page_assess_admin` | 6.5 |
| 3 | **Projects** | `page_projects_admin` | 6.5 |
| 4 | **Tinkercad** | `page_tinkercad` | 6.5 |

Surveys is four screens in one and is most of this lane. Do it first.

---

## The survey proof reports — the rule that matters most in this lane

**Every proof view shows three numbers: yes, no, and not asked. Never two.**

A student who joined on Day 5 did not answer *no* to Day 3's question. They were
never asked. Folding that into a denominator inflates every gain, and the
inflation is invisible — the number still looks like a number. This is the
easiest way for the whole proof to quietly become a lie.

So:

- Three numbers, always. Even when *not asked* is zero, it says zero.
- **Never a bare percentage.** Every one carries its denominator beside it —
  "87% (179 of 206)", never "87%".
- *Not asked* is never hidden, never zero-filled, never merged into *no*.

There is a test that fails the build if a proof endpoint returns only yes and a
total. `docs/survey-spec.md` section 5.1 is the full rule — read it.

Four views, all wanted: per question · one headline number · per student · split
by venue. Every count links to the named list behind it.

## Everywhere in this lane

`personal_email` is empty for all 209 students. Any list offering to contact
someone is **phone only, and says so on screen.**

The bulk loaders follow the same rules as Lane B's: parsed list shown before
saving, bad lines rejected with a clear message, count shown after.

## Assessment

Pre and post. Show **percent gain, never raw marks** — the two question sets are
different, so the raw numbers are not comparable and putting them side by side
invites exactly that mistake.

---

When all four are done, report and stop. Do not start another lane's screens,
even if you finish first.

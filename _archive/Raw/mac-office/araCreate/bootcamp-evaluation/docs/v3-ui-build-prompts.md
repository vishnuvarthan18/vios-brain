# v3 UI build — the agent's instructions

**How to use this: give the agent one line.**

To do the whole migration in one run:

> Read `docs/v3-ui-build-prompts.md` and do the full run.

To do one piece at a time:

> Read `docs/v3-ui-build-prompts.md` and do step 3.

That is the whole prompt either way. Everything the agent needs is below. Do not
paste anything else, and do not explain the project again — it is written here.

---

## The full run

Do steps 1 to 11 in order, in one session. **Do not do step 12.** Cutover is a
decision, not a build step, and it is Vishnu's to make after he has seen the
result.

**Do not stop to ask.** Every decision this migration needs is already written
down, here or in `docs/v3-ui-migration.md`. If something is genuinely undecided,
write it in the report, make the smallest reversible choice, and keep going.

**Finish each screen before starting the next.** A screen is finished when all
seven points in section 9 of the migration doc are true. Then:

1. Tick that screen's row in section 6 of `docs/v3-ui-migration.md`.
2. Commit the screen and the tick together, one commit per screen.

That tick is the resume marker. If this run is interrupted — context runs out,
the session ends, anything — the next run reads the ticks and carries on from
the first unticked row. So the tick must never be written before the screen is
actually done.

**If a screen will not come cleanly:** stop that screen, leave its row unticked,
write down what blocked it, and take the next one. Never force it. Never
refactor the old front end to make the new one easier.

**No Playwright. None.** Do not run it, do not write tests with it, do not take
screenshots, do not install browsers for it. It is slow, and Vishnu checks the
screens himself in a browser. The existing suites in `tests/` are left exactly
as they are — not run, not edited, not extended, during this migration.

What you do instead, for every screen you finish:

- The production build must pass with **no errors and no warnings**.
- Write a **Check list** in the report: the two or three things Vishnu should
  click to prove it works. Be specific — "hand in a photo on Day 2 as a
  non-lead", not "check the work page". Name the role and the day.

**You cannot claim a screen works because the code looks right.** Six items on
this project were marked DONE with an API built and no screen to reach them.
Nothing automated is watching for that now, so the Check list is the whole of
the safety net: it must describe a real path a person can actually follow. A
screen with no reachable route is not done, however complete its code is.

### The report, at the end

1. A table of all 26 screens: done, or not done and why.
2. Everything the old screen did that you could not reproduce. This list being
   empty is a claim, and it will be checked.
3. Every `--ac-*` token you needed and could not find.
4. The Check list for every finished screen, gathered in one place, so Vishnu
   can go through the lot in a single sitting.
5. Anything you noticed that is worth fixing but is not part of the move —
   listed, not built.

---

## Standing rules — these apply to EVERY step

Read `docs/v3-ui-migration.md` first. It is the checklist for this work and it
is authoritative. Do not re-derive what it already lists.

**Branches.** Work on `dev`. There are only two branches, `main` and `dev`.
Commit one screen at a time. Never commit a half-finished screen.

**Where things are.** The React front end is `web/` — Vite 8, React 19,
Tailwind 4, shadcn. It builds into `src/public/v3/` and is served at `/v3/`.
The old front end is `src/public/app.js` at `/`.

**The old front end is not to be edited or deleted** until every screen has
moved. Both run side by side. That is what makes this reversible.

**Nothing is redesigned during the move.** This is a restyle. If the old screen
does something, the new one does it too. New ideas go in a list at the end of
the step report — they do not get built.

**Light theme only.** No `dark:` classes. No dark variant is registered, so a
`dark:` class compiles to nothing and silently does nothing.

**No raw values in components.** No hex, no px, no font name. Every value points
at an `--ac-*` token in `src/public/ds/`. If a token is missing, say so in the
report — do not invent one.

**Two brand rules are already baked into `web/src/components/ui/button.jsx`:**
buttons are square, and Golden Sun is a background colour whose text is
graphite, never white. Use the component; do not restyle it.

**Use `web/src/lib/api.js`, never a bare `fetch`.** It turns every failure into
plain English.

**Navigation stays flat.** The 17 admin items keep their current order.
Regrouping into five groups is Phase 4 and happens after every screen has moved.

**Done means seen.** A step is finished only when all seven points in section 9
of `docs/v3-ui-migration.md` are true. An API with no screen to reach it is not
done. A test run reporting 0 pass / 0 fail is a failure, not a success.

**Report at the end of every step:** what you built, what you could not
reproduce from the old screen, which tokens were missing, and the Check list.
Do not claim a screen works because the code looks right.

**Commands.** `cd web && npm run dev` proxies `/api` to `localhost:3099`.
`cd web && npx vite build` builds. `emptyOutDir` is false on purpose.

---

## Step 1 — Finish the login

It is only partly built. Finish it against sections 6.1 and 4 of
`docs/v3-ui-migration.md`.

Missing: the background photograph. Port it exactly as `app.css` has it —
`login-bg-sm.jpg` below 900px, `login-bg.jpg` at 900px and up, `cover`, on
`min-height: 100dvh`, with the veil over it. Below 560px the position shifts to
`50% 28%` and the card sits at the bottom. The comment in `app.css` explains
why: the photo is 16:9, a tall phone crops it hard, and the arch has to be
pulled into the band that stays visible above the card.

Also add to `web/index.html`: the favicon
(`ds/assets/logos/aracreate-icon-default.svg`), `theme-color` `#f9bf3b`, and
`viewport-fit=cover` on the viewport meta.

Keep what is already there: a wrong code keeps the email and clears only the
code, the error sits above the form, the button disables and says "Signing in…".

Say in the Check list that the background needs looking at on a phone, at
about 560px, and on a laptop — the three sizes where it behaves differently.

## Step 2 — The app shell

Build the signed-in frame against section 3 of `docs/v3-ui-migration.md`.

Sidebar on a laptop, the same markup as a drawer below 900px, with a scrim,
outside-tap and Esc to close, and `aria-expanded`. Brand mark is
`ds/assets/logos/aracreate-icon-t-w-b-g.svg`. The foot shows name, role and team
code, then Log out, which asks first.

Navigation comes from `pages_for()` in `app.js`. Keep the order exactly,
including Attendance inserted third for a lead, and Survey and "Where you are"
appearing only while one is open, inserted at the front.

Read the URL hash on load so a refresh stays on the same page. Fall back to the
person's first tab if the hash names a page they may not see.

Port the `ICON` map. Every page must have its own icon — the old code falls back
to the `admin` icon, which is how a missing key hides. Compare the two `ICON`
key lists directly and confirm none is missing, rather than trusting the page to
look right.

## Step 3 — Student Work

Rebuild `page_projects` against section 6.2.

Must keep: today's card marked in gold and scrolled to, days that have not
arrived faded, and a non-lead told **who their lead is** rather than shown an
empty space. Tasks and projects both live on this screen.

## Step 4 — Student Today

Rebuild `page_home` against section 6.2.

Must keep: the "Start here" card on Day 1 with the three things to do, ticking
off as they are done; and the completion bar with the missing items **named**,
not just a percentage.

## Step 5 — Student Quiz

Rebuild `page_quiz` against section 6.2. This is the hardest student screen.
Read its four "must keep" points before writing anything.

A refresh mid-quiz goes straight back to the questions. Submit asks first and
names the blanks. A failed answer save reports beside its own question, retries
once, and offers Try again. The clock shows the real time immediately.

## Step 6 — Student Survey, and Attendance

Rebuild `page_survey` and `page_attend` against section 6.2.

Survey: Yes/No, saves the instant it is tapped, survives a refresh, tagged
Required, and does **not** lock the other items.

Attendance: the day list comes from `settings.total_days`, never hardcoded to 9.
It opens on today, labelled "Day 5 — today". Later days say "(not yet)". Keep
"Mark all present", "Clear all", and the live count.

Do **not** build the 09:00–10:00 window. The rules are not settled.

## Step 7 — Student Posts, Board, You, Where you are

Rebuild `page_posts`, `page_board`, `page_profile`, `page_assessment` against
section 6.3.

Board highlights your own row with a "you" badge and pins a "Your team — #3, 70
points" card on top. Posts are private: a student sees only their own, and
mentors cannot read post content at all.

`personal_email` is empty for all 209 students. Any screen offering to contact
someone says phone only.

## Step 8 — Staff daily screens

Rebuild `page_admin`, `page_marking`, `page_open`, `page_quizlive`,
`page_register` against section 6.4.

Marking: the comment box saves on blur as well as on a score click. With no
score picked it says "Pick a score — the comment saves with it" rather than
throwing the typing away.

## Step 9 — Admin content screens

Rebuild the six screens in section 6.5.

Every bulk loader is paste-many, one item per line, shows the parsed list before
saving, rejects bad lines with a clear message rather than skipping them
silently, and shows the count added.

The survey proof reports carry three numbers, always: yes, no, and not asked.
Every percentage carries its denominator beside it.

## Step 10 — Admin people screens

Rebuild the three screens in section 6.6.

Delete is a red danger button in the confirm box, names the thing, and says what
else goes with it. Delete must never look like Edit.

## Step 11 — Reports

Rebuild the four screens in section 6.7, plus the three chart renderers:
`attendance_chart`, `points_chart`, `spread_html`. They use the design system's
`.ac-chart__*` and `.ac-bars__*` classes, not `app.css`.

## Step 12 — Cutover

Only when every row in section 6 is ticked. Follow section 10 of the migration
doc exactly. Do not start Phase 4 in the same commit.

---

## Never

- Never run `load-eee.sql` or `load-ece.sql`
- Never change `start_date` on the server
- Never deploy — no deploy until the whole build is finished
- Never edit `src/public/app.js` before step 12

# Bootcamp Dashboard — design

The spec the app is built to. Read section 6 before you write any CSS.

---

## 1. What was actually wrong

The app was never ugly by accident. It sits on the araCreate design system,
which ships **90+ components**, and it was using about **twelve of them**.

Everything else was hand-rolled next to a better version that already existed:

| The app hand-rolled | The design system already had |
|---|---|
| `.skeleton` / `.sk-bar` grey bars | `.ac-skeleton`, shaped like the content that is coming |
| `.deptsw` custom toggle | `.ac-segmented`, with focus rings and the accessible fill |
| `<p class="hint">Nothing yet.</p>` | `.ac-empty` — art, title, sentence, one action |
| alerts that push the page down | `.ac-toast` / `.ac-toast-region` |
| a bare `<h1>` per page | `.ac-toolbar` — title, count, spacer, actions |
| plain text names in tables | `.ac-avatar` + `.ac-cell-stack` |
| numbers with no context | `.ac-progress-bar`, `.ac-stat__note` |
| a table of attendance numbers | `.ac-chart` + `.ac-bars` |
| "← Back" buttons | `.ac-breadcrumb` |

So the fix is not a new visual language. It is **using the one already paid
for.** That also makes it safe two days before go-live: every component below
is already tested, accessible and in the light theme.

---

## 2. Principles

1. **Never land anyone on a form.** Land them on what to do today.
2. **One job per screen**, said in the first line.
3. **The system owns the look.** If you are writing CSS for a button, a card,
   a badge or a table, stop and find the `.ac-*` class.
4. **Every state is designed** — loading, empty, error, success, offline,
   no-permission. A screen with only a happy path is unfinished.
5. **Phone first.** 206 students in classrooms, on their own phones, on
   college wifi. 390px is the design width; the desktop is the bonus.
6. **Say what happens next**, not what went wrong.
7. **Numbers carry context.** `70` means nothing. `70 points · #3 of 52 · +9
   today` means something.

---

## 3. Information architecture

Home is first for everyone. It is the only screen that answers
"what do I do right now".

| | Student | Team lead | Mentor | Admin |
|---|---|---|---|---|
| **Home** | ✓ | ✓ | ✓ | ✓ |
| My profile | ✓ | ✓ | | |
| My team | ✓ | ✓ | | |
| Projects | ✓ | ✓ (submits) | | |
| Attendance | | ✓ | | |
| Quiz | ✓ | ✓ | | |
| Leaderboard | ✓ | ✓ | ✓ | ✓ |
| Quiz results | ✓ | ✓ | ✓ | ✓ |
| My teams (scoring) | | | ✓ | ✓ |
| Students · Teams · Staff · Quizzes · Progress | | | | ✓ |

Nav labels and page headings use the **same words**. "Projects" in the nav is
"Projects" as the heading, never "Our projects".

---

## 4. Page inventory

| Page | Who | Its one job | Pattern |
|---|---|---|---|
| Home | all | What do I do today | Dashboard |
| My profile | student | Write today's post, hand in resumes | Dashboard + timeline |
| My team | student | Who we are, where we stand | Detail |
| Projects | student | Hand in today's work, see scores | Card stack |
| Attendance | lead | Tick today's six, save | Focus form |
| Quiz | student | Answer under a clock | Focus |
| Leaderboard | all | Where is my team | Ranked list |
| Quiz results | all | How every team did | List |
| My teams | mentor | Score what is waiting | List → Detail |
| Students / Teams / Staff | admin | Manage the roster | List |
| Quizzes | admin | Load questions, open the day | List → Builder |
| Progress | admin | Who is falling behind | List → Journey |
| Admin | admin | Run today | Dashboard |

---

## 5. Layout patterns

Five, and no sixth.

**Dashboard** — a banner stating the day, a row of stats, then cards ordered by
what the person should do first. Used by Home, Admin, My profile.

**List** — `.ac-toolbar` (title · count · spacer · actions), an optional
`.ac-filter-bar` (search, segmented filter), then one `.ac-card` holding an
`.ac-table`. Rows carry `.ac-cell-stack` so a name can bring its register
number without a second column.

**Detail** — `.ac-breadcrumb`, heading, stat row, then an `.ac-stack` of cards.
Never a "← Back" button; the breadcrumb is the way back.

**Focus** — one column, one task, nothing else competing. Attendance and the
quiz. The quiz keeps a sticky progress header.

**Modal** — `.ac-modal` for add/edit/confirm only. Anything longer than eight
fields becomes a page.

---

## 6. Component rules

Reach for these. The right-hand column is what not to do.

| You need | Use | Never |
|---|---|---|
| Page title + actions | `.ac-toolbar` | a bare `<h1>` and a floated button |
| Second-level nav | `.ac-breadcrumb` | "← Back" |
| Loading | `.ac-skeleton` shaped like the result | "Loading…" |
| Nothing to show | `.ac-empty` (title, sentence, one action) | "No data" |
| It worked | `.ac-toast` | an alert that shifts the page |
| It failed in a form | `.ac-alert--error` inside the form | a toast they will miss |
| It failed on the page | page-level alert **plus a retry button** | a dead screen |
| A person in a row | `.ac-avatar--sm` + `.ac-cell-stack` | plain text |
| A number | `.ac-stat` with `__note` for context | a bare figure |
| A proportion | `.ac-progress-bar` | a percentage in text |
| A choice of 2–4 | `.ac-segmented` | custom buttons |
| A status | `.ac-badge--success/warning/outline` | coloured text |
| Comparison across rows | `.ac-chart` + `.ac-bars` | a column of numbers |
| A tag list | `.ac-tag-group` / `.ac-tag` | comma-separated text |
| Ordered progress | `.ac-steps` with `data-ac-state` | a checklist of ticks |

**Colour discipline.** Golden Sun is one accent, used for one thing per
screen: the thing to do next. If two things are gold, neither is.

**Accessibility notes already settled by the system** — fills that carry text
use Golden **Soft** (6.63:1); Golden **Sun** is only for fills with no text on
them. Do not override this.

---

## 7. States

Every screen ships all six.

- **Loading** — `.ac-skeleton` in the shape of the answer. The page heading
  stays put, so nothing jumps when data lands.
- **Empty** — `.ac-empty`: what this is for, one sentence, one button. Never a
  dead end.
- **Error, form** — `.ac-alert--error` above the fields, title "Check this",
  the typed values kept.
- **Error, page** — the heading, the message, and **Try again**.
- **Offline** — "Lost connection. Check your wifi and try again." Never
  "Failed to fetch". Background saves report next to the thing they were
  saving and retry themselves once.
- **Success** — a toast, bottom-right, four seconds. The page does not move.

---

## 8. The four journeys

**Student, every morning.** Open → **Home**. The banner says which day it is.
Three cards say: post not written, project not handed in, quiz open. Each is
one tap to the thing. When all three are done the cards collapse to one line
and Home shows the team's rank instead.

**Team lead, every morning.** Same Home, plus a fourth card: attendance, which
opens on *today* and offers "Mark all present" — two taps for the usual case.

**Mentor, every evening.** Open → **Home**. One number: how many projects are
waiting. A list of teams under it, each linking to its scoring page. When it
hits zero the page says so, clearly, and stops asking for anything.

**Admin, every morning and evening.** Open → **Admin**. Today's box opens or
closes the day's quiz in one click. Below it: what needs attention, the
counts, attendance as a chart rather than a column of numbers, and the
department switch that filters everything on the page.

---

## 9. Motion and accessibility

- Transitions use `--ac-duration` and `--ac-ease`. Nothing animates longer
  than 200ms except the toast entrance.
- Everything respects `prefers-reduced-motion`.
- Tap targets are `--ac-target-min` (44px) on phones.
- Every control that toggles carries `aria-pressed`; every live region that
  updates carries `aria-live="polite"`.
- Focus is never removed, only restyled by the system.
- The phone menu traps nothing, but closes on Esc and on an outside tap, and
  the toggle carries `aria-expanded`.

---

## 10. Responsive

| Width | Nav | Tables | Cards |
|---|---|---|---|
| < 768px | overlay + backdrop | secondary columns drop (`.hide-sm`) | one column |
| 768–991px | overlay + backdrop | full | two columns |
| ≥ 992px | fixed rail, collapsible | full | grid |

At 390px there is **no horizontal page scroll**, on any screen. Tables scroll
inside their own `.ac-table-wrap`, never the page.

---

## 11. What shipped in this pass

Front end only. `public/app.js` and `public/app.css`. No server change, no
database change, no new endpoint — Home is composed from calls that already
existed.

**New**
- **Home**, for student, team lead and mentor. The day, what is left to do,
  one item flagged as next, then the numbers. An admin's Home is the Admin
  page, which was already a dashboard.
- Toasts for every success. Nothing shifts the page any more.
- Charts on Admin: attendance as a four-band distribution, points as a ranked
  top eight. One series each, direct-labelled, labels in text tokens.
- A two-panel sign-in on wide screens; the panel is dropped on a phone.

**Replaced with the design system**
`.skeleton` → `.ac-skeleton` · `.deptsw` → `.ac-segmented` ·
`.steps` and `.timeline` → `.ac-steps` · `.priv` → `.ac-alert` ·
`"Nothing found"` → `.ac-empty` · `.pagehead` → `.ac-toolbar` ·
`"← Back"` → `.ac-breadcrumb` · plain names → `.ac-avatar` + `.ac-cell-stack` ·
comma-joined skills → `.ac-tag-group` · bare figures → `.ac-progress-bar`.

**Two real bugs this found**
- The quiz header is a dark surface holding a `<strong>`. It did not re-point
  `--ac-text-heading`, so the quiz title rendered graphite-on-graphite and
  was invisible. The design system warns about exactly this for its nav and
  banner; the same rule applies to any dark surface you make yourself.
- `.ac-banner` points `--ac-surface-subtle` at graphite. `.ac-banner--accent`
  does not reset it, so a progress track inside the accent banner came out
  near-black. Reset it on the element.

---

## 12. Rollback

The tested pre-redesign UI is kept in `.pre-saas/`. One command:

```bash
./rollback-ui.sh
```

It restores `app.js`, `app.css` and `index.html` and prints a confirmation.
No server restart, no database change — reload the page and you are back.

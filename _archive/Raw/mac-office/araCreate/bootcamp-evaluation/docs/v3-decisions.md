# v3 — decisions log

Running list. Every decision taken with Vishnu, in order. Anything marked
**open** still needs an answer before build starts.

Companion docs: `docs/v3-restructure-plan.md` (what), `docs/v3-build-and-cutover.md`
(how and when), `docs/v3-agent-brief.md` (the instructions the dev agent works from).

---

## Direction

| # | Decision |
| --- | --- |
| D1 | Organise all of it — product flow, code structure, docs, way of working, **and the UI/UX structure** |
| D2 | **Dev and deploy only when students are not working.** No exceptions |
| D3 | The app must become a **reusable product** for future batches |
| D4 | **One build, one deploy.** No more piecemeal changes onto a live app |
| D5 | **No calendar deadlines on deploys.** A change is deployed when development and testing are fully finished — then it goes out in the next window with nobody working. Never scheduled to a date first |

## Now, while the bootcamp runs

| # | Decision |
| --- | --- |
| D6 | **Build the SQL chase-lists.** Read-only queries, CSV output — who has not handed in, who is absent, who never opened the quiz, with phone numbers. No deploy, no restart, zero risk |

## Daily survey — needed for this batch

| # | Decision |
| --- | --- |
| D7 | A survey is asked **before each day starts**, on **every day** — not just pre and post |
| D8 | Questions are **different each day**, written the evening before, like the quiz |
| D9 | **Yes / No questions only** |
| D10 | **Every student answers individually. Zero points, no timer, no right answer.** It is feedback, not a test |
| D11 | The survey is an **activity on the Open tab** — admin opens it **per venue**, by hand, like everything else. Nothing opens itself |
| D12 | For students it is **just another item in "My work"** — first in the list, tagged **Required**. The other items stay open. The five activity types are our language, not theirs |
| D13 | Loaded by **paste-many, one question per line**, same as the quiz loader |
| D14 | Results needed: **Yes/No counts per question** (split by venue) · **who answered what**, as a named list with phone numbers · **the same question tracked across days** · **on each student's profile page** |

## Deferred — built later, not now

| # | Decision |
| --- | --- |
| D15 | **Certificates** — ~~out of current scope~~ **BUILT 23 Sep.** Every student gets one (Vishnu, 23 Sep: all students, not an attendance threshold), they download their own PDF, issued by araCreate alone, content kept to one simple line. Generated once by an admin and stored on Drive, *not* rendered when a student clicks. See `2026-09-23-b-certificates.sql`, `src/routes/certificates.js`, `src/certificate/template.js` |
| D17 | **Certificates are served by this server, not Drive** — one click, and it works when Drive is down. Files live in `uploads/certificates/`, the only directory excluded from the deploy rsync `--delete`. The Drive copy is kept as the off-server backup, since `uploads/` is in neither the deploy nor the database dump |
| D18 | **No LinkedIn share button** — LinkedIn does not let any site write text into someone's post, and the only way to get the wording and picture in is a PUBLIC certificate page, which would put every student's name and programme on the open internet. Vishnu, 23 Sep: download is enough |
| D16 | **Journey page and transcript** — also deferred. When built: old CV vs new CV side by side, the 9 daily posts as a timeline, pre vs post assessment gain as a percent, and their Day 0 goal |

## UI rebuild — top-tier look

| # | Decision |
| --- | --- |
| D17 | The current UI is not good enough. Rebuild using an **open-source component library**, not hand-rolled CSS |
| D18 | Stack: **React + Tailwind + shadcn/ui**. Free, owned in-repo, fully themeable, strongest for tables and matrices |
| D19 | Theme: **araCreate brand** — Golden Sun as the single accent, araCreate colour rules, rebuilt as Tailwind tokens |
| D20 | **Light mode only** |
| D21 | **Phone-first web app.** Designed at 390px first, desktop is the bonus. Not a PWA, not offline |
| D22 | The old araCreate CSS design system is **replaced in this app only**. Nothing else uses it — no shared library, no migration |
| D23 | Built with **Vite + React, served as static files**. Backend — Node, Postgres, Caddy — untouched. Easy rollback |
| D24 | **Three screens first for approval**: Student Home, Admin Today, and the completion matrix. The other 20+ screens are built only after the look is signed off |

## Guardrails for the dev agent

| # | Decision |
| --- | --- |
| D25 | **The agent never deploys.** Local development only. No ssh to the server, no connection to the production database |
| D26 | Work happens on a **separate dev branch**, never on `main` |
| D27 | **Full local testing first.** Then Vishnu deploys, at night, when nobody is working |
| D28 | Local testing uses a **real production dump that Vishnu pulls to the Mac**. The agent never fetches it |
| D29 | The agent **reports at the end of each feature**, then stops and waits |

---

## Where that leaves the work

**Track 1 — now, no deploy.** SQL chase-lists. Read-only, usable immediately.

**Track 2 — now, one deploy when ready.** The daily survey, built on the current
app. Small and self-contained. Its UI is throwaway once v3 lands, and that is an
acceptable cost for having the data from this batch.

**Track 3 — the v3 rebuild.** Everything in the restructure plan, plus the full
React + shadcn front end. One build, one cutover, between batches.

**Deferred.** Journey page, transcript. (Certificates built 23 Sep — see D15.)

---

## Still open

1. What the survey questions are actually for — Vishnu had more to add.
2. Can a **team lead see which of their six members** finished today's work?
   Recommendation: yes for status, no for content.
3. Can a **mentor** open a student profile? Recommendation: yes, minus the daily posts.
4. What counts as **"done"** for a task with no hand-in type?
5. **Absent vs not-yet-marked** must be two different colours in the matrix.
6. Profile photos are on the server only, in no backup. Move to Drive or accept the risk.

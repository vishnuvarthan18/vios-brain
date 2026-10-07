# BUILD PLAN

**The plan of record.** Scope, stack, data model, milestones, risks.

**Revised 7 September 2026.** Scope locked by Vishnu. English only. See
[§1](#1-scope) for what was dropped and [§9](#9-decisions-on-record) for the
two that carry a cost.

| Part | Covers |
| --- | --- |
| [1 Scope](#1-scope) | In · out · the three doors |
| [2 Stack](#2-stack) | Choices and why |
| [3 Data model](#3-data-model) | Tables · status flow · grouping · permissions |
| [4 API](#4-api) | The public endpoints |
| [5 Widget](#5-widget) | States and behaviour |
| [6 Screens](#6-screens) | The dashboard, in build order |
| [7 Milestones](#7-milestones) | Five days, with acceptance |
| [8 Risks](#8-risks) | What is watched |
| [9 Decisions on record](#9-decisions-on-record) | What was dropped, and the cost |
| [10 Open decisions](#10-open-decisions) | In blocking order |

## 1 Scope

Build the working tool for B. Halle. Nothing else. No SaaS product is being
built here.

### In

- **The widget** — English only. Launcher, point at the problem, one question
  with five options in random order, optional note, thank you.
- **Screenshots** — captured automatically, shown to the tester before sending,
  with a "don't include it" option.
- **Report grid** — which pages have reported problems.
- **Report list** — the raw append-only feed.
- **Issues** — list and detail, with status, owner, priority and category.
- **Comments** — internal notes and client-visible comments, kept apart.
- **Event log** — who changed what, when. Append-only.
- **Roles** — staff, developer, client.
- **Pages, testers, assignments.**
- **String editor** — every tester-facing sentence editable in the app, no
  deploy.
- **CSV export** — of reports and of issues.

### Out — do not build

- **German, and any language switching.** English only. No locale mechanism, no
  translation, no following the site's language switcher.
- **The keyboard path to element selection.** Mouse and touch only.
- **The "this page was fine" button.** Problems only — a tester who finds
  nothing sends nothing.
- **Multiple targets per report.** One report points at one thing.
- **"Where did you look for it?"**
- **Read aloud, and the bigger-text button.**
- Signup, billing, plans, payment
- A multi-project or multi-customer switcher UI
- White-label, a public API, webhooks
- Kanban boards, sprints, time tracking, burndown charts
- Emails to testers of any kind, including reminders
- Screen recording, console log capture, network request capture
- Drawing or annotation tools
- Severity or priority questions put to the tester

### The three doors

Cheap now, expensive later. Do them even though nothing is being productised.

1. `org_id` and `project_id` on every table row. Behaves single-tenant now.
2. The widget carries a public key and fetches its own configuration. **No
   client-specific value in the widget source — not even now that there is only
   one language.**
3. `reports` are append-only; resolution state lives in `issues`.

The fourth door — strings per language — is reduced to strings from the API in
one language. The mechanism stays; the second language does not.

## 2 Stack

| Layer | Choice | Why |
| --- | --- | --- |
| Workspaces | npm, two under `src/` | One fewer tool to install |
| Web app + API | Next.js App Router | Dashboard and public API in one deployable |
| Database | Postgres — local `postgresql@17` for development, EU cloud region later | Version pinned so both behave the same |
| Data access | Drizzle, migrations in repo | Typed, reviewable |
| Auth | Staff, developer and client accounts, created by hand | **Testers never log in and never have an account** |
| Widget | Vanilla JS, zero dependencies, esbuild → one file | Runs on the client's live site. Small, defensive, boring |
| Widget hosting | Object storage behind a CDN, versioned `/v1.js` | Webflow's Assets panel rejects `.js` |
| Screenshots | `modern-screenshot` in a lazy-loaded chunk, uploaded by a self-signed URL to local disk behind a swappable interface | Never through the API. No cloud storage — Vishnu's instruction, 7 Sept. Cloud is a second implementation of the same interface |

Dashboard mutations go through server actions. Only the widget endpoints are
public.

## 3 Data model

Seven core tables — `organisations`, `users`, `projects`, `pages`, `testers`,
`assignments`, `reports` — every row carrying `org_id`.

`users.role` is `'staff' | 'developer' | 'client'`.

### Separate evidence from work

A `report` is what a tester sent. It never changes. An `issue` is what the team
works on. One issue holds one or more reports, and no report is ever hidden.

```
issues            id · org_id · project_id · page_id · ref · title · answer_id
                  group_key · category · status · priority · assignee_id
                  duplicate_of · reports_count · first_seen_at · last_seen_at
                  resolved_at · closed_at
                  unique (project_id, ref) · unique (project_id, group_key)

issue_reports     issue_id · report_id                  (many reports to one issue)

comments          issue_id · user_id · body · client_visible (default false)

issue_events      issue_id · user_id · kind · from_value · to_value
                  append-only audit; also sign-off evidence

config_revisions  project_id · config · user_id
                  so a bad string edit can be rolled back
```

### Status flow

```
new -> triaged -> in_progress -> fixed -> verified -> closed
                                     \-> wont_fix
                                     \-> duplicate
```

Only `verified`, `wont_fix` and `duplicate` close an issue. Testers never
verify — staff verify. Priority is set by the team and **never asked of the
tester**: experts disagree on severity 24–30% of the time, and severity does
not predict frequency.

### Grouping, server-side, on report insert

1. `group_key` = `page_id` + `answer_id` + normalised `target_text`
   (lowercase, collapse whitespace, trim to 120 characters).
2. An open issue with that key exists → attach the report, bump
   `reports_count` and `last_seen_at`, write an `issue_events` row.
3. No match → create the issue at `status = 'new'`, title built from the answer
   label and the element text.
4. No element (whole page) → the key is `page_id` + `answer_id` only.
5. **Never group across pages automatically.** A person can mark a duplicate.

### Permissions

| | staff | developer | client |
| --- | --- | --- | --- |
| Report grid, issues, reports, CSV | yes | yes | yes |
| Change status, assign, set priority | yes | own issues only | no |
| Client-visible comments | yes | yes | yes |
| Internal notes | yes | yes | no |
| Pages, testers, assignments, strings | yes | no | no |
| Copy tester invitation links | yes | no | no |
| Invite users | yes (owner) | no | no |

## 4 API

```
GET  /api/v1/config?key=pk_live_xxx&t=<token>   60s cache, CORS open
POST /api/v1/reports                            rate limit 60 per token per hour
POST /api/v1/uploads                            presigned screenshot upload URL
```

Four rules that do not get softened:

- **Unknown key → 404, and the widget renders nothing.** No error UI on the
  client's site, ever.
- **Invalid token → serve the config without the tester object**, store
  `tester_id = null`, widget still works.
- **URL matches no known page → not an error.** Store `page_id = null` and the
  raw URL. Testers wander.
- **Report POST fails → the tester still sees the thank-you screen.** Retry
  twice in memory.

## 5 Widget

```
idle -> pointing -> question -> detail -> consent -> sent
                        ^                                |
                        +---- "something else here" -----+
```

- **idle** — launcher, bottom right, fixed.
- **pointing** — launcher hides, a bar appears with the framing text and two
  escape buttons. `mousemove` → `elementFromPoint` → outline that element.
  Click in the **capture phase**, with `preventDefault`, `stopPropagation` and
  `stopImmediatePropagation`, or clicking a nav link navigates instead of
  selecting. **Touch: first tap highlights and asks "is that right?", second
  tap confirms.** One escape hatch: "it was the whole page". There is no
  "this page was fine" button — see [§9](#9-decisions-on-record).
- **question** — the five options in randomised order, plus "something else".
  A radio-style list, never a dropdown. One tap advances, no submit button.
  **The order shown is recorded on the report.**
- **detail** — optional textarea, then Send or Skip and send. Both submit.
- **consent** — the captured image shown with a "don't include it" option.
  **This comes after `detail`, immediately before sending, not mid-flow.** An
  earlier draft of this document put it between `question` and `detail`; the
  implementation is right and the document was wrong. Consent belongs directly
  before the act it authorises — "here is what we will send, is that all
  right?" — rather than interrupting the tester between two questions.
  Declining means no upload happens and the report is submitted with its
  `screenshot_key` pointing at nothing. That is a normal outcome, not an
  error, and is not logged as one.
- **sent** — thank you, and an offer to report something else on the page.

Every string comes from the API. Nothing hardcoded, not even error text.

## 6 Screens

In build order.

1. **Report grid** — pages down, testers across. Amber where a tester reported
   a problem, blank everywhere else, with row and column totals. The home
   screen. **A blank cell means no report, which is either "not looked at" or
   "looked at and fine" — the grid cannot tell them apart.** Do not label it
   "coverage" anywhere in the UI or the export; it is not coverage.
2. **Issues** — filter by status, page, answer, owner. Default: not closed.
3. **Issue detail** — every report beneath it with its own screenshot, the
   element text, status, owner, priority, category, comments split internal and
   client-visible, and the full event log.
4. **Reports** — the raw append-only feed with filters. The evidence view.
5. **Pages** — list, add, edit, bulk import from a URL list.
6. **Testers** — list, create, copy the invitation link. No passwords, ever.
7. **Assignments** — run the generator, then edit by hand. Three testers per
   page, spread evenly, with Home, Contact and 404 in every tester's set.
8. **Strings** — every tester-facing string, editable, with rollback.
9. **Team** — invite staff, developer, client.
10. **Export** — CSV of reports, CSV of issues.

## 7 Milestones

Five working days with an AI developer, on the condition that questions are
answered within minutes and no scope is added mid-build.

| Day | Milestone | Accept when |
| --- | --- | --- |
| 1 | **M0 Foundation** — repo, schema, migrations, seed, CI, widget size gate | Clean database migrates; seed gives a working project; CI fails a deliberately bloated widget build |
| 1 | **M1 Public API** — config, reports, validation, rate limit, grouping | Unknown key returns 404; a posted report creates an issue and an event; two matching reports give one issue with count 2 and both reports still readable |
| 2 | **M2 Widget** — all six states, on a local test page | Under 15 KB gzipped; option order differs between sessions and is stored; tap-to-confirm works; breaking the API leaves the host page byte-for-byte intact |
| 3 | **M2b Live site** — hardened on the published Webflow site | Works on a CMS product page, the 404 page, Safari and iPad; the token survives navigation without being written to the device |
| 3 | **M3 App read-only** — auth, report grid, report list, CSV | Grid matches a hand count; CSV opens clean in Excel |
| 4 | **M4 Issues and resolving** — list, detail, status, owner, comments, event log, three roles | A client login cannot see internal notes or change status; a developer can only move their own issues; **no action anywhere mutates or deletes a report** |
| 5 | **M5 Admin** — pages, testers, assignment generator, string editor, team invites | A string edited in the app changes the widget within 60 seconds with no deploy; the generator makes 147 assignments across 20 testers |
| 5 | **M6a Storage and upload** — key whitelist, self-signed upload URL, WebP verification, retention | Path traversal in all five forms rejected; an expired, tampered or reused signature rejected; a non-WebP or oversized body rejected; retention deletes files only and never a `reports` row |
| 6 | **M6b Capture, privacy and consent** — lazy-loaded chunk, stripping, consent screen | Safari produces a real image, **not the blank first capture**; a test **decodes the captured WebP and inspects pixels** to prove no `input`, `textarea`, `contenteditable` or `data-fb-block` content survives; a failed or blocked chunk still submits the report |

## 8 Risks

| Risk | What we do |
| --- | --- |
| The screenshot capture briefly attaches a clone of the page to the live document | Unavoidable — a detached node reports zero layout. **Unverified on the real Webflow site**: whether it disturbs Webflow's interactions, lazy-loading or analytics. First thing to check at M2b |
| Tests that check text and aria-labels can pass while the widget renders completely unstyled | This happened, from M0 to M6, undetected through four sign-offs. `computed-styles.spec.ts` now asserts real computed style across all five states |
| Safari's first screenshot is blank — documented across libraries, still open | Capture twice, keep the second. This is an acceptance item, not a nice-to-have |
| Cross-origin iframes cannot be captured or pointed into — YouTube, Maps, some cookie banners | Browser security, not a bug. Every tool has this limit. Do not attempt a workaround |
| Webflow class names change when a designer renames a style | The element fingerprint stores the element's **text**, not only a selector |
| Custom code runs only on the published Webflow site, never in Designer preview | Expect a publish cycle per iteration on day 3. Budget for it |
| Agent drift — an AI developer will "improve" the copy and add a kanban board | [agent-rules.md](agent-rules.md), read every session, plus a human review of every change |
| Five days assumes fast answers | Every unanswered question is dead time. This is the single biggest schedule risk and it is not a technical one |

## 9 Decisions on record

Two things were dropped that carry a cost. Recorded here so nobody
re-discovers them in three weeks and thinks it was an oversight.

**The keyboard path was dropped.** Element selection is mouse and touch only.
Consequences, stated plainly:

- **WCAG 2.1 AA cannot be claimed** for the tester interface. Do not put a
  conformance claim anywhere.
- A tester who cannot use a mouse or a touchscreen cannot file a report. If any
  of the 10–30 testers is in that position, they are excluded from the round.
- It can be added later. What it needs is not code but plain-word names for the
  parts of all 49 pages, and that was the reason it was dropped.

**The "this page was fine" button was dropped** (originally 2 September,
re-confirmed 7 September). Consequences:

- The main screen is a **report grid, not a coverage grid**. It shows where
  problems were found. It cannot evidence that a page was checked and was
  clean.
- A blank cell is ambiguous: not looked at, or looked at and fine. There is no
  way to tell from the data.
- The client sign-off export therefore proves what was found, not what was
  checked. Do not describe it as coverage to the client.
- `reports.outcome` stays in the schema but only ever holds `'problem'`. The
  column is left in place so the button can be added later without a
  migration.
- Adding it back later is about an hour of widget work.

**German was dropped.** The widget is English only. This matches the original
brief, which described the testers as German-speaking but reading English. The
locale mechanism is not built, so adding a second language later is a build,
not a configuration change.

## 10 Open decisions

In the order they block work.

1. **Does the client's Webflow plan allow custom code?** Blocks install, and
   therefore day 3. Ask Jakob.
2. **Does site-wide Webflow custom code run on the 404 page?** Blocks an M2b
   acceptance item. Five minutes: publish, visit a nonexistent URL, look for
   the script tag. The 404 is one of the 49 pages.
3. **Who sees the launcher** — gated on a valid tester token, or visible to
   everyone? Blocks the M2 launcher logic.
4. **The domain** for the API and the CDN. Blocks deployment on day 3.
5. **"Page opened" logging** — build it or not? Behavioural logging of
   identifiable people, so it needs the client's sign-off.
6. **How do we know a round is finished?** No confirmations and no reminders, so
   there is no completion signal. Manual chase, or accept partial coverage.

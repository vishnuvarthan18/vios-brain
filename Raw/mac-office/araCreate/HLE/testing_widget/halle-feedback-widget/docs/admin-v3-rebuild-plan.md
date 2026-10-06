# Admin v3 — flow simplification and UI rebuild: findings and plan

Written before implementation, as
`agent-task-admin-flow-simplify-and-ui-rebuild.md` §7 and "Report back"
require: item 7's findings, and the shape of items 4–6, for Vishnu and
Claude to sanity-check the direction before a large amount of code exists.

Status: **built, 9 September.** Signed off and implemented; §6 below records
what changed against this plan during the build. Nothing committed yet.

The real design system document arrived during the build and is at
`docs/halle-design-system-draft.md` — §3.3's reconstruction is superseded by
it, and the build used the real values (see §6.3).

Two decisions were put to Vishnu and answered (§0 below). Everything else
here is either a finding from the current code or a proposal.

---

## 0 Decisions taken

1. **Removing a tester** — revoke the link, keep the tester row and their
   reports. The old link stops working immediately; past reports keep their
   attribution. This is also the only option the database permits without
   weakening the append-only rule (§1.3).
2. **The new home screen** — an overview of counts above, with Queue
   remaining the working triage list. Each count links into the
   already-filtered Queue. Queue and Tracked items are not rebuilt.

---

## 1 What the code actually does today

Three findings materially change the task as written. Each was verified
against the code and, where it concerns the database, against a live
Postgres.

### 1.1 There is no page-assignment restriction to remove

Item 2 asks to "remove whatever server-side check currently restricts a
tester's report to only their assigned page(s) (if one exists)". **One does
not exist.** A tester's token already works on every page of the site.

- `app/api/v1/reports/route.ts` — the gate sequence is: body schema →
  project public key → tester token (soft: an invalid token still stores
  the report with `tester_id` null) → rate limit → insert. `assignments`
  is never queried, never imported.
- `lib/db/submit-report.ts:49` resolves the page by URL
  (`find_page_by_url`) purely to record `page_id`; no match is explicitly
  not an error, because "testers wander".
- `lib/db/testers.ts:20` `find_tester_by_token` joins nothing and filters
  on nothing but the token.
- `app/api/v1/config/route.ts` never receives the current URL, so it
  cannot gate on the page even in principle.
- The widget's launcher gate is `if (!config.tester) return;`
  (`src/widget/src/app.ts:155`) — the presence of a resolved tester, not
  an assignment.

So item 2 is **not** a behaviour change to the tester's experience. It is
the removal of a planning-and-counting feature that never restricted
anything. That is worth stating plainly, because it means item 2 carries
far less risk than it reads, and no tester-facing behaviour needs
re-testing for it.

What assignment is actually used for today, all display or planning:
`lib/db/assignment-generator.ts` (the whole file), the five files under
`app/app/admin/assignments/`, the nav link, `tester_progress` in
`lib/db/testers.ts`, the `assignedTotal`/`completed`/`assignedPages`
fields on the config response — and the report grid, §1.2.

### 1.2 Removing assignment blanks the current home screen

`app/app/page.tsx` — the `/app` home screen — is a pages × testers matrix
built **entirely** from `assignments` (`lib/db/grid.ts:45`: rows and
columns both come from `assignments innerJoin pages innerJoin testers`).
With assignment gone, `grid_pages` and `grid_testers` are both empty, the
screen renders its `"No assignments yet."` empty state, and every real
report becomes invisible on the home screen.

So items 2 and 4 are **coupled**: item 2 does not merely delete a nav
entry, it removes the home screen's reason to exist. This is the main
reason item 4 has to land in the same milestone rather than after it.

The grid also documents its own core flaw, in its own footnote: "A blank
cell means no report was filed — that page was either not looked at, or
looked at and found fine. This grid cannot tell the two apart." Rebuilding
rather than rebasing it is the right call.

### 1.3 A tester cannot be hard-deleted (probed, not assumed)

Item 3 says to decide "what happens to that tester's existing reports"
and not to guess. The database has already decided: two of the three
possible behaviours are impossible.

Probed against the test database:

| Attempt | Result |
| --- | --- |
| `update reports set tester_id = null` | **BLOCKED** — `reports is append-only except for status: UPDATE on reports is not permitted` |
| `delete from testers` (tester has reports) | **BLOCKED** — `violates foreign key constraint "reports_tester_id_testers_id_fk"` |
| `update testers set token = (secret removed) | **ALLOWED** |

`reports.tester_id` is nullable, but the append-only trigger
(migration 0001, narrowed in 0005 to permit only a `status` change)
forbids writing it. So "delete the tester and keep the reports
anonymised" would need a migration relaxing that trigger — which
`agent-rules.md §1.1` forbids and which would lose attribution
permanently.

Hence decision 0.1: **remove = revoke**. Implementation shape:
- Rotate `testers.token` to a fresh value, so the old invitation link
  resolves to no tester and the widget's launcher stops appearing.
- Add `testers.revoked_at timestamptz` (nullable) so admin can show the
  tester as revoked rather than silently listing a dead link.
- `find_tester_by_token` must additionally reject a tester with
  `revoked_at` set, so that even the newly-rotated token is inert.
- Reports keep `tester_id`, so the queue still shows who filed what.

### 1.4 Item 1 is a swap, not a removal

Item 1 reads as "remove the email field", but the form today asks for
**email only and has no name field at all**
(`admin/testers/create-tester-form.tsx:30`). The displayed name is
auto-numbered — `next_label()` in `lib/db/testers-admin.ts:26` produces
"Tester 01", "Tester 02".

So item 1 is: **add a name field, remove the email field**, and stop
auto-numbering. That is a genuine improvement for the admin triaging
reports — "Tester 03" tells them nothing about who filed a report,
whereas a real name does.

`testers.email` is confirmed safe to drop outright: nullable, no unique
constraint, no index, read in exactly one place (the admin table cell),
never used for notifications, login, identification or deduplication —
and "emails to testers of any kind" is listed under *Never in scope* in
both `agent-rules.md` and `build-plan.md`, so no future feature
resurrects it. It needs migration `0006` with a single
`ALTER TABLE "testers" DROP COLUMN "email";`.

A name should be **required** (it is the only identifying field left) and
**not** unique — two real testers can share a first name, and uniqueness
on a display name is the kind of constraint that produces a baffling
error at the worst moment. `next_label()` is deleted.

---

## 2 Item 7 — walking the flow as each person

Scenarios walked end to end against the current code, per audience. Only
problems not already covered by items 1–6 are listed; each is marked
**[flag]** (asking, per §7) or **[fix]** (proposed as part of this work).

### The tester (a non-technical person, e.g. Vishnu's own live test)

1. **The token silently dies on any fresh entry to the site.** [flag]
   `src/widget/src/token.ts` carries `?t=` forward by rewriting
   same-origin links and guarding the URL with `replaceState`. That covers
   clicking around the site. It does **not** cover: opening a bookmark,
   arriving from Google, typing a URL, following a link someone sent them,
   or reopening the browser next day. In every one of those the widget
   simply is not there — no launcher, no message, nothing to explain why.
   For a tester told "use this link to report problems", the tool
   vanishing is indistinguishable from the tool being broken.
   This gets *more* likely once assignment is removed and testers are
   encouraged to roam the whole site over days rather than working a
   short assigned list in one sitting.
   **Not covered by items 1–6.** Options: accept it and make the
   invitation email/message tell them to always start from the link;
   or let the widget remember the token for the session (which
   `agent-rules.md §1.3` currently forbids — "never written to the
   device"); or show a small "your testing link has expired, open it
   again" hint when a `?t=` was once seen but is now absent. Asking
   rather than deciding: the rule against device storage is explicit.
2. **Nothing confirms the report actually arrived.** The thank-you screen
   appears immediately, before the POST resolves — deliberate, and right.
   But if the request then fails, the tester has been told "thank you,
   that really helps" for a report that does not exist. [flag] Low
   frequency, and the alternative (making them wait) is worse. Flagging
   only so the trade-off is a decision rather than an accident.
3. **After removing assignment, the tester has no sense of scope.**
   Today the config sends `assignedTotal`/`completed`; the widget never
   renders them, so nothing visible changes when they go. Worth stating:
   removing assignment costs the tester *nothing they can currently see*.
   No replacement progress UI is needed. [no action]

### The admin triaging reports

4. **"Tester 03" is unidentifiable.** Covered by item 1 — noting it here
   because it is the strongest argument for making the name required.
5. **A revoked tester's reports must stay legible.** Decision 0.1 keeps
   attribution, so this is handled. [no action]
6. **The queue has no bulk action.** 49 pages × several testers will
   produce runs of near-identical reports (one broken shared header
   generates one report per page it appears on). Marking them one at a
   time is the obvious first thing to become tedious. **Not covered by
   items 1–6.** [flag] Not proposed for this milestone — flagging as the
   likely next request once the rebuilt UI is in real use.
7. **Deleted reports are unrecoverable from the UI.** `status='deleted'`
   is a soft delete (the row survives), but there is no screen that lists
   deleted reports, so a mis-click is not undoable by the admin. [fix]
   Cheap to add as a filter on the rebuilt dashboard/queue rather than a
   new screen.
8. **The page list holds 3 placeholder pages, not the real 49.** Already
   noted in the task doc as Vishnu's data task. The rebuilt UI should
   therefore not *look* broken with 3 pages, and should not assume 49.
   [no action, design constraint]

### B. Halle staff (reading, not triaging)

9. **There is no read-only view.** One login for everyone, no roles
   (`admin-v2-spec.md §1.4`), so anyone who can see reports can also
   change their status and remove testers. Deliberate for now — one
   person is running this. [flag] Only becomes a problem when B. Halle
   staff get their own logins; worth knowing before that happens, not
   worth building now.
10. **CSV export is the only way data leaves the app.** Now that it
    strips the tester token (previous milestone), it is safe to circulate.
    [no action]

---

## 3 Items 4–6 — the shape proposed

### 3.1 Navigation (item 5)

Left sidebar, three groups, replacing the current flat row of eight
undifferentiated top-bar links:

```
TRIAGE          Overview        <- new home, /app
                Queue           <- unchanged
                Tracked items   <- unchanged

LIBRARY         Reports         <- unchanged (all reports, filterable)
                Pages

SETTINGS        Testers
                Wording         <- renamed from "Strings"
```

"Assignments" is gone (item 2). "Strings" becomes "Wording" — it edits
the words the tester sees, and "strings" is a programmer's word on a
screen aimed at whoever runs the testing round.

### 3.2 Overview screen (item 4)

Per decision 0.2. Three bands:

1. **Status counts** — new / bugs / fixed, as large numbers. Each is a
   link into Queue or Reports already filtered to that status.
2. **By page type** — the five templates from `admin-v2-spec.md §6`
   (Home, Product Category, Product Detail, Contact, 404) in that fixed
   order, each with its new-report count and a proportional bar. Keeps
   the template-grouping concept item 4 asks to retain, and answers "which
   kind of page is generating problems" — which the old grid could not.
3. **Latest reports** — the most recent handful, each linking to its
   picture viewer.

Empty states matter here and are part of the work, not an afterthought:
with 3 placeholder pages and few reports this screen must read as "nothing
has come in yet", not as broken.

Data needs one new query module (`lib/db/overview.ts`) built on `reports`
and `pages`. `lib/db/grid.ts` and its test are deleted with the grid.

### 3.3 Design system (item 6)

The design system document the task refers to
(`claude/halle-design-system-draft.md`) **does not exist in this repo** —
same missing `claude/` directory as the four documents referenced by the
previous task. Its values are recoverable, though, from
`COMMIT_MSG_rebrand-admin.txt` and the CSS that commit produced, and
that is what §3.3 is based on. **If the real document still exists
somewhere, hand it over before this item is built** — it is the one item
here that cannot be done properly from the repo alone, since "matches it
exactly, not approximately" needs the actual source.

What the audit found, which explains "still looks messy" better than
"needs more polish" does:

- **No spacing or typography tokens exist at all.** `globals.css` defines
  colours and two radii, and nothing else. Every spacing value is written
  literally at each use site.
- Spacing values themselves are *already* on a 4/8/16/24/32 rhythm — so
  the mess is not rogue numbers, it is that the rhythm is convention only,
  with nothing to enforce it and no name to reach for. The next edit
  breaks it silently.
- **Typography is genuinely unsystematised**: `0.85rem`, `0.9rem`, `1rem`
  used interchangeably for what is visually the same "small text", mixed
  with `px` elsewhere. Three sizes where there should be one token.
- The widget is tighter (a clean 4/8/16 rhythm) but has three off-scale
  strays: `16.5px`, `10px`, `18px`. Item 6 asks to double-check the
  widget; that is the whole finding.

Proposed: add `--space-*` and `--text-*` scales as real tokens, snap the
three near-identical font sizes onto them, fix the widget's three strays,
then rebuild each admin screen against the tokens. Concretely this is a
token layer plus a per-screen pass, not a rewrite of the CSS.

---

## 4 Order of work, once signed off

Each step verified before the next; commits per step, none without asking.

1. Migration `0006`: drop `testers.email`, add `testers.revoked_at`.
2. Item 1 — name in, email out, `next_label()` deleted. Tests updated
   (11 call sites pass `{ email: null }`; one asserts email is stored and
   is deleted outright).
3. Item 3 — revoke action with a confirmation step;
   `find_tester_by_token` rejects revoked testers. New test: a revoked
   token resolves to nothing and its reports keep their attribution.
4. Item 2 — delete the assignments feature (generator, 5 UI files, nav
   link, `tester_progress`, the three config fields, the unused widget
   types) and update `admin-v2-spec.md` §§41-51/59/80/89 plus
   `widget-v2-spec.md` §§42-51. New test: a report for a *known but
   previously-unassigned* page is accepted — the regression guard for the
   new rule, which no current test covers.
5. Items 4/5/6 — token layer, then sidebar shell, then Overview, then a
   consistency pass over Pages / Testers / Wording / Reports / Queue /
   Tracked.

Rough size: steps 1–4 are small and mostly deletion (one migration, one
form, one action, ~10 files removed). Step 5 is the bulk of it.

## 5 What I need before building

1. **Sign-off on §3's direction** (nav structure, Overview shape, the
   design-token approach).
2. **The real design system document**, if it exists — see §3.3.
3. **Answers, or an explicit "leave it", on the four [flag] items in §2**:
   the token dying on fresh entry (§2.1, the one with real tester impact),
   the thank-you-before-confirmation trade-off (§2.2), queue bulk actions
   (§2.6), and read-only access for B. Halle staff (§2.9).


---

## 6 What was built, and where it differed from this plan

Written after the fact. Items 1-5 of the work order in §4 are done; the three
minor findings in §2 (report confirmation, queue bulk actions, read-only staff
access) were explicitly deferred to a later round by Vishnu and are NOT built.

### 6.1 Decisions confirmed by Vishnu

- **Removing a tester** — revoke, per §0.1. Built exactly as described.
- **The home screen** — overview above, Queue stays the worklist, per §0.2.
- **The expired-link problem (§2.1)** — Vishnu's decision: do *not* quietly
  remember the tester; show a clear message instead. Built as a docked notice
  with editable wording (see §6.2).

### 6.2 The expired-link notice

The half of this that needed care was the server contract, not the notice.
The config endpoint previously returned the same shape for "no token" and
"token that did not resolve", so the widget could not tell an ordinary
visitor from a tester whose link had died. It now returns
`tokenExpired: true` for the second case, and:

- **No token at all still renders absolutely nothing.** This is load-bearing
  and separately tested: the widget sits on the client's real public site, so
  an ordinary visitor must never see any of it.
- "Removed" and "never valid" are deliberately not distinguished — both mean
  the same thing to the person holding the link, and distinguishing them
  would leak whether a token ever existed.
- A `tokenExpired` response is `private, no-store`. Caching it publicly (the
  old condition keyed on whether a tester was *found*) would let a shared
  cache serve "your link has expired" to a tester whose link is fine — a
  worse failure than the silent one being fixed. Caught while writing the
  endpoint, and now has its own test.
- Both strings are editable in the Wording screen, like every other
  tester-facing string.

### 6.3 The design system (item 6)

Using the real document rather than the reconstruction changed the work
materially: it defines an **eight-step type scale** (42/26/24/22/20/18/16/12px),
a **spacing scale** (8·16·24·32·40·48·64·80, "nothing in between"), line
heights, letter spacing, and component sizes — none of which §3.3 could have
guessed.

What that exposed, beyond §3.3's findings:

- **No heading on any screen was a design-system size.** `globals.css` never
  set `font-size` on h1-h6, so every heading fell back to the browser's
  defaults (32/24/18.7px at this base). Now on the scale.
- **Three font sizes were off the scale entirely** — `0.85rem` (13.6px) and
  `0.9rem` (14.4px) are not scale values, and the document's rule is "no
  custom/odd sizes, round to the nearest size above". All ten uses now
  resolve to one token.
- **`color-scheme: light dark` was a latent bug.** The palette is light-only,
  so a viewer whose OS prefers dark got dark form controls and scrollbars
  against a light design. Now `light`, with a note that a real dark palette
  is a design decision the document does not yet make (§10 "Still missing").
- Spacing values were already on-rhythm but written literally at each use
  site — now tokenised, so the rhythm is enforced rather than coincidental.
  Four sub-scale values remain deliberately (a `-1px` border overlap, a 4px
  badge inset, two 2px gaps): the scale governs layout, not hairlines.

### 6.4 Problems only visible once rendered

Screenshotting each rebuilt screen caught four things the code and a passing
test suite did not:

1. **The Overview's four numbers did not add up** — 64+29+28 against a total
   of 149, because closed and deleted reports had no tile. The total tile now
   says what the remainder is.
2. **"20 of 49" on the template bars** read as "20 of 49 pages"; 49 was
   actually every report ever filed for that template. Removed.
3. **Queue tables sized to their content**, leaving them at a third of the
   column with the Bug/Delete buttons spilling outside the table border — and
   those two buttons, both 12px-radius, visually overlapped. Tables are now
   full-width with real gaps between row actions.
4. **Bare `<p>` and `<h1>` elements** inherited browser margins, so counts
   floated in large gaps and "Testers"/"Add a tester" collided. Every screen
   now uses the same `.page` / `.page-head` structure.

### 6.5 Two bugs the build caught in passing

- **A client component importing from `lib/db/`** pulled the postgres driver
  (and `node:fs`, `perf_hooks`) into the browser bundle. `next build` fails on
  it. The constant moved to a driver-free module, mirroring what
  `lib/page-types.ts` already exists to solve.
- **Two new strings were not editable in admin.** The Wording screen has an
  explicit field list, and nothing checked it against `DEFAULT_STRINGS` — so
  a string could be shown to testers with no way to change it, silently
  breaking that screen's entire promise. Now guarded by a test in both
  directions (missing fields, and fields for strings that no longer exist).

### 6.6 Removing assignment: smaller than it looked

Confirmed during the build, as §1.1 predicted: there was never any
enforcement to remove. The deletion covered the generator, five admin UI
files, the nav entry, `tester_progress`, three config fields, the unused
widget types, the old report grid, the `assignments` table (migration 0008)
and 23 tests. No tester-facing behaviour changed.

The dev fixture did need rewriting — it generated its reports *from* the
assignments table, so it now builds them directly from testers x pages,
still index-based so `make db-fixture` stays reproducible.

A regression guard was added where none existed: a report on a **known page
the tester was never assigned to** is accepted, with the page still recorded.

### 6.7 Verification

378 web tests (37 files), 44 widget tests, lint and typecheck clean,
production build succeeds, widget size budget unchanged. Migrations 0006-0008
applied to dev and test databases. Every rebuilt screen was screenshotted and
looked at, and the expired-link notice was screenshotted on a plain host page
(added as `tests/widget/host-page-plain.html`, since the existing host page is
deliberately hostile and useless for looking at anything).

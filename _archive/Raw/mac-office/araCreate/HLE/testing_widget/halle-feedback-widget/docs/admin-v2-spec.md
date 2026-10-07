# ADMIN / BACKEND v2 — SIMPLIFICATION

**Decided by Vishnu, 8 September 2026.** This replaces the roles, permissions,
assignment and multi-step workflow described in `build-plan.md` and built in
milestones M3–M5. Read with `widget-v2-spec.md`, which drives the report
payload this depends on, and with `v2-build-plan-for-agent.md`, which turns
both into milestones and settles anything either of them leaves open.

---

## 1 The flow

```
Tester sends a report
        |
        v
     Queue  ------ Delete -----> gone
        |
      Bug
        |
        v
   Tracked item
        |
   Fix it   Close it
```

1. Every report a tester sends lands in a queue. Nothing happens to it
   automatically.
2. From the queue, there are exactly two actions: **Bug** (it becomes a
   tracked item) or **Delete** (it is gone, no further trace needed).
3. A tracked item has exactly two actions: **Fix it** or **Close it**. No
   further states, no assignment.
4. **One login for everyone.** Whoever logs in sees the picture and the full
   technical detail (`meta` from `widget-v2-spec.md §5` — browser, screen,
   console errors, the lot) on every report. Nothing is hidden by role,
   because there are no roles.

## 2 What this removes — decided knowingly, 8 September

**Correction, same day:** the first draft of this document told the agent to
remove "the assignment generator" as part of this simplification. That was
wrong — there are two unrelated things both called "assignment" in this
project, and they got conflated. Fixed below before this went anywhere near
the agent.

- **Issue assignment** — giving a tracked bug to a specific developer to fix.
  This is what today's decision removes, along with roles.
- **Tester-page assignment** — deciding which of the 49 pages each tester is
  asked to check. This is a completely different feature, part of running
  the test round itself, and **was not touched by this document**. The
  testers table, and whatever assigns pages to testers, stay.

  > **SUPERSEDED, 9 September (evening).** Tester-page assignment has since
  > been removed as well —
  > `agent-task-admin-flow-simplify-and-ui-rebuild.md` §2 reverses the
  > decision recorded above. A tester's invitation link now works on **every**
  > page of the site; there is no assigned set, no assignment generator and no
  > assignments table (migration 0008 drops it). Reports still record which
  > page they came from, via `reports.page_id` resolved by URL — that is
  > independent of assignment and unchanged.
  >
  > Worth knowing if you are reading this for history: assignment never
  > *restricted* anything on the server. The report endpoint never consulted
  > the assignments table, and the widget's launcher was gated on a resolved
  > tester token, not on an assignment. It only ever fed a planning generator,
  > an admin list and the old report grid.

The app already built (M3–M5) has three roles (`staff` / `developer` /
`client`), permissions per role, issue assignment, a team list,
duplicate-linking between reports, categories, and a multi-step issue
transition table (new → in progress → fixed → and other states) with an event
log and comments.

**All of it except the testers table is being
stripped out**, not left dormant. (As first written this sentence also spared
tester-page assignment; that has since been removed too — see the superseded
note above.) This is a deliberate
choice — the alternative (leave it in place, unused) was offered and turned
down. Reasons on the record, so this doesn't get treated as an oversight
later:

- One person is running this for now. Roles and assignment solve a
  multi-person problem that does not currently exist.
- Less code in the app to maintain while the v2 widget changes are still
  being built.

**Consequence for `session-handover.md` and `PROJECT-INDEX.md`:** both still
list "three roles" as a decision on record. That line is now stale — same
kind of stale-doc problem this project has been burned by twice before
(Better Auth, and the screenshot milestone state — which went stale again on
8 Sept and had to be corrected a second time, see
`v2-build-plan-for-agent.md §0.1`). It should be corrected there, not just
here, before an agent reads the old line and builds against it.

## 3 What stays

- **Testers.** ~~And tester-page assignment~~ — see the superseded note in
  §2: assignment was removed on 9 September, and a tester's link now works on
  every page of the site. The testers table itself stays, and gained a
  `revoked_at` column (removing a tester revokes their link rather than
  deleting rows their reports point at).
- Report storage — the report itself, its `comment`, `mode`, `target_*`,
  `markup`, and `meta` (per `widget-v2-spec.md §5`).
- The picture. Storage, signed upload and the 180-day retention sweep are
  **already built** (M6a). What is still missing is the **viewer** — the
  single biggest gap in the admin app, and it does not go away with this
  simplification. See §5 and `v2-build-plan-for-agent.md §5.3`.
- CSV export — simplified. It no longer needs a role column or an assignment
  column, since neither exists.
- The tester-facing string editor and the page list (M5) — unaffected by
  this document, they don't touch roles. **One exception:** the four consent
  strings are removed with the consent screen, `widget-v2-spec.md §11`.
- Login itself — still needed, just one kind, not three.

## 4 Report lifecycle, plainly

| State | Set by | Meaning |
| --- | --- | --- |
| (none — sits in the queue) | automatic, on arrival | Not yet looked at |
| `deleted` | Delete action | Not a real report — spam, test, mistake. No further trace needed |
| `bug` | Bug action | Confirmed real, now a tracked item |
| `fixed` | Fix it | Done |
| `closed` | Confirm fix | A stakeholder checked the fix on the site. Only reachable from `fixed` (5 Oct 2026, see below) |

**Corrected 5 October 2026 (Vishnu).** `closed` does not mean "not being
fixed": it is the step after `fixed`, when a stakeholder has checked the fix.
The steps are now in order — Processing (`bug`) → Fixed → Closed — and the
server refuses a step out of order (`move_bug`, lib/db/report-status.ts): a
bug cannot be closed before it is marked fixed, and a failed check or a
closed bug that comes back returns to Processing. Each report shows a step
line and who acts next.

No transition table, no permissions check on who can move a report between
these — there is one login, so anyone who can log in can do any of these.

## 5 Resolved since this document was first written

- **What names a report in the list:** page name + number. Example:
  "Contact page #14".
- **Grouping for screenshot-mode reports:** not grouped. Every report is its
  own item, even two on the same page. This does not conflict with §6 below
  — §6 is about how the *list* is organised for browsing, not about merging
  reports into each other.
- **A report can arrive with no picture.** Not because the tester declined —
  that choice is removed (`widget-v2-spec.md §11`) — but because a capture
  can fail and a failed capture must never block a report
  (`agent-rules.md §1.11`). Both lists and the viewer must show such a
  report normally, with a plain "no picture was taken" placeholder rather
  than a broken image or an empty row.

## 6 Page templates — grouping the list, decided 8 September

The 49 pages fall into 5 templates. This is not new — it was worked out
earlier in the project (`feedback-questions-per-page.md`) for a different
reason (the old 227-checkbox spec needed different questions per template)
and holds up unchanged for this purpose:

| Template | Pages |
| --- | --- |
| Home | 1 |
| Product Category | 6 |
| Product Detail | 40 |
| Contact | 1 |
| 404 / Not Found | 1 |

**Decision:** the exact page stays on every report — which of the 40
products, specifically — because that is what a developer needs to
reproduce the bug. The template is an added way to **organise the list**,
not a replacement for the exact page. In practice: the report list groups or
sorts by template first, so all the Product Detail bugs sit together and a
template-wide problem (a broken price table on every product page) is
obvious at a glance, rather than looking like 40 unrelated one-off reports.

**Build note carried over from the earlier research, still true:** a page's
template cannot always be told from its URL alone — product slugs don't
reliably match product names, and both languages (if German ever comes back)
share the same slugs. Read the template from a tag already set on the page in
Webflow; fall back to the URL pattern only if that tag is missing.

**Schema consequence:** the pages table needs a `template` column (one of the
5 values above). Every report already links to a page, so the report
inherits its template through that link — no new column needed on `reports`
itself.

## 7 Still open

1. **The picture viewer.** Now designed — `v2-build-plan-for-agent.md §5.3`.
   No longer open.
2. **Filters and search** on the queue and the tracked-item list — by page,
   template, mode, date, state. Now designed —
   `v2-build-plan-for-agent.md §5.4`. No longer open.

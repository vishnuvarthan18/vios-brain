# QUALITY GATE

**A milestone is not done because the tests are green. It is done when
deliberate attempts to break it failed.**

This exists because a checklist you grade yourself against is not a standard.
Every item below is a command, a test or a proof — not a judgement. Run the
whole gate before each commit and record the result in
`docs/overnight-log.md`.

| Part | Covers |
| --- | --- |
| [1 Mutation proof](#1-mutation-proof) | Break it on purpose, prove a test screams |
| [2 Attack list](#2-attack-list) | The eleven holes to hunt, per milestone |
| [3 Concurrency](#3-concurrency) | The races that silently corrupt data |
| [4 Loops and runaway code](#4-loops-and-runaway-code) | Bounded everything |
| [5 Adversarial review pass](#5-adversarial-review-pass) | Read it as an attacker, not an author |
| [6 The gate](#6-the-gate) | What must be true to commit |

## 1 Mutation proof

Test count proves nothing. A test that cannot fail is decoration.

For every safety-critical rule, **introduce a real violation, run the suite,
confirm it fails, then revert.** You already did this in M0 and it found two
escapes in your own guard. Do it every time, for each of these:

| Rule | The violation to introduce |
| --- | --- |
| Reports append-only | Export a `delete_report` that actually deletes |
| Tenant scope | Drop the `project_id` term from one scoped query |
| Status transitions | Allow `new -> closed` directly |
| Client cannot see internal notes | Return internal comments to a client session |
| Developer moves only own issues | Drop the assignee check |
| Client cannot change status | Remove the role check on the status action |
| Client comment forced visible | Let a client post `client_visible: false` |
| CSV quoting | Stop escaping the embedded double quote |
| CSV formula injection | Stop prefixing a leading `=` |
| Storage key validation | Accept a key containing `..` |
| Config validation | Save a config missing a required string |
| Widget size gate | Pad `v1.js` past 15 KB |

For each: record the violation, the test that caught it, and confirm the
revert is clean. **If a violation does not break any test, the test suite is
wrong — write the missing test before moving on.** That is the finding, not
an inconvenience.

## 2 Attack list

Hunt these deliberately per milestone. For each, either a test proving it is
closed, or a line in `docs/blocked.md` saying it is open and why.

1. **Authorisation bypass** — for every action, attempt it as each of the
   three roles and as no session. Nine combinations minimum per action.
   Server-side only; a hidden button is not a permission.
2. **Object reference across tenants (IDOR)** — take a valid id from one
   project and use it in every route and action of another. Must 404, not 403,
   and must not confirm the row exists.
3. **Unsafe query construction** — no string interpolation into SQL anywhere.
   Grep for template literals reaching `sql`/`execute`. All parameters bound.
4. **CSV injection** — a note beginning `=`, `+`, `-`, `@`, tab or CR must be
   neutralised in both exports.
5. **Path traversal** — storage keys with `..`, absolute paths, URL-encoded
   traversal, and a null byte. All rejected before touching the filesystem.
6. **Unvalidated input reaching the database** — every route handler, server
   action and file read parses with zod first. A malformed body is a 400 that
   writes nothing.
7. **Error leakage** — force a database error and confirm the client sees a
   generic message, never a stack trace, SQL, a file path or a column name.
8. **Session forgery** — tamper with one byte of the cookie signature, expire
   it, and use a cookie for a user who has since been disabled. All three must
   fail closed.
9. **Login brute force** — the lockout actually triggers, and the error text
   does not reveal whether the email exists.
10. **N+1 queries** — count the queries behind the report grid with 20 testers
    and 49 pages. It must be a bounded number, not one per cell. State the
    number in the log.
11. **Mass assignment** — a form posting extra fields (`role`, `org_id`,
    `status`) must not have them applied. Whitelist, never spread.

## 3 Concurrency

These are the failures that leave bad data behind and no error anywhere.

- **Two identical reports at once.** Fire 20 simultaneous POSTs with the same
  `group_key`. Assert **exactly one** issue exists afterwards, with
  `reports_count` 20 and 20 rows in `issue_reports`. The
  `unique (project_id, group_key)` constraint means the naive
  read-then-insert loses this race — use an upsert or catch the unique
  violation and retry the attach. Prove it under real concurrency, not in
  sequence.
- **Two reports racing the ref counter.** Assert no duplicate `ref` and no
  gaps that indicate a lost update.
- **Partial writes.** Force a failure between the report insert and the issue
  attach. Assert the whole thing rolled back — no orphan report, no issue with
  a wrong count. Every multi-write operation is in one transaction.
- **Assignment generator run twice at once.** No duplicate assignments.
- **Two string-editor saves at once.** Both revisions recorded; the config
  matches one of them, not a merge of both.

## 4 Loops and runaway code

- Every retry has a hard cap and a backoff. No unbounded `while`.
- Every outbound request and every filesystem call has a timeout.
- No recursion without a depth limit, especially in DOM walking and selector
  building in the widget.
- The widget's link-rewriting MutationObserver must not react to its own
  mutations — prove no feedback loop by counting observer callbacks on a page
  that adds links.
- The retention job processes in bounded batches and cannot spin.
- No polling anywhere. Nothing that runs "until done" without a ceiling.
- Grep the diff for `while (true)`, `for (;;)`, `setInterval` and unbounded
  recursion before each commit.

## 5 Adversarial review pass

After a milestone is written and green, **review it as though someone else
wrote it and you are being paid to find the flaw.** Not a re-read — a hunt.

Fix the standard of proof: for each file changed, name the one input that
would break it. If you cannot think of one, you have not looked hard enough,
because there is always one.

Specifically ask:

- What happens on the empty case? Zero pages, zero testers, zero reports, a
  tester with no assignments, an issue with no reports.
- What happens at the boundary? 1 character, 120 characters, 121 characters,
  the exact rate limit, exactly 90 days.
- What happens with the wrong type? A string where a uuid is expected, `null`,
  an array, an object, a number as a string.
- What does a second user, mid-action, break?
- Which of these lines would still be here in a year, and which is a hack that
  will rot? Say so plainly in the log rather than leaving it for someone to
  discover.

Write the findings to `docs/overnight-log.md` **including the ones you fixed**.
A milestone whose review found nothing is a review that did not happen — say
that too, honestly, rather than reporting a clean sweep.

## 6 The gate

All of this, before every commit:

```
make lint          green
make build         green
make test          green
make test-widget   green
make size          green, both budgets
```

Plus:

- Every §1 mutation introduced, caught, reverted — recorded.
- Every §2 item either tested closed or logged open.
- Every §3 concurrency test written and passing.
- §4 greps clean.
- §5 review done and its findings written down.
- No `TODO`, `FIXME`, commented-out code, or `console.log` in shipped paths.
- The working tree clean after the commit.

**If the gate does not pass, do not commit that milestone.** Log why, leave it
in a clean state, and move to the next one. An honest red is worth more than a
green that was arranged.

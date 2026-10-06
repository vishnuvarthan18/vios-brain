# Merge debt

Edits that are **correct on their own branch and wrong only once branches
meet**. None of them belongs to any lane's queue, which is exactly why they get
lost: every lane's tests pass, every lane is done, and the defect appears at the
merge with nobody assigned to it.

**The rule: the lane that creates the debt records it the moment it writes the
code, not at merge time.** By merge time the reason has been forgotten and only
the symptom is left.

One row per item. Tick it when the merge that resolves it has landed **and been
verified**, not when it is queued.

| # | Debt | Created by | Resolved at | Status |
| --- | --- | --- | --- | --- |
| M1 | `scripts/chase/run.sh` needs a line for query 10 | Lane A | `survey` + `chase-lists` → `main` | OPEN |
| M2 | `tests/gates.js` takeover | Lane A wrote it, Lane B owns it | `survey` → `main`, then `v3-dev` rebase | OPEN |
| M3 | `CROSS JOIN (VALUES ('ECE'),('EEE'))` sites in `scripts/chase/` | pre-dates the split | Phase B2 | OPEN |
| M4 | Unreachable survey branch in `isOpenFor()` — B4 must not carry it forward as if it were the gate | Lane A | Phase B4 | OPEN |

---

## M1 — `run.sh` does not know about query 10

**What.** `scripts/chase/10-no-survey-answer.sql` exists on `survey`.
`scripts/chase/run.sh` exists on `chase-lists` and lists nine queries. Neither
branch is wrong by itself: the query cannot run on `chase-lists` because the
survey tables are not there, and `run.sh` cannot list a file that does not
exist on its own branch.

**The failure if missed.** `run.sh` silently produces nine CSVs instead of ten.
No error, no empty file — the tenth list simply is not there, and the person
chasing a room has no idea a list is missing. This is the worst shape of bug in
the chase set, because its whole job is to be complete.

**The fix, at merge.** One line in `run.sh`, after query 09:

```sh
run_one 10-no-survey-answer.sql        10-no-survey-answer     "10 no survey answer"
```

Then run `./scripts/chase/run.sh 1` and count **ten** CSVs plus the flags file.

---

## M2 — `tests/gates.js` was written by Lane A and is owned by Lane B

**What.** Lane A wrote `tests/gates.js` and its `Makefile` line before
`docs/lanes.md` assigned that file to Lane B. It is committed and pushed on
`survey`. Lane A has not touched it since and will not. `v3-dev` was cut from
`cb7ac5d`, which predates it, so Lane B does not have the file.

**The failure if missed.** Lane B writes a second gate lint. Two files with
overlapping greps and different budgets, and the one that fails first wins
whichever argument. Or worse, the budgets disagree and someone lowers the
wrong one, which quietly unpins a gate.

**The fix, at merge.** Per Vishnu's ruling: Lane B takes Lane A's file as the
starting point and does not write a second. The four budgets stay as
calibrated, and the survival split — which phase clears each budget — is the
part worth keeping. If Lane B has already written one, keep both sides and
reconcile only the numbers.

The budgets as calibrated on 19 Sep:

| Budget | Today | Cleared by |
| --- | --- | --- |
| activity type gates, server-side | 17 | B4, by hand |
| venue literals, server-side | 13 | B2, by hand |
| venue literals, `app.js` | 24 | C1/C2, by deletion |
| page routes, `app.js` | 10 | C2, by deletion |

---

## M3 — venue literals in the chase scripts

**What.** `scripts/chase/` on `chase-lists` hard-codes both venues in several
places, mostly as `CROSS JOIN (VALUES ('ECE'),('EEE')) AS v(dept)`. Phase B2
replaces the dept enum with a `venues` table, and these are on a different
branch from the code B2 will be editing.

**The failure if missed.** A third venue is added, `venues` has three rows, the
app is correct — and every chase list silently covers only two. Same shape as
M1: no error, just a list that is quietly incomplete.

**The fix.** In B2's scope, already written into the queue. Not done until
adding a third venue is a single INSERT **and** the chase scripts pick it up
without being edited again.

`tests/gates.js` does not currently count these, because it greps `src/` and
the chase scripts live in `scripts/`. Worth extending when B2 starts, so the
budget covers them too.


---

## M4 — the unreachable survey branch in `isOpenFor()`

**What.** `isOpenFor()` has a `if (item_type === 'survey' && item_id)` branch
returning false. It is unreachable: the `if (any) return false` check above it
returns first, because every survey is released through the Open screen and so
always has a release row somewhere.

**How it is known.** By sabotage, not by reading. Changing the branch's
`return false` to `return true` leaves the venue-leak suite entirely green.
Sabotaging `if (any) return false` instead turns five checks red across all
four leaking surfaces. The second line is load-bearing; the first is a default
that never fires.

**Why it was kept rather than deleted.** It makes the default for an unreleased
survey explicit. Deleted, a survey falls through to
`return item_type === 'attendance'`, which answers false today by accident
rather than by intent and would flip the moment that line is edited.

**The failure if missed — and this is the dangerous one.** T3-B4 replaces
`isOpenFor()` with `releases.is_open_for(activity, venue)`. Whoever writes it
will read this function top to bottom, find a branch containing plausible
per-type survey logic, and carry it forward — while possibly dropping the
`any` rule above it, which is the part that actually prevents one venue seeing
another's work. That reintroduces the exact leak T2-14 closed, in the function
whose entire job is preventing it.

This is the same shape as the four enumerating-gate bugs: **old logic nobody
updated, kept because it looked load-bearing.** Here it is the inverse —
logic that looks load-bearing and is not.

**The fix, at B4.** Carry forward the `any` rule: once any venue has a release
row for an item, that table is the only authority. The per-type branch is a
default for the unreleased case and nothing more. `tests/survey-venue-leak.js`
must stay green through B4, and must be re-run with the same sabotage
afterwards — against the new gate, not the old one.

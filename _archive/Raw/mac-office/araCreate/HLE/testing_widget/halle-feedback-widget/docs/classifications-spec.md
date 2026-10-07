# Classifications — spec

Agreed with Vishnu, 2 October 2026.

## What it is

Today a Queue report can only be marked **Bug** (or deleted). The team needs
other kinds too — wording, design, a suggestion — and wants to make its own.

## Decisions

| Question | Decision |
| --- | --- |
| How does a type relate to the status flow? | **Bug keeps its flow** (Processing → Fixed → Closed). Every other type is separate, with two steps: **Open → Done**. |
| Who makes the types? | Anyone logged in, on a new **Admin → Classifications** screen: add, rename, colour, hide. No hard delete. |
| Can a type be changed later? | Yes, from the report popup. The history records it, with who and when. |
| The reports already marked Bug? | They are Bug. Nothing to move: Bug is built in, not a row. |
| Where do the other types show? | A new sidebar page, **Classified**, one tab per type, with Open / Done under it. Tracked items stays bugs-only. |

## How it works

- **Queue popup:** the Bug button stays, and one button per visible custom
  type sits next to it. Delete stays.
- **Bug is built in.** It cannot be renamed or hidden, and its name is
  reserved. It is not stored as a row: a report is a Bug when its status is
  `bug`, `fixed` or `closed`, exactly as today.
- **A custom-type report** has status `open` or `done`, and a row in
  `report_classifications` saying which type.
- **Changing type:** to a custom type → `open`; to Bug → `bug` (Processing).
  Custom → custom keeps Open or Done as it was.
- **Hidden type:** not offered for new reports. Reports already carrying it
  keep it, and its tab stays on Classified while it has any.
- **Colours:** a short fixed list, every one readable as text on white
  (4.5:1). Red is kept for Bug.

## Data

- `classifications` — id, org_id, project_id, name, colour, hidden_at,
  created_at. Name unique per project, ignoring case.
- `report_classifications` — one row per custom-type report: report_id
  (primary key), org_id, project_id, classification_id, updated_at.
- `reports.status` — gains `open` and `done`. Still the only column on
  `reports` that ever changes, so the append-only rule (agent-rules.md §1.1)
  and its trigger are untouched.
- `report_events` — gains `from_classification_id` / `to_classification_id`,
  so the history can say "changed it from Bug to Wording". Null means Bug or
  the queue, read with the status beside it.

One rule, kept in code (`lib/db/report-status.ts`) and covered by tests: a
report has a `report_classifications` row **if and only if** its status is
`open` or `done`. Every change to status and type happens in one
transaction, with its history row.

## Also changed

- CSV export: a new `type` column (Bug, the custom name, or empty).
- The report popup shows the type next to the status.
- `launch-reset` clears `report_classifications` with the reports, and keeps
  the types themselves, like pages.

## Not in this change

- Custom steps per type (e.g. To write → Reviewed → Published).
- Reordering types (they list in the order they were made).
- A Type filter on the Queue: Queue reports have no type yet.

---
name: decide
description: Write a decision record with options, the call, a checkable prediction and a scheduled revisit — or, with --revisit, score past predictions against what actually happened. Use for "/decide", "help me decide", "log this decision", "revisit my decisions".
---

# /decide — decisions with a scoreboard

Read `CLAUDE.md` first. Records live in `30-Areas/decisions/`.

## Mode A: a new decision

1. Clarify the question until it is answerable in one sentence. Ask, don't assume.
2. Lay out the real options with upside, downside and cost. Include "do nothing".
3. Ask for the call. You may recommend; you do not decide.
4. Write `30-Areas/decisions/YYYY-MM-DD-slug.md` from the decision template.
5. **The prediction section is mandatory and must be checkable.** Push back on
   vague predictions. "It'll go well" is not acceptable; "by 2026-11-20 at least 2
   of 5 pitched clients have signed" is. Record `confidence` (0-1) and
   "I would call this wrong if:".
6. Set `review_on` — usually 30, 90 or 365 days out.
7. Append to `90-System/logs/decisions.jsonl`:
   `{"at":"<iso>","slug":"...","question":"...","call":"...","confidence":0.7,"review_on":"YYYY-MM-DD","prediction":"..."}`

## Mode B: `--revisit`

1. Read `decisions.jsonl`; find every record whose `review_on` is today or past
   and whose note has an empty `## Outcome`.
2. For each: state the prediction, then gather actual evidence from the vault
   (project notes, dailies, meeting logs). Do not accept a self-report.
3. Fill `## Outcome` with: what happened, was the prediction right/wrong/partly,
   and — the useful part — **what the error tells us about how Vishnu mis-predicts**.
4. Set `status: stable`, append a History line, and log the score.
5. Report a running calibration summary: of N scored predictions at ~70%
   confidence, M came true.

Never rewrite a past prediction. That is the one thing that would make this
worthless.

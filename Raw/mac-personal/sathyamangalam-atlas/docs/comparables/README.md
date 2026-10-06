# Worldwide Atlas Comparables — Task 1 research library

`worldwide-atlas-comparables.html` is a self-contained browser for the Task 1
worldwide inventory of comparable projects: search, facet filters, every field
the report carried, and every link it gave. Open it by double-clicking — the data
is embedded in the file, so it needs no server and no network.

## Layout

```
sources/                                   ← the delivered research files, verbatim, never edited
  Sathyamangalam_Atlas_Research_Report_2.md  ← merged master (Tasks 1–4 + Rounds 2–3). Parsed from.
  Task1_Worldwide_Inventory.md               ← the standalone Task 1 delivery. Same project rows,
                                               but four of its six continent headings are glued to
                                               the tail of a preceding line by a merge bug, so the
                                               master is what the build reads.
  Sathyamangalam_Atlas_Round3.md             ← Round 3 standalone
comparables.json                           ← GENERATED. Structured rows + counts + provenance.
worldwide-atlas-comparables.html           ← GENERATED. The library page.
```

Regenerate both after any change to the source markdown:

```bash
python3 scripts/build_comparables_html.py
```

Never hand-edit the `.html` or `.json` — the next build overwrites them.

## What the numbers mean

`602` is a count of **table rows**, not of projects:

| | |
|---|---|
| 545 | profiled projects — full seven-column entries |
| 57 | dead / dormant entries — three- and four-column tables with no scale, team, subject or geography |
| **602** | **total table rows** |
| 585 | distinct project names (17 names appear twice — kept, not merged) |
| 533 | rows that give a project URL (69 do not) |
| 67 | further leads the report could not verify — bullet prose, no table row, so **outside** the 602 |
| 143 | links in Round 2/3 prose that appear in no Task 1 table |

The 17 repeats are mostly real overlap rather than sloppiness: Mexico and the
Caribbean were each covered by two regional passes, and several projects are
profiled as live in one table and listed as dead in another. Both rows are kept
and cross-linked, because the differing assessments are themselves the finding.

## Two rules the build follows

1. **Nothing is invented.** A field the source does not state is emitted as
   `null` and rendered "not stated in source" — never back-filled with "Unclear"
   or "Other". The report nests its dead/dormant tables under whichever country
   heading came last; those rows get no country here, because that country is not
   theirs.
2. **Every derived facet records its rule.** Status, scale, subject and domain
   facets are keyword-derived from the report's prose, and each row shows the rule
   and token that bucketed it, with the raw text alongside. Rows no rule matched
   are shown as stated rather than forced into a bucket: 11 status, 148 scale,
   95 subject. The facets are a browsing aid; the prose is the evidence.

## Provenance

Task 1 of the deep-research brief in
`docs/claude-project/sathyamangalam/deep-research-brief-comparable-projects-2026-08-25.md`,
compiled 25 August 2026, self-described as "six independent research passes (one
per world region)". The analyses of the same report already on file live in
`docs/claude-project/sathyamangalam/deep-research-report-ai1-*-analysis-2026-08-26.md`.

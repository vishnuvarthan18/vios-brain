---
type: system
status: stable
created: 2026-08-20
updated: 2026-08-20
tags: [vos, schema]
---

# Frontmatter schema (closed lists)

This file is the vocabulary. Nothing outside these lists is allowed without a
deliberate decision. Adding a value here is a real change — do it consciously.

## Required on every note

```yaml
type:    note          # see list below — required
status:  active        # see list below — required
created: 2026-08-20    # ISO date, never changes
updated: 2026-08-20    # ISO date, bump on every edit
```

## `type` — closed list

| type | Lives in | Meaning |
|---|---|---|
| `inbox` | `00-Inbox/` | Unprocessed capture |
| `daily` | `10-Journal/` | One day |
| `weekly` | `10-Journal/` | Week review |
| `monthly` | `10-Journal/` | Month review |
| `project` | `20-Projects/` | Has a finish line |
| `area` | `30-Areas/` | Ongoing, no finish line |
| `note` | `40-Library/` | An idea, concept, how-to, learning note |
| `person` | `50-People/` | A human |
| `meeting` | `20-Projects/` or `30-Areas/` | A specific conversation |
| `decision` | `30-Areas/decisions/` | A decision record |
| `source` | `60-Raw/` | Immutable captured material |
| `system` | `90-System/` | Vault machinery |

## `status` — closed list

`inbox` → `active` → `stable` → `archived`
plus `draft`, `blocked`, `superseded`, `dropped`.

- `draft` — started, not usable yet
- `active` — being worked on now
- `blocked` — waiting on someone/something (add `blocked_by`)
- `stable` — finished and trusted, no action needed
- `superseded` — replaced; add `superseded_by`
- `archived` — done or dead; lives in `99-Archive/`
- `dropped` — deliberately abandoned (keep the note, it is evidence)

## Optional fields

```yaml
tags:        [brand-strategy, pricing]   # LIST. topics only. lowercase-kebab.
people:      ["[[Anita Rao]]"]           # links, quoted
project:     "[[20-Projects/acme-rebrand/README]]"
source:      60-Raw/transcripts/2026-08-20-acme-call.md   # path, for anything claimed
generated:   { by: claude/opus-5, at: 2026-08-20T09:14:00Z }   # if an agent wrote it
verified:    [ { by: "human:vishnu", at: 2026-08-20 } ]        # you read it and agree
stale_after: 2027-02-20                  # after this date, treat with suspicion
review_on:   2026-11-20                  # for decisions: when to score the prediction
superseded_by: "[[Newer Note]]"
blocked_by:  "waiting on Acme legal"
confidence:  0.7                         # decisions only, 0-1
```

## Rules that silently break things if ignored

- `tags` must be a **list**, never a bare string. `tags: [a, b]` not `tags: a, b`.
- Reserved keys — never repurpose: `tags`, `aliases`, `cssclasses`.
- Lowercase kebab-case for field names and tag values. `date-finished`, not `Date_Finished`.
- Dates ISO 8601 only: `2026-08-20`. Never `20/08/2026`.
- Numbers unquoted, or they sort as text.
- Wikilinks inside YAML must be quoted: `project: "[[Thing]]"`.
- Plural field name for multi-value (`people`, `tags`), singular for single (`status`, `project`).

## Tag discipline

Tags are for **topics only**, open vocabulary, and nothing depends on them.
Type, status and lifecycle live in frontmatter and folders instead. Cap: if you
pass ~30 tags in total, prune before adding more. Everything else is search.

## Naming

- Dailies: `2026-08-20.md`. Weeklies: `2026-W34.md`. Monthlies: `2026-08.md`.
- Raw: `YYYY-MM-DD-slug.md` — date first, always.
- People: `Firstname Lastname.md`.
- Projects: folder `kebab-slug/` with `README.md` inside.
- Library notes: title the note as the **claim**, not the topic.
  "Retainers beat project fees for small studios" beats "Pricing".

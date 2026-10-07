# Calendar App — Data Model Reference

**The full field-level schema reference for calendar-be's 7 MongoDB collections already exists — [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5-§6 — and is not duplicated here.** That document was written by reading every `*.schema.ts` file directly, covers every field/type/index/relationship for `accounts`/`calendars`/`events`/`syncconfigs`/`syncgroupmetas`/`synchistories`/`auditlogs`, the migration strategy, and the platform's hard-delete-only policy, in comprehensive detail. Rewriting that content in a second document would just create two sources of truth to keep in sync. This page is the calendar-scoped fast index into it, plus one thing DATABASE-DESIGN.md deliberately doesn't cover: what each collection means operationally, from calendar-be's own point of view.

---

## 1. Where to find what

| Need | Go to |
|---|---|
| Field-by-field schema for any collection (types, defaults, indexes) | [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5.1–§5.7 |
| How the 7 collections relate to each other (relationship diagram) | [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5 (top) |
| The `Calendar.accountId` vs. `Event.accountId` ID-type inconsistency | [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5 (top) — a verified, live gap, not resolved as of this writing |
| Migration inventory + idempotency guarantees | [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5.8 — one idempotent `StartupMigrationService`, not versioned migration files |
| Soft-delete vs. hard-delete policy | [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §6 — hard-delete only, no `deletedAt`/`isDeleted` field anywhere in this platform |
| How collections fit into calendar-be's module/service architecture | [`calendar/ARCHITECTURE.md`](ARCHITECTURE.md) §6 |
| The exact algorithms that read/write `Event`/`SyncHistory` during a sync | [`calendar/SYNC-FLOW.md`](SYNC-FLOW.md) |
| The exact ordered cleanup sequence when an account is unlinked | [`calendar/DELETE-CASCADE.md`](DELETE-CASCADE.md) |

---

## 2. What each collection means, operationally

A one-line summary of *why* each collection exists, for someone orienting themselves before diving into DATABASE-DESIGN.md's field tables:

| Collection | Operational role |
|---|---|
| `accounts` | The credential/connection record — one per linked Google or Microsoft calendar account. Everything else in this data model exists to serve syncs *between* accounts' calendars. |
| `calendars` | A calendar as calendar-be knows it — one document per calendar the app has seen, whether or not it's actively synced. Also where Google's webhook channel state lives (`channelId`/`channelToken`/`expiration`). |
| `events` | Two different things share this collection: real synced events copied *content-wise* from a source, and blocker events calendar-be itself created on a target. The `extendedProperties.private.isSource` flag is what distinguishes a blocker from everything else calendar-be reads — not its title or description, which are configurable per sync (see `syncgroupmetas` below) and no longer a fixed string. |
| `syncconfigs` | The actual configuration a user set up — "keep target calendar X blocked out based on what's busy on source calendar Y." One document per source→target pairing. |
| `syncgroupmetas` | The sync's user-facing identity and content controls — one document per **target** calendar (not per source→target pairing, unlike `syncconfigs`), holding a display `name` and the three `showTitle`/`showDescription`/`showLocation` toggles `blocker-content.util.ts` resolves a blocker's real content from. Absence of a row (a target that's never been named/edited) is meaningful, not an error — every toggle defaults to `true`. See [`calendar/SYNC-FLOW.md`](SYNC-FLOW.md) §10. |
| `synchistories` | An audit trail, but a partial one — see [`calendar/SYNC-FLOW.md`](SYNC-FLOW.md) §4: only initial-sync runs and delta-processing *failures* write here. A healthy, ongoing incremental sync produces no new rows at all. Don't read "no recent history" as "sync is broken." |
| `auditlogs` | A separate, narrower audit trail from `synchistories` — privileged *account-level* actions only (`account.delete`, `sync.stop`), not sync activity. Easy to conflate the two by name; they track different things entirely. |

---

## Related documents

- [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5-§6 — the authoritative field-level schema this page indexes into
- [`calendar/ARCHITECTURE.md`](ARCHITECTURE.md), [`calendar/SYNC-FLOW.md`](SYNC-FLOW.md), [`calendar/DELETE-CASCADE.md`](DELETE-CASCADE.md) — how these collections are actually used
- [ADR-003](../adr/003-mongodb-for-calendar.md) — why calendar-be uses MongoDB in the first place
- [`calendar/API.md`](API.md), [`calendar/LINKED-ACCOUNTS.md`](LINKED-ACCOUNTS.md) — the HTTP surface and feature that read/write these collections

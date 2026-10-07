# ADR-003 — MongoDB for calendar data vs. PostgreSQL

**Status**: Decided, implemented. **Note on sourcing**: unlike ADR-001, no original decision document for this choice was found anywhere in the workspace — this ADR is written from the shape of the implemented schema, not from a preserved rationale. Flagged explicitly rather than presenting inferred reasoning as a verified historical record.

## Context

Every other backend on the platform (`core-be`, `admin-be`) uses PostgreSQL. `calendar-be` is the one exception, using MongoDB (`arm-calendar` database) for accounts, calendars, events, sync configs, and sync history — see [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5 for the full schema.

## Decision

MongoDB for all calendar-domain data.

## Inferred rationale (not sourced from an original decision record)

Calendar event data verified in the actual schema shape is genuinely variable and provider-specific in ways that don't map cleanly onto fixed relational columns:

- `Event.start`/`end` are typed as `calendar_v3.Schema$EventDateTime` — a nested object shape, not a scalar, and one that differs for all-day vs. timed events.
- `Event.extendedProperties.private.{sourceCalendarId, isSource}` is arbitrary nested metadata used to mark and trace blocker events — exactly the kind of schema-on-read flexibility a document store accommodates without a migration, that a relational table would need a JSON column (or a schema change) for.
- `recurrence` is stored as a raw `string[]` of RRULE lines, passed through unmodified from the provider (see [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §1) — again, provider-shaped data stored close to its original form rather than decomposed into relational structure.
- Cross-provider syncs (Google ↔ Microsoft) mean the same collections hold documents whose exact field population differs by provider (Microsoft's `Calendar` documents lack a `timeZone` field Google's have, for instance) — a natural fit for a schema-flexible store, awkward for strict relational columns that would need to be nullable per-provider.

## Consequences

- **No foreign-key enforcement** — cross-collection relationships (`Calendar.accountId` → `Account`, `Event.calendarId` → `Calendar`) are maintained entirely at the application level, with **a verified real inconsistency as a result**: `Calendar.accountId` stores the provider's own account ID while `Event.accountId` stores `Account._id` (Mongo's ObjectId) — two different ID types under the same field name across collections, confirmed in [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5 and independently flagged as a live e2e-test gap in [`calendar/DELETE-CASCADE.md`](../calendar/DELETE-CASCADE.md) §7. A relational schema with real foreign keys would very likely have caught this class of bug at the schema level; MongoDB's lack of enforcement didn't cause it, but didn't catch it either.
- **Delete cascades are application-level, not database-level** — account unlink requires an explicitly ordered cleanup sequence across 5 collections ([`calendar/DELETE-CASCADE.md`](../calendar/DELETE-CASCADE.md)) rather than `ON DELETE CASCADE`, unlike `core_be`'s Postgres tables which do get that for free (see [DATABASE-DESIGN.md](../DATABASE-DESIGN.md)'s delete-policy table).
- **This is the one service on the platform requiring both a different backup/restore tool and a different connection-health check pattern** than the two Postgres services — see [INFRASTRUCTURE.md](../INFRASTRUCTURE.md) §9's note that the restore-drill tooling only covers Postgres today, not MongoDB.

## Related documents

- [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5 — full schema, the `accountId` ID-type inconsistency
- [`calendar/ARCHITECTURE.md`](../calendar/ARCHITECTURE.md) §6 — how the collections fit into calendar-be's module structure
- [`calendar/DELETE-CASCADE.md`](../calendar/DELETE-CASCADE.md) — the application-level cascade this decision requires

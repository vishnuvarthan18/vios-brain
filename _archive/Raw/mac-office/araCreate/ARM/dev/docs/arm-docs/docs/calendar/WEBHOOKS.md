# Calendar App — Webhook Architecture

Google Calendar push notifications are calendar-be's primary sync trigger, and a stateful external dependency that degrades silently when not maintained. This document covers channel registration, the inbound intake path, renewal, and — most importantly — exactly where the silent-failure risk lives and how narrow it actually is. [SYNC-FLOW.md](SYNC-FLOW.md) §9 covers what happens to the data once a notification is validated (delta processing); this document stops at validation and channel lifecycle.

Everything below is verified against `src/backend/src/` directly, as of 2026-08-11.

---

## 1. Channel registration

Google's push-notification model: the app tells Google "watch this calendar, POST changes to this URL," Google replies with a channel (`channelId`, `resourceId`, `expiration`), and Google pushes a notification to that URL on every change until the channel expires or is explicitly stopped.

`calendar/google-calendar-api.service.ts`'s `watchEvents()` is the only place the actual `calendar.events.watch` call happens:

| Field | Value | Source |
|---|---|---|
| `id` (channel ID) | `crypto.randomUUID()`, app-generated | Caller (`sync-orchestrator.service.ts` for initial registration, `webhook-renewal.processor.ts` for renewal) — Google never generates this, the app owns it and uses it later to look up the calendar |
| `address` | `env.NOTIFICATION_WEBHOOK_URL` | Points at calendar-be's own `POST /v1/events/notifications` route (§3). **`env.ts`'s own hardcoded fallback default is `http://localhost:4001/events/notifications` — missing the `/v1/` prefix, a latent bug** (calendar-be sets a global `v1` prefix, per `main.ts`). Not currently exercised: every shipped compose file either sets this var explicitly with the correct `/v1/` path or requires it with no fallback at all — see [KONG-CONFIG.md](../KONG-CONFIG.md) §4 |
| `expiration` | `Date.now() + 604_600_000` ms ≈ **6.999 days** | **Hardcoded literal in `watchEvents()`**, sent explicitly on every `watch` call — this is an app-requested TTL, not Google's own default kicking in |
| `token` (channel token) | `(secret removed)(32).toString('hex')`, app-generated | Same caller sites as channel ID — a 32-byte random hex secret Google echoes back on every push as `x-goog-channel-token`, verified on intake (§3) |

All four values (`channelId`, `channelToken`, `resourceId`, `expiration`) are stored on the `Calendar` document, plain (not encrypted at rest — unlike account tokens).

**Registration happens at three points**, all generating a fresh channel ID + token each time (channels are never reused):
1. **Initial sync**, per source calendar (`sync-orchestrator.service.ts`) — only attempted if `accessRole` is `owner`/`writer`; a `ForbiddenException`/403 (the path Microsoft's `watchEvents` always throws through) falls back to a 5-minute poll scheduler instead.
2. **Initial sync**, target calendar (NR-01, same file) — same webhook-or-poll-fallback logic, registered after the whole source loop.
3. **Daily renewal cron** (§4) — stops the old channel, registers a brand-new one.

---

## 2. `stopChannel`

Calls Google's `calendar.channels.stop({id, resourceId})` — a generic channel-teardown call, not calendar-specific; it stops by opaque channel/resource ID regardless of what it was watching. Called from renewal (stopping the old channel before creating its replacement), sync stop/unlink, sync-config updates that remove a source, and cross-target migration (F-32, cancelling the old target's source webhooks).

Microsoft's `stopChannel` is a stub — Microsoft has no push-notification support (§6 covers how Microsoft still gets tamper-restoration coverage despite this, via the poll fallback).

---

## 3. Inbound intake

`POST /v1/events/notifications` (`events/events.controller.ts`, `@Controller('events')` under calendar-be's global `v1` prefix — verified against `main.ts`'s `app.setGlobalPrefix('v1')` and Kong's own hardcoded rewrite target in [KONG-CONFIG.md](../KONG-CONFIG.md) §4) — **not** reached through the session service proxy (Google's push service can't carry a browser session cookie), unauthenticated by design (Google calls it, not a logged-in user), `@SkipThrottle()`.

**The controller itself does no validation at all.** It reads the raw request headers, enqueues them onto the `webhook-processing` BullMQ queue (3 attempts, exponential backoff, 5s base), and returns `200 OK` **immediately** — before any auth or token check has run. Every real validation step happens later, asynchronously, in `WebhookProcessorService.handleWebhook`:

1. **Parse** — reads exactly three headers: `x-goog-resource-id`, `x-goog-channel-id`, `x-goog-resource-state`. Missing `resourceId`/`channelId` → drop silently. (`x-goog-message-number` is never read anywhere in source.)
2. **Sync handshake no-op** — `x-goog-resource-state: sync` (Google's automatic ping when a channel is first created) short-circuits immediately: no channel lookup, no token check, nothing else runs. This is expected and not an error.
3. **Channel lookup** — resolve `(channelId, resourceId)` to a `Calendar` document; no match → drop silently.
4. **Token verification** — only if the stored `Calendar.channelToken` is set (older channels registered before this existed skip the check):
   ```
   incoming.length === stored.length && crypto.timingSafeEqual(incoming, stored)
   ```
   Length-checked first (`timingSafeEqual` throws on mismatched lengths), wrapped in try/catch. Any failure → drop silently, no error surfaced back to Google.
5. **Source-vs-target dispatch** — see §6.
6. **Source account existence + `onSync` flag**, then **`syncToken` presence** (drops with a warning if initial sync hasn't produced one yet).

Every failure mode above is a **silent drop**, not an error response — by design, since the `200 OK` was already sent by the controller before any of this runs. This means a misconfigured or stale channel doesn't generate any Google-visible error; it just stops producing effects. Keep this in mind when debugging "sync silently stopped" — the signal is application logs, not HTTP status codes Google saw.

---

## 4. Renewal — and where the real gap is

Google channels expire ~7 days after registration (the hardcoded TTL in §1). `webhook/webhook-renewal.service.ts`'s daily cron (`@Cron('0 0 * * *')`, midnight) finds channels expiring within a 48-hour window and enqueues a `renew` job per channel (3 attempts, exponential backoff, 30s base). `webhook/webhook-renewal.processor.ts` does the work: stop the old channel (best-effort — failure only logged), generate a fresh channel ID + token, call `createSubscription`, and — **only on success** — overwrite the stored `channelId`/`channelToken`/`resourceId`/`expiration` with the new values.

**This is the confirmed, currently-open gap (`TASK-045`, still open per `TASK_SHEET.md`).** If `createSubscription` throws during renewal — no try/catch wraps that call — the exception fails the BullMQ job (retried per the 3-attempt policy above, then dead-lettered), and **the DB is never updated**. The old channel record stays in place, but it was already `stopChannel`'d against Google earlier in the same run — so the stored `channelId`/`resourceId` now point at a channel that's dead on Google's side and will never receive another push. Nothing marks this calendar as needing a poll fallback. **After 3 exhausted renewal attempts, this calendar's sync silently stops updating, with no automated recovery and no user-visible error**, until someone notices and force-renews it (§5) or the calendar is dropped and re-added.

**This gap is narrower than it might sound — it's specific to renewal, not to registration.** *Initial* webhook registration (source or target, §1) does have a poll fallback built in: a `ForbiddenException`/403 on the first `watchEvents` call falls back to a 5-minute poll scheduler automatically. Only a channel that was already successfully registered and then **fails to renew** ends up in the no-fallback state described above. Don't conflate the two — a newly-connected account with a webhook-unsupported calendar (e.g. Microsoft, or a Google calendar the account doesn't own) is fine; it's polling by design from the start.

**Manual recovery**: `POST /webhook/renew-all` (`webhook/webhook-controller.ts`) force-renews every channel regardless of expiry window — the mechanism this doc's §5 assumes you'd reach for after changing `NOTIFICATION_WEBHOOK_URL`, and the practical workaround for a calendar stuck in the gap above (it doesn't detect the gap, but re-running it against a known-affected calendar recovers it). This route is not admin-only in the RBAC sense — it's gated by `InternalSecretGuard`, which requires an `x-session-secret` header matching `SESSION_INTERNAL_SECRET`. A plain authenticated admin session cannot call it, and neither can a browser at all: the session service proxy strips that header, so it must be called directly (bypassing the proxy) with the secret attached.

---

## 5. 410 Gone — sync token invalidation

Google returns `410` on an incremental `events.list` call (using a stored `syncToken`) when the token has expired or the client fell too far behind. Two independent code paths handle it identically in shape:

- **Webhook-triggered** (`WebhookProcessorService.handleWebhook`): catches the 410, clears the stored `syncToken`, re-runs a **full** listing (not incremental) as fallback, persists the new sync token from that full listing *before* delta processing runs (deliberately — avoids double-processing on a mid-loop crash), then runs `sweepStaleBlockers` (§6 of [SYNC-FLOW.md](SYNC-FLOW.md)) to catch source events that vanished without an explicit cancellation.
- **Poll-triggered**: same pattern, delegated to a shared `fullyResyncCalendar()` helper — also reused by Microsoft's monthly delta-chain renewal, since Graph delta chains fail in a conceptually similar way (they expire outright rather than returning a 410, but the recovery is the same: full re-fetch + sweep).
- **Target-calendar (NR-01) path** has its own inline 410 handling, same clear-and-refetch shape, but does **not** run the stale-blocker sweep — there's no blocker-staleness concept on the target side; tamper restoration is separate logic entirely.

A 410-triggered full refetch differs from a brand-new initial sync in one important way: it reuses `applyDeltaEvents`, which is idempotent against existing blockers (matched by source event ID) — so it doesn't recreate blockers that already exist, only reconciles what changed and sweeps what disappeared.

---

## 6. NR-01 — target-calendar tamper restoration

Users can manually delete or edit a blocker event directly in the target calendar. NR-01 detects and restores that. It uses **the same intake route** as source-calendar webhooks — there is no separate controller or endpoint. Disambiguation happens purely by database lookup inside `WebhookProcessorService.handleWebhook`, *after* the token-verification gate (§3 step 4) already ran identically for both:

- The notification's `(channelId, resourceId)` resolves to *some* `Calendar` document, regardless of its role.
- If that calendar is a **source** in any active `SyncConfig`, it always takes the normal delta-sync path — even if the same calendar is also someone's target elsewhere (multi-hop sync chains proceed as source first).
- Only if it is **not** a source but **is** a target does it route to the tamper-restoration handler.

Because Microsoft's `watchEvents` always throws, every Microsoft-connected target calendar falls back to the poll path automatically (§1) — `pollTargetCalendar` calls the identical tamper-restoration handler, so Microsoft targets get the same protection as Google targets, just on a 5-minute poll cadence instead of push.

---

## 7. Monitoring — there isn't any dedicated to this

No Grafana dashboard, alert, or metric specifically tracks webhook-renewal failures, channels stuck in the §4 gap, or the dead-letter queue's contents. [REDIS-ARCHITECTURE.md](../REDIS-ARCHITECTURE.md) documents the `webhook-renewal`/`webhook-processing` BullMQ queues' existence and retry config, but — consistent with that document's own finding that Redis metrics are scraped with no dedicated dashboard — there's no alerting layered on top. [MONITORING.md](../MONITORING.md) confirms only `calendar-be` has OpenTelemetry tracing wired in among the platform's backends, but tracing individual requests isn't the same as an alert firing when renewals start failing.

**What actually exists today, for an on-call engineer**: pino log lines from the renewal processor on each attempt/failure, and the `dead-letter` BullMQ queue accumulating jobs after 3 exhausted renewal attempts — both require someone to be looking, not something that pages anyone. Building an alert on "`webhook-renewal` job failure count > 0" or "`Calendar.expiration` in the past with no successful renewal since" would close this gap; neither exists yet.

---

## 8. Related documents

- [SYNC-FLOW.md](SYNC-FLOW.md) §4, §6, §9 — renewal cron/queue config, stale-blocker sweep, and what delta processing does once a notification passes validation
- [ARCHITECTURE.md](ARCHITECTURE.md) §4 — where this fits in the module map, Microsoft's monthly delta-renewal cron as the parallel mechanism
- `apps/arm-app-calendar/docs/failure-modes.md` — dependency-outage behavior (Redis down mid-renewal, Google API down); note its §6 (Microsoft Graph) is now stale — it predates the Microsoft calendar-sync integration this doc and `ARCHITECTURE.md` describe
- [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §5.2 — `Calendar` schema field table (`channelId`/`channelToken`/`resourceId`/`expiration`)
- [`troubleshooting/calendar-sync-not-working.md`](../troubleshooting/calendar-sync-not-working.md) §3 — the `TASK-045` renewal gap as a triage symptom

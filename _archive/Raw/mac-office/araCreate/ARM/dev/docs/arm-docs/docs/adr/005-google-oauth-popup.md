# ADR-005 — OAuth popup + BroadcastChannel for linking calendar accounts

**Status**: Decided, implemented — but the implementation has drifted from a single clean pattern into two coexisting ones. This ADR records both the original decision and the drift.

## Context

Linking a second Google/Microsoft calendar account (the "Add Account" flow) needs to run an OAuth consent redirect without navigating the user away from the app they're using, and the parent window needs to learn when that flow finishes so it can refresh its data — without the popup ever handing the parent any token material.

## Decision

A popup window carries the OAuth consent flow; on success, the popup's landing page signals completion to the parent via `BroadcastChannel`, then closes itself. The completion payload is deliberately **type-only** — no tokens, no email, no account ID cross the browser boundary; the parent just refetches its own data once notified.

## Why not the generic proxy, and why a dedicated session service route

The OAuth consent redirect is a `302` — a generic axios-based proxy follows redirects server-side, so it can never hand the actual redirect back to the browser. This is why Add Account uses dedicated session service routes (`/session/calendar/add-account(/microsoft)`) with `maxRedirects: 0` instead of the normal `/session/proxy/...` path — documented in depth in [`core/arm-session/README.md`](../../../../core/arm-session/README.md)'s "Adding a second calendar account" and [`calendar/GOOGLE-OAUTH.md`](../calendar/GOOGLE-OAUTH.md) §3.

## Consequences — the drift from the plan implied by "not postMessage" in the title

The completion-signal mechanism didn't stay singular. Two different call sites use two different mechanisms today, verified in [`calendar/GOOGLE-OAUTH.md`](../calendar/GOOGLE-OAUTH.md) §6:

- `calendar-account-page.tsx` listens on **both** `BroadcastChannel` and a legacy same-origin `window.postMessage`, and additionally polls whether the popup has closed every 500ms as a fallback.
- The Settings page's `useOAuthPopup` listens **only** on `window.postMessage`, with no close-poll fallback.

**This split has a real local-dev consequence**: `BroadcastChannel` is same-origin only, and locally the shell (`localhost:3000`) and the session service success page (`localhost:5001`) are different origins — so the channel message often never arrives in local dev specifically. The account page recovers via its close-poll fallback; Settings' `useOAuthPopup` has no such fallback and can miss the success signal locally until the user manually refreshes. This is a known, documented gap ([`calendar/GOOGLE-OAUTH.md`](../calendar/GOOGLE-OAUTH.md) §12), not something this ADR is newly surfacing — recorded here because it's a direct consequence of the original completion-signal decision interacting with local dev's origin split, worth knowing before "fixing" either listener in isolation.

A separate legacy path (`auth-callback.tsx`, used for error/reconnect-success pages) still uses `postMessage(..., '*')` — a weaker pattern (any origin can theoretically receive it) than either of the two above, and not what the primary Add Account success path uses today.

## Related documents

- [`calendar/GOOGLE-OAUTH.md`](../calendar/GOOGLE-OAUTH.md) §5–§7 — full completion-signal mechanics, the local-dev origin gotcha, known pitfalls
- [`core/arm-session/README.md`](../../../../core/arm-session/README.md) — the dedicated-route pattern this flow depends on
- [`troubleshooting/auth-failures.md`](../troubleshooting/auth-failures.md) §2 — popup-specific cookie/completion-signal troubleshooting

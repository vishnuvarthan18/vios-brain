# Phase 7: UX Bug Fixes

> Fix verified user-facing bugs from QA testing. These are visual and interaction issues that real users will hit — no security or data integrity risk, but they erode trust.

**Priority:** P3 — After core stabilization, before external users
**Effort:** ~4 hours total
**Prerequisite:** Phase 1 complete (services running), Phase 2 recommended (security holes closed)
**Source:** Bug Sheet — Verification Report (2026-08-21)
**Repos touched:** `arm-app-calendar` (frontend + backend), `arm-ui-library`, `arm-session`

---

## Bug Triage Summary

| ID | Verdict | Severity | Task |
|---|---|---|---|
| BUG-01 | Invalid — popup closes correctly | — | No fix |
| BUG-02 | Valid — no success toast on account add | Major | T-7.1 |
| BUG-03 | Valid — no error on re-adding linked account | Major | T-7.2 |
| BUG-04 | Partially valid — race condition in popup hook | Minor | T-7.3 |
| BUG-05 | Invalid — merged calendar logic is correct | — | No fix |
| BUG-06 | Valid — date numbers clipped in week header | Major | T-7.4 |
| BUG-07 | Valid — sidebar text overflows into grid | Major | T-7.5 |
| BUG-08 | Invalid — "Busy" title is by design | — | No fix |

**5 valid bugs, 3 invalid (closed). 5 tasks.**

---

## Tasks

### T-7.1: Add success toast after account connect

**Source:** BUG-02
**Repo:** arm-app-calendar
**File:** `apps/arm-app-calendar/src/frontend/src/features/calendar/components/calendar-account-page.tsx`
**Bug:** The `onDone` callback (line ~91) calls `reloadAll()` and opens the import picker, but never shows a success message. All `<Toast>` instances on this page are wired to error states only. No green/success toast exists.

- [ ] Add state: `const [successMsg, setSuccessMsg] = useState<string | null>(null);`
- [ ] In the `onDone` callback, after `reloadAll()`, set the success message:
  ```typescript
  setSuccessMsg("Account connected successfully");
  ```
- [ ] Add a success `<Toast>` alongside the existing error toasts:
  ```tsx
  {successMsg && (
    <Toast msg={successMsg} color="green" durationMs={4000} onDone={() => setSuccessMsg(null)} />
  )}
  ```
- [ ] Verify in browser: add a Google or Microsoft account, confirm the green toast appears
- [ ] Verify: existing error toasts still work (disconnect network, try adding account)

**Commit:** `fix(calendar): show success toast after account connect`
**Effort:** 15 minutes

---

### T-7.2: Detect and surface re-adding an already-linked account

**Source:** BUG-03
**Repos:** arm-app-calendar (backend + frontend), arm-session
**Bug (two layers):**
1. **Backend** — `google-oauth.service.ts:82-87` only rejects accounts owned by a *different* user. Same-user re-add silently upserts tokens via `createOrUpdate()` with no signal back to the UI.
2. **Frontend error path** — Even if the backend threw, the session service discards the message at `session.controller.ts:312-317` (generic `error=oauth_failed` redirect), and `use-connect-account-popup.ts` doesn't handle `GOOGLE_AUTH_FAILED` messages (lines 40-48 only listen for success types).

**Files:**
- `apps/arm-app-calendar/src/backend/src/google/google-oauth.service.ts` (~line 82-93)
- `core/arm-session/src/session/session.controller.ts` (~line 306-317)
- `apps/arm-app-calendar/src/frontend/src/features/calendar/hooks/use-connect-account-popup.ts`
- `apps/arm-app-calendar/src/frontend/src/features/calendar/components/calendar-account-page.tsx`

#### Step 1: Backend — signal "already linked" on same-user re-add

- [ ] In `google-oauth.service.ts`, after the cross-user check (line ~87), add same-user detection:
  ```typescript
  if (existing && existing.userId === userId) {
    await this.accountService.createOrUpdate({ ... });
    return { alreadyLinked: true };
  }
  ```
- [ ] Update return type to `Promise<{ alreadyLinked?: boolean }>` (or equivalent)
- [ ] Apply the same pattern to `microsoft-oauth.service.ts` for Microsoft accounts
- [ ] Run backend tests: `cd apps/arm-app-calendar/src/backend && pnpm jest google-oauth --no-coverage`

**Commit (calendar-be):** `fix(google): signal already-linked on same-user account re-add`

#### Step 2: session service — forward the flag in BroadcastChannel message

- [ ] In `session.controller.ts` (~line 306), include `alreadyLinked` in the inline script's `postMessage`:
  ```javascript
  'ch.postMessage({type:"GOOGLE_AUTH_SUCCESS",alreadyLinked:' +
    (response.data?.alreadyLinked ? 'true' : 'false') + '});'
  ```
- [ ] Apply same change to the Microsoft callback path

**Commit (session):** `fix(proxy): forward alreadyLinked flag in OAuth broadcast message`

#### Step 3: Frontend — show appropriate message

- [ ] In `use-connect-account-popup.ts`, pass `alreadyLinked` through to `onDone`:
  ```typescript
  export function useConnectAccountPopup(
    onDone: (opts?: { alreadyLinked?: boolean }) => void,
  ) { ... }
  ```
- [ ] In `calendar-account-page.tsx`, use the flag to distinguish reconnect from new:
  ```typescript
  const { connect, connecting, connectingProvider } = useConnectAccountPopup(async (opts) => {
    reloadAll();
    if (opts?.alreadyLinked) {
      setSuccessMsg("Account tokens refreshed — already connected");
    } else {
      setSuccessMsg("Account connected successfully");
    }
    // ... existing import picker logic
  });
  ```
- [ ] Verify in browser: re-add an already-linked account, confirm the "already connected" message appears
- [ ] Verify: first-time add still shows the original success message

**Commit (calendar-fe):** `fix(calendar): distinguish reconnect from new account in success toast`
**Effort:** 1.5 hours (3 repos, 3 commits)

---

### T-7.3: Fix double-invocation race in popup close hook

**Source:** BUG-04
**Repo:** arm-app-calendar
**File:** `apps/arm-app-calendar/src/frontend/src/features/calendar/hooks/use-connect-account-popup.ts`
**Bug:** The 500ms closed-popup poll (line ~64-74) can fire before the BroadcastChannel message is delivered, causing `finish()` to run from the poll path first. Both paths can call `onDoneRef.current()`, causing a double `reloadAll()`. On Safari, BroadcastChannel may not deliver at all after cross-origin popup navigation, so the poll path is the only one that fires.

- [ ] Add a `didFinishRef` guard to prevent double invocation:
  ```typescript
  const didFinishRef = useRef(false);

  const finish = useCallback(() => {
    if (didFinishRef.current) return;
    didFinishRef.current = true;
    setConnectingProvider(null);
    popupRef.current?.close();
    onDoneRef.current();
  }, []);
  ```
- [ ] Reset the guard when `connect()` is called:
  ```typescript
  const connect = useCallback((provider: "google" | "microsoft") => {
    didFinishRef.current = false;
    setConnectingProvider(provider);
    // ...
  }, []);
  ```
- [ ] Verify in browser: add an account, confirm `reloadAll()` fires exactly once (check network tab for duplicate API calls)

**Commit:** `fix(calendar): prevent double finish callback in connect-account popup hook`
**Effort:** 20 minutes

---

### T-7.4: Fix clipped date numbers in weekly calendar header

**Source:** BUG-06
**Repo:** arm-ui-library
**File:** `library/arm-ui-library/src/components/big-calendar-overrides.css`
**Bug:** `CalendarDayHeader.tsx` renders a two-row layout (~44px: weekday label + date circle). The parent row (`.rbc-time-header`) height is controlled by react-big-calendar's internal layout which assumes one line of text. No `min-height` is set to accommodate the two-row custom header. The date numbers are vertically clipped.

- [ ] Add a min-height rule after the existing `.rbc-header` overflow fix (~line 39):
  ```css
  /* CalendarDayHeader renders a two-row layout (weekday label + date circle)
     that is taller than RBC's assumed single-line header. Force the header
     content row to accommodate both rows without clipping. */
  .rbc-radix-scope .rbc-time-header-content > .rbc-row:first-child {
    min-height: calc(var(--space-6) + var(--font-size-1) + var(--space-3));
  }
  ```
- [ ] If the design system tokens aren't available at this selector level, use a fixed `min-height: 52px` as fallback
- [ ] Build the library: `cd library/arm-ui-library && pnpm build`
- [ ] Verify in browser: open the weekly view, confirm date numbers are fully visible (not clipped)
- [ ] Check month and day views are unaffected

**Commit:** `fix: set min-height on week header row to prevent date clipping`
**Effort:** 20 minutes

---

### T-7.5: Fix sidebar text overflow into calendar grid

**Source:** BUG-07
**Repo:** arm-ui-library
**File:** `library/arm-ui-library/src/components/CalendarMergedView.tsx`
**Bug:** The sidebar `Box` (line ~117-128) has `overflowY: "auto"` but no `overflowX` constraint. The footer paragraph ("On your target calendars...") at line ~264 has no width limit. If the sidebar width is narrower than the text's natural width, text overflows horizontally into the calendar grid. The `boxShadow` on the sidebar creates a stacking context that renders above the grid.

- [ ] Change `overflowY: "auto"` to `overflow: "auto"` on the sidebar Box (line ~127):
  ```tsx
  // Change:
  overflowY: "auto",
  // To:
  overflow: "auto",
  ```
- [ ] This clips both horizontal and vertical overflow within the sidebar boundary. The horizontal scrollbar won't actually appear because all internal content uses truncate or wraps naturally — this just prevents the edge case where a long unwrapped string escapes the box.
- [ ] Build the library: `cd library/arm-ui-library && pnpm build`
- [ ] Verify in browser: open the merged calendar view, resize the window narrow, confirm sidebar text doesn't overflow into the grid
- [ ] Check that vertical scrolling inside the sidebar still works

**Commit:** `fix: clip sidebar overflow on both axes to prevent grid bleed`
**Effort:** 10 minutes

---

## Repo and Commit Map

This phase touches 3 repos. Each repo gets its own commits — never cross-repo commits.

| Task | Repo | File(s) | Commit |
|---|---|---|---|
| T-7.1 | arm-app-calendar | `src/frontend/.../calendar-account-page.tsx` | `fix(calendar): show success toast after account connect` |
| T-7.2 step 1 | arm-app-calendar | `src/backend/.../google-oauth.service.ts` | `fix(google): signal already-linked on same-user account re-add` |
| T-7.2 step 2 | arm-session | `src/session/session.controller.ts` | `fix(proxy): forward alreadyLinked flag in OAuth broadcast message` |
| T-7.2 step 3 | arm-app-calendar | `src/frontend/.../use-connect-account-popup.ts`, `calendar-account-page.tsx` | `fix(calendar): distinguish reconnect from new account in success toast` |
| T-7.3 | arm-app-calendar | `src/frontend/.../use-connect-account-popup.ts` | `fix(calendar): prevent double finish callback in connect-account popup hook` |
| T-7.4 | arm-ui-library | `src/components/big-calendar-overrides.css` | `fix: set min-height on week header row to prevent date clipping` |
| T-7.5 | arm-ui-library | `src/components/CalendarMergedView.tsx` | `fix: clip sidebar overflow on both axes to prevent grid bleed` |

**Total: 7 commits across 3 repos.**

---

## Verification

All fixes in this phase are visual — they must be verified in the browser, not just with unit tests.

```bash
# 1. Build the UI library after T-7.4 and T-7.5
cd library/arm-ui-library && pnpm build

# 2. Rebuild calendar frontend to pick up library changes
make local-dev-rebuild SERVICE=calendar-frontend

# 3. Rebuild calendar backend after T-7.2 step 1
make local-dev-rebuild SERVICE=calendar-backend

# 4. Rebuild session service after T-7.2 step 2
make local-dev-rebuild SERVICE=session

# 5. Open browser to localhost:3000
# - Navigate to calendar → Manage Syncs
# - Add a Google account → green success toast should appear (T-7.1)
# - Add the same Google account again → "already connected" message (T-7.2)
# - Check network tab: only one burst of API calls on add (T-7.3)
# - Navigate to weekly view → date numbers fully visible (T-7.4)
# - Navigate to merged view → resize window narrow → sidebar doesn't bleed (T-7.5)
```

---

## Completion Checklist

- [ ] All 5 tasks done (7 commits across 3 repos)
- [ ] Success toast appears on account connect
- [ ] Re-adding an already-linked account shows appropriate message (not silent upsert)
- [ ] Popup close hook fires exactly once (no double reload)
- [ ] Weekly calendar header shows full date numbers (not clipped)
- [ ] Merged view sidebar doesn't overflow into the calendar grid
- [ ] UI library rebuilt and published locally
- [ ] All changes committed locally, nothing pushed

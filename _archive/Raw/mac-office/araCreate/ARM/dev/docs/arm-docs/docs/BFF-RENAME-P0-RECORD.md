# BFF → Session Rename — Phase 0 Record

> Pre-flight record and rollback note for the rename.
> Plan: [BFF-RENAME-TASKS.md](BFF-RENAME-TASKS.md) · Mapping: [BFF-RENAME-INVENTORY.md](BFF-RENAME-INVENTORY.md)
>
> Captured 2026-08-25. **No source code has been modified.**

---

## Status

Phase 0 is **complete**. The items that needed you, a server, or a decision were
**parked into [Phase 7 — Cutover readiness](BFF-RENAME-TASKS.md#phase-7--cutover-readiness)**,
which runs in parallel with Phases 1–6 and must land before the Phase 8 cutover.

| ID | Task | Status |
|---|---|---|
| P0-1 | Capture rollback anchors | ✅ Done — [§1](#1-rollback-anchors) |
| P0-2 | Create the branch in all 8 repos | ✅ Done — [§1](#1-rollback-anchors) |
| P0-3 | Capture the local baseline | ✅ Done — [§2](#2-local-environment-at-baseline), [§3](#3-test-baseline--action-required) |
| P0-4 | Draft the user notice | ✅ Done — [§4](#4-drafted-user-notice) |

**Parked to Phase 7:**

| Now | Was | Task |
|---|---|---|
| P7-1 | P0-1 | Book the maintenance window |
| P7-2 | P0-2 | Schedule the notice drafted in [§4](#4-drafted-user-notice) |
| P7-3 | P0-3 | Record server image tags — [§5](#5-commands-you-need-to-run-on-the-server) |
| P7-4 | P0-4 | Snapshot the server `.env` — [§5](#5-commands-you-need-to-run-on-the-server) |
| P7-6 | — | Decide: fix the 10 `calendar-be` failures, or carry them |
| P7-7 | — | Establish a `notification` test baseline |
| P7-8 | — | Confirm `arm-tool-cli`'s branch base |
| P7-9 | — | Commit or stash `arm-core-be`'s WIP |

---

## 1. Rollback anchors

Branch created everywhere: **`refactor/rename-bff-to-session`**

Created with `git branch` and **not** checked out, so every repo is still on the branch
it was on. Nothing was committed, staged, or pushed.

| Repo (local path) | GitHub repo | Was on | Rollback SHA | New branch base |
|---|---|---|---|---|
| `core/arm-bff` | `arm-core-bff` | `dev` | `55e210c` | `55e210c` |
| `core/arm-core-be` | `arm-core-be` | `dev` | `a9b0971` | `a9b0971` |
| `core/arm-core-fe` | `arm-core-fe` | `dev` | `00affac` | `00affac` |
| `apps/arm-app-calendar` | `arm-app-calendar` | `dev` | `7990cfd` | `7990cfd` |
| `apps/arm-admin` | `arm-app-admin` | `dev` | `bb76101` | `bb76101` |
| `deploy/arm-deploy-make` | `arm-devops-makefile` | `main` | `bf1b81f` | `bf1b81f` |
| `docs/arm-docs` | `arm-docs` | `main` | `ff6d9bc` | `ff6d9bc` |
| `arm-tool-cli` | `arm-tool-cli` | `fix/dead-exports` | `ae5402e` | `570d3d7` ⚠️ |

> ⚠️ **`arm-tool-cli` was branched from `dev` (`570d3d7`), not from its current HEAD.**
> It is sitting on `fix/dead-exports`, one unmerged commit ahead of `dev`
> (`ae5402e refactor: drop exports and constants that nothing imports`). Basing the
> rename on that would entangle it with unrelated WIP. Confirm this is what you want
> before starting Phase 5.

> ⚠️ **`core/arm-core-be` has uncommitted source changes** — `src/common/generic.controller.ts`,
> `src/users/users.controller.ts`, and a new `src/users/users.controller.http.spec.ts`.
> These are unrelated to the rename and will follow the working tree onto whichever
> branch you check out. Commit or stash them before starting Phase 3a.
>
> Every other repo's dirty state is untracked `CLAUDE.md` / `TASK_SHEET.md` files only.

---

## 2. Local environment at baseline

Only **3 of 8** app services were running:

| Service | State |
|---|---|
| `arm-core-be-local` | Up, healthy |
| `arm-calendar-be-local` | Up, healthy |
| `arm-notification-local` | Up, **unhealthy** ⚠️ *pre-existing* |
| `arm-bff-local` | **not running** |
| `arm-core-fe-local` | **not running** |
| `arm-calendar-fe-local` | **not running** |
| `arm-admin-be-local` | **not running** |
| `arm-admin-fe-local` | **not running** |

Infra (`kafka`, `mongodb`, `postgres`, `redis`) all healthy.

> `arm-notification-local` being unhealthy is recorded here deliberately so it is **not
> misattributed to the rename** at P8-3.

A full-stack baseline needs `make arm-run` to bring up the remaining five. Not run —
starting containers was outside what Phase 0 asked for.

---

## 3. Test baseline — action required

`make test` was run from `deploy/arm-deploy-make`. **It does not pass today.**

| Suite | Result |
|---|---|
| `core-backend` | ✅ 35 suites, 301 tests — all pass |
| `arm-bff` | ✅ 11 suites, 92 tests — all pass |
| `calendar-backend` | ❌ **5 suites failed** / 59 passed · **10 tests failed** / 600 passed |
| `notification` | ⚠️ **never ran** — `make test` aborted at exit 2 |

### The 10 pre-existing failures (all in `calendar-backend`)

```
GoogleClientFactory — getGoogleToken › no refresh, no token, when account not authenticated
GoogleClientFactory — refreshFromCoreAuth › catches CoreAuthService.getGoogleToken error
GoogleClientFactory — refreshFromCoreAuth › returns early when CoreAuth reports no token
MicrosoftService.generateAddAccountUrl › builds authorize URL with tenant/client/redirect/scopes/state
AuthController — Microsoft add-account › microsoftAddAccountCallback exchanges code and state
SyncQueryService › previewCalendar › defaults source status to PENDING, target falls back to "[deleted]"
SyncQueryService › previewCalendar › returns null target and empty sources when no SyncConfigs match
WebhookProcessorService.handleWebhook › dispatches to Microsoft when Google returns null
WebhookProcessorService.pollTargetCalendar › recognizes Microsoft-shaped blocker via summary fallback
WebhookProcessorService.pollTargetCalendar › recognizes Google-shaped blocker via extendedProperties
```

**None of these touch the BFF contract.** They are Google/Microsoft client, sync-query
and webhook-processor failures. The BFF-related `ERROR` lines in the log come from
core-be's `auth.controller.spec.ts`, which **passes** — those are deliberately logged
errors on negative-path assertions.

### Consequence for the plan

**P8-2 as written ("`make test` passes") is not achievable and must be restated as
"the same 10 failures, no new ones."** The task sheet has been updated accordingly.

Two things to decide before Phase 7:

1. **Fix these 10 first, or carry them?** Carrying them means the rename's verification
   signal is "compare against a known-bad baseline", which is workable but weaker. One
   of them — `AuthController — Microsoft add-account` — sits in the add-account flow
   that P3-7 touches, so a real regression there could hide behind a pre-existing failure.
2. **`notification` has no baseline at all.** Run `make test-notification` on its own to
   establish one, since `make test` aborts before reaching it.

Full log: `p0-test-baseline.log` (scratchpad, not committed).

### ⚠️ This baseline was itself incomplete — since fixed

`make test` covered **backend jest suites only**. It ran no frontend tests, no admin
tests at all, and never typechecked anything. Three pre-existing `tsc -b` errors on
`dev` were therefore invisible to it.

Fixed on 2026-08-25: the three errors were corrected, and `make test` was extended to
cover all 8 services plus a `typecheck` target. See
[BFF-RENAME-TASKS.md § Baseline gap closed](BFF-RENAME-TASKS.md#baseline-gap-closed).

**A baseline captured with the old `make test` understates what was passing.** The 10
`calendar-be` failures above were accurate at the time; the frontend picture was absent.

**Superseded 2026-08-25:** all 10 `calendar-be` failures were subsequently fixed (each
was a stale test double or assertion, not a production bug) and the workspace now runs
**1,329 tests, 0 failing**, with `make test` exiting 0. See
[BFF-RENAME-TASKS.md § Phase 10a](BFF-RENAME-TASKS.md#10a--the-10-calendar-be-failures--fixed).

---

## 4. Drafted user notice

> **Subject: ARM scheduled maintenance — you will need to sign in again**
>
> On **[DATE]** between **[START]** and **[END]** we are carrying out planned
> maintenance on ARM.
>
> **What to expect**
>
> - ARM will be briefly unavailable during the window.
> - **You will be signed out and will need to log in again afterwards.** This is
>   expected and does not mean anything is wrong with your account.
> - If you are midway through connecting a Google or Microsoft calendar when the
>   window begins, that connection will not complete — simply start it again once
>   we are back.
>
> **What is not affected**
>
> - Your calendars, events, sync settings and connected accounts are unchanged.
> - You do not need to reconnect Google or Microsoft accounts already linked.
> - No password or credential changes are required.
>
> **After the window**
>
> Go to [URL] and sign in as normal. If anything looks wrong, contact [CONTACT].

Fill in `[DATE]`, `[START]`, `[END]`, `[URL]`, `[CONTACT]` and schedule it.

---

## 5. Commands you need to run on the server

Neither is reachable from this workspace — there is no server host configured in the
local `.env`.

**P0-3 — record image tags for rollback:**

```sh
docker ps --format '{{.Names}}\t{{.Image}}\t{{.CreatedAt}}' \
  | grep -E 'arm-(core-backend|calendar-backend|admin-backend|bff|notification-service|core-frontend|calendar-frontend|admin-frontend)' \
  | tee ~/arm-rollback-images-$(date +%F).txt
```

**P0-4 — snapshot the server `.env`:**

```sh
cp /path/to/arm/.env ~/arm-env-backup-$(date +%F).env
chmod 600 ~/arm-env-backup-$(date +%F).env
```

> Secret **values** are preserved across the rename — only key names change
> (`BFF_*` → `SESSION_*`, `SESSION_SECRET` → `SESSION_COOKIE_SECRET`). Rollback is a
> rename-back, not a secret re-issue. Keep the snapshot anyway.

Paste both results into §1 of this document once captured.

---

## 6. Phase 0 exit criteria — met

- [x] Branches cut in all 8 repos
- [x] Rollback SHAs recorded
- [x] Local baseline captured, and its failures understood
- [x] User notice drafted

Everything still outstanding lives in
[Phase 7 — Cutover readiness](BFF-RENAME-TASKS.md#phase-7--cutover-readiness).

**Phase 1 is not blocked by any of it** — it is confined to `arm-session` and is
compiler-checked. Phase 7 gates the Phase 8 cutover, not the code work.

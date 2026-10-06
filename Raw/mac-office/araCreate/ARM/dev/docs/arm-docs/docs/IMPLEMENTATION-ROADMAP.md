# ARM Platform — Implementation Roadmap

> Master plan connecting the Failure Analysis, Production Audit, Infrastructure Auto-Tune, and Object Storage specs into a phased, task-level execution plan. This document tells you what to do, in what order, how to run it, and how to commit it.

**Date:** 2026-08-21
**Target:** 100 concurrent users on a single Hetzner CX31 (7.6 GB RAM)

---

## Source Documents

| Doc | What it is | Where |
|---|---|---|
| Failure Analysis | 8 failure scenarios, 3 tiers of urgency, per-container memory snapshot | [FAILURE-ANALYSIS-2026-08-21.md](FAILURE-ANALYSIS-2026-08-21.md) |
| Production Audit | 114 bugs (19C, 35H, 35M, 25L) + 14 infra issues across all 7 repos | [PRODUCTION-AUDIT-2026-08-21.md](PRODUCTION-AUDIT-2026-08-21.md) |
| Infra Auto-Tune | `make tune-infra` design spec — deterministic infrastructure right-sizing | [architecture/infra-auto-tune-design.md](architecture/infra-auto-tune-design.md) |
| Object Storage | Hetzner S3 media storage — move profile pictures out of Postgres | [architecture/object-storage-design.md](architecture/object-storage-design.md) |
| MS Graph Webhooks | Real-time Microsoft calendar sync (design + implementation plan) | [calendar specs](../../../apps/arm-app-calendar/docs/superpowers/specs/2026-08-21-microsoft-graph-webhooks-design.md) |
| QA Bug Sheet | 8 bugs triaged (5 valid, 3 invalid) — visual/interaction issues | [PHASE-7-UX-BUGS.md](roadmap/PHASE-7-UX-BUGS.md) |

---

## How the 4 Docs Connect

```
                     ┌────────────────────────────────────┐
                     │        FAILURE ANALYSIS             │
                     │  "What breaks, when, and why"       │
                     │  8 scenarios, 3 tiers of urgency    │
                     └────────┬───────────────┬────────────┘
                              │               │
              Tier 1 fixes    │               │  Tier 2+3 fixes
              feed into       │               │  are automated by
                              ▼               ▼
               ┌──────────────────┐   ┌───────────────────────┐
               │ PRODUCTION AUDIT │   │  INFRA AUTO-TUNE      │
               │ "114 bugs across │   │  "make tune-infra"    │
               │  all 7 repos"    │   │  right-sizes Kong,    │
               │                  │   │  Kafka, Postgres,     │
               │ Bugs cause the   │   │  MongoDB, Redis       │
               │ failures. Fixing │   │  automatically"       │
               │ bugs prevents    │   │                       │
               │ scenarios.       │   │  Prevents Scenarios   │
               │                  │   │  1, 2, 4, 5           │
               └──────┬───────────┘   └───────────┬───────────┘
                      │                           │
                      │  Some bugs block          │  Auto-tune must
                      │  feature work             │  run before features
                      │                           │  add load
                      ▼                           ▼
               ┌──────────────────────────────────────────────┐
               │           OBJECT STORAGE + MS WEBHOOKS        │
               │  "New capabilities, built on stable ground"   │
               │                                               │
               │  Depends on:                                  │
               │  - Infra stabilized (Phase 1-2)               │
               │  - Security bugs fixed (Phase 2)              │
               │  - Body size limits in place (Phase 2)        │
               │  - Sync engine hardened (Phase 4)             │
               │  - Postgres not under memory pressure (Tune)  │
               └──────────────────────────────────────────────┘
```

**The dependency chain:** Fix what's about to break -> fix what's exploitable -> automate infra sizing -> harden the sync engine -> build new features -> operational maturity.

---

## Phase Summary

| Phase | Sheet | Goal | Tasks | Effort | Prerequisite |
|---|---|---|---|---|---|
| **1** | [PHASE-1-EMERGENCY.md](roadmap/PHASE-1-EMERGENCY.md) | Prevent first outage | 12 | ~3h | None |
| **2** | [PHASE-2-SECURITY.md](roadmap/PHASE-2-SECURITY.md) | Close exploitable holes | 18 | ~8h | Phase 1 |
| **3** | [PHASE-3-INFRA-TUNING.md](roadmap/PHASE-3-INFRA-TUNING.md) | Automate resource tuning, handle 50 users | 14 | ~12h | Phase 1 |
| **4** | [PHASE-4-SYNC-ENGINE.md](roadmap/PHASE-4-SYNC-ENGINE.md) | Data integrity under load | 16 | ~10h | Phase 2 |
| **5** | [PHASE-5-FEATURES.md](roadmap/PHASE-5-FEATURES.md) | MS Webhooks + Object Storage | 12 | ~16h | Phase 3+4 |
| **6** | [PHASE-6-OPERATIONS.md](roadmap/PHASE-6-OPERATIONS.md) | 100-user readiness, remaining bugs | 20 | ~12h | Phase 3 |
| **7** | [PHASE-7-UX-BUGS.md](roadmap/PHASE-7-UX-BUGS.md) | QA-verified visual and interaction bugs | 5 | ~4h | Phase 1 |

**Total: 97 tasks, ~65 hours.**

### Parallelism

Phases 3 and 4 can run in parallel (infra vs app code, different repos). Phase 6 can start as soon as Phase 3 completes. Phase 5 waits for both 3 and 4.

```
Phase 1 ──→ Phase 2 ──→ Phase 4 ──┐
   │                                ├──→ Phase 5
   │    ┌──→ Phase 7 (anytime)      │
   └──→ Phase 3 ──→ Phase 6 ──────┘
              (parallel with 2+4)
```

Phase 7 (UX bugs) can run anytime after Phase 1 — it's independent of the security, infra, and sync phases. It touches frontend and UI library code that doesn't conflict with the backend work in Phases 2-4.

---

## How to Approach This Work

### General method

1. **One task at a time.** Each phase sheet lists tasks in priority order — work top to bottom. Don't skip ahead to a later phase because it looks more interesting; the dependency chain exists for a reason.
2. **Verify after every task.** Run the test suite for the service you touched before moving on. Don't batch 5 fixes and then discover the second one broke something.
3. **Read the file before changing it.** The audit found 114 bugs by reading every file. You're fixing those bugs — read the surrounding code, understand the context, then make the smallest change that fixes the issue.
4. **Server-side tasks are separate from code tasks.** Phase 1 has both (e.g. T-1.1 is a compose file edit, T-1.3 is a server SSH command). Don't mix them in the same commit. Server-side manual steps (swap, log rotation) have no commit — they're run on the server directly.
5. **Never stop to wait for human input.** If a task needs a manual step (SSH into the server, create a Hetzner bucket, fill in credentials, make a design decision you're unsure about), skip that task, note it as "SKIPPED — needs human action: <reason>" in the phase sheet, and move on to the next task immediately. Do not block the entire phase waiting for one answer. Skipped tasks are collected and addressed later in a single batch. Keep moving.

### Which repo to work in

ARM is a multi-repo workspace. Each task's phase sheet names the file path, which tells you which repo you're in:

| Path prefix | Repo | Branch |
|---|---|---|
| `core/arm-core-be/` | arm-core-be | `dev` |
| `core/arm-session/` | arm-session | `dev` |
| `core/arm-core-fe/` | arm-core-fe | `dev` |
| `apps/arm-app-calendar/` | arm-app-calendar | `dev` |
| `apps/arm-admin/` | arm-admin | `dev` |
| `services/arm-service-notification/` | arm-service-notification | `dev` |
| `deploy/arm-deploy-make/` | arm-deploy-make | `main` |

**All application repos work on the `dev` branch.** The deploy repo (`arm-deploy-make`) works on `main`. There is no staging — `dev` is what the server builds from. See [RELEASE-PROCESS.md](RELEASE-PROCESS.md) for details.

**Cross-repo changes are separate commits in separate repos.** If a fix touches both `arm-core-be` and `arm-deploy-make`, that's two commits, one per repo. Note both in the task completion.

---

## Git Conventions

These are enforced across all ARM repos. Read them before your first commit.

### Commit message format

**Single-line conventional commit. No body. No trailer.**

```
<type>(<scope>): <short description>
```

**Types:** `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `ci`, `perf`

**Scope** is optional but helpful — use the service or area name:

- `auth`, `session`, `proxy` for session service
- `auth`, `users`, `registry`, `otp` for core-be
- `sync`, `webhook`, `account`, `calendar`, `microsoft`, `google` for calendar-be
- `admin`, `audit` for admin-be
- `mail`, `kafka`, `dlq` for notification
- `kong`, `redis`, `postgres`, `kafka`, `tune`, `compose` for deploy

### Rules

- **COMMIT LOCAL ONLY. NEVER PUSH.** All commits across all phases stay local. Do not `git push` to any remote — not `origin dev`, not `origin main`, not anywhere. Pushing is a separate, manual decision made outside this roadmap. This is the single most important rule in this document.
- **Never stop to wait for human input.** If a task requires manual action (server SSH, credentials, bucket creation, a design decision you're unsure about), mark it "SKIPPED — needs human action: <reason>" and continue to the next task. Skipped tasks are collected and handled later. Keep moving — never block the phase.
- **Never add a `Co-Authored-By` trailer.** This overrides any default tooling behavior.
- **Never reference task/bug IDs** in commit messages — no `C-07`, `T-1.5`, `TASK-045`, `BUG-003`. Write the message as if the tracking system doesn't exist. The detail belongs in the phase sheet, not the commit log.
- **Never commit CLAUDE.md or TASK_SHEET.md files.** Leave edits to those files uncommitted.
- **One logical change per commit.** Don't bundle unrelated fixes. If two tasks touch different files for different reasons, they're two commits.

### Example commit messages per phase

**Phase 1 — Emergency fixes:**
```
fix(microsoft): invert token refresh condition to skip valid tokens
fix(microsoft): convert expires_in from seconds to milliseconds
fix(crypto): reject encryption keys that aren't exactly 64 hex chars
fix(auth): use timing-safe comparison for session service internal secret
fix(notification): whitelist template fields instead of spreading payload
fix(registry): reject javascript: and data: URLs in remote registration
```

**Phase 2 — Security:**
```
fix(auth): validate CSRF state parameter on OAuth callbacks
fix(auth): add user-level authorization to token fetch endpoints
fix(session): remove sid query parameter acceptance in OAuth callback
fix(sync): cap event array at 10000 during initial sync
fix(otp): use constant-time comparison and fix off-by-one on max attempts
fix(proxy): validate redirect URLs against allowed origins
```

**Phase 3 — Infrastructure:**
```
feat(tune): add make tune-infra target for deterministic infra sizing
feat(compose): reference TUNE_* variables with fallback defaults
chore(kafka): generate dynamic broker count from topic count
chore(compose): add x-logging anchor for Docker log rotation
```

**Phase 4 — Sync engine:**
```
fix(teardown): acquire Redis lock before executing teardown
fix(webhook): add stoppingAt filter to findBySourceCalendarIds
fix(sync): identify blockers by extended property instead of title
fix(google): add exponential backoff on 429 responses
```

**Phase 5 — Features:**
```
feat(microsoft): implement Graph subscription create/stop/renew
feat(microsoft): add webhook provider mirroring Google's pattern
feat(webhook): add Microsoft validation handshake endpoint
feat(media): add S3 upload service for Hetzner Object Storage
feat(media): replace base64 avatar flow with S3 upload
```

**Phase 6 — Operations:**
```
fix(health): verify dependency readiness in health endpoints
chore(redis): enable Sentinel for session failover
fix(admin): add RBAC to audit log queries
fix(notification): add SMTP retry with exponential backoff
```

**Phase 7 — UX bugs:**
```
fix(calendar): show success toast after account connect
fix(google): signal already-linked on same-user account re-add
fix(proxy): forward alreadyLinked flag in OAuth broadcast message
fix(calendar): prevent double finish callback in connect-account popup hook
fix: set min-height on week header row to prevent date clipping
fix: clip sidebar overflow on both axes to prevent grid bleed
```

---

## How to Run and Verify

### Before starting any phase

```bash
# From the workspace root (arm/)
make env-check                    # verify .env is consistent
make local-dev-ps                 # verify all services are running
```

### Running tests per service

Each task's phase sheet says which test to run. Here's the full reference:

```bash
# Core-be
cd core/arm-core-be
pnpm test                                          # all tests
pnpm jest <pattern> --no-coverage                  # single test file

# session service
cd core/arm-session
pnpm test
pnpm jest <pattern> --no-coverage

# Calendar-be
cd apps/arm-app-calendar/src/backend
pnpm test
pnpm jest <pattern> --no-coverage

# Admin-be
cd apps/arm-admin/src/backend
pnpm test
pnpm jest <pattern> --no-coverage

# Notification
cd services/arm-service-notification
pnpm test
pnpm jest <pattern> --no-coverage
```

### After code changes that affect containers

```bash
# Rebuild the specific service you changed
make local-dev-rebuild SERVICE=<service-name>

# Service names: core-backend, arm-session, calendar-backend, admin-backend,
#                notification-service, core-frontend, calendar-frontend, admin-frontend
```

### After compose/infra changes

```bash
# Validate compose files parse correctly
cd deploy/arm-deploy-make
docker compose -f docker-compose.infra.prod.yml config --quiet
docker compose -f docker-compose.yml config --quiet

# Restart infra if you changed infra compose
make infra-down && make infra-up-prod && make infra-wait-prod

# For local dev compose changes
make local-dev-stop && make local-dev
```

### Server-side manual tasks (Phase 1: swap, log rotation)

```bash
# SSH into the server first
ssh <your-server>

# T-1.2: Docker log rotation
sudo tee /etc/docker/daemon.json <<'EOF'
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "50m", "max-file": "3" }
}
EOF
sudo systemctl restart docker

# T-1.3: Add swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

# Verify
free -h                                            # shows swap
docker inspect --format='{{.HostConfig.LogConfig}}' arm-kong  # shows log rotation
```

### Deploying to the server

**Not part of this roadmap.** All commits stay local. Pushing and deploying is a separate decision made manually after reviewing the local commits. When the time comes, the deploy process is documented in [RELEASE-PROCESS.md](RELEASE-PROCESS.md) and [CICD.md](CICD.md).

---

## Prompts for Agentic Execution

When using Claude Code (or any agentic tool) to execute these tasks, use prompts structured like this:

### Starting a phase

```
Read roadmap/PHASE-1-EMERGENCY.md and execute the tasks in order.

Rules:
- COMMIT LOCAL ONLY. NEVER PUSH. No git push to any remote, ever.
- One task at a time. Run the verification step before moving to the next task.
- If a task needs my input, a manual step, or something you can't do — skip it,
  note "SKIPPED — needs human action: <reason>" and move to the next task. Never stop working.
- Commit messages: single-line conventional commit, no body, no trailer, no task IDs.
- Never commit CLAUDE.md or TASK_SHEET.md.
- Cross-repo changes are separate commits.
- Server-side tasks (swap, log rotation) — just tell me the commands to run; don't try to SSH.
```

### Resuming mid-phase

```
Continue with Phase 1 from task T-1.7 (token encryption key padding).
The file is apps/arm-app-calendar/src/backend/src/common/crypto/token-encryption.service.ts.
Read the file, understand the current getKey() implementation, then fix it
to reject keys that aren't exactly 64 hex characters.
Run the tests after: cd apps/arm-app-calendar/src/backend && pnpm jest token-encryption --no-coverage
```

### Running a parallel phase

```
Phase 3 (infra tuning) and Phase 4 (sync engine) can run in parallel.
Start with Phase 3. Work in deploy/arm-deploy-make on the main branch.
Read roadmap/PHASE-3-INFRA-TUNING.md and begin with T-3.1 (create make/tune.mk).
```

### Asking for a specific fix

```
Fix the inverted Microsoft token refresh condition.
File: apps/arm-app-calendar/src/backend/src/microsoft/microsoft-client.factory.ts
Function: refreshIfExpired() around line 57.
The condition checks `tokenExpiryAt > new Date()` and returns early —
this skips refresh when the token IS expired. Invert the logic.
Run: cd apps/arm-app-calendar/src/backend && pnpm jest microsoft-client.factory --no-coverage
Commit: fix(microsoft): invert token refresh condition to skip valid tokens
```

---

## Failure Scenario Coverage

Each failure scenario from the Failure Analysis maps to specific phases:

| Scenario | What Breaks | Fixed By |
|---|---|---|
| 1: Kong OOM (10-20 users) | All API routes 502 | Phase 1 (T-1.1) + Phase 3 (auto-tune) |
| 2: Server-wide OOM (50-80 users) | OOMKiller takes out Kafka or Postgres | Phase 1 (T-1.3 swap) + Phase 3 (right-sizing) |
| 3: Cascading auth failure (30+ users) | Mass logout every 15 min | Phase 2 (T-2.7, T-2.8) + Phase 6 (T-6.4) |
| 4: Disk full (2-4 weeks) | Everything dies | Phase 1 (T-1.2 log rotation) + Phase 3 (auto-tune) |
| 5: Redis fills up | Platform inaccessible | Phase 1 (T-1.4 AOF) + Phase 3 (auto-tune maxmemory) |
| 6: Google API quota (20+ users) | Syncs fail 24 hours | Phase 4 (T-4.10 backoff) |
| 7: Redis restart = sessions lost | All users logged out | Phase 6 (T-6.6 Sentinel) |
| 8: MongoDB OOM corruption | Partial writes, orphaned data | Phase 3 (wiredTiger) + Phase 4 (T-4.3) |

---

## Audit Bug Coverage

All 114 code bugs + 14 infrastructure issues assigned to a phase:

| Severity | Total | Ph1 | Ph2 | Ph3 | Ph4 | Ph5 | Ph6 | Covered |
|---|---|---|---|---|---|---|---|---|
| Critical (19) | 19 | 4 | 12 | 0 | 3 | 0 | 0 | **19/19** |
| High (35) | 35 | 1 | 10 | 0 | 14 | 0 | 10 | **35/35** |
| Medium (35) | 35 | 0 | 0 | 2 | 8 | 2 | 23 | **35/35** |
| Low (25) | 25 | 0 | 0 | 0 | 0 | 0 | 25 | **25/25** |
| Infra (14) | 14 | 4 | 0 | 6 | 0 | 0 | 4 | **14/14** |

---

## Milestones

| Milestone | After Phase | What you can say |
|---|---|---|
| **"Won't crash tomorrow"** | Phase 1 | Kong won't OOM, disk won't fill, Redis persists sessions, Microsoft sync actually works |
| **"Safe for external users"** | Phase 2 | No exploitable CSRF, session fixation, template injection, or timing attacks |
| **"Handles 50 users"** | Phase 3 | Infrastructure right-sized automatically, 1.75 GB RAM recovered, Kafka at 1 broker |
| **"Sync data is reliable"** | Phase 4 | No orphaned blockers, no silent failures, no retry storms, Google 429 handled |
| **"Feature-complete for v1"** | Phase 5 | Microsoft real-time sync, profile pictures on S3, no base64 in database |
| **"Ready for 100 users"** | Phase 6 | Redis Sentinel, backups automated, all 114 bugs closed, health checks real |
| **"Polished for users"** | Phase 7 | QA bugs fixed, success feedback on account add, no visual clipping or overflow |

---

## What NOT to Do

- **Don't push. Ever.** All work is committed locally. Pushing is a separate, manual decision. If you're using an agent, the agent must never run `git push`. This is the #1 rule.
- **Don't skip Phase 1.** Everything else assumes Kong isn't about to OOM and Redis persists data.
- **Don't start Phase 5 before Phase 4.** The MS webhook code uses the same sync engine being hardened in Phase 4. Building on an unreliable engine wastes time.
- **Don't batch all fixes into one giant commit.** Each fix is independently verifiable and revertible. One commit per logical change.
- **Don't "improve" adjacent code** while fixing a bug. Touch only what the task says to touch. If you notice something else, note it — don't fix it in the same commit.
- **Don't deploy Phase 2 security fixes without Phase 1 infra fixes.** A server that OOMs under load can't benefit from CSRF validation.
- **Don't run migrations (Phase 5 T-5.12) without a backup.** The base64-to-S3 migration reads every user row. Take a Postgres backup first (`make backup-postgres`).

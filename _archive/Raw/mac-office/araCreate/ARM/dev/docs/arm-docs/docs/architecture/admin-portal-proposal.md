# ARM Platform — Admin Portal
## Product Proposal

**Document Version:** 1.0  
**Date:** June 2026  
**Prepared by:** araCreate Engineering  
**Status:** Proposal — Pending Approval

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [The Problem We Are Solving](#2-the-problem-we-are-solving)
3. [What We Are Building](#3-what-we-are-building)
4. [How It Fits Into the Existing Platform](#4-how-it-fits-into-the-existing-platform)
5. [Feature Breakdown](#5-feature-breakdown)
6. [What the Admin Portal Will Look Like](#6-what-the-admin-portal-will-look-like)
7. [Implementation Plan](#7-implementation-plan)
8. [Technology Choices](#8-technology-choices)
9. [What We Need to Get Started](#9-what-we-need-to-get-started)
10. [Success Criteria](#10-success-criteria)
11. [Risks and How We Will Handle Them](#11-risks-and-how-we-will-handle-them)
12. [Summary and Next Steps](#12-summary-and-next-steps)

---

## 1. Executive Summary

The ARM platform currently has no centralized way to manage users, monitor system health, or investigate operational issues. When something goes wrong today, the engineering team must SSH into the server, run terminal commands, and manually inspect database records. There is no audit trail of who did what, no unified view of system performance, and no ability for non-technical operators to manage the platform.

We propose building the **ARM Admin Portal** — a secure, role-controlled web interface that gives the platform team complete visibility and management control over the entire ARM system from a single browser tab.

The Admin Portal will be built as a new section of the existing ARM web application, using the same technology, the same design system, and the same authentication infrastructure that already exists. This means no duplicate login, no separate deployment complexity, and no new infrastructure required for the core features.

The build is divided into five phases over an estimated **15–16 weeks**, starting with the highest-priority capability — user management — and progressively adding observability, monitoring, reporting, and advanced security controls.

---

## 2. The Problem We Are Solving

### 2.1 Today's Reality

Right now, every operational task on the ARM platform requires direct server access:

| Task | How It Is Done Today |
|---|---|
| Check if all services are running | SSH → `docker ps` |
| View application errors | SSH → `docker logs arm-calendar-backend-dev` |
| Disable a user account | SSH → `psql` → `UPDATE users SET isActive = false WHERE...` |
| Check calendar sync failures | SSH → `mongosh` → run query manually |
| See how many users are active | SSH → `psql` → `SELECT COUNT(*)...` |
| Revoke a user's session | SSH → `redis-cli` → find and delete session key |
| Check Kafka consumer lag | SSH → run `kafka-topics.sh` inside container |

This is slow, error-prone, and requires engineering knowledge to perform even basic tasks. More importantly, there is no record of who made what change or when.

### 2.2 The Gaps

**No visibility.** There is no single screen that shows the health of the platform. If a service degrades, the first signal is usually a user complaint.

**No audit trail.** If a user account is modified in the database, there is no record of who did it, why, or what changed. This is a compliance and accountability gap.

**No operational delegation.** Because every task requires terminal access, only engineers can operate the platform. Customer support, operations managers, and non-technical team members are blocked from performing even simple tasks like disabling a spam account.

**No proactive alerts.** Issues are discovered reactively. There is no monitoring that says "the calendar sync failure rate has doubled in the last hour."

**No user activity insight.** There is no way to see how many people are using the platform, when they are most active, or which features are most used.

### 2.3 Why Now

The platform is growing. More users, more applications, and more services will make these gaps more painful over time. The right moment to build the operational foundation is before the complexity scales further — not after the first major incident exposes the lack of tooling.

---

## 3. What We Are Building

The ARM Admin Portal is a **secure, role-controlled web interface** that provides the platform operations team with:

- A real-time view of the health of every service, database, and queue
- The ability to manage users — create, update, disable, assign roles — without database access
- A live log viewer across all services simultaneously
- API traffic metrics and error tracking
- Calendar sync operational dashboards
- A permanent, searchable audit trail of all admin actions

### 3.1 In One Sentence

> The Admin Portal turns today's terminal-based operations into a structured, accountable, and accessible web interface.

### 3.2 What It Is Not

- It is **not** a replacement for Grafana, Datadog, or enterprise APM tools. It is purpose-built for the ARM platform's specific operations.
- It is **not** a separate product. It lives inside the existing ARM web application as an admin-only section.
- It is **not** built from scratch on new infrastructure. It reuses every piece of the existing platform — the same auth, the same design system, the same deployment pipeline.

---

## 4. How It Fits Into the Existing Platform

The ARM platform already uses a **microfrontend architecture** — the main web application (core shell) loads separate frontend modules dynamically. The calendar application is already loaded this way. The Admin Portal will follow the exact same pattern.

```
ARM Web Application (arametrics.app)
│
├── /                    →  Core App (home, settings, profile)
├── /calendar            →  Calendar Module  ← already deployed as MFE
└── /admin               →  Admin Portal     ← new, same pattern
```

On the backend, a new dedicated service (`arm-admin-be`) will handle all admin operations. It will:

- Connect to the existing PostgreSQL, MongoDB, and Redis databases using **read-only access** for inspection
- Write to a new audit log table
- Aggregate metrics from all services
- Be accessible only through the existing session mechanism — so no separate login

```
Existing Infrastructure
│
├── arm-core-be         (auth, users, MFE registry)
├── arm-session             (session management, proxy)
├── arm-calendar-be     (calendar sync)
├── arm-notification    (email delivery)
│
└── NEW: arm-admin-be   (admin operations, audit log, metrics aggregation)
    └── NEW: arm-admin-fe (admin portal UI, loaded as MFE remote)
```

**Key point:** The existing platform does not need to be restructured. The Admin Portal is additive.

---

## 5. Feature Breakdown

### 5.1 User Management

**What it does:**  
Complete user lifecycle management from a web UI — no database access required.

**Specific capabilities:**

| Capability | Description |
|---|---|
| User directory | Search, filter, and paginate all users. Filter by status, auth provider, role. |
| User profile view | Full user details: name, email, registration date, auth method, linked Google accounts, last active |
| Create user | Add a new user with local auth credentials |
| Edit user | Update name, email, active status |
| Disable / Enable account | Immediately prevent login without deleting data |
| Assign roles | Promote to admin or operator; demote to regular user |
| View active sessions | See all active login sessions for a user, with device and IP metadata |
| Revoke sessions | Force logout from specific sessions or all sessions simultaneously |
| View linked accounts | See all connected Google accounts per user |
| Revoke Google account | Remove a linked Google account on behalf of a user |

**Why this matters:** Today, disabling a spam account or a departed employee's access requires a database query. With this feature, any operator can do it in under 10 seconds with a confirmation click — and the action is recorded in the audit log.

---

### 5.2 Platform Health Overview

**What it does:**  
A single screen showing the live status of every service, database, and queue.

**Specific capabilities:**

| Capability | Description |
|---|---|
| Service status board | Green / yellow / red status for: core-be, calendar-be, arm-session, notification service, Kong |
| Database connectivity | PostgreSQL, MongoDB, Redis — connected or disconnected |
| Kafka broker status | Broker reachable, consumer group status, message lag |
| Queue depths | BullMQ queue job counts: pending, active, completed, failed |
| Uptime indicators | How long each service has been running since last restart |

**Why this matters:** Right now, knowing if all services are healthy requires five separate `docker logs` commands. This replaces all of them with a single page that refreshes automatically.

---

### 5.3 API Traffic and Performance Metrics

**What it does:**  
Live view of API request volume, response times, and error rates across all services.

**Specific capabilities:**

| Capability | Description |
|---|---|
| Request rate | Requests per second per service, updated every 15 seconds |
| Status code breakdown | Count of 2xx, 4xx, 5xx responses per service |
| Error rate trend | Percentage of failed requests over the last 1 hour |
| Slowest endpoints | Top 10 endpoints by p95 response time |
| Top endpoints by volume | Top 10 most-called endpoints |
| Kong traffic stats | Requests passing through the API gateway |

**Why this matters:** Without this, the first sign of a performance degradation is user complaints. This makes degradation visible before users notice.

---

### 5.4 Calendar Sync Operations Dashboard

**What it does:**  
A dedicated operational view for the calendar sync system — the most complex part of the ARM platform.

**Specific capabilities:**

| Capability | Description |
|---|---|
| Active syncs | All currently active sync configurations with source → target pairs |
| Sync health status | Per-sync status: healthy, failing, no webhook, polling fallback |
| Recent sync history | Last 20 sync runs per configuration with events synced, errors, duration |
| Failed sync alerts | List of syncs with `invalid_grant` or other persistent errors |
| Queue job status | Initial sync jobs, poll jobs, webhook renewal jobs — counts and states |
| Accounts with expired tokens | Users whose Google OAuth tokens need re-authentication |
| Webhook registration status | Which calendars have active webhooks vs polling fallback |

**Why this matters:** The calendar sync system has multiple failure modes, all of which were previously invisible. This dashboard surfaces them immediately, enabling the team to proactively reach out to affected users rather than waiting for support tickets.

---

### 5.5 Application Log Viewer

**What it does:**  
A searchable, filterable view of application logs from all services in one place.

**Specific capabilities:**

| Capability | Description |
|---|---|
| Multi-service log view | See logs from all services simultaneously or select individual services |
| Severity filter | Filter by ERROR, WARN, INFO, DEBUG |
| Time range selector | View logs from the last 15 minutes, 1 hour, 6 hours, 24 hours |
| Search | Full-text search across log content |
| Context expansion | Click any log line to see the full JSON log object |
| User filter | Filter logs by user ID to trace a specific user's activity |
| Correlation ID filter | Filter logs by correlation ID to trace a specific request across services |

**Why this matters:** Today, investigating an issue requires jumping between multiple terminal windows running `docker logs`. This consolidates everything into one searchable interface.

---

### 5.6 Error Tracking and Exception Reporting

**What it does:**  
A consolidated view of all errors and exceptions thrown across the platform.

**Specific capabilities:**

| Capability | Description |
|---|---|
| Error feed | Live feed of all ERROR-level log entries across services |
| Error grouping | Repeated errors with the same message grouped and counted |
| Error rate chart | Error frequency over time, per service |
| BullMQ failed jobs | All failed background jobs with full error detail and job payload |
| Calendar sync failures | Sync-specific errors with user context and failure reason |
| Acknowledge errors | Mark an error group as known/resolved |

**Why this matters:** Currently, errors accumulate silently in `docker logs` with no way to know their frequency or pattern. This turns reactive debugging into proactive monitoring.

---

### 5.7 Audit Log

**What it does:**  
A permanent, searchable record of every action performed through the Admin Portal.

**Specific capabilities:**

| Capability | Description |
|---|---|
| Complete history | Every admin action recorded: who did it, when, what changed, from which IP |
| Searchable | Filter by admin user, action type, target user, date range |
| Immutable | Audit records can never be edited or deleted |
| Before/after state | For every change, shows what the value was before and what it became |
| Export | Download audit log as CSV for compliance or legal review |

**Example audit records:**

```
2026-06-02 14:32:11  kishor@aracreate.group  DISABLED USER    user: rahul@aracreate.group
2026-06-02 14:35:44  kishor@aracreate.group  ROLE CHANGED     user: gobinath@aracreate.group  user → admin
2026-06-02 15:01:22  kishor@aracreate.group  SESSION REVOKED  user: vishnu@aracreate.group  (2 sessions)
```

**Why this matters:** There is currently zero record of any administrative action. This is a compliance gap and a security gap. The audit log closes both.

---

### 5.8 Infrastructure Monitoring

**What it does:**  
Container-level resource usage and database statistics.

**Specific capabilities:**

| Capability | Description |
|---|---|
| Container CPU / memory | Live and historical resource usage per container |
| Docker host utilization | Total CPU, memory, and disk usage of the host server |
| Redis memory and keys | Memory consumed, key count by pattern, TTL distribution |
| MongoDB collection sizes | Document count and storage size per collection |
| PostgreSQL table sizes | Row counts and storage per table |
| Kafka consumer lag | Message backlog per topic and consumer group |

**Why this matters:** Without this, the first sign of a resource problem is a crashed service. This enables the team to see resource pressure building before it causes an outage.

---

### 5.9 Authentication and Security Activity

**What it does:**  
A dedicated view of login events, OAuth activity, and security-relevant actions.

**Specific capabilities:**

| Capability | Description |
|---|---|
| Login history | All successful and failed login attempts with timestamp, IP, and user agent |
| OAuth events | Google account link, unlink, and token refresh activity |
| Failed login summary | Count of failed attempts per user over time |
| Session analytics | Sessions created vs. expired vs. revoked per day |
| Suspicious activity | Users or IPs with abnormally high failure rates |

**Why this matters:** There is no current visibility into authentication events. This is needed for both security awareness and incident response.

---

## 6. What the Admin Portal Will Look Like

The Admin Portal uses the same `@aracreate/test-arm-ui` design system as the rest of the ARM platform. It is visually consistent with the existing application and requires no new design assets.

### 6.1 Navigation Structure

```
/admin
│
├── Overview          ← Platform health summary (default landing page)
├── Users
│   ├── User List
│   ├── User Detail
│   └── Create User
├── Monitoring
│   ├── Service Health
│   ├── API Traffic
│   └── Infrastructure
├── Calendar Ops
│   ├── Sync Status
│   ├── Queue Jobs
│   └── OAuth Tokens
├── Logs
│   ├── Application Logs
│   └── Error Tracker
├── Security
│   ├── Auth Activity
│   └── Session Manager
└── Audit Log
```

### 6.2 Access Control

The `/admin` route is completely invisible to regular users. It only appears in the navigation for users with the `admin` or `operator` role. If a regular user navigates to `/admin` directly, they are redirected to the home page.

Certain actions within the portal are further restricted to `admin` role only — operators can view and investigate but cannot make structural changes like deleting users or assigning the admin role.

| Action | Admin | Operator |
|---|---|---|
| View all dashboards | ✅ | ✅ |
| View user list and profiles | ✅ | ✅ |
| Enable / disable user | ✅ | ✅ |
| Revoke sessions | ✅ | ✅ |
| Assign roles | ✅ | ❌ |
| Delete user | ✅ | ❌ |
| Export audit log | ✅ | ❌ |
| View audit log | ✅ | ✅ |

---

## 7. Implementation Plan

The build is divided into five focused phases. Each phase produces working, deployable functionality — there is no long period where nothing is visible.

### Phase 1 — User Management Foundation
**Duration: 4 weeks**

The most operationally urgent capability. Eliminates the need for direct database access for all user management tasks.

**Delivered at end of Phase 1:**
- Admin role on the User model (schema migration)
- Admin Portal UI accessible at `/admin` for users with the `admin` role
- Full user directory: search, filter, paginate
- User profile view with complete detail
- Create user, edit user, disable/enable user
- Role assignment (admin, operator, user)
- Audit log recording every admin action
- CLI seed script to bootstrap the first admin user
- Kong route and session-service proxy for admin service

**Why first:** This eliminates the most common operational pain point immediately and establishes the security foundation everything else builds on.

---

### Phase 2 — Observability and Health Monitoring
**Duration: 4 weeks**

Establishes live visibility into the platform's operational state.

**Delivered at end of Phase 2:**
- Platform Overview dashboard (service health, queue depths, active sessions)
- API Traffic dashboard (request rate, error rate, response times per service)
- Calendar Sync Operations dashboard (sync status, failed syncs, expired tokens)
- Infrastructure dashboard (container CPU/memory, Redis, MongoDB, PostgreSQL stats)
- BullMQ queue inspector (job states, failed job details)
- Kafka consumer lag monitoring

**Why second:** Once user management is in place, the next priority is making the system's operational state visible. These features require no additional infrastructure — only the metrics and health endpoints that already exist on every service.

---

### Phase 3 — Logs and Error Tracking
**Duration: 3 weeks**

Centralizes log access and surfaces errors without SSH.

**Delivered at end of Phase 3:**
- Application log viewer across all services (last 1000 lines, filterable)
- Log filtering by service, severity, time range, user ID, correlation ID
- Error tracker — grouped errors with count, trend, and full stack trace
- BullMQ failed jobs with full context
- Calendar sync failure feed with user and error context

**Why third:** With health and metrics visible, the next need is the ability to investigate issues without terminal access.

---

### Phase 4 — Session Management and Auth Security
**Duration: 3 weeks**

Gives operators control over active sessions and visibility into auth activity.

**Delivered at end of Phase 4:**
- Per-user active session list (device, IP, created at, last active)
- Single-session and all-session revocation
- Force logout for disabled accounts (immediate effect via deny-list)
- Auth activity feed (logins, OAuth events, failures)
- Suspicious login activity surface (repeated failures, unusual IPs)
- Session analytics (created vs. expired vs. revoked per day)

**Why fourth:** Session management requires a minor update to how the session service stores session metadata in Redis. This is kept until after the foundational dashboards are in place.

---

### Phase 5 — Reporting and Advanced Controls
**Duration: 2–4 weeks**

Adds scheduled reporting, historical data, and advanced security controls.

**Delivered at end of Phase 5:**
- Historical time-series metrics (requires Prometheus deployment)
- Historical log search (requires Grafana Loki deployment)
- Scheduled daily summary report (email digest via existing notification service)
- Audit log export to CSV
- User analytics dashboard (registrations over time, activity patterns)
- Alert thresholds (configurable error rate / failure rate alerts)
- MFA enforcement for admin accounts (stretch goal)

---

### Timeline Summary

```
Week 1–4    Phase 1   User Management + Admin Foundation
Week 5–8    Phase 2   Observability + Health Monitoring
Week 9–11   Phase 3   Log Viewer + Error Tracking
Week 12–14  Phase 4   Session Management + Auth Security
Week 15–16  Phase 5   Reporting + Advanced Controls
```

---

## 8. Technology Choices

All technology choices are deliberate extensions of what the ARM platform already uses. There are no new frameworks, no new languages, and no dependency on unfamiliar tools.

### 8.1 Admin Backend (`arm-admin-be`)

| Concern | Choice | Reason |
|---|---|---|
| Framework | NestJS (TypeScript) | Identical to every other service. Same guards, interceptors, and patterns. |
| PostgreSQL ORM | TypeORM | Already used in core-be. |
| MongoDB client | Mongoose | Already used in calendar-be. |
| Redis client | ioredis | Already used in arm-session. |
| Kafka client | KafkaJS | Already used platform-wide. |
| Queue inspection | @nestjs/bullmq | Already used in calendar-be. |

### 8.2 Admin Frontend (`arm-admin-fe`)

| Concern | Choice | Reason |
|---|---|---|
| Framework | React 19 + Vite + TypeScript | Identical to calendar-fe and core-fe. |
| Module Federation | @module-federation/vite | Same as calendar-fe. Loads into existing core shell. |
| Component library | @aracreate/test-arm-ui | Shared design system. Visual consistency guaranteed. |
| Charts | Recharts | Already used in core-fe. No new dependency. |
| Data tables | TanStack Table | Industry standard for complex sortable/filterable tables. |
| State management | Zustand | Already used across all ARM frontends. |

### 8.3 Infrastructure Additions

For Phase 1–4, **no new infrastructure** is required. All features are powered by:
- Existing PostgreSQL, MongoDB, Redis
- Existing Prometheus `/metrics` endpoints on each NestJS service
- Existing Docker log output (tailed by admin-be)
- New `arm-admin-be` container
- New `arm-admin-fe` container (or served as MFE remote from core-fe)

For Phase 5, optional infrastructure additions:
- **Prometheus** — for historical time-series metrics storage (30-day retention)
- **Grafana Loki** — for persistent, searchable log storage

These are standalone containers added to `docker-compose.infra.yml` and are entirely optional — Phase 5 features degrade gracefully without them (no history, but current-state views still work).

---

## 9. What We Need to Get Started

### 9.1 Decisions Required

Before development begins, three decisions need to be made:

**1. Who is the first admin user?**  
The admin role is bootstrapped via a one-time seed script. The email address of the first admin needs to be designated.

**2. Where will the Admin Portal be hosted?**  
Option A: As a path on the existing domain — `arametrics.app/admin`  
Option B: On a separate subdomain — `admin.arametrics.app`  
Both are technically equivalent. The subdomain option is slightly more secure (cookie isolation) but requires an additional DNS entry and TLS certificate.

**3. What is the operator role scope?**  
The proposal distinguishes `admin` (full access) from `operator` (read + limited write). The specific list of operator-permitted actions should be reviewed and agreed before development to avoid mid-build changes.

### 9.2 New Repositories

Two new repositories following the existing ARM repository structure:
- `arm-admin-be` — NestJS service, same structure as `arm-core-be`
- `arm-admin-fe` — React + Vite + MFE remote, same structure as `arm-app-calendar-fe`

### 9.3 Infrastructure Changes (Minimal)

The following changes to existing deployment configuration are needed:

| Change | What | When |
|---|---|---|
| PostgreSQL | Add `role` column to `users` table | Before Phase 1 development |
| PostgreSQL | Create `audit_logs` table in new schema | Before Phase 1 development |
| PostgreSQL | Create read-only DB user for admin-be | Before Phase 1 development |
| MongoDB | Create read-only DB user for admin-be | Before Phase 2 development |
| Kong | Add `/api/admin/*` route | Before Phase 1 deployment |
| arm-session | Add `ADMIN_BE_URL` to `SERVICE_ENV_MAP` | Before Phase 1 deployment |
| docker-compose | Add `arm-admin-be` and `arm-admin-fe` service definitions | Before Phase 1 deployment |
| repos.conf | Add new repos to deploy makefile | Before Phase 1 deployment |

### 9.4 Team

- **1 backend engineer** — arm-admin-be development
- **1 frontend engineer** — arm-admin-fe development (can be same engineer in later phases)
- **Code review** from existing team for security-sensitive features (role assignment, session revocation)

---

## 10. Success Criteria

The Admin Portal is considered successful when the following are true:

### By end of Phase 1
- [ ] An admin can disable a user account from the web UI in under 30 seconds
- [ ] Every admin action is recorded in the audit log with no exceptions
- [ ] No database terminal access is needed for any user management task
- [ ] Regular users cannot access any admin route or endpoint

### By end of Phase 2
- [ ] The status of all services is visible on a single screen without SSH
- [ ] A spike in error rate is visible within 60 seconds of it occurring
- [ ] Calendar sync failures are surfaced in the portal before a support ticket is raised
- [ ] An operator can identify the cause of a sync failure without inspecting the database directly

### By end of Phase 3
- [ ] An engineer can trace a request across all services by correlation ID without SSH
- [ ] All `invalid_grant` errors across all calendar accounts are visible in one place

### By end of Phase 4
- [ ] A disabled account is fully locked out (sessions revoked) within 60 seconds of the disable action
- [ ] Auth anomalies (repeated failed logins) are visible in the security dashboard

### By end of Phase 5
- [ ] A daily operational summary email is delivered to the admin team automatically
- [ ] The audit log can be exported for any date range for compliance purposes

---

## 11. Risks and How We Will Handle Them

### Risk 1 — Admin Bootstrap Circular Dependency
**The problem:** The admin portal enforces that only admin users can assign the admin role. But initially no one has the admin role.  
**How we handle it:** A one-time CLI seed script (`make seed-admin EMAIL=...`) directly sets the role in PostgreSQL. This is a deployment-time operation, documented, and runs once.

### Risk 2 — Disabled Account Takes Up to 15 Minutes to Take Effect
**The problem:** JWTs are valid for 15 minutes. A disabled user may continue making API calls until their token expires.  
**How we handle it:** When a user is disabled, their user ID is added to a Redis deny-list with a 15-minute TTL. Kong's JWT guard (or session service) checks this list. Effective lockout becomes near-immediate.

### Risk 3 — Admin Portal Becomes a High-Value Attack Target
**The problem:** Concentrating privileged operations in one place makes that surface attractive to attackers. A compromised admin account could affect all users.  
**How we handle it:** Admin sessions have a shorter TTL than regular sessions (configurable, default 2 hours). Admin login triggers an audit log entry. Suspicious admin activity (unusual IP, high-volume actions) surfaces in the security dashboard. MFA is planned for Phase 5.

### Risk 4 — Heavy Database Queries From Admin Portal Affecting Production Performance
**The problem:** Admin reporting queries (user counts over time, sync history aggregations) could slow down the same databases used by production traffic.  
**How we handle it:** Admin-be connects with a read-only database user. All admin queries include row limits and are executed with lower connection priority where possible. Expensive aggregation queries are rate-limited and scheduled during low-traffic windows in Phase 5.

### Risk 5 — Log Volume Growing Faster Than Storage
**The problem:** Centralizing logs from 6+ services could create significant storage growth without a retention policy.  
**How we handle it:** The Phase 3 log viewer tails the last N lines only — no persistent log storage added. Phase 5's Loki deployment includes configurable retention (default 30 days). Storage growth is bounded and predictable.

---

## 12. Summary and Next Steps

### What We Are Building

A **secure, role-controlled Admin Portal** integrated into the existing ARM platform that gives the operations team:

- Complete user management from the browser — no database access needed
- Real-time visibility into every service, queue, database, and infrastructure component
- Centralized log access and error tracking
- Active session management and auth security monitoring
- A permanent audit trail of all administrative actions

### How We Are Building It

- As a new section of the existing ARM web application — no separate product, no duplicate login
- Using the exact same technology stack as every other ARM service
- Without restructuring anything that currently exists
- In five focused phases over 15–16 weeks, each delivering working functionality

### Why It Fits the ARM Platform

The Admin Portal does not fight the existing architecture — it extends it. The microfrontend pattern is already proven with the calendar module. The session service session mechanism handles authentication automatically. The Prometheus metrics endpoints and structured JSON logging are already in place. The Admin Portal is the operational layer the platform was designed to support but has not yet built.

---

### Immediate Next Steps

| Step | Owner | Timeline |
|---|---|---|
| Review and approve this proposal | Team lead | This week |
| Decide on subdomain vs. path routing | Team | This week |
| Designate first admin user email | Team lead | This week |
| Create `arm-admin-be` and `arm-admin-fe` repositories | Engineering | Week 1 |
| Write and apply database migration for `role` column | Engineering | Week 1 |
| Create read-only PostgreSQL and MongoDB users | DevOps / Engineering | Week 1 |
| Add admin routes to Kong config | Engineering | Week 1 |
| Begin Phase 1 development | Engineering | Week 1 |

---

*This proposal was prepared based on a full review of the ARM platform codebase, deployment configuration, database schemas, authentication flows, and existing observability infrastructure.*

*araCreate Engineering — June 2026*

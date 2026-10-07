---
title: halle-feedback-widget/DECISIONS
type: decisions
zone: ai
project: halle-feedback-widget
date: 2026-10-02
---
## For future agent
Decision record for [[halle-feedback-widget]]. Each entry: date, decision, why, who. Newest at the bottom. Superseded decisions stay, marked (superseded by #N).

# DECISIONS — halle-feedback-widget

<!-- ## D1 · YYYY-MM-DD · decision title
- Decision:
- Why:
- Options considered:
- Decided by: Vishnu / agent
-->

## D1 · 2026-10-02 · Bare domain sends visitors to the dashboard
- Decision: `/` redirects to `/app`; the existing middleware sends logged-out visitors to `/login`.
- Why: Vishnu wanted apps.b-halle.de to go to login; going via `/app` also lets logged-in staff skip the login page.
- Options considered: redirect `/` straight to `/login` (logged-in users would see login again).
- Decided by: agent, approved by Vishnu (deploy "yes").

## D2 · 2026-10-02 · Do not cancel the old server yet
- Decision: keep the old IONOS M+ server until 2026-10-14 (14 clean days), its Apache log shows no real visitors, a final archive is taken, and the new contract's 30-day money-back window has passed.
- Why: migration plan Part 4 + IONOS condition; the old server is the only fallback.
- Options considered: cancel now (rejected — no final archive, too early).
- Decided by: Vishnu ("ok save all"), on agent recommendation.


## D3 · 2026-10-02 · Classifications: Bug keeps its flow, custom types are Open → Done
- Decision: Bug stays built in (Processing → Fixed → Closed, Tracked items). Team-made types live on a separate Classified page with two steps, Open → Done; managed on Admin → Classifications; hidden, never deleted; type changes recorded in history.
- Why: Vishnu: "only bug goes like that, other need to be separate".
- Options considered: every type through Bug's flow; each type with its own custom steps (more work).
- Decided by: Vishnu, on agent options.

## D4 · 2026-10-02 · Admin flag for managing logins
- Decision: one `users.is_admin` flag; only admins open Users. Partly overrides admin-v2-spec §1.4 ("no roles"); recorded in repo agent-rules §5.
- Why: if every login could manage logins, the client's login could disable anyone.
- Options considered: every login manages logins.
- Decided by: Vishnu, on agent recommendation.

## D5 · 2026-10-03 · 100% araCreate conventions, including snake_case on the live contract
- Decision: snake_case for every name the project owns, including the widget↔server JSON and saved string keys (migration 0015). Only library/platform names are exempt.
- Why: Vishnu: "it should follow the conventions 100%".
- Options considered: internal names only, with the live keys planned later (agent's recommendation); only new code.
- Decided by: Vishnu. Cost: server, widget and migration must deploy together.

## D6 · 2026-10-03 · Seven feature commits rebuilt from a backup, then pushed
- Decision: work split into one commit per concern (users screen + own settings merged: too intertwined), each checked on its own; pushed to `dev` on Vishnu's "push".
- Why: git conventions — one concern per commit, explicit instruction to push.
- Decided by: Vishnu ("yes", "push").

## D7 · 2026-10-05 · Closed means checked, not "won't fix" — reverses admin-v2-spec
- Decision: a bug's Closed status means a stakeholder checked the developer's fix on the live site, not "not being fixed" as `admin-v2-spec.md` originally said. Steps are now enforced in order: Processing → Fixed → Closed; closing a bug nobody marked fixed is refused server-side, not just hidden in the UI.
- Why: Vishnu corrected the agent's own wrong reading of "Closed" mid-conversation — the original spec's wording was simply inaccurate to how the team actually uses it.
- Options considered: none — this is a factual correction of an existing document, not a tradeoff.
- Decided by: Vishnu, spec text updated in the same commit (`ab8ad7f`).

## D8 · 2026-10-05 · This project's STATE.md, not the repo, is the source of truth for "which server is production"
- Decision: any agent working on this project — with or without viOS loaded — checks this project's `STATE.md` "Now" section before touching any real server, reading or deploying. If the repo's own docs disagree with STATE.md, STATE.md wins until proven otherwise by directly checking the server. The repo's docs are patched with warning banners (commit `f5f12f0`) but are not trusted to stay current on their own.
- Why: a 2026-10-05 session working from the repo alone deployed to the retired old server (212.227.213.174) because the repo's "read this first" doc (`docs/session-handover.md`) still called it live, three weeks after the real migration. This file already had the correct server, unchanged, the whole time — the session simply never checked it.
- Options considered: rely on fixing the repo's docs alone (rejected — still a single point of failure if a future doc update is missed again); make the repo's CI/tooling block a deploy to the wrong IP automatically (not attempted, more work than the actual gap needed closing today).
- Decided by: Vishnu ("clear that should not happen again, save all info"), agent executing.

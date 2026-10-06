---
title: aracreate-server/DECISIONS
type: decisions
zone: ai
project: aracreate-server
date: 2026-10-05
---
## For future agent
Decision record for [[aracreate-server]]. Each entry: date, decision, why, who. Newest at the bottom. Superseded decisions stay, marked (superseded by #N).

# DECISIONS — aracreate-server

## D1 · 2026-10-05 · Add Kishor's SSH key to root
- Decision: Appended Kishor's ed25519 public key (comment `kishor-aracreate`) to root's `authorized_keys` on `212.227.213.174`, giving him root SSH access.
- Why: Vishnu asked to give Kishor, an araCreate teammate, root access to the server.
- Options considered: IONOS web console (not available — no account login on hand); password-based SSH (rejected by the server itself, key-only).
- Decided by: Vishnu

---
title: aracreate-server/STATE
type: state
zone: ai
project: aracreate-server
as_of: 2026-10-05
updated_by: claude-code
status: active
---
## For future agent
Current state of [[aracreate-server]]. READ THIS FIRST when resuming. Always true "as of" the date above. Update at every handoff.

# STATE — aracreate-server

## Now
- Server: hostname `my-vps`, IP `212.227.213.174`. Root login is SSH-key only (password auth disabled).
- Root's `authorized_keys` has 2 entries: Vishnu's Mac key, and Kishor's key (comment `kishor-aracreate`) — added 2026-10-05.
- Root password exists but is **not stored here** (viOS rule: no secrets in vault). Vishnu has it; it was shared in a chat session, so consider rotating it.
- A different IP, `217.160.93.75`, was tried first by mistake (or is unrelated) — it rejected the password entirely over SSH (publickey only, no matching key). Not the same server. Unresolved what it is.

## Next (in order)
1. Confirm with Kishor that he can log in with his own private key.
2. Consider rotating the root password on `212.227.213.174` since it was shared in a chat.
3. Clarify what `217.160.93.75` is, if anything, to Vishnu.

## Blockers / questions for Vishnu
- What is `217.160.93.75`? Not reachable with credentials on hand — may be a typo or an unrelated server.

## How to run / test
- `ssh (secret removed)` — works directly for anyone whose public key is in `authorized_keys` (no password needed).

## Key files
- n/a (remote server, no local repo)

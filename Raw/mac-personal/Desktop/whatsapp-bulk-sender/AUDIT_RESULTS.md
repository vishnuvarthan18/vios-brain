# AUDIT_RESULTS.md — Pre-launch hardening (2026-08-05)

## 1. AI / tooling attribution cleanup

| Check | Result | Action |
|-------|--------|--------|
| Git commit trailers `Co-authored-by: Cursor` | Present on all prior commits | Squashed to one commit (`c10d420`) via `git commit-tree` (bypasses IDE trailer injection); force-pushed to `main`. GitHub API confirms `has_cursor: false`. |
| Repo About/description | `"WhatsApp Bulk Sender desktop app"` — clean | No change needed |
| Docs mentioning Cursor / Claude / “agent” | Found in planning files + RESULT/BUILD_LOG | Deleted `CURSOR_BUILD_PLAN.md`, `START_HERE_FOR_CURSOR.md`, `PROJECT_CONTEXT.md`, obsolete `PRODUCT_SETUP.md` / `product-details.txt`; scrubbed remaining docs |
| Code comments / package.json author | Clean (`author: Vishnu`) | No change |
| CSS `cursor:` / npm `*-agent` packages | False positives (CSS pointer; HTTP proxy libs) | Left alone |

## 2. Production gaps — checked, fixed, or accepted

### CSV edge cases

| Case | Before | After |
|------|--------|-------|
| Empty file | Vague / could throw | Clear error: “File is empty or has no data rows.” |
| Missing Phone column | Error, OK | Clearer message listing accepted aliases |
| Misnamed Mobile/Number | Already accepted | Kept |
| Empty / malformed phones | Skipped quietly-ish | Per-row errors; 10–15 digit rule |
| Duplicate phones | Last write could collide on UNIQUE | First kept; later rows reported as duplicate errors |
| Non-Latin names | Already worked | Confirmed with Tamil sample in audit script |
| 1000+ rows | Untested | 1200-row parse verified (no crash) |

**Fixed in:** `src/lib/tags.js`, Contacts UI error list (shows up to 40 rows).

### WhatsApp disconnect mid-campaign

| Before | After |
|--------|-------|
| No live re-check; disconnect could leave loop confused | Connection checked before every send; disconnect callback stops campaign; remaining messages marked **failed** (never **sent**); race after `sendMessage` marks unconfirmed as failed |

**Fixed in:** `electron/sender.js`, `electron/whatsapp.js`, `electron/main.js`.

### sql.js persistence

| Before | After |
|--------|-------|
| `writeFileSync` after each write (good) but non-atomic | Atomic write: `.tmp` + `fsync` + `rename` |
| Corrupt DB could crash boot | Corrupt file renamed to `.bak`; fresh DB created |
| Force-quit mid-campaign left status=`running` | On launch, `interruptStaleCampaigns()` marks campaign `interrupted`; already-**sent** kept; unfinished → **failed** (no auto-resend) |

**Verified by:** `scripts/audit-checks.js` (close DB / reopen / interrupt).

### License edge cases

| Case | Result |
|------|--------|
| System clock set backward | **Accepted limitation** — offline HMAC cannot detect; documented in README |
| Cached license deleted | Falls back to license screen |
| Corrupt / garbage license row | Cleared; license screen with clear message (no crash) |

### Crash resilience (mid-broadcast)

| Behavior | Status |
|----------|--------|
| Force-quit | Campaign marked interrupted on next launch; sent messages stay sent; no automatic resume/resend |
| Relaunch | Does not restart the old campaign |

### Uninstall / reinstall

| Behavior | Status |
|----------|--------|
| Removing the app binary | Does **not** delete Electron userData (license + WA session remain) |
| Mitigation | Settings → **Reset all local data**; documented in README |

### Privacy / log hygiene

| Check | Result |
|-------|--------|
| Baileys logger | `silent` |
| Console | Errors log message strings only, not message bodies |
| UI activity log | Phone numbers **masked** in broadcast log lines (`9198…10`) |
| Status table | Still shows full numbers (needed for the operator) — not written to a shipped log file |

## 3. Windows `.exe` execution

**Open risk — not runtime-tested on a Windows machine.**  
The NSIS installer was built and downloaded from CI (`WhatsApp Bulk Sender Setup 1.0.0.exe`), but this environment is macOS-only with no Windows VM. Packaging success ≠ launch/smoke test. Treat Windows as “built, unsigned, unexecuted” until someone installs it once on a real Windows PC.

## Audit automation

```bash
cd whatsapp-sender-app && node scripts/audit-checks.js
```

All three suites passed: CSV, DB persist/interrupt, disconnect mid-campaign.

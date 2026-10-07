# W2D — Session Log, 2026-08-21 (Maestro E2E test plan drafted)

Continuation of `claude/SESSION_LOG_2026-08-20.md`. Short session — no
code changes, no dev-agent run happened yet. This log exists so the next
session (or tomorrow's test run) has the drafted task ready to go.

---

## What happened today

User wants full-app automated E2E testing before going further. Decided
on **Maestro** (https://maestro.mobile.dev) for React Native/Expo E2E,
run against the Android emulator + Firebase emulator + existing seed data.

A complete dev-agent task prompt was drafted covering:

- Maestro CLI install + ADB connectivity, documented in a new
  `MAESTRO_SETUP.md`
- `.maestro/` flow files for: OTP sign-in, new-signup category+role
  migration screen, pre-migration account migration screen, Manufacturer
  post-gating (sell-used allowed, requirement blocked, Catalog allowed),
  Vendor post-gating (requirement allowed, sell-listing blocked, Catalog
  blocked), Manufacturer responding to a Vendor requirement + reveal
  screen trust-context assertions, Vendor confirming NO general Needs feed
  is shown, per-business reveal-cap dedupe (same business revealed twice
  from different listings ≠ double charge, per DECISIONS.md §6), public
  profile screen view, Settings navigation through to (not completing)
  Delete Account
- Each flow must assert on real visible content, not just "no crash"
- Run against Firebase emulator + `scripts/seed.mjs`, extending seed data
  minimally if it lacks both a Vendor and Manufacturer fixture with known
  credentials
- New npm script `test:e2e`
- Agent must show the full flow list and confirm Maestro + emulator
  connectivity BEFORE writing all flow files (this session's requested
  checkpoint)
- If a flow fails on a real app bug, log it to `E2E_FINDINGS.md` and leave
  the flow in place as a regression test — do not silently patch app code
- Commit + push once suite runs and results are shown

**This session (Cowork) is not connected to the actual `w2d-app` repo or
an Android emulator** — it can review/refine prompts and read project
docs, but cannot execute the dev-agent task itself. The task must be
pasted into the actual dev agent (Cursor) working in the real repo.

### Three preconditions flagged before running tomorrow

| Item | Risk if skipped |
|---|---|
| Android emulator/device up and ADB-reachable | Maestro stalls silently with no target |
| Firebase emulator suite running with seed data loaded | Every flow fails at first Firestore call |
| A real "test-OTP" flow already exists from manual testing | If it doesn't actually exist yet, the first flow (OTP sign-in) blocks everything else — confirm before pasting the prompt, or amend step 2 to read the OTP from the Emulator UI (`127.0.0.1:4000/auth`) per the standing gotcha in DECISIONS.md §13 |

---

## ⚠️ FIRST THING NEXT SESSION (tomorrow's test run)

1. Before pasting the dev-agent prompt, confirm the three preconditions
   above are actually true in the real environment — don't assume.
2. Paste the Maestro setup + flow-writing task (full text is in this log's
   originating chat, or can be re-derived from the bullet list above —
   ask if it needs to be reconstructed verbatim).
3. **Review step 7's output (planned flow list + Maestro/emulator
   connectivity confirmation) before letting it write all flow files** —
   this was an explicit checkpoint the user asked for, not optional.
4. After the suite runs: read `E2E_FINDINGS.md` if it was created — any
   real bugs found there are new information, not yet reflected in
   DECISIONS.md or prior session logs. Triage those before touching
   anything else.
5. Still outstanding from 2026-08-20's overnight run review (unresolved,
   carried forward): confirm Phase 3 (UI revamp) actually ran, and
   specifically re-verify the §6 reveal-grant security fix's rules-test
   coverage is real, not just claimed. E2E testing is a good moment to
   fold that verification in rather than treating it as separate work.

---

## Standing environment notes (carried from 2026-08-20 log, still true)

- Folder is `~/Desktop/mura/w2d/b2b/w2d-app`.
- Repos: `wedding2day-app` (main app) and `w2d-admin` (admin dashboard) —
  both on GitHub, private, separate remotes.
- Emulator ports: Auth 9099 · Firestore 8080 · Storage 9199 · UI 4000.
  Start with `firebase emulators:start` from `w2d-app`, leave running.
- Dev-client build requires `expo start --dev-client` (Metro) running
  separately for the app to load any JS on device.
- Firebase emulator Auth does not send real SMS — read OTP codes from
  `http://127.0.0.1:4000/auth` during local dev.
- DECISIONS.md last updated 2026-08-20 (role split restored, §0b/§7/§9).
  No changes to DECISIONS.md today — this session was test-planning only.

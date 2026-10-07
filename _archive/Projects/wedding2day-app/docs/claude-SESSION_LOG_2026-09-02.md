# W2D — Session Log, 2026-09-02 (Overnight run execution, APK build, tester login fix, role-mapping concern flagged)

Continuation of `claude/SESSION_LOG_2026-09-01.md`. This session covered: field validation of the role-model audit, execution of the 16-batch overnight run, EAS APK build/distribution, live phone-auth debugging for testers, and an unresolved concern raised by the founder about the Vendor/Manufacturer category mapping.

## What happened

1. **Code audit executed** (Cursor agent, 9 finders + verifiers + completeness critic against commit `a50a4f8`). Found the role model mostly built but with one critical hole: `users.role` was self-writable client-side, defeating every role gate. Also found an `interests` doc-id binding gap, `postType` mutability gap, phone-field standing permission gap, unfiltered Available feed, and Play Store release-gate items incomplete. Full detail in `claude/AUDIT_2026-09-01_ROLE_SECURITY.md`.

2. **Overnight run executed** — 16 batches (security fixes 1-4, gaps 5-6, release-gate prep 7-8, w2d-admin parity 9, test run 10-11, Gluestack UI migration 12-16). Result: 15/16 complete, 1 blocked (visual QA on UI migration — blocked on local JDK version mismatch + missing dev-client rebuild, not yet verified visually). Two deliberate deviations from literal instructions, both logged and reasoned in `E2E_FINDINGS.md`/`OVERNIGHT_RUN_LOG.md`:
   - Batch 1: bound `role` to `category` instead of fully blocking self-update (full block would have stranded pre-role accounts with no server-side fix available, Blaze still blocked).
   - Batch 5: skipped literal query-level Available feed filter (Firestore can't express the needed null-or-match condition safely against unbackfilled data).
   - Found and fixed 3 stale tests that had been asserting the phone-write hole as correct behavior.
   - `test:emulated` wrapper added (`firebase emulators:exec`) so `npm test` is self-contained; `firebase-tools` pinned as an explicit devDependency (exact version, not caret).

3. **Firestore rules deployed to production** (`wedding2day-a99ea`) — the critical role-lockdown fix is now live, not just committed.

4. **EAS preview APK built and distributed.** `eas.json` preview profile now explicitly declares `buildType: apk` (was previously an implicit default). Build: commit `6edaa41`, SDK 54, expires 2026-09-16. Link: https://expo.dev/accounts/vishnu18/projects/w2d/builds/3f8bb6ba-30d9-470e-a774-c8cec17b4dc2

5. **Tester phone-auth debugging (live, this session):**
   - Phone sign-in provider was fully disabled in Firebase Console — enabled it.
   - Android SHA-1 fingerprint was never registered — pulled from `eas-cli credentials` (preview profile) and added to Firebase Project Settings: `70:95:9A:57:50:D4:4B:2B:69:6F:10:BC:27:5A:96:E1:A7:16:5D:04`.
   - Added 6 Firebase test phone numbers (`(phone removed)`–`99996`, code `123456` each) so testers don't depend on real SMS delivery or the SMS region-policy block.
   - **Result: login now works.** Verified via screenshot — tester reached the "Complete your profile" screen successfully using a test number.

6. **WhatsApp OTP status reaffirmed unbuilt and not attempted this session** — confirmed still blocked on Meta Business verification (status unknown, not investigated further) and the Firebase Blaze billing bug. Agreed direction: ship with SMS OTP for now, treat WhatsApp as a parallel non-blocking track.

## Open items carried forward

- **D1** (from `E2E_FINDINGS.md`): public profile (`§6a`) publishes phone number reveal-free while `§6` spent a full pass bounding reveals elsewhere. Founder decision needed — not yet made.
- **D4**: `W2D_Test_Plan.xlsx` (48 test cases) not found anywhere on the dev machine (exhaustive home-directory search). Either lives elsewhere or was never saved locally. Not yet resolved — `TEST_SHEET_RUN.md` stands in as a partial substitute.
- **Visual QA (Batch 16) still not done** — Gluestack UI migration (batches 12-15) has not been visually verified on a running app. Blocked on JDK 17 install + dev-client rebuild (react-native-svg arrived as a new native module with Gluestack).
- **Firebase Hosting deploy for Privacy Policy URL** — prepared, not deployed to production yet.
- **Contacts migration to production** — mentioned as pending, not run this session.
- **⚠️ FLAGGED, NOT YET DIAGNOSED: founder raised a concern this session that the Vendor/Manufacturer category-to-role mapping (§9's table) is "too wrong" — exact nature of the problem not yet specified.** Founder asked to save this and discuss later rather than resolve now. **Next session must open with this** before any other work — do not assume which row(s) are wrong; ask the founder directly which category/role assignment(s) are the issue.

## Next step

Resume by asking the founder specifically what's wrong with the category/role mapping (§9) — do not guess or pre-emptively "fix" any row. Everything else above (D1, D4, visual QA, hosting deploy, contacts migration) remains queued behind that.

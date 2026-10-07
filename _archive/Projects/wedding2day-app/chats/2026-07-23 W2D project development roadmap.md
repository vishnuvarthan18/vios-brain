---
tags: chat
date: 2026-07-23
source: Claude personal account
uuid: efe2bea3-5318-4333-bc83-aa99657cf00b
---
# W2D project development roadmap

## Summary
**Conversation Overview**

The person is resuming work on a project called W2D, a mobile application built with React Native (Expo), Firebase (Auth/Firestore/Storage/FCM), NativeWind, and Cursor Pro. The session began with the person listing four sequential tasks: git backup, updating userType to three options, implementing a post-type picker with four forms, and a two-tab rework.

Claude flagged a critical mismatch: the system prompt describes a FlutterFlow + Supabase + Twilio stack with a separate admin dashboard required for v1, but the actual locked project state is React Native (Expo) + Firebase with no admin app in v1 (Firebase Console used directly instead). Claude explicitly stated it would work from the v1.1 locked state, not the system prompt's outdated description. The v1.1 scope includes five post types via a `postType` field on a `listings` collection, three userTypes, and a two-tab Available/Needs structure.

Claude also identified a blocker: `DECISIONS.md` and `PRODUCT_CONTEXT.md` referenced by the person are not present in Project knowledge—only a file labeled `v1` (an old scope note) is attached. Claude requested the person paste or upload `DECISIONS.md` before proceeding. On sequencing, Claude affirmed the person's instinct to do the git backup first, noting that the subsequent three tasks are all destructive reworks of existing working screens. No files were shared and no tasks were completed; the conversation ended at the blocker-resolution step.

## Chat

**Vishnu:** Resuming W2D. Read DECISIONS.md and PRODUCT_CONTEXT.md in Project knowledge.
Next up: (1) git backup, (2) userType → 3 options, (3) post-type picker + 4 forms, (4) two-tab rework.

**Claude:** I can't read those files — Project knowledge here contains only one file (`v1`, the old scope note). No `DECISIONS.md` or `PRODUCT_CONTEXT.md` is attached.

Two corrections before we start:

| Your system prompt says | Actual locked state |
|---|---|
| FlutterFlow + Supabase + Twilio | React Native (Expo) + Firebase (Auth/Firestore/Storage/FCM) + NativeWind, Cursor Pro |
| Admin dashboard = separate FlutterFlow web project, required for v1 | Admin = Firebase Console directly, no admin app in v1 |
| v1 = resale-only, single `listings` collection, 2 userTypes | v1.1 = 5 post types via `postType` field on `listings`, 3 userTypes, two-tab Available/Needs |

I'll work off the v1.1 locked state, not the FlutterFlow prompt.

**Blocker to resolve first:** `DECISIONS.md` lives at `~/Desktop/w2d/DECISIONS.md` on your Mac. It isn't in Project knowledge, so I can't verify current file state against it. Either paste its contents, or upload it here.

**On your 4-item list — one sequencing note:** item (1) git backup should absolutely go first, since (2), (3), (4) are all destructive reworks of existing working screens.

Confirm and I'll give you step 1 of the git backup.

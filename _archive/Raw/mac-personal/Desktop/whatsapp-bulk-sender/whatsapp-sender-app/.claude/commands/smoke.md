---
description: Prepare and check the live-QR smoke test (I run the GUI part)
---

Help me run a safe live WhatsApp smoke test. You cannot launch the GUI or scan the QR — that is my job. You do the prep and checks.

1. Run `npm run verify` to confirm green.
2. Confirm the mock E2E send+suppression path passes (`tests/e2e/mock-campaign.test.ts`).
3. Generate a **safe 20-contact opt-in test CSV** in the correct import format (only numbers I own or have consent for — remind me of this).
4. Print me a step-by-step runbook: start with `npm run dev` (real transport), scan QR, import the test CSV, send one small campaign, and the exact health signals to watch (per-number score, failures, blocks) with clear go/no-go criteria.
5. Do NOT disable pacing/warm-up/caps to "speed up" the test.

---
description: Audit that all consent-first guardrails are still intact
---

Audit the codebase against the hard guardrails in CLAUDE.md and report a pass/fail table. Check that:

1. Pacing, daily caps, and warm-up cannot be set to zero/off (see `src/shared/settingsClamp.ts`); no UI exposes a disable.
2. The suppression/opt-out gate runs on every send path (no bypass).
3. Consent attestation is still required on contact import.
4. No detection-evasion, fingerprint-spoofing, scraping, or number-generation code exists anywhere.
5. Session tokens use `safeStorage`; renderer never imports main/db directly.

Report findings only — do not change code unless I ask.

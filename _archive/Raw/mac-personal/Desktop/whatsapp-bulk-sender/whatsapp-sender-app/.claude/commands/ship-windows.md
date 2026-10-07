---
description: Build a distributable Windows package (unsigned OK for now)
---

Produce a Windows distributable.

1. Run `npm run verify` first — if red, stop and report; do not build on a broken tree.
2. Run `npm run build:win` (electron-builder, `--publish never`).
3. Report the output artifact path and size, and note that it is **unsigned** — users will see a dismissible "Windows protected your PC" SmartScreen warning until an Authenticode cert is added.
4. Do NOT attempt macOS signing or notarization (parked). Do NOT touch backend files. If the build needs a cert or my machine, stop and tell me exactly what you need.

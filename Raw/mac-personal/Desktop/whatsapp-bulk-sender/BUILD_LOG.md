# Build log

## SQLite native module (2026-08-05)

`better-sqlite3` failed to compile on Node 24 (missing Python `distutils`). Switched to `sql.js` (WASM). App now uses atomic temp-file + rename persistence after every write.

## WhatsApp linked-device scan during automated QA

QR generation was verified with Baileys. A physical phone scan was not available in the automated environment; Connect/session restore is wired via `useMultiFileAuthState`.

## Windows installer

Built successfully on GitHub Actions (`windows-latest`). Wine cross-build was not needed. See `RESULT.md` for the artifact download URL.

## Publish owner correction

`package.json` publish config was corrected to GitHub user `vishnuvarthan18`. Mac DMG was rebuilt so bundled `app-update.yml` matches.

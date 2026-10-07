# SCRIPTS

Helper scripts, and the `motd` banner printed by `make help`.

| File | Purpose |
| --- | --- |
| `motd` | ANSI Shadow banner of the project name, plus the standard file header |
| `gen-motd.sh` | Regenerates `motd`. Run by `make motd` |
| `set-identity.sh` | One-time placeholder replacement for a new project. **Delete after use** |

Scripts here are for humans running them by hand or for a `make` target to
call. Anything the application needs at runtime belongs in `src/`.

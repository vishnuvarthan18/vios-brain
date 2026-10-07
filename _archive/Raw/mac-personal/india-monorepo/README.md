# India Data Platform

One repository for the whole platform.

| Folder | What it is |
|---|---|
| `core/` | Shared database, API and storage (Postgres + PostGIS) |
| `protected-areas/` | Protected-areas harvest engine and the map site |
| `docs/` | Platform plan, decisions log, next-phase plan |
| `engines/culture` `extinct` `forest` `geo` `laws` `species` `water` | The other data engines |
| `ops-console/` | Private admin dashboard |

Each folder keeps its own README, Dockerfile and tests. The full git history of
every former repository is preserved (see `git log -- <folder>`).

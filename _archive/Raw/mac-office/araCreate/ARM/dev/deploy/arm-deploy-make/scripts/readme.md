# SCRIPTS

Helper scripts for platform orchestration. Nine files, and they do not all run in the same place — three need Docker on the host they run from (two `docker exec` into the prod database containers, one `docker run`s the aws-cli image), three run as root on a specific server, and the rest run wherever the workspace is checked out.

Local / workspace:

- `motd` — not a script: the ANSI Shadow banner `make help` prints, referenced as `MOTD` in the root `Makefile`.
- `arm-setup.sh` — the interactive credential-collection wizard `make arm` calls (Phase 3); it needs a TTY, writes `.arm/setup-inputs` and never touches `.env` — the `arm` target does that.
- `check-frontend-conformance.sh` — run by `make fe-check` from a checkout (it walks `$ROOT`, the workspace root); asserts every Module Federation frontend builds its UnoCSS presets from `acPreset` and that each `singleton: true` shared dep resolves to the host's version, two drifts that never fail a build on their own.

Application host (needs the prod containers):

- `backup-postgres.sh` — `docker exec arm-postgres-prod pg_dump` of `arm_core` and `arm_admin` into `backups/postgres/`, so it only works on the host running that container.
- `backup-mongodb.sh` — `docker exec arm-mongodb-prod mongodump` of `arm-calendar` into `backups/mongodb/`; same host requirement, and `MONGO_ROOT_PASSWORD` must be set.
- `backup-sync-s3.sh` — syncs the local `backups/` directory to S3-compatible object storage via the `amazon/aws-cli` image, run by `make backup-sync` on whichever host holds the backups; inert until the `BACKUP_S3_*` vars are set.

Database host:

- `db-provision-native.sh` — creates the ARM databases, roles and monitoring principals natively on the database host (Postgres over the local unix socket as the `postgres` OS user, Mongo on `127.0.0.1:27017`), replacing what the container entrypoints and `infra-wait-db` used to do; it must run as root there and needs a sudoers policy that actually forwards `sudo --preserve-env` (it probes this up front, because a stripped variable would create a role with a blank password). **It has never been run** — review every statement in its header against the live host before the first `apply`.
- `db-firewall.sh` — restricts 5432/27017 to named source IPs by writing rules into the `DOCKER-USER` iptables chain as root, which is the only chain that sees traffic to *Docker-published* ports. Native listeners never traverse it, and the live database host (`arm-htz-srvr-db`) now runs Postgres and MongoDB as native packages with no Docker at all — see the header of `docker-compose.infra.db.yml` — so on that host this script filters nothing despite its name. Source control there is host firewall plus `pg_hba.conf`.

Gateway / VPN host:

- `vpn-wireguard-setup.sh` — provisions the WireGuard server and NAT gateway for the estate and issues client peer configs; run as root on the VPN host, which must already be attached to the `arm-private` Hetzner network. It refuses to overwrite an existing `wg0.conf`, so `add-peer` is the path after the one-time `setup`.

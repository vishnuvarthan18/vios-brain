# Server plan (OVH VPS-2)

**Status 2026-10-03: foundation applied; crawling NOT enabled.** Created `/srv/semmozhi/{app,data,backups}` (owned by `ubuntu`), copied the collector code to `app/`, built the `semmozhi-crawler` image (`vps/docker-compose.server.yml`, crawler only, no ports, 2 GB / 2 CPU limit, profile `tools` so it never starts by itself). Verified: `engines.run list` and `status` work (10 engines, 0 records). No cron, no crawl, no change to the other project's containers, volumes or nginx.

The other project runs its own harvest job on this server (`water-engine-harvest-run`), so schedule ours at a different hour and keep the limits.

## What the server is
OVH VPS-2, Oregon: 4 vCPU, 8 GB memory, 75 GB disk (13 GB used), Ubuntu 26.04. It already runs another project (data platform, ops screens, notes app, password manager, nginx). **None of that is touched.** No Semmozhi code or data is on it yet. Firewall: only 22, 80, 443 open; everything else listens on localhost.

## Rule: two places only
- **Cloudflare:** websites (production, staging) and the private admin site.
- **Server:** the collector and its data. No public Semmozhi pages, no open ports.

## Layout (isolated, additive)
```
/srv/semmozhi/
  app/       code (git clone, branch main)
  data/      crawl output, latest file per source (+ archive/ for older runs)
  backups/   weekly copy of data/ and the dedup databases
```
One Docker Compose project named `semmozhi` with one `crawler` container. Limits: 2 GB memory, 2 CPUs. No published ports. Runs at low priority from cron, nightly, after the user approves the source list.

## Flow
collector (server) -> clean + dedupe + add licence/URL -> website JSON -> pull request into `dev` -> staging -> you check -> Run workflow -> production.
The server never publishes by itself; it only proposes to `dev`.

## Still to do before enabling crawling
1. OVH snapshot taken (panel -> Backup -> Snapshot).
2. Sources chosen (see `DATA_AUDIT.md`: ~10 text sources).
3. Topic engines run once on the laptop and reviewed.
4. `vps/` files updated: they still describe the old "server hosts the website" layout (9 engines, Caddy web container).

## Not changed on the existing server
Any container, volume (`core_pgdata`, `core_miniodata`), nginx config, the notes repo, the password manager.

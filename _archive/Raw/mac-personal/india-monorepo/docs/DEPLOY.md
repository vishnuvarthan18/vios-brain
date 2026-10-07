# Deploying the web console

The new console (`web/`) is deployed by hand-triggered GitHub Actions. Nothing
deploys on push.

It runs as its own container, `ops-web`, on `127.0.0.1:8020` of the VPS —
**next to** the Python ops-console (8010). Nothing existing is replaced until
you change the proxy for `ops.vidivu.in`.

## One-time setup

1. **A dedicated deploy key** (do not reuse your personal key):

       ssh-keygen -t ed25519 -f deploy_key -C "github-deploy-ops-web" -N ""
       ssh-copy-id -i deploy_key.pub ubuntu@40.160.137.239

2. **GitHub secrets** (repo > Settings > Secrets and variables > Actions):

   | Secret | Value |
   |---|---|
   | `VPS_HOST` | `40.160.137.239` |
   | `VPS_USER` | `ubuntu` |
   | `VPS_SSH_KEY` | contents of `deploy_key` (the private file) |
   | `VPS_KNOWN_HOSTS` | output of `ssh-keyscan -t ed25519 40.160.137.239` |

   Then delete the local `deploy_key` files.

3. **On the VPS**, create `~/ops-web/.env` once (copy `web/.env.example`):

   - `POSTGRES_HOST=postgres` (the container joins `core-infra_default`)
   - `POSTGRES_USER=ops_console` and its password — the read-only role
   - `ADMIN_USERNAME`, `ADMIN_PASSWORD_HASH` (escape each `$` as `\$`),
     `SESSION_SECRET` (new random value), `SESSION_EPOCH=1`
   - `chmod 600 ~/ops-web/.env`

## Deploying (automatic)

Push to `main` and the pipeline does the rest:

1. **CI - web** runs: type-check, lint, build, the privacy check against a real
   PostGIS database, and a Docker image build.
2. If (and only if) CI passes, **Deploy - web console** starts by itself. It
   builds the image, sends it to the VPS over SSH, starts it, and then checks
   that `/healthz` reports **the exact commit just built**.
3. If that check fails, the previous image is put back automatically and the
   run fails (you get GitHub's failure email).

Only changes under `web/` or `docs/` trigger it. To redeploy by hand (or deploy
a different commit): GitHub > Actions > **Deploy - web console** > Run workflow.

To confirm what is live: `curl http://127.0.0.1:8020/healthz` on the server
returns `{"ok":true,"sha":"<commit>"}`.

Reach it the same way as the current console:

    ssh -L 8020:127.0.0.1:8020 ubuntu@40.160.137.239   # then http://127.0.0.1:8020

## Switching `ops.vidivu.in` to it (later, by hand)

Point the reverse proxy for that hostname at `127.0.0.1:8020` instead of 8010.
Keep the old console running until you are happy; switching back is one line.

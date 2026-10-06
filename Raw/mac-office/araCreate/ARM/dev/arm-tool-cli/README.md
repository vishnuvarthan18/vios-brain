# ARM CLI

Scaffolds new repositories for the **ARM platform**. It generates the repo layout,
Module Federation wiring, and — critically — the `make descriptor` targets that
plug a new service into `arm-deploy-make`'s compose generation.

This is an ARM-specific tool, not a general-purpose scaffolder. It deliberately
does **not** generate `docker-compose.yml`, Kubernetes manifests, or Kafka/Redis
deployments: those are owned centrally by `deploy/arm-deploy-make`, which renders
every service's compose block from five shared `KIND`+`FE_BE` templates using the
descriptor each repo publishes about itself.

---

## What it generates

| `--kind` | Layout | Descriptor lives in |
| --- | --- | --- |
| `app` | `src/frontend/` + `src/backend/` in one repo, like `arm-admin` and `arm-app-calendar` | each half's own `Makefile` |
| `service` | single package at the repo root, like `arm-service-notification` | the root `Makefile` |

Both kinds get a repo-root `Makefile` carrying the standard target set
(`help`, `install`, `setup`, `dev`, `build`, `test`, `lint`, `lint-fix`,
`release`, `clean`).

**Stack:** React 19 + Vite 7 + UnoCSS (frontend), NestJS (backend), with optional
PostgreSQL or MongoDB. Databases are shared platform infrastructure — a generated
service connects to `arm-postgres-dev` / `arm-mongodb-dev` rather than
provisioning its own container.

---

## Usage

### Interactive

```bash
arm-cli
```

Prompts for project name, type, tech stack, port, database and Git setup.

### Non-interactive

Everything can come from flags, which is what CI and scripted regeneration use:

```bash
# A full ARM app — both halves, both descriptors
arm-cli projects --kind app --db PostgreSQL --port 10003 --frontend-port 10002 -y

# A standalone service
arm-cli audit --kind service --port 4300 -y

# A single backend, no ARM repo layout
arm-cli my-api --type backend --db PostgreSQL --port 4100 -y --skip-git
```

### Options

| Flag | Description |
| --- | --- |
| `--kind app\|service` | ARM repo kind; selects the compose template category |
| `--type frontend\|backend` | Project type for a single-package repo |
| `--service-type core\|service` | Module Federation role for a frontend |
| `--port N` | Port the app (or backend half) listens on |
| `--frontend-port N` | Frontend dev server port; apps only |
| `--core-port N` | Port the Core app runs on; required with `--service-type service` |
| `--db PostgreSQL\|MongoDB\|None` | Database for a backend |
| `-y, --non-interactive` | Run with no prompts |
| `--skip-git` | Skip Git initialisation |
| `--verbose` | Enable debug logging |
| `-h, --help` / `-v, --version` | Help and version |

---

## Development

```bash
make install     # Install dependencies
make clistart    # Run from source via tsx — fastest loop, no build
make build       # tsc + tsc-alias + copy templates into dist
make run         # Build, then run the compiled CLI
make link        # Build and link globally, so `arm-cli` works anywhere
make test        # Run the test suite
make rebuild     # Clean, install, build, link
```

`make build` is three steps for a reason: `tsc` alone leaves `@/*` path aliases
and barrel imports in the emitted JavaScript that Node's ESM resolver cannot
resolve, and `src/templates` is excluded from compilation so it has to be copied
in separately. Running `tsc` on its own produces a `dist` that exits non-zero on
first import.

---

## Registering a generated repo

Generating the repo is not the whole job. A new ARM service also has to be
registered in files this CLI does not own:

- `deploy/arm-deploy-make/repos.conf` — so `make clone-repos` fetches it
- `deploy/arm-deploy-make/services.conf` — one row per half
- `core/arm-core-fe/vite.config.ts` — the Module Federation remote entry
- `core/arm-core-fe/src/routes/app.router.tsx` — the mounted route
- `core/arm-bff/src/bff/proxy/proxy.service.ts` — the `SERVICE_ENV_MAP` entry
- `.env.example` — `VITE_<NAME>_REMOTE_URL` and `<NAME>_BE_URL`

These are edited by hand today. Emitting a reviewable patch for them is planned.

---

## Verifying a generated repo

From the workspace root, against a generated repo added to `services.conf`:

```bash
orchestrator describe        --manifest services.conf --root . --only <name>
orchestrator generate-compose --manifest services.conf --root . --only <name> \
    --out out.yml --env-file .env --templates-dir <templates> --env-scope local
```

`describe` validates the descriptor; `generate-compose` renders the compose block
the platform will actually use. See
`deploy/arm-deploy-make/docs/service-descriptor.md` for the full contract.

---

## Licence

See [LICENSE](LICENSE).

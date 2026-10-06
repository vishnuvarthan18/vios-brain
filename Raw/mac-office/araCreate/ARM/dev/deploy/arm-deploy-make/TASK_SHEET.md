# arm-deploy-make — Task Sheet

> Tasks for `deploy/arm-deploy-make/`.

---

## Status Overview

| Task | Title | Status |
| --- | --- | --- |
| DEV-001 | Frontend containers mount only `src/`, so new `public/` assets 404 until rebuild | ✅ Fixed |
| DEV-002 | `local-dev-rebuild` leaves every dependent container in `Created` when infra is down | ✅ Fixed |
| DEV-003 | Edge and observability layers are only ever exercised on the server | 🟡 Partly closed |
| SEC-006 | Kong's `pre-function` calls `require 'cjson'`, which its Lua sandbox forbids — every `/api/*` request carrying a token 500s | ✅ Fixed |
| SEC-007 | Kong's JWT secret is the literal string `${KONG_JWT_SECRET}` — declarative config does not interpolate env vars | ✅ Fixed |
| DEV-004 | Shell frontend OOM-killed at 1g during Vite dep optimization | ✅ Fixed |
| DEV-005 | `_ensure-*` / `_wait-*` targets print a red ERROR and then `ready`, and exit 0 — DEV-002's bug in ten more places | ✅ Fixed |

---

## DEV-005

**`_ensure-*` / `_wait-*` targets report success after failing — DEV-002's defect in ten more places**

**Status:** ✅ Fixed — verified 2026-09-06. All ten `&& printf "." || \` sites in `make/infra.mk`
now take the braced form `|| { printf ...; exit 1; }`, which runs in the current shell and so
propagates. Confirmed the chain exits 1 rather than 0.

The "Related" half is closed too: `infra-up-db` / `infra-wait-db` / `infra-down-db` /
`infra-status-db` carry both `EXPECT_ROLE := db` and `EXPECT_PROVISIONING := compose`, so they
**refuse** on the live native database host rather than reporting a healthy tier having created
nothing. `tests/test-estate-guards.sh` asserts the target-specific variables actually reach the
guards' recipes rather than assuming Make's propagation.
native database host.

DEV-002 fixed one instance of a swallowed failure in `local-dev-rebuild`. The same pattern is
still present **ten times** in `make/infra.mk`:

```make
docker exec $(CONTAINER_POSTGRES_PROD) psql ... -c "CREATE DATABASE $$db;" > /dev/null 2>&1 && printf "." || \
printf "\n$(RED)ERROR: Failed to create database $$db$(RESET)\n"; \
```

`printf` succeeds, so the `&&`/`||` chain exits **0**. Verified:

```console
$ sh -c 'false > /dev/null 2>&1 && printf "." || printf "\nERROR\n"; echo "chain exit=$?"'
ERROR
chain exit=0
```

The target then runs its final line unconditionally:

```make
@printf " $(GREEN)ready$(RESET)\n"
```

So a failed `CREATE DATABASE` or `CREATE ROLE` prints a red ERROR **and then prints `ready`**, and
`infra-wait-db` goes on to print "Databases are healthy and bootstrapped" and exit 0. The error
scrolls past in green-looking output and nothing downstream notices.

**Why it matters more now.** `arm-htz-srvr-db` runs Postgres and MongoDB as native packages with
no Docker installed, so every `docker exec` in these targets fails immediately. Run against that
host, `infra-wait-db` reports a healthy, bootstrapped database tier having created nothing at all.

**Affected:** `_ensure-postgres-databases`, `_ensure-postgres-databases-prod`,
`_ensure-monitoring-roles`, `_ensure-monitoring-roles-prod`, and the `_wait-*` targets sharing the
idiom — ten occurrences of `&& printf "." || \` in total.

**Fix.** The same change DEV-002 took: braces run in the current shell, so
`cmd || { printf ...; exit 1; }` propagates. A subshell — `cmd || (printf ...; exit 1)` — does
not, which is what DEV-002 was actually about. Decide per call site whether a failure should abort
the target or merely be reported; the monitoring-role steps are deliberately optional when the
password is unset, and that skip path should stay non-fatal.

**Related.** `scripts/db-provision-native.sh` now does this work natively for the database host
and does not use `docker exec`. Redirecting `infra-wait-db` at it, or refusing to run it against a
non-Docker host, is the other half of this fix.

---

## DEV-004

**Shell frontend OOM-killed at 1g during Vite dep optimization**

**Status:** ✅ Fixed — 2026-08-19. `orchestrator/templates/core-frontend.yml.tmpl` raised to `memory: 2g`.

After a component-library upgrade, `core-fe` stopped booting in its container: the browser reported
`504 (Outdated Optimize Dep)` then `Failed to fetch dynamically imported module: .../remoteEntry.js`,
and the shell rendered as a completely blank page. The container log said only:

```
Error during dependency optimization:
The service was stopped
```

`docker inspect arm-core-fe-local --format '{{.State.OOMKilled}}'` settled it: **true**. esbuild's
child process was being killed at the 1g limit mid-optimize, and neither the browser nor Vite ever says
"out of memory".

The shell is the Module Federation host, so its optimize pass covers its own Radix / MF / observability
graph *and* every dependency it shares with the remotes. It idles at ~620 MiB, which left almost no
headroom under 1g for the spike. The remotes were already raised 512m → 1g for the same reason when
Radix landed — this is that problem reaching the host one version later.

**Diagnosis note, because this cost an hour:** a merely stale Vite dep cache produces the *identical*
browser symptom, so `504 (Outdated Optimize Dep)` does not distinguish "clear the cache" from "raise
the limit". Check `OOMKilled` first. Clearing `node_modules/.vite` while esbuild is mid-flight makes
things worse, and neither `docker restart` nor `--force-recreate` helps when the cause is the limit.

---

## DEV-002

**`local-dev-rebuild` leaves every dependent container in `Created` when infra is down**

**Status:** ✅ Fixed 2026-08-19 — `infra-check` gate added, and the swallowed failures now propagate

`make local-dev-rebuild SERVICE=core-fe` recreated core-fe *and* every container that shares a
dependency chain with it, then left five of them (`core-fe`, `calendar-fe`, `admin-fe`, `admin-be`,
`session`) in Docker's `Created` state — object exists, never started — while exiting 0. The dev
environment was down with no error surfaced by the command that caused it.

The root cause was **not** the rebuild itself: the infra stack had no `redis` or `mongodb` container
running at all, so `core-be` failed its health check on 101 Redis reconnects and `calendar-be` on
`getaddrinfo ENOTFOUND arm-mongodb-dev`. Compose then refused to start everything that depends on
them. `make infra-up && make infra-wait`, then `make local-dev`, brought all 12 back healthy.

**Fix, in the two parts this was filed as:**

1. **New `infra-check` target** (`make/infra.mk`) — read-only, fails with `make infra-up && make
   infra-wait` as the remedy, and is now a prerequisite of `local-dev-rebuild`. Deliberately does *not*
   start infra itself: a targeted single-service rebuild shouldn't silently bring up four databases.
   `arm-run`'s inline check keeps its own copy because it *does* auto-start, which is right there and
   wrong here.
2. **The swallowed failures now propagate.** The root cause was shell semantics, not logic:
   `cmd || (printf ...; exit 1)` runs the handler in a **subshell**, so `exit 1` left only the subshell
   and the `;`-chained recipe carried on to the health poll and exited 0. Changed to
   `cmd || { printf ...; exit 1; }` — braces run in the current shell — on both the `build` and `up -d`
   steps. The health poll now also treats `created`/`exited`/`dead` as immediate failures rather than
   polling a container that will never start, dumps the last 20 log lines on either failure path, and
   exits 1 on timeout instead of printing a yellow warning and succeeding.

Also fixed in passing: the timeout message read `Not healthy after s` because `${MAX}` (single `$`) is a
*Make* variable reference, which expands to empty. It needed `$${MAX}`.

**Verified.** With `arm-redis-dev` stopped, `make local-dev-rebuild SERVICE=core-fe` now exits 2, names
the unhealthy container, and — the point of part 1 — aborts before the first `stop`, so all 8 app
containers were left untouched and nothing landed in `created` (diffed container states before/after).
Happy path still exits 0 (`admin-fe`, healthy in 10s). The subshell-vs-brace semantics were confirmed
separately with a minimal two-target Makefile: the old form exits 0 after its handler runs, the new form
exits 2.

Also confirmed while here: rebuilding **does** pick up `public/`, which is the fix for DEV-001's
symptom — `core-fe/public/arm-logo-inverted.svg` went from a 200-with-HTML SPA fallback to
`200 image/svg+xml`, and the auth pages' brand logo renders again.

---

## DEV-001

**Frontend containers mount only `src/`, so new `public/` assets 404 until rebuild**

**Status:** ✅ Fixed 2026-08-19 — `public/` is now bind-mounted alongside `src/` in both local FE templates

`generated/docker-compose.core-fe.yml` bind-mounts one path:

```yaml
volumes:
  - /…/core/arm-core-fe/src:/app/src:ro
```

`public/` is not mounted, so it is whatever the image was built with. Adding a file there on the host
does nothing until `make local-dev-rebuild SERVICE=core-fe`, and the failure is quiet: Vite has no
such file, the SPA fallback answers with `index.html` at **status 200**, and the browser reports a
broken image rather than a 404.

Live example: `core-fe/public/arm-logo-inverted.svg` (which the shared `BrandPanel` requests by that
exact absolute path) renders as broken alt text on `/auth/login` and `/auth/signup`, while
`public/logo.svg` — present when the image was built — serves fine. Nothing is wrong with the code or
the asset; only the container is stale.

**Fix:** `public/` is bind-mounted read-only next to `src/` in both local frontend templates —
`orchestrator/templates/core-frontend.yml.tmpl` (core-fe) and `app-frontend.yml.tmpl` (calendar-fe,
admin-fe). All three FE repos already had a `public/` directory, so no bind mount creates a root-owned
empty one, and all three use Vite's default `publicDir`, so `/app/public` is the path Vite actually
serves. The prod templates are untouched — they build static assets into the image and mount nothing.

Verified by adding a file to `public/` with the stack already running, then requesting it with no
rebuild: `core-fe` went from `200 text/html` (692-byte SPA fallback) to `200 image/svg+xml` with the
exact 108 bytes; `calendar-fe` and `admin-fe` serve theirs under their configured `base`
(`/calendar/`, `/admin/`). Deleting the file propagates too. Pre-existing assets, both MF
`remoteEntry.js` bundles, and SPA routing all still work.

Worth knowing for next time: the misleading `200`-with-HTML answer only appears for paths **inside**
the app's Vite `base`. Request a missing asset outside it and you get an honest `404`, which is why the
remotes looked like they behaved differently at first.

---

## DEV-003

**Edge and observability layers are only ever exercised on the server**

**Status:** 🟡 Partly closed 2026-08-19 — `make edge-up` covers the edge half; observability is still server-only

Local dev runs no Kong, no Caddy, and no Prometheus/Loki/Alloy/Tempo/Grafana. With dev and
staging gone, the first place any change to those lands is the server. The `X-User-Id` JWT fallback in the
auth guards is the active path locally, so Kong's header-injection path in particular has no pre-server
exercise at all.

**Edge half: done.** `make edge-up` runs Kong and Caddy in front of the already-running local stack
(`docker-compose.edge.local.yml`). It mounts the real `kong/kong.yml` and `Caddyfile.prod` unmodified —
local containers are joined to `arm-edge-local` under network *aliases* matching the server's compose
service names (`core-backend`, `calendar-backend`, `admin-backend`, `arm-session`), and they already listen
on exactly the internal ports those configs expect. So there is no second Kong config to drift, which
matters: a hand-written "local edge" file would recreate the very problem deleting dev/staging solved.
It publishes only 80/443/8000/8100, so it runs alongside `make local-dev` with no port conflict.

**It paid for itself immediately** — first run surfaced SEC-006 and SEC-007, two live bugs in the
server's Kong config that no local workflow could ever have hit.

**Note the original framing here was wrong.** "This is a compose invocation rather than a new
environment" does not hold for the *full* stack: `docker-compose.yml` wants ports 3000/3001/10000 (all
taken by the local stack), its service blocks point at `arm-*-prod` infra hostnames baked into eight
repos' descriptors, and `Caddyfile.prod` needs a real domain for ACME. The overlay sidesteps all three;
running the whole server stack locally does not.

**Observability half: still open.** Prometheus/Grafana/Loki/Alloy/Tempo remain server-only. Grafana would
need a host port other than 3000, and `prometheus.prod.yml` scrapes prod container names — the same alias
trick would likely work, but it is untried.

**Also still true:** with no pre-server environment, a schema migration's first real run is on the server.
`make rollback` is images-only. Gate any migration deploy on `make backup-all` + `make restore-postgres-drill`.

A related consequence: with no pre-server environment, a schema migration's first real run is on the
server. `make rollback` is images-only and says so. Gate any deploy carrying a migration on
`make backup-all` plus `make restore-postgres-drill`.

---

## SEC-006

**Kong's `pre-function` calls `require 'cjson'`, which its Lua sandbox forbids**

> Numbered SEC-006/SEC-007 to continue the platform-wide sequence in
> `docs/arm-docs/docs/SECURITY.md` §11, where SEC-001–SEC-005 are already taken by earlier
> findings. SEC-001 there is the unrelated 2026-07-02 `X-User-Id` trust bug.

**Status:** ✅ Fixed 2026-08-19 — `KONG_UNTRUSTED_LUA_SANDBOX_REQUIRES: cjson` added to Kong in both the server stack and the overlay

Every request to `/api/core/*`, `/api/calendar/*` or `/api/admin/*` that carries an `Authorization`
header returns **500**, from Kong itself:

```
[pre-function] sandbox.lua:79: require 'cjson' not allowed within sandbox
```

Kong runs `pre-function` Lua under `untrusted_lua = sandbox` (its default; the server sets no override),
and the sandbox blocks `require`. The X-User-Id injection block in `kong/kong.yml` calls
`require("cjson").decode(json)`, so it throws on every tokened request. Requests with **no** token still
401 correctly, because the Lua is guarded by `if auth then` and never reaches the `require`.

**This has never worked.** Same image (`kong:3.7-ubuntu`), same config, no sandbox override on the
server — verified against `docker-compose.yml`.

Not noticed because the frontends call `/session/*`, which `Caddyfile.prod` routes straight to `arm-session:5000`,
bypassing Kong. Kong's `/api/*` surface is effectively unused today.

**Fix applied:** `KONG_UNTRUSTED_LUA_SANDBOX_REQUIRES: cjson` on Kong's environment in both
`docker-compose.yml` and `docker-compose.edge.local.yml` (they must stay identical or the overlay stops
reproducing server behaviour). The sandbox stays on; exactly one module is allowlisted.

Two alternatives were rejected. `KONG_UNTRUSTED_LUA: on` disables the sandbox wholesale. Rewriting the
Lua to pattern-match `sub` out of the raw JSON avoids the config change but extracts an *identity* claim
with string patterns — a crafted payload such as `{"x":{"sub":"admin"},"sub":"real"}` could yield the
wrong match, which is a worse bug than the one being fixed.

**Verified end to end** with a header-echo container aliased as `core-backend` behind Kong, so the exact
upstream request could be inspected:

```
PATH /v1/health                       ← strip_path removed /api/core
HDR x-user-id: sec006-verify-user     ← the sub claim, injected — works for the first time
HDR x-consumer-username: arm-platform ← jwt plugin authenticated the consumer
```

Zero `not allowed within sandbox` errors in Kong's log afterwards; `/api/core` and `/api/admin` went
200, `/api/calendar` 401s because calendar-be re-verifies the JWT against the real secret (SEC-007).

**The safety argument in `KONG-CONFIG.md` §1 was confirmed while the harness was up.** `pre-function`
does run before `jwt` and does stage `X-User-Id` from an unverified payload — but with a tampered
signature the upstream received *nothing*: request count at the echo was unchanged across both a
no-token and a bad-signature request. The `jwt` plugin halts the access phase before proxying, exactly
as documented.

**Caution — this fix makes SEC-007 reachable.** The `cjson` crash was accidentally fail-closed: every
tokened `/api/*` request 500d, so nothing got through Kong at all. Now that requests flow, a token forged
with the literal `${KONG_JWT_SECRET}` passes Kong (demonstrated above — the verification token was signed
with exactly that public string). Backends still re-verify because `TRUST_PROXY_HEADERS` is unset, so
there is no bypass today, but **SEC-007 should be fixed before this reaches the server.**

---

## SEC-007

**Kong's JWT secret is the literal string `${KONG_JWT_SECRET}`**

**Status:** ✅ Fixed 2026-08-19 — Kong's config is now rendered from a template with the real secret substituted

`kong/kong.yml` declares the consumer credential as `secret: (secret removed) **Kong does not
interpolate environment variables in declarative config**, so the credential's secret is that literal
23-character string — a value committed to this repo and readable by anyone with the source.

Demonstrated with SEC-006 temporarily patched, both against `/api/core/v1/health`:

| Token signed with | Kong's response |
|---|---|
| the real `JWT_ACCESS_SECRET` | **401** — every legitimate core-be token is rejected |
| the literal `${KONG_JWT_SECRET}` | **200** — accepted, and its `sub` injected as `X-User-Id` |

**Severity today: high but not currently exploitable end-to-end.** `TRUST_PROXY_HEADERS` is set nowhere
(not in `.env`, `.env.example`, or any compose file), so the backends fall through to verifying the JWT
themselves against the real secret and reject the forged token. The dual-path guard is what's saving it.

**It becomes a full impersonation bypass the moment `TRUST_PROXY_HEADERS=true` is set on the server** —
which is the documented intent for that flag, and exactly what `SECURITY.md` §3 describes as Kong's
"trusted outright" path. Anyone could then mint a token for any `sub` using a secret that is public.

**Two native mechanisms were tried first and both failed**, verified against `kong:3.7-ubuntu`:
`${VAR}` is loaded as literal text, and `{vault://env/kong-jwt-secret}` is *also* literal —
`jwt_secrets.secret` is not a referenceable field in DB-less mode, so Kong's env-vault backend never
resolves it. Rendering is the only thing that works.

**Fix:** `kong/kong.yml` is now `kong/kong.yml.tmpl`, carrying `{{KONG_JWT_SECRET}}`, and
`make generate-kong-config` (new `make/kong.mk`) renders it to `generated/kong.yml` with the real
`JWT_ACCESS_SECRET` substituted. Both compose files mount the rendered file. Wired as a prerequisite of
`ci-deploy-prod`, `up`, and `edge-up`, so no path can start Kong without rendering first.

The rename is deliberate: a file called `kong.yml` that *looked* like live config, but held an
unsubstituted placeholder Kong used as an HMAC key, is exactly how this survived. `.tmpl` makes it
impossible to mistake.

Three safety properties, each verified:

| Property | How |
|---|---|
| Cannot ship an unsubstituted placeholder | render greps the output for `{{` and fails, deleting the partial file — tested with a deliberately unwired placeholder, exit 2 |
| Cannot ship an empty key | render aborts if `JWT_ACCESS_SECRET` is blank — tested, exit 2, `.env` left untouched |
| Secret cannot be committed or read by others | `generated/` is gitignored (`git check-ignore` confirms), file written under `umask 077` → mode `0600`, and `git grep` finds the value in no tracked file |

Substitution uses bash `${line//search/replace}`, not `sed` — `JWT_ACCESS_SECRET` is base64 and routinely
contains `/` and `+`, which breaks a `s///` command, and sed's replacement text is re-parsed as part of
the command it is spliced into. Same reasoning as `orchestrator/lib/generate-compose.sh`.

`KONG_JWT_SECRET` was removed from both compose files: Kong never read it, and leaving it implied
otherwise.

**Verified — the results invert, which is the whole fix:**

| Token signed with | Before | After |
|---|---|---|
| the real `JWT_ACCESS_SECRET` | 401 | **200** |
| the literal `${KONG_JWT_SECRET}` | 200 | **401** |
| no token | 401 | 401 |

And end to end through a header-echo aliased as `core-backend`, a legitimate core-be-signed token now
yields `x-user-id: sec007-verified` at the upstream with `PATH /v1/health` — SEC-006 and SEC-007 together
mean Kong's identity injection works correctly for the first time.

---

## Notes

**No outstanding tasks otherwise.** The TPL-001–010 template-conformance pass and BUG-001–007 are all complete;
their entries were deleted on 2026-08-18 at the maintainer's request. This file was never committed, so
that text is **not** recoverable from git.

Conventions settled by that work, since they are judgment calls someone will otherwise re-litigate:
`make/` is this repo's `src/` equivalent and keeps its name (Makefiles conventionally live there, and 4
CI workflows plus 13 `include` lines hardcode the path); `.mk`/`.sh` files do carry SPDX headers here,
because they are this repo's actual deliverable rather than incidental tooling; and `make test-self`
(the repo's own Make-logic tests, no Docker, no network) is deliberately separate from `test`/`test-all`,
which delegate to the 7 platform repos' suites.

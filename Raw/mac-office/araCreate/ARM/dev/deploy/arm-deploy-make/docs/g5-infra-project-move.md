# Runbook — moving Redis and Kafka to the `arm-infra-app` project

> **✅ Executed 2026-09-05.** Both containers now run under `arm-infra-app`; Postgres and MongoDB
> stayed under `arm-infra-prod`. Verified: network `91ffe6a5…` adopted with the same id and
> `attached=22`, Redis `DBSIZE` 328 against a 329 baseline (TTL'd keys expire — non-zero is the
> test), `maxmemory` 768 MB, all six Kafka topics, 34 Redis clients across 6 addresses and 3 Kafka
> consumer groups reconnected, 0 restarts, 23 containers healthy, site 200. Kept below as the
> record, including two things that did not go to plan — see *What actually happened*.

**Purpose.** Free the container names `arm-redis-prod` and `arm-kafka1-prod` from compose
project `arm-infra-prod` so `make infra-up-app` can create them under `arm-infra-app`. This is
the gate that currently blocks merging `feat-separate-DB`: `ci-deploy-prod` depends on
`infra-up-app`, and that target fails with a name conflict while the old project holds those
names.

**Verified against the live host 2026-09-04.** Every claim below was checked, not assumed;
where something was tested by experiment the result is quoted.

---

## What actually happened — two corrections

**1 · The Redis eviction policy would have silently regressed.** This runbook said the recreate
"leaves Redis on `noeviction` — exactly what it runs on today. Nothing regresses." That was wrong
on the day: Redis was running `volatile-lru`, applied by a live `CONFIG SET` back on 2026-09-02.
That is runtime-only state and does **not** survive a container recreate, and
`TUNE_REDIS_MAXMEMORY_POLICY` is unset in the host `.env`, so the compose fallback would have put
it back to `noeviction` — undoing a Phase 0 fix, silently, with nothing to show it had happened.

Step 3 was therefore run with the variable passed inline:

```sh
TUNE_REDIS_MAXMEMORY_POLICY=volatile-lru \
  docker compose -p arm-infra-app -f docker-compose.infra.prod.yml up -d redis kafka1
```

Confirmed `volatile-lru` afterwards. **The general lesson: a `CONFIG SET` is not a fix, it is a
reprieve.** Before recreating any container, check whether what it runs today came from its
compose file or from a live command.

**2 · The warning was about volumes, not the network.** The predicted network-adoption warning did
not appear; two volume warnings did:

```
WARN volume "arm_kafka1_prod_data" already exists but was created for project
     "arm-infra-prod" (expected "arm-infra-app"). Use `external: true` to use an existing volume
```

Same adoption behaviour, different resource — and the reassuring reading is the right one: it says
the **existing** volumes were reused, which is exactly what preserves the data. Compose warned and
proceeded. Expect these rather than the network line.

---

## This runbook deliberately does NOT use `docker compose down`

The platform plan's step 1 says to run:

```sh
docker compose -p arm-infra-prod -f docker-compose.infra.prod.yml down
```

**Do not do that yet.** That project owns four containers, not two:

| Container | Service | Needed after this move? |
|---|---|---|
| `arm-redis-prod` | `redis` | moves to `arm-infra-app` |
| `arm-kafka1-prod` | `kafka1` | moves to `arm-infra-app` |
| `arm-postgres-prod` | `postgres` | **still serving production** |
| `arm-mongodb-prod` | `mongodb` | **still serving production** |

`down` removes all four. The native database tier on `10.0.0.5` is **not** serving yet — its
`5432` does not accept connections, and the tasks to bind the engines to the private address and
open the port to the app host are both still open. So `down` would take production's databases
away with nothing to replace them.

Postgres and MongoDB come out during the database cutover, after TLS is enforced and the native
tier answers — not here. This runbook moves the two services that can move safely.

## Two hazards, both tested

**Network adoption — resolved, adoption works.** `arm-infra-network` is owned by project
`arm-infra-prod` and has 23 containers attached, including all the app containers, which consume
it as `external: true`. `docker-compose.infra.app.yml` declares it *non*-external with the same
name, so the question was whether Compose adopts it or refuses on the project-label mismatch.
Tested with two throwaway projects: Compose **warns and proceeds**, exit 0.

```
warning: a network with name <net> exists but was not created for project "<consumer>".
         Set `external: true` to use an existing network
```

Expect that warning during step 3. It is not an error and needs no action. (Tested on Compose
5.5.0; the host runs 5.1.4. The check long predates both.)

**`down` removing an in-use network — resolved, harmless.** Also tested: `down` on the owning
project *attempts* removal, Docker refuses because containers are attached, Compose prints the
misleading `No resource found to remove` and continues. The network survives. This runbook does
not use `down`, but the finding matters for the later cutover step that will.

## Data is preserved

Both compose files pin the same volumes by explicit `name:`, so the recreated containers reattach
to the existing data rather than starting empty:

| Volume | `infra.prod.yml` | `infra.app.yml` |
|---|---|---|
| `arm_redis_prod_data` | `name: arm_redis_prod_data` | `name: arm_redis_prod_data` |
| `arm_kafka1_prod_data` | `name: arm_kafka1_prod_data` | same, via `generated/docker-compose.kafka.yml` |

Redis additionally runs `--appendonly yes`, so its dataset is on disk and survives the swap.
Both volumes exist on the host now. Removing a *container* never removes a named volume.

`arm_kafka2_prod_data` and `arm_kafka3_prod_data` also exist on the host — leftovers from an
earlier three-broker tuning. Only `kafka1` runs today. Leave them; they are inert and deleting
them is a separate, deliberate decision.

## Impact window

Between steps 2 and 3 Redis and Kafka are **down**. Everything else keeps running.

- **Redis** — the session service reads it on every request, so requests fail for the duration.
  Sessions themselves survive (AOF on the reused volume), so users are not logged out; requests
  made during the gap error. `calendar-be`'s BullMQ queues also pause.
- **Kafka** — `core-be` produces welcome and login-OTP emails; `notification` consumes them.
  Sends attempted during the gap fail rather than queue.

Keep the window to the time between two commands. Do not pause between step 2 and step 3.

## Preconditions

1. WireGuard tunnel up, and a shell on the app host:
   `ssh -i ~/.ssh/id_ed25519 deploy@10.0.0.2` — note **`id_ed25519`**; the estate key
   (`hetzner-prod-arm`) is rejected for `deploy` on this host, though it works for `root`.
   `deploy` is sufficient for every command here: they are all docker-group operations, and
   `deploy` has no passwordless sudo on this box anyway.
2. `cd ~/arm-deploy`
3. `generated/docker-compose.kafka.yml` must exist — both compose files include it. The deploy
   workflow wipes `generated/` on every run, so if the last action here was a deploy, run
   `make tune-infra` first. It reads only this host's CPU and RAM and needs no `.env`.
   Present and dated 2026-09-02 as of this writing.
4. Announce the window if anyone is using the platform.

## Correction — step 3 cannot use `make infra-up-app`

**Checked on the host 2026-09-04, after this runbook was first written.** The server's checkout
is on `main` at `687ad0e`, and on `main` neither `docker-compose.infra.app.yml` nor the
`infra-up-app` target exists — both are `feat-separate-DB` work. `make infra-up-app` therefore
fails with "No rule to make target".

That is worse than a wasted command. Run step 2 first and Redis and Kafka are already down when
step 3 fails, leaving only the rollback. **Do not run step 2 before reading this section.**

The circularity is the point: this move is what unblocks the merge, so the merge cannot be what
delivers the file the move needs.

**Use the file that is already there, with the project overridden on the command line.** `-p`
takes precedence over a compose file's top-level `name:`, so `docker-compose.infra.prod.yml`
creates the two services under `arm-infra-app` without any new file being copied to the host:

```sh
docker compose -p arm-infra-app -f docker-compose.infra.prod.yml up -d redis kafka1
```

Verified on the host, read-only, before recommending it:

| Requirement | Check |
|---|---|
| That file resolves both services | `config --services` returns `kafka1 mongodb postgres redis` — Kafka arrives through the same `include:` |
| Data is reattached, not recreated | Both volumes are pinned by explicit `name:` — `arm_redis_prod_data`, `arm_kafka1_prod_data` — identical in both files |
| The network is adopted, not replaced | `arm-infra-network` is declared non-external with an explicit `name:`, the same shape already tested. Expect the adoption warning |
| `-p` beats the file's `name:` | Compose 5.1.4 on this host; command-line project name has precedence |

**One behavioural difference, and it is in the safe direction.** `infra.prod.yml` defaults Redis
to `--maxmemory-policy noeviction`; `infra.app.yml` defaults it to `volatile-lru`. The host sets
neither `TUNE_REDIS_*` variable, so this route leaves Redis on `noeviction` — exactly what it
runs on today. The `volatile-lru` fix arrives when the post-merge deploy's `infra-up-app`
recreates the container from `infra.app.yml`. Nothing regresses; a pending improvement simply
stays pending.

**Then skip `make infra-wait-app` too** — it does not exist on `main` either. The Kafka topics
it would create already exist on the reused volume; confirm them in step 4 rather than creating
them.

---

## Sequence

**Step 1 — record the starting state.** Keep this output; it is what you compare against.

```sh
docker ps -a --format '{{.Names}}\t{{.State}}\t{{.Label "com.docker.compose.project"}}'
docker volume ls --format '{{.Name}}' | grep -E 'redis|kafka'
docker network inspect arm-infra-network --format 'attached={{len .Containers}}'
```

Expect four containers under `arm-infra-prod`, the volumes above, and 23 attached.

**Step 2 — free the two names.** Per-service, so Postgres and MongoDB are untouched. `-s` stops
first; `-f` skips the prompt. This removes containers only, never volumes.

```sh
docker compose -p arm-infra-prod -f docker-compose.infra.prod.yml rm -s -f redis kafka1
```

Confirm both names are gone and the databases are still up:

```sh
docker ps -a --format '{{.Names}}' | grep -E 'arm-redis-prod|arm-kafka1-prod'   # expect no output
docker ps --format '{{.Names}}' | grep -E 'arm-postgres-prod|arm-mongodb-prod'  # expect both
```

**Step 3 — recreate under the new project.** Immediately. See the correction section above for
why this is not `make infra-up-app`.

```sh
docker compose -p arm-infra-app -f docker-compose.infra.prod.yml up -d redis kafka1
```

The network adoption warning appears here. It is not an error and needs no action.

**Step 4 — verify.**

```sh
# both containers back, now under arm-infra-app
docker ps --format '{{.Names}}\t{{.Label "com.docker.compose.project"}}' | grep -E 'redis|kafka'

# the network was adopted, not replaced — same id, attachment count restored
docker network inspect arm-infra-network --format 'id={{.Id}} attached={{len .Containers}}'

# Redis kept its data: a non-zero keyspace means the volume was reused.
# Measured 2026-09-04, immediately before the move: DBSIZE=329, network
# attached=23, six Kafka topics. Compare against those, not against zero.
docker exec arm-redis-prod redis-cli -a "$(grep -E '^REDIS_PASSWORD=' .env | cut -d= -f2-)" DBSIZE

# Kafka kept its topics
docker exec arm-kafka1-prod /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

# the platform still answers
make health
```

A `DBSIZE` of 0 means Redis came up on an empty volume — stop and investigate before proceeding;
that would mean sessions and queued jobs were lost.

## Rollback

Nothing here is destructive, and the reverse is symmetric. To put both services back under the
old project:

```sh
docker compose -p arm-infra-app -f docker-compose.infra.prod.yml rm -s -f redis kafka1
docker compose -p arm-infra-prod -f docker-compose.infra.prod.yml up -d redis kafka1
```

> **Corrected 2026-09-05.** The first line used to name `docker-compose.infra.app.yml`, which is
> `feat-separate-DB` work and **does not exist in the server's checkout** — verified on the host,
> which is on `main` at `687ad0e`. The rollback would have failed with "no such file" at exactly
> the moment it was needed, with Redis and Kafka already down. Step 3 creates the containers from
> `infra.prod.yml`, so the rollback has to remove them with the same file. The main path already
> carried this correction; it had not been carried into the rollback.

Volumes are untouched throughout, so a rollback loses nothing either.

## What this unblocks, and what it does not

**Unblocks:** `make infra-up-app` stops failing on the name conflict, which is what
`ci-deploy-prod` calls. With that clear, `feat-separate-DB` can merge.

**Does not do:** the database cutover. Postgres and MongoDB stay on this host, under
`arm-infra-prod`, serving production. Moving them needs the native tier reachable, TLS enforced
and refusing plaintext, a working native dump-and-restore path, and the connection-identity check
that proves each application is really talking to `10.0.0.5` — none of which this runbook
touches.

**Authelia — resolved 2026-09-05, and not by the merge.** This section used to say the merge
deletes the credentials file on the host. It does not: `ci-deploy-prod` never runs `down` and
omits `--remove-orphans` on purpose, so a service dropped from the manifest keeps running as an
orphan. Authelia was removed by hand instead — container, `arm_authelia` database, volume,
on-disk directory and the three `.env` lines. Backups are in
`~/arm-estate-captures/2026-09-05/authelia-decom/`.

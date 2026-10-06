# orchestrator

Manifest-driven service install/dev engine. Standalone bash, no build step, no Node dependency — matches `arm-deploy-make`'s own bootstrap, which needs nothing but Docker/Make/bash before it starts cloning and building the actual app repos.

This directory has no ARM-specific values in it. Everything project-specific — service list, paths, commands — is supplied by the caller via `--manifest`/`--root`. It's written this way on purpose: it's designed to be relocated into its own repo later without a rewrite, once it's proven out here.

How a service actually *runs* (its Compose/container definition) is not this engine's concern — that's owned by each service's own repo (see `services/arm-service-notification/docker-compose.local-dev.yml` for the current example, pulled into `arm-deploy-make`'s compose file via `include:`). This engine only covers install/dev.

## Manifest format

Tab-delimited text file, one service per row, 4 columns:

```
NAME  PATH  INSTALL_CMD  DEV_CMD
```

- `NAME` — short id used on the command line (`orchestrator dev <NAME>`).
- `PATH` — relative to the `--root` passed by the caller.
- `INSTALL_CMD` / `DEV_CMD` — shell commands run with `PATH` as the working directory. Not tied to any language or package manager.

Lines starting with `#` and blank lines are ignored.

## Subcommands

```
orchestrator install --manifest <path> --root <path> [--only <name>]
orchestrator dev <name> --manifest <path> --root <path>
```

`install` accepts `--only <name>` to install just one service instead of every row.

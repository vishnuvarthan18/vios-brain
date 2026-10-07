# Registering arm-tool-cli in the ARM workspace

This CLI is not yet part of the workspace: it was cloned in by hand and is
absent from `repos.conf`, so `make clone-repos` will not fetch it for anyone
else.

`workspace-registration.patch` in this directory adds it, plus the two doc
entries the platform's own docs expect. Verified with `git apply --check`.

## Applying

From the workspace root (`arm/`, not this repo):

```bash
git apply --check arm-tool-cli/workspace-registration.patch
git apply arm-tool-cli/workspace-registration.patch
```

The workspace root is not itself a git repository — each of the seven repos has
its own history — so `git apply` runs from the root against workspace-relative
paths, and the resulting changes land as uncommitted edits in three different
repos. Commit them separately, in each repo, following that repo's conventions.

## What it changes

| File | Change |
| --- | --- |
| `deploy/arm-deploy-make/repos.conf` | `arm-tool-cli` → `tools/arm-tool-cli` on `dev` |
| `CLAUDE.md` | a row in the key-documents table |
| `docs/arm-docs/docs/ARCHITECTURE.md` | a short "Scaffolding a new service" section |

## What it deliberately does not change

- **`services.conf`** — this is a toolchain member, not a service. It has no
  port, no container, no compose block, and no `make descriptor` target.
- **The directory it lives in.** The patch registers it at `tools/arm-tool-cli`,
  but the repo is currently checked out at the workspace root. Move it to
  `tools/` when you apply this, or edit the path in the patch to match where you
  want it.

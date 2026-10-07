# 08 — CI/CD & Deployment

> Changes to `deploy-production.yml` for multi-server deployment + security hardening.

---

## 8.1 Current (Single Server)

GitHub Actions SSHes into one server and runs everything:
```
SSH → make ci-deploy-prod → infra-up → generate-compose → docker compose up
```

## 8.2 Multi-Server Deploy Sequence

Add secrets: `PROD_DB_SERVER_HOST`, `PROD_OBSERVE_SERVER_HOST`, `PROD_BACKUP_SERVER_HOST` (S2 keeps existing `PROD_SERVER_HOST`).

**Prepare:** rsync deploy files to S1, S2, S3 (each gets what it needs)

**Deploy:**
1. SSH S1: `make infra-up-multi` + `make infra-wait-multi`
2. SSH S2: `make app-deploy-multi IMAGE_TAG=$TAG`
3. SSH S3: `make observe-deploy-multi`

**Verify:** Check containers on all 3 servers (same health-check polling pattern)

**Rollback:** Only S2 (app images) — databases on S1 not touched. Same `make rollback TAG=<sha>` pattern.

---

## 8.3 What's Already Good

- Secrets via `env:` binding (not `${{ }}` interpolation)
- `.env` via `scp`, never inline shell
- Concurrency control (no parallel deploys)
- Automatic rollback on verify failure
- `IMAGE_TAG` is commit SHA (deterministic rollback)

## 8.4 Harden Further

| Action | Why |
|---|---|
| Pin Actions to SHA (`actions/checkout@<sha>`) | Tags can be repointed; SHA is immutable |
| Add `permissions:` block | Least-privilege per job |
| Rotate `PROD_SSH_PRIVATE_KEY` annually | Limit blast radius |
| IP allowlist on SSH | Only GitHub Actions runners + team IPs |

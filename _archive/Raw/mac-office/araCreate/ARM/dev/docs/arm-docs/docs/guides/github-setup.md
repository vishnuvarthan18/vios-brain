# ARM Platform — GitHub Secrets & Variables Setup

This document lists every secret and variable you must configure in GitHub before the deploy workflows can run.

GitHub organises these under:
**Repository → Settings → Secrets and variables → Actions**

There is one named environment in the workflows: `production` — the server. Its secrets and variables are prefixed `PROD_`.

> The `development` and `staging` GitHub environments, and every `DEV_`/`STAGE_`-prefixed secret and variable, became unused on 2026-08-19 when those environments were removed ([DEPLOY-ORCHESTRATION.md](../DEPLOY-ORCHESTRATION.md) §2). Deleting them from GitHub is a manual cleanup step this document does not cover.

---

## How GitHub Actions uses these values

| Workflow | Environment | Branch trigger |
|----------|-------------|----------------|
| `deploy-production.yml` | `production` | push to `main` |

The workflow writes a `.env` file on the server from these values, then runs `make ci-deploy-prod`.

---

## Quick reference — how to generate secrets

```bash
# JWT and session secrets — 32-byte base64 string
openssl rand -base64 32

# AES-256 token encryption key — exactly 32 bytes, hex-encoded
openssl rand -hex 16

# SSH key pair — run on your local machine, add public key to server's authorized_keys
ssh-keygen -t ed25519 -C "github-actions-deploy"
```

---

## Secrets

> Stored encrypted. Never visible after saving. Use for passwords, keys, tokens.

### Server access

| Secret name | Description |
|-------------|-------------|
| `PROD_SSH_PRIVATE_KEY` | Private SSH key GitHub Actions uses to connect to the server. Add the matching public key to `~/.ssh/authorized_keys` on the server. |
| `PROD_SERVER_HOST` | IP address or hostname of the deployment server (e.g. `192.168.1.10` or `server.example.com`). |
| `PROD_SERVER_USER` | SSH username on the server (e.g. `ubuntu`, `deploy`). |

### Database

| Secret name | Description |
|-------------|-------------|
| `PROD_POSTGRES_PASSWORD` | PostgreSQL `arm_core` database password. Matches `POSTGRES_USER` below. |
| `PROD_MONGODB_URI` | Full MongoDB connection string including credentials. Format: `mongodb://(secret removed)@localhost:27017/arm-calendar?authSource=admin` |

> **Infra-only secrets** — used by `docker-compose.infra.yml` to initialise Mongo and Redis.
> Set these in `.env` on the server directly (or add to the workflow if you manage the infra from CI).

| Secret name | Description |
|-------------|-------------|
| `MONGO_ROOT_PASSWORD` | MongoDB root password. Must match the password inside `MONGODB_URI`. |
| `REDIS_PASSWORD` | Redis `requirepass` password. Used by both the infra compose and the app services. |

### JWT and session

| Secret name | Description |
|-------------|-------------|
| `PROD_JWT_ACCESS_SECRET` | Signing secret for short-lived JWT access tokens. Generate: `openssl rand -base64 32` |
| `PROD_JWT_REFRESH_SECRET` | Signing secret for refresh tokens (different from access secret). Generate: `openssl rand -base64 32` |
| `PROD_SESSION_SECRET` | Session secret (kept for safety even though express-session is removed). Generate: `openssl rand -base64 32` |

### Encryption

| Secret name | Description |
|-------------|-------------|
| `PROD_GOOGLE_TOKEN_ENCRYPTION_KEY` | AES-256 key used to encrypt Google OAuth tokens at rest in both PostgreSQL and MongoDB. Must be exactly 32 bytes. Generate: `openssl rand -hex 16`. **⚠️ Not yet in deploy workflows — must be added manually to `.env` on the server and to the workflow `Write .env` step.** |

### Google OAuth

| Secret name | Description |
|-------------|-------------|
| `PROD_GOOGLE_CLIENT_ID` | OAuth 2.0 client ID from Google Cloud Console → Credentials. |
| `PROD_GOOGLE_CLIENT_SECRET` | OAuth 2.0 client secret from Google Cloud Console → Credentials. |

### SMTP (email delivery)

| Secret name | Description |
|-------------|-------------|
| `PROD_SMTP_HOST` | SMTP server hostname (e.g. `smtp.gmail.com`, `smtp.sendgrid.net`). |
| `PROD_SMTP_USER` | SMTP login username / sender email address. |
| `PROD_SMTP_PASS` | SMTP password or app-specific password. |

### GitHub access

| Secret name | Description |
|-------------|-------------|
| `PROD_GITHUB_TOKEN` | GitHub Personal Access Token (PAT) used by the server to clone private repos. Scope: `repo` (read). The deploy workflow injects it into `GITHUB_BASE_URL`. Not needed if repos are public or if you use a deploy key. |

---

## Variables

> Plain text, visible after saving. Use for non-sensitive configuration (URLs, usernames, TTLs).

### Database

| Variable name | Default | Description |
|---------------|---------|-------------|
| `PROD_POSTGRES_USER` | `admin` | PostgreSQL username. |
| `PROD_POSTGRES_DB` | `arm_core` | PostgreSQL database name. |

### JWT TTLs

| Variable name | Default | Description |
|---------------|---------|-------------|
| `PROD_JWT_ACCESS_TOKEN_TTL` | `15m` | Access token lifetime. Short-lived; 15m recommended. |
| `PROD_JWT_REFRESH_TOKEN_TTL` | `30d` | Refresh token lifetime. Stored in httpOnly cookie. |

### SMTP

| Variable name | Default | Description |
|---------------|---------|-------------|
| `PROD_SMTP_PORT` | `587` | SMTP port. Use `587` for STARTTLS, `465` for SSL. |

### GitHub

| Variable name | Description |
|---------------|-------------|
| `PROD_GITHUB_BASE_URL` | Base URL for cloning repos, e.g. `https://github.com/aracreate-group`. |

### Google OAuth callback URLs

These must exactly match the **Authorized redirect URIs** configured in Google Cloud Console.

| Variable name | DEV | STAGE | PROD | Example value |
|---------------|:---:|:-----:|:----:|---------------|
| `PROD_GOOGLE_CORE_CALLBACK_URL` | `https://app.example.com/api/core/auth/google/callback` |
| `PROD_GOOGLE_CALENDAR_CALLBACK_URL` | `https://app.example.com/api/calendar/auth/google/callback` |

### Application URLs

Replace `YOUR_SERVER_IP` or domain with the server's actual address.

| Variable name | Description |
|---------------|-------------|
| `PROD_FRONTEND_URL` | Core frontend root URL, e.g. `https://app.example.com` |
| `PROD_CALENDAR_FRONTEND_URL` | Calendar frontend URL, e.g. `https://app.example.com/calendar` |
| `PROD_CORE_API_URL` | Core backend API URL (via Kong), e.g. `https://app.example.com/api/core` |
| `PROD_CALENDAR_API_URL` | Calendar backend API URL (via Kong), e.g. `https://app.example.com/api/calendar` |
| `PROD_NOTIFICATION_WEBHOOK_URL` | Google push notification webhook URL. **Must be HTTPS on the server.** e.g. `https://app.example.com/api/calendar/events/notifications` |

### Module Federation remotes

| Variable name | Description |
|---------------|-------------|
| `PROD_VITE_CALENDAR_REMOTE_URL` | Calendar MFE remote entry URL, e.g. `https://app.example.com/calendar/remoteEntry.js` |
| `PROD_VITE_CORE_REMOTE_URL` | Core shell remote entry URL, e.g. `https://app.example.com/remoteEntry.js` |

---

## Full checklist — server

GitHub environment name: **`production`** · Branch trigger: `main`

### Secrets
- [ ] `PROD_SSH_PRIVATE_KEY`
- [ ] `PROD_SERVER_HOST`
- [ ] `PROD_SERVER_USER`
- [ ] `PROD_POSTGRES_PASSWORD`
- [ ] `PROD_MONGODB_URI`
- [ ] `PROD_JWT_ACCESS_SECRET`
- [ ] `PROD_JWT_REFRESH_SECRET`
- [ ] `PROD_SESSION_SECRET`
- [ ] `PROD_GOOGLE_TOKEN_ENCRYPTION_KEY` ← generate: `openssl rand -hex 16`
- [ ] `PROD_GOOGLE_CLIENT_ID`
- [ ] `PROD_GOOGLE_CLIENT_SECRET`
- [ ] `PROD_SMTP_HOST`
- [ ] `PROD_SMTP_USER`
- [ ] `PROD_SMTP_PASS`
- [ ] `PROD_REDIS_PASSWORD`
- [ ] `PROD_MONGO_ROOT_PASSWORD`

### Variables
- [ ] `PROD_POSTGRES_USER`
- [ ] `PROD_POSTGRES_DB`
- [ ] `PROD_JWT_ACCESS_TOKEN_TTL`
- [ ] `PROD_JWT_REFRESH_TOKEN_TTL`
- [ ] `PROD_SMTP_PORT`
- [ ] `PROD_GITHUB_BASE_URL`
- [ ] `PROD_GOOGLE_CORE_CALLBACK_URL`
- [ ] `PROD_GOOGLE_CALENDAR_CALLBACK_URL`
- [ ] `PROD_FRONTEND_URL`
- [ ] `PROD_CALENDAR_FRONTEND_URL`
- [ ] `PROD_CORE_API_URL`
- [ ] `PROD_CALENDAR_API_URL`
- [ ] `PROD_NOTIFICATION_WEBHOOK_URL`
- [ ] `PROD_VITE_CALENDAR_REMOTE_URL`
- [ ] `PROD_VITE_CORE_REMOTE_URL`
- [ ] `PROD_MONGO_ROOT_USER` (default value: `arm_admin`)

---

## Reminders before go-live

| Item | Note |
|------|------|
| Never reuse a local-development secret on the server | Generate server secrets fresh; `make env-init` does this. |
| Add Google OAuth redirect URIs in Google Cloud Console | The server's callback URL must be registered on the OAuth client. |
| Use a different Google OAuth client for local development (recommended) | Keeps local traffic isolated from the server's consent screens and quotas. |
| Restrict Google OAuth scopes if not all are needed | Current scopes: `calendar`, `calendar.events`, `user.email`, `user.profile`. |

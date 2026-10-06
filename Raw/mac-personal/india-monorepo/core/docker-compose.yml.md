---
source: personal Mac ~/india-monorepo/core/docker-compose.yml
---

# Core infrastructure for the India data platform (PLAN.md Phase B + C.3).
#
# Deployed to ~/core-infra/ on the VPS. Every published port binds to
# 127.0.0.1 deliberately — Docker's iptables rules are evaluated BEFORE
# ufw's INPUT chain, so a 0.0.0.0 bind would be reachable from the internet
# even with ufw denying everything. See DECISIONS.md D-2. Reach these from
# the Mac over an SSH tunnel.

name: core-infra

services:
  postgres:
    image: postgis/postgis:17-3.5
    container_name: core-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      # Data lives on a named volume, so PGDATA must sit in a subdirectory
      # of the mount point rather than at its root.
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "127.0.0.1:${POSTGRES_HOST_PORT:-5432}:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s
    # 4 GB box shared with MinIO and the API — leave headroom rather than
    # letting Postgres assume it owns the machine.
    command:
      - postgres
      - -c
      - shared_buffers=512MB
      - -c
      - effective_cache_size=1536MB
      - -c
      - maintenance_work_mem=128MB
      - -c
      - work_mem=8MB
      - -c
      - max_connections=60
      - -c
      - random_page_cost=1.1
      - -c
      - log_min_duration_statement=2000

  minio:
    image: minio/minio:RELEASE.2025-09-07T16-13-09Z
    container_name: core-minio
    restart: unless-stopped
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    volumes:
      - miniodata:/data
    ports:
      - "127.0.0.1:${MINIO_API_HOST_PORT:-9000}:9000"
      - "127.0.0.1:${MINIO_CONSOLE_HOST_PORT:-9001}:9001"
    healthcheck:
      test: ["CMD-SHELL", "mc ready local || exit 1"]
      interval: 15s
      timeout: 10s
      retries: 10
      start_period: 20s

  core-api:
    build:
      context: .
      dockerfile: Dockerfile
    image: india-data-core:latest
    container_name: core-api
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
      minio:
        condition: service_healthy
    environment:
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      MINIO_ENDPOINT: http://minio:9000
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
      RAW_ARCHIVE_BUCKET: ${RAW_ARCHIVE_BUCKET:-raw-archive}
      CORE_API_ENV: ${CORE_API_ENV:-production}
      LOG_LEVEL: ${LOG_LEVEL:-INFO}
    ports:
      - "127.0.0.1:${CORE_API_HOST_PORT:-8000}:8000"

volumes:
  pgdata:
    name: core_pgdata
  miniodata:
    name: core_miniodata

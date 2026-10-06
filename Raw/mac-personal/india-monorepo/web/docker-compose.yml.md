---
source: personal Mac ~/india-monorepo/web/docker-compose.yml
---

# The new web console. Runs NEXT TO the Python ops-console (which uses 8010) on
# 127.0.0.1:8020, so nothing existing is replaced until you switch the proxy.
# Same rule as the other services: bind to 127.0.0.1 only (DECISIONS D-2 —
# Docker's iptables rules bypass ufw, so a 0.0.0.0 bind is public).
name: ops-web

services:
  ops-web:
    image: india-ops-web:latest      # loaded by the deploy workflow
    container_name: ops-web
    restart: unless-stopped
    env_file: .env
    environment:
      POSTGRES_HOST: ${POSTGRES_HOST:-postgres}
    ports:
      - "127.0.0.1:${OPS_WEB_HOST_PORT:-8020}:8020"
    networks:
      - core

networks:
  core:
    external: true
    name: core-infra_default

---
source: personal Mac ~/india-monorepo/protected-areas/harvest-engine/docker-compose.yml
---

# Protected Areas engine. Not a long-running service — every service here is
# a one-shot task invoked by a systemd timer (PLAN.md §3.4: systemd timers,
# never GitHub Actions cron).
#
# Joins the core stack's existing network so the engine reaches the core API
# by service name. The engine holds NO database or object-store credentials:
# one per-engine API key is the whole of its authority.

name: pa-engine

services:
  harvest:
    build: .
    image: pa-engine:latest
    profiles: ["task"]
    env_file:
      - /home/ubuntu/core-infra/harvest-secrets.env
    environment:
      CORE_API_BASE_URL: http://core-api:8000
      CORE_API_KEY: ${CORE_API_KEY_PROTECTED_AREAS}
      # Quoted: the default contains a colon, which YAML would otherwise
      # read as a nested mapping.
      HARVEST_USER_AGENT: "${HARVEST_USER_AGENT:-india-data-platform/0.1 (+contact: vishnu@aracreate.group)}"
    networks:
      - core
    command: ["python", "scripts/run_harvest.py"]

networks:
  core:
    external: true
    name: core-infra_default

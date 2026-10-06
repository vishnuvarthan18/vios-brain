---
source: personal Mac ~/india-monorepo/engines/water/docker-compose.yml
---

# Water Systems engine (PLAN.md Phase F). One-shot tasks driven by a systemd
# timer, joining the core stack's network. Like every engine, it holds no
# database or object-store credentials — one per-engine API key is its whole
# authority.
name: water-engine

services:
  harvest:
    build: .
    image: water-engine:latest
    profiles: ["task"]
    environment:
      CORE_API_BASE_URL: http://core-api:8000
      CORE_API_KEY: ${CORE_API_KEY_WATER_SYSTEMS}
      HARVEST_USER_AGENT: "${HARVEST_USER_AGENT:-india-data-platform/0.1 (+contact: vishnu@aracreate.group)}"
    networks: [core]
    command: ["python", "scripts/harvest_nwdp_catalog.py"]

networks:
  core:
    external: true
    name: core-infra_default

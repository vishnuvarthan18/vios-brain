---
source: personal Mac ~/india-monorepo/engines/forest/docker-compose.yml
---

name: forest-engine
services:
  harvest:
    build: .
    image: forest-engine:latest
    profiles: ["task"]
    # Upstream source credentials (DATA_GOV_IN_API_KEY for the FSI ISFR
    # harvest) come from the shared secrets file, exactly as the PA engine
    # takes them. The engine's own authority over the core is still only
    # CORE_API_KEY below — it holds no database or object-store credentials.
    env_file:
      - /home/ubuntu/core-infra/harvest-secrets.env
    environment:
      CORE_API_BASE_URL: http://core-api:8000
      CORE_API_KEY: ${CORE_API_KEY_FORESTS_LAND}
      HARVEST_USER_AGENT: "${HARVEST_USER_AGENT:-india-data-platform/0.1 (+contact: vishnu@aracreate.group)}"
    networks: [core]
networks:
  core:
    external: true
    name: core-infra_default

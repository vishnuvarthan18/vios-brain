---
source: personal Mac ~/india-monorepo/engines/geo/docker-compose.yml
---

name: geo-engine
services:
  harvest:
    build: .
    image: geo-engine:latest
    profiles: ["task"]
    environment:
      CORE_API_BASE_URL: http://core-api:8000
      CORE_API_KEY: ${CORE_API_KEY_MOUNTAINS_GEOGRAPHY}
      HARVEST_USER_AGENT: "${HARVEST_USER_AGENT:-india-data-platform/0.1 (+contact: vishnu@aracreate.group)}"
    networks: [core]
networks:
  core:
    external: true
    name: core-infra_default

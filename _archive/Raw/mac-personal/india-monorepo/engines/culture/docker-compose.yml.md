---
source: personal Mac ~/india-monorepo/engines/culture/docker-compose.yml
---

name: culture-engine
services:
  harvest:
    build: .
    image: culture-engine:latest
    profiles: ["task"]
    # DATA_GOV_IN_API_KEY for harvest_fra_jk.py, same shared-secrets file
    # the Forests/PA engines already use. The engine's own authority over
    # the core is still only CORE_API_KEY below.
    env_file:
      - /home/ubuntu/core-infra/harvest-secrets.env
    environment:
      CORE_API_BASE_URL: http://core-api:8000
      CORE_API_KEY: ${CORE_API_KEY_TRIBAL_CULTURE}
      HARVEST_USER_AGENT: "${HARVEST_USER_AGENT:-india-data-platform/0.1 (+contact: vishnu@aracreate.group)}"
    networks: [core]
networks:
  core:
    external: true
    name: core-infra_default

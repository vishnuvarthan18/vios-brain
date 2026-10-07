---
source: personal Mac ~/india-monorepo/engines/species/docker-compose.yml
---

name: species-engine
services:
  harvest:
    build: .
    image: species-engine:latest
    profiles: ["task"]
    environment:
      CORE_API_BASE_URL: http://core-api:8000
      CORE_API_KEY: ${CORE_API_KEY_LIVING_SPECIES}
      HARVEST_USER_AGENT: "${HARVEST_USER_AGENT:-india-data-platform/0.1 (+contact: vishnu@aracreate.group)}"
    networks: [core]
    command: ["python", "scripts/harvest_species_checklists.py"]
networks:
  core:
    external: true
    name: core-infra_default

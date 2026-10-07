---
source: personal Mac ~/Downloads/tamil_harvest/v2/vps/docker-compose.yml
---

# Run from the project root:  docker compose -f vps/docker-compose.yml build
services:
  crawler:
    build: { context: .., dockerfile: vps/Dockerfile }
    image: semmozhi-crawler
    volumes:
      - /srv/semmozhi/data:/data          # all crawled data + state lives here, NOT in git
      - /srv/semmozhi/site:/site          # the built website (served by Caddy)
    restart: "no"
    profiles: ["tools"]                    # started on demand by cron: `docker compose run --rm crawler ...`
  web:
    image: caddy:2
    restart: unless-stopped
    ports: ["80:80", "443:443"]
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - /srv/semmozhi/site:/srv/site:ro
      - caddy_data:/data
volumes:
  caddy_data:

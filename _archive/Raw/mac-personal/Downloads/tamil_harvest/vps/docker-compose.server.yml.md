---
source: personal Mac ~/Downloads/tamil_harvest/vps/docker-compose.server.yml
---

# Semmozhi collector on the shared OVH server. Crawler only: NO ports, NO web server (the website is on Cloudflare).
# Everything lives under /srv/semmozhi. Run from /srv/semmozhi/app:
#   docker compose -f vps/docker-compose.server.yml build
#   docker compose -f vps/docker-compose.server.yml run --rm crawler python3 -m engines.run list
name: semmozhi
services:
  crawler:
    build: { context: .., dockerfile: vps/Dockerfile }
    image: semmozhi-crawler
    container_name: semmozhi-crawler
    volumes:
      - /srv/semmozhi/data:/data
    restart: "no"
    profiles: ["tools"]        # only runs when asked; never starts by itself
    mem_limit: 2g
    cpus: 2
    network_mode: bridge
    security_opt: ["no-new-privileges:true"]

---
title: vios-local-troubleshooting
type: wiki
zone: ai
date: 2026-10-01
status: solid
sources: [install session 2026-10-01]
tags: [wiki, vios, troubleshooting]
---
## For future agent
Problems hit while installing viOS locally on Vishnu's Mac and the fixes that worked (as of 2026-10-01).

# viOS local — troubleshooting

- **Command pasted with `# comment` fails in zsh** ("unknown file attribute") → paste commands without inline comments.
- **vios-api keeps restarting, "unable to open database file"** → data volume owned by wrong user. Fix:
  `docker run --rm -u 0 -v vios-local_vios-api-data:/data --entrypoint sh vios/vios-api:local -c "chown -R $(id -u):$(id -g) /data" && docker restart vios-local-api`
  (now automatic via `vios-api-init`).
- **No chat password / create-user prints nothing** → temporarily set `ALLOW_REGISTRATION: "true"` in local/docker-compose.yml, `local/bin/vios-local up`, register at http://chat.localhost:8088/register, then set it back to false.
- **Chat says 401 / "x-api-key header is required"** → no AI key. `local/bin/vios-local keys add gemini` (free key at aistudio.google.com/apikey).
- **OpenCode won't start** → `npm approve-scripts opencode-ai && npm install -g opencode-ai`.
- **Check health** → `local/bin/vios-local status`; logs → `docker logs --tail 40 vios-local-<service>`.

## Related
- [[vios]] · [[personal-ai-os-research]]

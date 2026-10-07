# DOCS

Supporting documentation for platform orchestration — architecture notes, observability stack design, and reference material. The written record around the deploy tooling, not the tooling itself. Group related files into subfolders as they grow.

| File | Purpose |
|---|---|
| [service-descriptor.md](service-descriptor.md) | The descriptor contract each service repo implements, and how compose blocks are generated from it |
| [../guard/README.md](../guard/README.md) | The estate gateway's configuration and its rebuild procedure. Split DNS, NAT and ufw; deliberately holds no keys |
| [g5-infra-project-move.md](g5-infra-project-move.md) | Runbook: move Redis and Kafka from the `arm-infra-prod` compose project to `arm-infra-app`, which is what unblocks `infra-up-app` and the `feat-separate-DB` merge. Explains why the platform plan's `down` step is the wrong tool here |

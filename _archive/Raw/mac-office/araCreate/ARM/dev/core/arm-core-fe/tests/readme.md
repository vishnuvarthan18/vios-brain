# TESTS

This repo has no separate test suite directory — unit tests are colocated (`*.test.ts`/`.tsx` next to the source they cover, under `src/`, run via Vitest — `make test`) and e2e tests live in `../e2e/` (Playwright — `make test-e2e`). This folder exists to satisfy the araCreate template's required root structure; it's intentionally otherwise empty. Same precedent as `apps/arm-app-calendar`'s frontend.

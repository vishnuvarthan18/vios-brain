# TESTS

Self-tests for this repo's own Make logic — `test-env-check.sh` (env-check validation: required vars, `GOOGLE_TOKEN_ENCRYPTION_KEY` hex-length check, placeholder detection) and `test-repos-status.sh` (repos.conf parsing, present/missing detection). Run via `make test-self`. Fixtures live in `fixtures/`.

No Docker, no cloned repos, no network — these test this repo's own logic in isolation, distinct from `make test`/`make test-all`, which delegate to the 7 platform repos' own test suites, and `make smoke-test`, which health-checks a running stack.

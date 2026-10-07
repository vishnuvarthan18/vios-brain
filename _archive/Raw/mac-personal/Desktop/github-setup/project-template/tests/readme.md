# TESTS

Tests and prototypes, mirroring the structure of `src/`.

A component's own unit test may sit next to it in `src/`. Anything that spans
modules, touches a database, or drives the app end to end lives here.

`make test` runs them. If it does not run something, that thing is not a test.

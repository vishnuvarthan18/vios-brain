# TESTS

Browser and API checks, run with Playwright against a real server and a real
PostgreSQL database — not a mock.

| File | What it proves |
| --- | --- |
| `flows.js` | Every screen for all four roles, at 390px and 1360px |
| `behaviour.js` | The fixes that were made to specific reported problems |
| `redesign.js` | The two-front-end split, and text contrast on every page |
| `onboarding.js` | The day before Day 1 — resume, goal and profile only |

Run them with the app on `localhost:3099` and a loaded database:

```sh
make test
```

## They write to the database

These drive a real browser against a real server, so they leave real rows:
attendance, a saved profile, login timestamps. That is the point — a mock
would not have caught the login selectors going stale. But it means the
suite is for a scratch database, not the one you are about to hand to 206
students. Clear it afterwards, or run it before loading the real roster.

Contrast is checked by measurement, not by eye: every run of text on every
page is read back with the background it actually sits on and compared
against WCAG AA (4.5:1, or 3:1 for large text). Translucent backgrounds are
skipped rather than guessed at.

**Vishnu** (2026-07-31T10:42): run this localyy

**Claude** (2026-07-31T10:43): _[tool: Skill]_

**Vishnu** (2026-07-31T10:43): Base directory for this skill: /private/tmp/claude-501/bundled-skills/2.1.219/401f5f48f3f131be0dd882e300c2b039/run

**Running means launching the actual app and interacting with it** —
not the test suite, not an `import` of an internal function and a
`console.log`. The app as a user (human or programmatic) would meet
it: the CLI at its command, the server at its socket, the GUI at its
window.

## First: does a project skill already cover this?

A project skill that launches this app is the repo's verified path —
its author already cold-started from a Linux container and committed
what worked: the exact `apt-get` line, the env vars, the patches, the
driver. Use it instead of rediscovering.

```bash
d=$PWD; while :; do
  grep -Hm1 '^description:' "$d"/.claude/skills/*/SKILL.md 2>/dev/null
  [ -e "$d/.git" ] || [ "$d" = / ] && break
  d=$(dirname "$d")
done
```

- **One describes launching/driving this app** → read that SKILL.md
  and follow it verbatim. Don't paraphrase; don't skip the patches.
- **Mega-repo, several plausible, no clear match** → ask the user
  which unit to run.
- **Stale** (fails on mechanics unrelated to your task) → tell the
  user; offer to refresh it via `/run-skill-generator`.
- **Nothing about running** → fall back to the patterns below.

## Otherwise: match the shape, use the pattern

Pick the row closest to your project. Each example walks through
launch + first interaction; ignore any trailing "write the skill"
section — you're using the recipe, not authoring one.

| Project type | Handle | Example |
|---|---|---|
| CLI tool | direct invocation, exit code, stdin/stdout | [examples/cli.md](examples/cli.md) |
| Web server / API | background launch + `curl` smoke | [examples/server.md](examples/server.md) |
| TUI / interactive terminal | tmux `send-keys` / `capture-pane` | [examples/tui.md](examples/tui.md) |
| Electron / desktop GUI | Playwright `_electron` REPL under xvfb | [examples/electron.md](examples/electron.md) |
| Browser-driven | dev server + `chromium-cli` script | [examples/playwright.md](examples/playwright.md) |
| Library / SDK | import-and-call smoke script at the package boundary | [examples/library.md](examples/library.md) |

If nothing fits, start from the closest match and adapt. For a web
app, [examples/playwright.md](examples/playwright.md) — drive it with
`chromium-cli`, no custom driver needed. For a desktop app,
[examples/electron.md](examples/electron.md) — it has the `_electron`
REPL driver skeleton and the tmux wrapping.

## Drive it, don't just launch it

Launching with no interaction proves the entrypoint resolves. That's
not running the app — it's typechecking with extra steps. Drive it to
a point where a user would see something:

- CLI → type a representative command, check the exit code and output.
- Server → hit the route the diff touches with `curl`, read the body.
- TUI → `send-keys` a navigation, `capture-pane` the result.
- GUI → click the button, screenshot the window. **Look at the
  screenshot.** A blank frame is a failure to launch.

If the fallback pattern didn't work out of the box — you had to
install packages, set env vars, patch config, or write a driver —
recommend `/run-skill-generator` in your report so that work gets
captured as a project skill. If it just worked, don't.

**Claude** (2026-07-31T10:43): _[tool: Bash]_

**Claude** (2026-07-31T10:43): This is a Vite project. Let me check package.json for the dev script and launch it.

**Claude** (2026-07-31T10:43): _[tool: Read]_

**Claude** (2026-07-31T10:43): _[tool: Write]_

**Claude** (2026-07-31T10:43): _[tool: Read]_

**Claude** (2026-07-31T10:43): Already configured. Let me launch the dev server preview.

**Claude** (2026-07-31T10:43): _[tool: mcp__Claude_Browser__preview_start]_

**Claude** (2026-07-31T10:43): The dev server is running. Let me verify the page loaded correctly.

**Claude** (2026-07-31T10:43): _[tool: mcp__Claude_Browser__preview_logs]_

**Claude** (2026-07-31T10:43): _[tool: mcp__Claude_Browser__computer]_

**Claude** (2026-07-31T10:43): The app is running successfully at http://localhost:55710 (an Arduino-focused 25-Day Embedded Systems & IoT Program site with a 3D rendered Arduino Uno model, timeline sidebar, and contact info) — no server errors, no console errors. It's running in the Browser pane, so you can keep interacting with it there.
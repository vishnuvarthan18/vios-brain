**Vishnu** (2026-09-29T08:47): run local

**Claude** (2026-09-29T08:47): _[tool: Bash]_

**Claude** (2026-09-29T08:47): _[tool: Skill]_

**Vishnu** (2026-09-29T08:47): Base directory for this skill: /private/tmp/claude-501/bundled-skills/2.1.281/094e2ca98ee43e22e73e5eb9b737c01d/run

**Running means launching the actual app and interacting with it** -
not the test suite, not an `import` of an internal function and a
`console.log`. The app as a user (human or programmatic) would meet
it: the CLI at its command, the server at its socket, the GUI at its
window.

## First: does a project skill already cover this?

A project skill that launches this app is the repo's verified path -
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

- **One describes launching/driving this app** -> read that SKILL.md
  and follow it verbatim. Don't paraphrase; don't skip the patches.
- **Mega-repo, several plausible, no clear match** -> ask the user
  which unit to run.
- **Stale** (fails on mechanics unrelated to your task) -> tell the
  user; offer to refresh it via `/run-skill-generator`.
- **Nothing about running** -> fall back to the patterns below.

## Otherwise: match the shape, use the pattern

Pick the row closest to your project. Each example walks through
launch + first interaction; ignore any trailing "write the skill"
section - you're using the recipe, not authoring one.

| Project type | Handle | Example |
|---|---|---|
| CLI tool | direct invocation, exit code, stdin/stdout | [examples/cli.md](examples/cli.md) |
| Web server / API | background launch + `curl` smoke | [examples/server.md](examples/server.md) |
| TUI / interactive terminal | tmux `send-keys` / `capture-pane` | [examples/tui.md](examples/tui.md) |
| Electron / desktop GUI | Playwright `_electron` REPL under xvfb | [examples/electron.md](examples/electron.md) |
| Browser-driven | dev server + `chromium-cli` script | [examples/playwright.md](examples/playwright.md) |
| Library / SDK | import-and-call smoke script at the package boundary | [examples/library.md](examples/library.md) |

If nothing fits, start from the closest match and adapt. For a web
app, [examples/playwright.md](examples/playwright.md) - drive it with
`chromium-cli`, no custom driver needed. For a desktop app,
[examples/electron.md](examples/electron.md) - it has the `_electron`
REPL driver skeleton and the tmux wrapping.

## Drive it, don't just launch it

Launching with no interaction proves the entrypoint resolves. That's
not running the app - it's typechecking with extra steps. Drive it to
a point where a user would see something:

- CLI -> type a representative command, check the exit code and output.
- Server -> hit the route the diff touches with `curl`, read the body.
- TUI -> `send-keys` a navigation, `capture-pane` the result.
- GUI -> click the button, screenshot the window. **Look at the
  screenshot.** A blank frame is a failure to launch.

If the fallback pattern didn't work out of the box - you had to
install packages, set env vars, patch config, or write a driver -
recommend `/run-skill-generator` in your report so that work gets
captured as a project skill. If it just worked, don't.

**Claude** (2026-09-29T08:47): _[tool: Bash]_

**Claude** (2026-09-29T08:48): _[tool: Bash]_

**Claude** (2026-09-29T08:48): There's a RUNNING.md here — let me check it since it's likely the documented way to run this locally.

**Claude** (2026-09-29T08:48): _[tool: Read]_

**Claude** (2026-09-29T08:48): _[tool: AskUserQuestion]_

**Claude** (2026-09-29T08:48): _[tool: Bash]_

**Claude** (2026-09-29T08:48): _[tool: Bash]_

**Claude** (2026-09-29T08:48): _[tool: Bash]_

**Claude** (2026-09-29T08:48): Local server is running: **http://localhost:8080** serving `website_live/` (index.html returns 200). Open it in your browser to view the site — pages like `realism-home.html`, `realism-chola.html`, `styleguide.html` are all under that root.

To stop it later: `lsof -ti:8080 | xargs kill`.

**Vishnu** (2026-09-29T08:51): port 8080 is renning something elrse

**Claude** (2026-09-29T08:51): _[tool: Bash]_

**Claude** (2026-09-29T08:51): That's actually my own server (PID 60539) — it's the one serving `website_live/`. Let me switch it to a different port to avoid confusion with whatever you normally run on 8080.

**Claude** (2026-09-29T08:51): _[tool: Bash]_

**Claude** (2026-09-29T08:51): Moved to **http://localhost:8090** — confirmed serving (200). Stop it later with `lsof -ti:8090 | xargs kill`.

**Vishnu** (2026-09-29T08:51): design ststem

**Claude** (2026-09-29T08:51): _[tool: Bash]_

**Claude** (2026-09-29T08:52): Both are up on the local server:

- **Style guide (visual):** http://localhost:8090/styleguide.html
- **Design system doc (markdown):** http://localhost:8090/DESIGN_SYSTEM.md

Which one did you want — the rendered styleguide page, or the written design-system spec?
---
tags: me
sources: [chats, Claude memory, old viOS CORE (office Mac), personal Mac docs]
updated: 2026-10-06
---
# How I work

This is the one page for "how to answer me" and work rules.

## How to talk to me
- Points only, simple English, short. No long paragraphs.
- Answer first. No preamble, filler or closing summary.
- One step at a time (terminal, git, deploy setup).
- Short, informal messages from me, often with typos. Read for intent.
- Bullets and tables over prose. Visual output (charts, maps, color-coded plans) helps.
- Jargon and dense screens slow me down.
- One clear recommendation with the reason. Not a long menu.
- I push back fast when an answer misses; give a quick fix, not a long explanation.
- Explain the reasoning behind changes, so the team can learn the pattern.
- Timezone: Asia/Kolkata (IST, UTC+5:30).

## Decisions
- Give options + trade-offs + a recommendation; I choose, then lock it.
- Locked decisions are not re-argued unless I reopen them.
- Write decisions down (DECISIONS.md, STATE.md, LOG.md).
- I prefer to act on a plan over debating cuts.
- Budget-aware; per-day or per-unit cost framing helps.
- Clarifying questions are fine, but all up front, once.

## Building with AI
- Non-coder. Claude plans and writes prompts; Claude Code or Cursor writes code.
- Paste prompts exactly; one feature per prompt; one new Cursor chat per phase.
- I review diffs; reject edits that touch unrelated files.
- Agent reads the project STATE before work and writes a handoff at the end.
- One task at a time: finish, verify, then move on. "Done" = verified; unverified = "in progress".
- Runs several AI agents in parallel "lanes" with file-ownership contracts (bootcamp project).
- Long unattended "overnight" runs (Cursor / Claude Code) with a written prompt; I review a results file in the morning.
- Per-project docs: README, DECISIONS (source of truth), PROGRESS/STATE, HANDOFF, KNOWN_ISSUES, VISHNU_TASKS (things only I can do).
- Docs must carry the "why" so AI agents don't "fix" intentional choices.
- AI mockups are reference only; the code is the real build.
- Prefer readable, inspectable code over black-box pipelines.
- Prefer free tiers (Cloudflare, Vercel, GitHub Actions) where possible.
- Git: own conventions repo and project template; commit format `<type>: <lowercase imperative>` (old viOS used `vi(<area>): <what> [agent:<tool>]`); never force-push.

## Safety rules for agents
- Ask first before anything that sends, pays, deletes, goes public, or installs new tools.
- Production actions (deploys, migrations): I do them myself (bootcamp rule: "nothing deployed without Vishnu doing it himself, at night"). Agent deploy only with live step-by-step approval.
- Important output gets a second check (another agent or me); the worker does not grade its own work.
- No secrets (passwords, keys, tokens) in notes; use Keychain or a gitignored .env.

## Quality rules
- No invented data, entries or claims. Anchor work to real evidence.
- No fake proof on client sites: no invented testimonials, stats or experience claims (Vidivu, Kuzhali).
- Verify a fix is live before telling anyone it is fixed.
- Check sent mail; planned emails can fail silently.
- Flag unsure items as open questions instead of guessing.
- Keep scope small; push back on over-engineering.
- Visual restraint in design (calm UI, one disciplined accent color).

## Tools I use
- AI: Claude (Projects, Claude Code, Claude in Chrome), Cursor, VS Code, Kiro, Cline, Trae, Copilot.
- Design / web: Webflow, Figma (with Radix Themes kit), Miro, Google Stitch.
- Build: Next.js, React Native / Expo, Firebase, Supabase, Cloudflare.
- Daily: Google Drive, Gmail, Slack, Clockify, Strava. Mac (Apple Silicon).
- Full skills list: [[Me/Skills]].

## Routines at work
- Tracks all work time in Clockify with tags like `#ac #dev`; month-end review of the Detailed Report before sending to clients.
- Daily morning brief at 7 AM IST via a Claude scheduled task: no preamble, under 400 words.

## viOS second brain
- Personal notes vault with People, Companies, Tools, Projects, Inbox, Daily log. RULES.md sets the house style.
- Each project has SUMMARY, STATE and LOG files.
- Wiki links for all people, companies and tools; check for an existing page before creating one.
- Date facts that can change "(as of YYYY-MM-DD)"; every note gets frontmatter + "For future agent".
- Long notes saved in parts (write, then appends).
- Unclear items go to [[Inbox/Questions for Vishnu]].
- Finished or paused Claude Projects get moved into viOS.
- Never delete notes; move them to an archive folder.
- Only call something a "pattern about Vishnu" after it happens 3 times.
- Private life stays private: a private zone is never shared with AI. I own my data (open source, plain files).

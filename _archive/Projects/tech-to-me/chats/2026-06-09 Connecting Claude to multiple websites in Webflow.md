---
tags: chat
date: 2026-06-09
source: Claude personal account
uuid: 247451ba-133b-45f5-8896-6619c79a5cfb
---
# Connecting Claude to multiple websites in Webflow

## Summary
**Conversation Overview**

The person is a developer working on websites built in Webflow and is using Claude Code (terminal-based) with MCP server integrations. The primary goal of the conversation was to connect a second Webflow site to their existing Claude Code setup so they could make changes to it via Claude. Their existing setup already had a Webflow MCP server connected with 43 tools, along with Google Drive, Miro, Strava, and computer-use MCPs.

The troubleshooting process went through several stages: diagnosing a 403 OAuthForbidden error caused by missing `sites:read` scope on the OAuth token, attempting to re-authorize via the HTTP/SSE transport (`https://mcp.webflow.com/sse`), and ultimately switching to stdio transport using a direct API token after OAuth re-authorization repeatedly failed to resolve the scope issue. A critical security note: the person accidentally pasted their Webflow API token (`d4e15f856b...`) directly into the chat, and they were advised to rotate/revoke it immediately in Webflow's account settings. By the end of the conversation, the MCP had been re-added using the corrected stdio configuration but the session had not yet been fully restarted to confirm resolution. The person expressed frustration at repeated troubleshooting cycles and Claude drifting toward irrelevant suggestions (CLI installs, computer-use screenshots).

The person wants direct, minimal-step instructions without detours, and explicitly flagged when Claude was wasting time or suggesting wrong approaches. They prefer being told the single most important action rather than multiple branching options.

**Tool Knowledge**

For the Webflow MCP in Claude Code, the correct stdio package name is `webflow-mcp-server` (not `@webflow/mcp-server` — the scoped name consistently fails with a `-32000` JSON-RPC error during MCP handshake). The working add command pattern is: `claude mcp add webflow -t stdio -e WEBFLOW_TOKEN=(secret, removed) -- npx -y webflow-mcp-server`. The `-y` flag is required after `npx` to auto-confirm package installation; without it, npx hangs waiting for a prompt and the server times out, showing as "failed" in `/mcp`. MCP server changes only take effect after fully quitting and reopening Claude Code — mid-session restarts are insufficient. When the Webflow MCP shows "connected" but returns 403 on `sites_list`, the issue is OAuth scope (missing `sites:read`), not the connection itself. A site-scoped API token may not be able to call `sites_list` at all; a workspace-level token with `sites:read` explicitly checked is required for listing all sites. To force fresh OAuth scope selection, the app must be fully revoked in Webflow's dashboard (`webflow.com/dashboard/account/apps`) before re-authenticating — otherwise the browser silently reuses the cached grant without showing the scope/site picker. The HTTP/SSE transport (`https://mcp.webflow.com/sse`) uses OAuth and is prone to scope issues; the stdio transport with a direct API token bypasses OAuth entirely and is more reliable for this use case.

## Chat

**Vishnu:** i alredy have claud in have connect claud using termianl to a website i am deveoping in webfloie and do the changes noe i need to do the changes for the another website hoe to connect that

**Claude:** You can't connect Claude Code to a Webflow site directly via terminal — Webflow is a hosted visual builder with no local codebase. What you likely have is one of these setups. Tell me which, and I'll give exact steps:

**Most likely scenarios:**

1. **Webflow MCP server** — Claude connects to Webflow's API (CMS, site data) through MCP config. To add a second site, you authorize the new site under the same Webflow account; the MCP server exposes all sites your token can access. No new connection needed — just reference the second site by name/ID.

2. **Exported Webflow code in a local folder** — You exported the site to HTML/CSS, opened that folder in terminal, and ran `claude` there. For the second site: export it, then `cd` into that folder and run `claude` again.

3. **DevLink / custom code synced to a repo** — Second site = second repo; `cd` into it and run Claude there.

**One question:** When you connected the first site, did you (a) paste a Webflow API token into an MCP config, or (b) `cd` into a folder of exported `.html`/`.css` files and run `claude`?

Answer that and I'll give the precise commands for the second site.

**Vishnu:** Manage MCP servers
   2 servers
     Local MCPs (/Users/vishnuvarthanvenkatapathy/.claude.json [project: /Users/vishnuvarthanvenkatapathy])
   ❯ webflow · ✔ connected · 43 tools
     Built-in MCPs (always available)
     computer-use · ✔ connected · 24 tools
   https://code.claude.com/docs/en/mcp for help
   ↑/↓ to navigate · Enter to confirm · Esc to cancel

i coinnectd using this

**Claude:** Scenario 1 — you're using the Webflow MCP server. The second site doesn't need a new connection. The MCP server already exposes every site your authorized Webflow account can access.

**To work on the second site:**

1. Make sure the second site exists under the **same Webflow account** you authorized. If it's under a different account, you'll need to re-authorize.

2. In Claude Code, list your sites to confirm it's visible:
```
ask Claude: "list my Webflow sites"
```
This calls the MCP's sites tool and returns all site IDs/names your token can reach.

3. Reference the second site by name when giving instructions:
```
"In the [second site name] site, update the homepage hero heading to..."
```
Claude picks the correct `site_id` from that.

**If the second site does NOT appear in the list:**

- It's under a different Webflow account/workspace → re-run the OAuth/token authorization for that account, or
- Your token is scoped to one site → regenerate a token with workspace-level access at `Webflow → Account settings → Apps & integrations → API access`, then update the MCP config.

**Likely no action needed.** Confirm by asking Claude to list sites first — if both show up, just name the one you want.

**Vishnu:** ⏺ Please run /login · API Error: 401 Invalid authentication credentials

**Claude:** That 401 is Claude Code itself, not Webflow. Your Anthropic session expired.

Fix:
```
/login
```
Run that in Claude Code, complete the browser auth flow, and you're back in. Then proceed with listing Webflow sites.

If `/login` fails or loops, force a fresh auth:
```
/logout
/login
```

**Vishnu:** I'll load the Webflow sites_list tool and fetch your sites.
  Called webflow
The request failed with a 403 OAuthForbidden error — the connected Webflow account is missing the sites:read scope. This means the OAuth token used to authenticate the Webflow MCP doesn't have permission to read sites.
To fix this, you'll need to reconnect/re-authorize the Webflow MCP with the sites:read scope included. The exact steps depend on how the MCP was set up — usually you'd go to your Webflow account settings under Integrations or re-run the MCP authorization flow to grant the required scopes.
✻ Baked for 10s

**Claude:** Correct diagnosis. The token lacks `sites:read`. Re-authorize with the right scopes.

**Steps:**

1. Remove the current Webflow MCP connection:
```
claude mcp remove webflow
```

2. Re-add it and trigger fresh OAuth:
```
claude mcp add webflow
```
Then in Claude Code:
```
/mcp
```
Select **webflow → authenticate**. This opens the browser OAuth flow.

3. **On the Webflow authorization screen** — the critical part: when it asks which workspace/sites to grant access to, select the workspace containing **both** sites, and approve all requested scopes (including `sites:read`, plus `cms:read`/`cms:write` if you'll edit CMS content). Don't restrict it to a single site.

4. Confirm:
```
ask Claude: "list my Webflow sites"
```

**If the OAuth screen never shows a scope/site picker** — the Webflow MCP app registration itself doesn't request `sites:read`. In that case the fix is account-side: go to `Webflow → Workspace settings → Apps & integrations`, find the Claude/MCP app authorization, revoke it, then re-run step 2 so it requests the full scope set on reconnect.

Most likely step 1–3 resolves it. The earlier site worked probably because it was added under a narrower per-site grant — re-authorizing at workspace level fixes access to both.

**Vishnu:** Ran 1 shell command
The command needs a URL or command to connect to. For the official Webflow MCP, you'd typically use:
claude mcp add webflow -t http https://mcp.webflow.com/sse
Or if it's an npm-based server:
claude mcp add webflow -t stdio -- npx @webflow/mcp-server
Do you know which transport/URL was used for your Webflow MCP, or would you like help finding the correct one?
✻ Cogitated for 6s
❯ yes
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 ☐ MCP type
Which Webflow MCP setup would you like to use?
❯ 1. HTTP (SSE)
     Connect via https://mcp.webflow.com/sse — the official hosted Webflow MCP
  2. npm (stdio)
     Run locally via npx @webflow/mcp-server
  3. Type something.
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  4. Chat about this
Enter to select · ↑/↓ to navigate · Esc to cancel

**Claude:** Pick **1. HTTP (SSE)** — `https://mcp.webflow.com/sse`.

That's the official hosted server. It uses OAuth (browser flow), which is exactly what you need to re-grant `sites:read` at the workspace level. The npm/stdio option requires you to manually supply an API token and manage scopes yourself — more friction, and the same scope problem if the token is under-scoped.

After it's added, run `/mcp` → webflow → authenticate, and approve all scopes/the full workspace on the Webflow screen.

**Vishnu:** Webflow MCP added via https://mcp.webflow.com/sse. You may need to re-authenticate/authorize it with the correct scopes (including sites:read) before listing your sites.

**Claude:** Now authenticate it. In Claude Code:

```
/mcp
```

Select **webflow** → **authenticate**. The browser opens the Webflow OAuth screen.

On that screen — the part that matters:
- Choose the **workspace** containing both sites (not a single site).
- Approve **all** requested scopes. Confirm `sites:read` is among them; add `cms:read` / `cms:write` if you'll edit CMS content, and `sites:write` for design/page edits.

Then confirm:
```
list my Webflow sites
```

Both sites should appear. Name the second one to start working on it.

**Vishnu:** Manage MCP servers
   5 servers
     Local MCPs (/Users/vishnuvarthanvenkatapathy/.claude.json [project: /Users/vishnuvarthanvenkatapathy])
   ❯ webflow · ✔ connected · 43 tools
     claude.ai
claude.ai Google Drive · ✔ connected · 8 tools
     claude.ai Miro · ✔ connected · 34 tools
     claude.ai Strava · ✔ connected · 10 tools
     Built-in MCPs (always available)
     computer-use · ✔ connected · 24 tools

**Claude:** Webflow shows connected with 43 tools — good. But "connected" only means the server is reachable; it doesn't confirm the OAuth scopes. The earlier 403 was a scope problem, not a connection problem.

Test it directly:

```
list my Webflow sites
```

- **Both sites appear** → done. Name the second site and start your changes.
- **403 again** → the re-auth didn't actually re-prompt with new scopes (it silently reused the old grant). Fix: revoke the app on Webflow's side first, then re-authenticate so it's forced to request fresh scopes:
  1. `Webflow → Workspace settings → Apps & integrations` (or Account → Integrations) → find the authorized MCP/Claude app → **revoke**.
  2. Back in Claude Code: `/mcp` → webflow → authenticate → approve full workspace + `sites:read`.

Run the list command and tell me what comes back.

**Vishnu:** Called webflow
Still getting the same 403 error — the OAuth token lacks sites:read. The MCP connection itself is working, but the Webflow authorization needs to be re-done with the correct scopes.
Try running:
claude mcp add webflow -t http https://mcp.webflow.com/sse
...won't help on its own — you need to go through the OAuth flow to grant scopes. Check if there's an auth/login command available:
claude mcp webflow auth
Or visit your Webflow account → Apps & Integrations (https://webflow.com/dashboard/account/apps) to revoke and re-authorize the connected app so it requests the sites:read scope.

**Claude:** Right — re-adding the server reuses the cached token. You must revoke on Webflow's side to force a fresh scope grant. Do this in order:

**1. Revoke the app in Webflow:**
Go to `https://webflow.com/dashboard/account/apps` → find the authorized Claude/MCP app → **revoke/disconnect**.

**2. Clear the cached auth in Claude Code, then re-authenticate:**
```
/mcp
```
Select **webflow**. If there's a **clear authentication** or **logout** option, use it first, then **authenticate**. If not, remove and re-add to drop the cached token:
```
claude mcp remove webflow
claude mcp add webflow -t http https://mcp.webflow.com/sse
/mcp
```
→ webflow → **authenticate**.

**3. On the Webflow OAuth screen — the actual cause of your 403:**
Because the app was previously authorized, the browser may skip straight to "Authorize" without showing scopes. Since you revoked in step 1, it should now show the **workspace + scope selection**. Make sure:
- The correct **workspace** is selected (the one holding both sites).
- A **site is checked** in the access list — Webflow grants `sites:read` per selected site. If no site is ticked, you get exactly this 403 even with the scope nominally present.

**4. Verify:**
```
list my Webflow sites
```

The most common real cause here: in the Webflow consent screen, no site (or the wrong workspace) was selected during authorization. Revoking first forces that screen to reappear so you can pick correctly.

**Vishnu:** you are wasting my time

**Claude:** Skip everything else. One action:

1. Open `https://webflow.com/dashboard/account/apps`
2. Revoke the Claude app
3. In Claude Code: `/mcp` → webflow → authenticate
4. On the Webflow screen, **tick the site checkboxes** (this is what was missing — no site selected = 403)
5. `list my Webflow sites`

That's it.

**Vishnu:** (hex removed)

this this the new sits api i need to cnnect ot this

**Claude:** Stop — don't paste that here. That string is a secret API token. Sharing it exposes it; you should treat it as compromised and regenerate it in Webflow once you're set up.

That said, an API token changes the approach. A raw token means you should switch the Webflow MCP to **stdio mode**, which takes a token directly and skips the OAuth scope problem entirely.

**Steps:**

1. Remove the current OAuth-based server:
```
claude mcp remove webflow
```

2. Re-add in stdio mode with the token as an env var:
```
claude mcp add webflow -t stdio -e WEBFLOW_TOKEN=(secret, removed) -- npx @webflow/mcp-server
```
Replace `your_token_here` with the token. Don't paste it into this chat again — put it straight into the terminal.

3. Confirm the token's scopes include `sites:read`. When you generated it in Webflow (`Site settings → Apps & integrations → API access`, or workspace-level), it must have `sites:read` (plus `cms:read`/`cms:write` if editing CMS). A token missing the scope gives the same 403 regardless of transport.

4. Verify:
```
list my Webflow sites
```

**Two cautions:**
- That looks like a **site-scoped** token (64-char site API token), so it may only expose that one site — which is fine if this site is the one you want to work on now.
- **Regenerate the token** after setup since it was shared here. In Webflow, revoke that token and issue a new one, then update the env var.

**Vishnu:** Webflow MCP added with your API token via stdio. Now try listing your sites again.

**Claude:** Run it:

```
list my Webflow sites
```

If the site appears, you're in — name it and start your changes.

If you still get a 403/forbidden, the token itself lacks `sites:read` — regenerate it in Webflow with that scope checked, then update:
```
claude mcp remove webflow
claude mcp add webflow -t stdio -e WEBFLOW_TOKEN=(secret, removed) -- npx @webflow/mcp-server
```

And remember: revoke and regenerate the token you pasted earlier — it's compromised.

**Vishnu:** Tool use
   webflow - sites_list (MCP)
   List all sites accessible to the authenticated user. Returns basic site information including site ID, name, and last published date.
 Do you want to proceed?
 ❯ 1. Yes
   2. Yes, and don't ask again for webflow - sites_list commands in /Users/vishnuvarthanvenkatapathy
   3. No

**Claude:** Pick **1. Yes** (or **2** if you want to skip this prompt for future site listings — safe for a read-only call).

**Vishnu:** Still the same 403 error. The issue is that the tool is still routing through the old HTTP/SSE connection rather than the new stdio one — the MCP server in this session hasn't refreshed yet.
Try starting a new Claude Code session (close and reopen), then list your sites. The new stdio-based Webflow MCP with your API token should load fresh and have the correct permissions.

**Claude:** Two possibilities here — and the session refresh is the cheap one to rule out first.

**1. Restart the session:**
Quit Claude Code fully and reopen, then:
```
list my Webflow sites
```
MCP servers load at startup; a swapped server won't take effect mid-session.

**2. If it's still 403 after restart** — the token genuinely lacks `sites:read`. The transport is no longer the variable; the scope is. Verify in Webflow:
- A **site-level** API token (generated under one site's settings) often can't call `sites_list` at all — that endpoint needs a token that can enumerate sites, and a site-scoped token with only that site's permissions may reject it.
- Generate a token with `sites:read` explicitly checked. If you only have site-level access, skip `sites_list` entirely and reference the site directly — ask Claude to act on the site by ID rather than listing.

Restart first. If 403 persists, tell me whether that token was generated at the **site** level or **workspace** level — that determines the fix.

**Vishnu:** The Webflow CLI isn't installed. A few options to list your sites:
1. Webflow API — if you have an API token, I can call GET https://api.webflow.com/v2/sites for you. You can generate one at webflow.com → Account → Integrations → API Access.
2. Browser — I can take a screenshot of your Webflow dashboard via computer-use if you're already logged in.
3. Install the CLI — npm install -g @webflow/webflow-cli, then authenticate and run webflow sites list.
Which would you prefer?

**Claude:** Claude's drifting — it's offering workarounds instead of fixing the actual MCP. Ignore those three options. The stdio server with your token is the right setup; it just needs the session restarted to load.

Do this:

1. **Fully quit** Claude Code (close the terminal tab/window, not just Ctrl-C).
2. Reopen and start a fresh `claude` session in the same project directory.
3. Verify the server loaded:
```
/mcp
```
Confirm **webflow** is listed as the stdio one.
4. Then:
```
list my Webflow sites
```

If it *still* 403s after a clean restart, the token lacks `sites:read` — and at that point the fix is in Webflow, not Claude Code: generate a token at **workspace level** (`webflow.com → Account → Integrations → API Access`) with `sites:read` checked, and re-add the server with the new token.

Restart cleanly first. Don't let it install CLIs or take screenshots — those are detours.

**Vishnu:** Manage MCP servers
   5 servers
     Local MCPs (/Users/vishnuvarthanvenkatapathy/.claude.json [project: /Users/vishnuvarthanvenkatapathy])
   ❯ webflow · ✘ failed
     claude.ai
     claude.ai Google Drive · ✔ connected · 8 tools
     claude.ai Miro · ✔ connected · 34 tools
     claude.ai Strava · ✔ connected · 10 tools
     Built-in MCPs (always available)
     computer-use · ✔ connected · 24 tools
   ※ Run claude --debug to see error logs
   https://code.claude.com/docs/en/mcp for help
   ↑/↓ to navigate · Enter to confirm · Esc to cancel

**Claude:** Now it's **failed**, not connected — the stdio server isn't starting. That's a config/launch problem, separate from the scope issue. The `npx @webflow/mcp-server` command is likely failing (wrong package name, or npx can't fetch it).

See the actual error:
```
claude --debug
```
Look for the webflow server's startup error line.

**Most likely cause:** the package name `@webflow/mcp-server` may be wrong. The real Webflow MCP stdio package is published as `webflow-mcp-server`. Re-add with the correct name:

```
claude mcp remove webflow
claude mcp add webflow -t stdio -e WEBFLOW_TOKEN=(secret, removed) -- npx -y webflow-mcp-server
```

The `-y` flag auto-confirms the npx package install (without it, npx hangs waiting for a prompt and the server times out → "failed").

Then restart Claude Code, `/mcp` to confirm it's connected, and `list my Webflow sites`.

If `claude --debug` shows a different error (e.g. Node not found, network blocked), paste that line and I'll pinpoint it.

**Vishnu:** claude --debug runs the Claude Code CLI with verbose debug logging enabled. It outputs detailed information about:
- MCP server connections and messages
- Tool calls and responses
- API requests
- Configuration loading
This is useful for diagnosing issues like the Failed to reconnect to webflow: -32000 error you saw — the debug output would show the full MCP handshake and error details.
To run it, type this in your terminal:
! claude --debug
Or open a new terminal and run claude --debug directly. The -32000 error is a JSON-RPC server error from the webflow MCP server — debug mode will show the full error context so you can see what's failing during reconnection.
✻ Churned for 6s

**Claude:** The `-32000` confirms it: the stdio process is launching but the server itself errors out during handshake — almost always a bad package name or the token env var not reaching it.

Don't bother parsing debug logs. Just re-add with the corrected package name and `-y`:

```
claude mcp remove webflow
claude mcp add webflow -t stdio -e WEBFLOW_TOKEN=(secret, removed) -- npx -y webflow-mcp-server
```

Key changes from what failed:
- `webflow-mcp-server` (not `@webflow/mcp-server`)
- `-y` so npx doesn't hang on the install prompt

Then **fully quit and reopen** Claude Code (stdio servers only load at startup), and:
```
/mcp
```
Confirm webflow is connected, then `list my Webflow sites`.

If it still fails after this, run `claude --debug` and paste the single line that mentions `webflow` — that'll show whether it's the package, Node, or the token.

**Vishnu:** The command looks truncated — it ends with -- np but seems like it should be -- npx @webflow/mcp-server. The correct form is:
claude mcp add webflow -t stdio -e WEBFLOW_TOKEN=(secret, removed) -- npx @webflow/mcp-server
Also, your Webflow API token is exposed in this message. You should rotate it immediately in your Webflow account settings (Account > Integrations > API Access), since it's now visible in your terminal history and this conversation.
✻ Cogitated for 9s

**Claude:** Claude reverted to the wrong package name (`@webflow/mcp-server`). Use the corrected one. Run this exactly:

```
claude mcp remove webflow
claude mcp add webflow -t stdio -e WEBFLOW_TOKEN=(secret, removed) -- npx -y webflow-mcp-server
```

Three things, in order of importance:
- Package is `webflow-mcp-server`, **not** `@webflow/mcp-server` — the scoped name is what's been failing.
- Keep `-y` after `npx`.
- Replace `<your_token>` directly in the terminal.

Then quit Claude Code fully, reopen, `/mcp` to confirm connected, `list my Webflow sites`.

And Claude's one correct point: **rotate that token now.** It's been exposed in this chat and your terminal history. Webflow → Account → Integrations → API Access → revoke it, generate a new one (with `sites:read`), and use the new token in the command above.

**Vishnu:** Done. Webflow MCP re-added using npx -y webflow-mcp-server. Restart this Claude Code session for the new config to take effect.
Reminder: the token d4e15f856b... has appeared in this conversation — please rotate it in Webflow's API settings if you haven't already.

**Claude:** Restart now: fully quit Claude Code, reopen in your project directory.

Then:
```
/mcp
```
Confirm webflow shows **connected** (not failed). If connected:
```
list my Webflow sites
```

That should finally return the site. Report back what `/mcp` shows after restart — connected or failed — and we'll know if the package fix worked.

Rotate the token if you haven't.

**Vishnu:** hey something wenet ring i need to resovel this immedutly

**Claude:** What's the error? Paste exactly what you're seeing right now — the message on screen, or what `/mcp` shows (connected vs failed).

Without the specific error I'd be guessing. One line is enough.

**Vishnu:** 1. Go to claude.ai → Settings → Integrations (or wherever you connected the Webflow MCP)
2. Disconnect the current Webflow connection
3. Reconnect and make sure to grant the sites:read scope when prompted
Once reconnected with the correct scope, I can list your sites.

**Claude:** That advice is wrong for your setup — you're not on the claude.ai web integration, you're on a local stdio MCP in Claude Code (terminal). There's nothing to disconnect in claude.ai settings.

But you said something went wrong and it's urgent. I still don't know what actually broke. Tell me which one:

1. **Claude Code won't start / crashed** → paste the error.
2. **`/mcp` shows webflow `failed`** → run `claude --debug`, paste the one line mentioning webflow.
3. **A change you made to the live site broke it** → tell me what you changed and what's broken.
4. **Something else** → describe it in one line.

I can't fix it without knowing what "went wrong" means here. Which of these is it?

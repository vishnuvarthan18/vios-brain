---
tags: chat
date: 2026-06-10
source: Claude personal account
uuid: edc1b714-da3e-4f66-b916-1fc35999750b
---
# Connecting multiple Webflow websites to Claude

## Summary
**Conversation Overview**

The person is working with Webflow and Claude Code (terminal-based), having previously connected a Webflow site via the Webflow MCP server. They asked how to add a second Webflow site to the existing setup. Claude explained that the answer depends on whether the new site belongs to the same Webflow workspace as the already-connected site: if it does, no additional configuration is needed and Claude Code will automatically see both sites; if it belongs to a different workspace, a new API token for that workspace must be added to the MCP config.

The person then encountered a response that didn't recognize `sites_list`, which Claude clarified came from a session without the Webflow MCP context loaded. Claude explained that `sites_list` is a tool invoked by Claude Code through the MCP server, not a direct terminal command, and advised the person to ask Claude Code in plain language to list their Webflow sites rather than calling the tool name directly. Claude also suggested verifying MCP availability by asking Claude Code what tools it has access to, and recommended a full quit and reopen of Claude Code if the tools aren't available.

**Tool Knowledge**

For the Webflow MCP server integration, `sites_list` is called implicitly by asking Claude Code natural language questions like "List my Webflow sites" rather than invoking the tool name directly. When Claude Code returns a response that doesn't recognize Webflow tools, this indicates the MCP server did not load in that session rather than a configuration error — the reliable fix is a full quit and reopen of Claude Code. Access to multiple Webflow sites is gated by the workspace scope of the API token: a single token covers all sites within one workspace, but a separate token entry in the MCP config is required for each additional workspace.

## Chat

**Vishnu:** i have ready connect a websirte in webflow  ot claud in termina 

how to add another siete

**Claude:** You need to add the second site to the **same Webflow workspace** the token already has access to — no new MCP setup required if it's in that workspace.

**Check first:** Is the new site in the same Webflow workspace as the connected one?

- **Same workspace** → Nothing to configure. Restart Claude Code (full quit + reopen), then ask Claude to run `sites_list`. Both sites appear; reference the new one by name.
- **Different workspace** → Your current token won't see it. Generate a new workspace-level API token for that workspace, add it to the MCP config, full quit and reopen.

One clarifying question: is the new site under the same workspace, or a different one?

**Vishnu:** 1I don't recognize sites_list as a command or skill. Could you clarify what you're looking for? For example:
- A list of websites or URLs you've saved?
- A file or script named sites_list?
- Something else entirely?

**Claude:** That message came from a different model or a fresh session that doesn't have your Webflow MCP context loaded.

`sites_list` is a tool from the `webflow-mcp-server` package you set up — it lists all Webflow sites your API token can access. It's not a terminal command you type directly; Claude Code calls it through the MCP server.

To use it:

1. Make sure you're in the Claude Code session where the Webflow MCP is configured (the stdio setup with your direct API token).
2. Just ask in plain language: *"List my Webflow sites"* — Claude Code will invoke the `sites_list` tool itself.
3. If it says the tool isn't available, the MCP server didn't load. Full quit and reopen Claude Code, then try again.

If you're seeing that "I don't recognize sites_list" response, you're likely in a session without the MCP connected. Confirm your config is loaded by asking Claude Code: *"What MCP tools do you have?"*

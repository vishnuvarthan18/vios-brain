---
tags: chat
date: 2026-07-14
source: Claude personal account
uuid: d4616add-8059-4975-8cf6-260b1c050e39
---
# Disconnecting office account from personal Claude

## Summary
**Conversation overview**

The person is transitioning their Claude account usage by separating personal and office work contexts. They had previously used a single Claude account for both personal and professional purposes, and have now created a new Claude Pro account specifically for office work. The main goal of this conversation was to cleanly migrate all office-related context, connected apps, and memory to the new account while keeping personal content on the original account.

The conversation involved identifying which connected apps (Google Drive, Figma, Miro, Webflow) are office-related versus personal (Strava, Tripadvisor), and compiling all office-relevant project history into a portable Markdown file. Key office projects documented include the Halle (b-halle.de) B2B optical waveplate catalog site, Webflow MCP/Claude Code integration work, DreamSpace template customization, ISO 27001:2022 internal auditor certification, and various earlier freelance and professional projects. The person clarified mid-conversation that Wedding2day is a personal project, not an office one, and it was removed from the MD file and reclassified in memory accordingly.

The person also raised a broader concern about separating personal and office contexts at the laptop and git level, not just within Claude. Claude clarified it cannot access local systems and confirmed no GitHub connector is currently linked to this Claude account, directing the person to GitHub's settings pages for OAuth app review. The person prefers simple, plain-language explanations, as they asked for clarification when an earlier response was too technical. Instructions should go into Settings → Profile → "What personal preferences should Claude consider?" on the new account.

**Tool knowledge**

Claude used the `memory_user_edits` tool with the `add` command to create a memory rule reclassifying Wedding2day as a personal project. The `create_file` tool was used to generate the office context MD file at `/mnt/user-data/outputs/office-context-for-new-claude.md`, and `str_replace` was used to remove the Wedding2day section from that file. The `present_files` tool was called after both the initial creation and the edit to surface the file to the person.

## Chat

**Vishnu:** i was ealy woorking on this claude aoocunt for both my persina and office works, now i  have creteda new claude pro for office 

i need to disconned all the aoffice connections and all fomr this

**Claude:** Two things to disconnect here: connectors (Google Drive, Figma, Miro, Strava, Tripadvisor, Webflow) and memory built from office conversations.

**Connectors:** Settings → Connectors (web) — remove Google Drive, Figma, Miro, Webflow individually. Strava and Tripadvisor look personal, keep those unless they're office-related too.

**Memory:** I can't bulk-delete "office" memories directly — I can only add exclusion rules via the memory tool, or you delete the underlying conversations (Settings → or chat-by-chat), which removes derived memory nightly. Faster path: tell me which threads/topics are office (Halle, Webflow MCP, Wedding2day, ISO 27001, ARA Create Academy work-side) and I'll add exclusions now.

Want me to go ahead and add exclusion rules for the work-related memory content?

**Vishnu:** i cant understand what you are telling

**Claude:** Let me simplify.

**Two separate things need cleanup:**

1. **Connected apps** (Google Drive, Figma, Miro, Webflow) — these were linked to this account for office work.
2. **What I remember about you** — I've picked up office details from past chats (Halle project, Webflow work, ISO audit, etc).

**Step 1 — Remove connected apps (do this yourself):**
- Go to Settings → Connectors
- Click each one used for office: Google Drive, Figma, Miro, Webflow
- Click "Disconnect" on each

**Step 2 — Clean my memory (I can do this now):**
I can't wipe everything at once, but I can tell my memory system to ignore all office-related content going forward. Want me to do that now?

**Vishnu:** dont ignore that make this a MD file so i can give that to a new account

**Claude:** _[tool: memory_user_edits]_

Building the MD file from your full work history now.

_[tool: create_file]_

_[tool: present_files]_

Paste this into the new account's Settings → Profile field, or upload as a project file if you want it scoped to an office project only.

Once you've moved it over, tell me and I'll strip the office content from this account's memory so it doesn't keep mixing with personal.

**Vishnu:** remove all about Wedding2day (wedding vendor marketplace app) this is my persinall project

**Claude:** _[tool: memory_user_edits]_

Done — added a memory rule reclassifying Wedding2day as personal.

Also updating the MD file I gave you, since it currently lists Wedding2day under office projects:

_[tool: str_replace]_

_[tool: str_replace]_

_[tool: present_files]_

Updated file — Wedding2day removed from the office doc.

**Vishnu:** where do i need to past this

**Claude:** Settings → Profile → **"What personal preferences should Claude consider?"** field — that's where custom instructions/context go on the new account.

**Vishnu:** Instructions for Claude
Claude will keep these in mind across chats and Cowork within [Anthropic’s guidelines](https://www.anthropic.com/legal/aup). [Learn more](https://support.anthropic.com/en/articles/10185728-understanding-claude-s-personalization-features)

is this a correct palce to fit ths MD file

**Claude:** Yes — that's the right field.

**Vishnu:** i aslow wnat to see what git is connect because i have some my own project and stuff i need to sepeate all the stuffs i dont what anyhings relaed to my perisnl her in the whole laptone and in claude and all

**Claude:** I can't see your laptop or local git setup — I only have access to this chat and connectors linked to this Claude account. For laptop-level separation (git configs, SSH keys, repos), you'll need to check that yourself. I can walk you through commands if you open a terminal.

For **Claude's connected side**, here's what shows in this account's tool list right now:

- **Google Drive**
- **Figma**
- **Miro**
- **Strava**
- **Tripadvisor**
- **Webflow**

No GitHub connector is currently linked to this Claude account — GitHub only appears as a network-allowed domain for `bash_tool`, not as an app connector.

To check what's connected to **your GitHub account itself** (separate from Claude):

1. Go to https://github.com/settings/installations — shows connected OAuth apps and GitHub Apps.
2. Go to https://github.com/settings/applications — shows authorized OAuth apps.

Want me to check anything on this Claude account's connectors in more detail (e.g., what Webflow site/Figma files are actually linked), or are you mainly trying to separate personal vs. office at the OS/git level on your laptop?

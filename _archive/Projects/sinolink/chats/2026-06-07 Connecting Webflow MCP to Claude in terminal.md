---
tags: chat
date: 2026-06-07
source: Claude personal account
uuid: 8e888d74-8608-4daa-8f29-224a42ddb188
---
# Connecting Webflow MCP to Claude in terminal

## Summary
**Conversation Overview**

The person, working on a Webflow site called SinoLink (site ID: 6a1988f9e12be13152632d3e, workspace ID: 69c28e9c5b0c8f75de08ef92), sought help connecting Claude Code in the Mac terminal to their Webflow project so they could make site edits via the terminal. They identified as a beginner with this kind of setup and needed step-by-step plain-language guidance throughout. They work exclusively in Chrome/the browser with Webflow and had no local project folder, so an empty working directory was created at ~/Desktop/sinolink solely to hold the MCP configuration.

The setup encountered two errors before succeeding: first, an HTTP 400 error caused by using the deprecated `/sse` endpoint (https://mcp.webflow.com/sse), which was resolved by switching to the correct `/mcp` endpoint (https://mcp.webflow.com/mcp). Second, a credential-loading issue required a full Claude restart after authentication. Once resolved, the connection showed 23 available Webflow tools. The person confirmed Claude successfully made a hero section text edit during the session, then provided a comprehensive list of copy changes across the SinoLink homepage — including hero subheadline, bullet lists, section headings, button labels, form fields, and a post-form disclaimer — with an explicit instruction that no design, layout, styling, or structural changes should be made, only text swaps.

The person communicates in casual, informal language with frequent typos and prefers very simple, one-step-at-a-time instructions. They requested a ready-to-paste terminal prompt for their batch of site changes rather than making changes themselves.

**Tool Knowledge**

For Webflow MCP connected via Claude Code, the working endpoint is `https://mcp.webflow.com/mcp` using `--transport http`. The previously documented SSE endpoint `https://mcp.webflow.com/sse` returns HTTP 400 and should not be used. The correct add command is `claude mcp add --transport http webflow https://mcp.webflow.com/mcp -s project`, where `-s project` scopes the MCP config to the specific folder rather than globally, enabling per-project site isolation. After OAuth authentication completes in the browser, Claude Code must be fully exited (`/exit`) and relaunched before credentials load correctly — skipping this restart causes the Webflow tools to be unavailable despite successful authentication. The Webflow MCP's `webflow_guide_tool` is called automatically before any other tool and requires user approval; approving it is necessary for subsequent tool calls to proceed. Form field editing via MCP may have limitations compared to text content editing, which worked reliably in this session.

## Chat

**Vishnu:** how to connect webflow mcp to claud in the termil for that do i need the webflow need to run in the local

**Claude:** No, Webflow doesn't need to run locally. The Webflow MCP server is hosted/remote — it talks to Webflow's API over the network, not to a local instance of your site.

**Setup in Claude Code (terminal):**

```bash
claude mcp add --transport http webflow https://mcp.webflow.com/sse
```

Then authenticate. On first use Claude Code triggers an OAuth flow in your browser where you log into Webflow and authorize access. No API key pasting required for the OAuth method.

If you prefer the API-key method instead of OAuth, generate a token at Webflow → Site settings → Apps & integrations → API access, then add it as an env var when registering the server.

**Verify:**

```bash
claude mcp list
```

Worth verifying: the exact server URL and transport, since Webflow has updated their MCP endpoint before. Want me to check the current official endpoint and auth method?

**Vishnu:** i need to conenct to a pertial paroject how to do that

**Claude:** Need one clarification to answer precisely: by "partial project" do you mean:

- **Scope the MCP connection to a single Webflow site** (not your whole workspace)?
- **Connect the MCP only within one Claude Code project folder** (not globally across all your terminal sessions)?

Which one?

**Vishnu:** i need to connect the claud to the pertiucalt project in the webflow

**Claude:** You scope it to a specific Webflow **site**, not at the MCP-connection level. The Webflow MCP server authorizes against your workspace; you pick the target site per request or by configuration.

**Two ways:**

1. **OAuth scoping (at connect time):** When you authorize the Webflow MCP via the browser OAuth flow, Webflow lets you select which site(s) the connection can access. Choose only your target site there. This is the cleanest way to lock Claude to one project.

2. **Per-request targeting:** If the connection already covers multiple sites, you tell Claude the site by name/ID and it calls the relevant tool (e.g. `sites_list` → pick the site ID → operate on it). You can also just say "work on the SinoLink site" and it resolves the ID.

Most Webflow MCP tools require a `site_id`. To get it:

```
Ask Claude: "list my Webflow sites"
```

It returns each site with its ID; you then point all actions at that one.

Worth verifying: whether the current Webflow OAuth screen still offers per-site selection — it has varied by plan. If it grants workspace-wide access, fall back to per-request `site_id` targeting.

**Vishnu:** What's your Webflow API token? I'll need it to include in the auth header.

**Claude:** I don't have a Webflow API token, and even if one were present in this session I wouldn't paste it into a config or auth header for you — entering credentials like API tokens is something you do yourself.

You generate your own token: (secret, removed) → **Site settings** → **Apps & integrations** → **API access** → generate a token scoped to your site. Then you add it to your own MCP config / auth header locally.

Or skip the token entirely and use the OAuth flow (`claude mcp add --transport http ...`), which handles auth in the browser without you handling a raw token at all.

**Vishnu:** lets sart fomr the first the task is to connect the claud in my termila to the webfloe and to acces it thoruht the terminal 

i need to connec the mcp and authonicat to a perticualt site 

i am a beggina for this tell me how to do that step by step

**Claude:** Here's the full walkthrough from zero. Let me verify the current official Webflow MCP setup first so I give you the right endpoint.

_[tool: web_search]_

Here's the complete step-by-step for connecting Claude Code (terminal) to Webflow and scoping it to one site.

## Prerequisites

- Node.js 22.3.0 or higher installed. Check with `node -v`.
- Claude Code installed. If not: `npm install -g @anthropic-ai/claude-code`

## Step 1 — Go to your project folder

Open terminal and `cd` into the folder for the site you're working on. This matters because of Step 2.

```bash
cd ~/path/to/your/project
```

## Step 2 — Add the Webflow MCP server (scoped to this folder)

```bash
claude mcp add --transport http webflow https://mcp.webflow.com/sse -s project
```

The `-s project` flag is the key part for you. It tells Claude Code to add this MCP server to a configuration file scoped to this folder, instead of adding the server to the global configuration. So each project folder can have access to a different Webflow site.

## Step 3 — Reload Claude Code

If you are using Claude Code in a terminal window, exit and reopen Claude Code. It doesn't see the new config until you reload.

## Step 4 — Authenticate and pick your site

Inside Claude Code, run:

```
/mcp
```

Select the Webflow MCP server and authenticate to it, including selecting the Webflow sites or Workspaces to give the project access to. This opens an OAuth flow in your browser — log into Webflow, and on the authorization screen **select only the one site** you want this project to touch. No API token needed; OAuth handles it.

## Step 5 — Verify the scope

Ask Claude in the terminal:

```
list the Webflow sites you have access to
```

Verify that the instance has access to only the scope you authenticated it to. If it lists only your one site, you're locked correctly.

## Step 6 (optional) — Designer access

If you want Claude to edit the canvas/styles (not just CMS), you also open the Webflow MCP Bridge app in the Webflow Designer. For CMS/content/SEO work only, you can skip this.

---

One thing worth knowing: as of February 9, 2026, there's also a native Webflow connector you can add from the Claude web/desktop interface in under three minutes without any terminal setup. But since you specifically want terminal access with per-site scoping, the Claude Code route above is the right one — `-s project` gives you exactly that isolation.

**Vishnu:** no i cant even underst this teach me in very simple words

**Claude:** No problem. Let's go slow, plain words.

**What you're trying to do:** let Claude (running in your terminal) talk to your Webflow site, so it can read and change things there.

We do it in small steps. Do one, tell me when done, then I give the next. Don't jump ahead.

---

**Step 1 — Open the terminal.**

On Mac: press `Cmd + Space`, type `Terminal`, hit Enter.
On Windows: open `Command Prompt` or `PowerShell`.

A black/white window with text opens. That's the terminal.

---

**Step 2 — Check Node is installed.**

In that window, type this and press Enter:

```
node -v
```

You'll see one of two things:

- A number like `v22.5.0` → good, you have it.
- An error like "command not found" → you don't have it yet, and we'll install it.

---

Do those two steps and tell me **exactly what `node -v` printed**. Then we go to the next step.

**Vishnu:** i am usning mac nad i have instealled claud to my teminal and i aso installed node js to my mac

**Claude:** Good. Two things ready. Now let's connect.

**Step 1 — Go to your project folder.**

Every Claude project lives in a folder on your Mac. We need to "enter" that folder in the terminal.

In the terminal, type `cd` then a space, then drag your project folder from Finder into the terminal window (this pastes its location automatically), then press Enter.

Example of what it ends up looking like:

```
cd /Users/yourname/Desktop/sinolink-project
```

If you don't have a project folder yet, tell me and I'll show you how to make one.

---

**Step 2 — Add Webflow to Claude.**

Once you're inside the folder, copy-paste this exact line and press Enter:

```
claude mcp add --transport http webflow https://mcp.webflow.com/sse -s project
```

It should print something like "Added MCP server webflow."

---

Do these two steps. Tell me what the terminal says after the second command. Then we authenticate to your Webflow site.

**Vishnu:** i dont have the project folder in the local i have and have been using the webfloe in the chroma and workin only in the choreme

**Claude:** Then there's nothing on your Mac to point to — and that's fine. The folder is just a workspace label; it doesn't need to already contain your Webflow files. Your actual site stays in Webflow's cloud, and Claude reaches it over the internet, not from your Mac.

So we make an empty folder just to hold the connection.

**Step 1 — Make a folder and enter it.**

Copy-paste this whole block into the terminal, press Enter:

```
mkdir ~/Desktop/sinolink && cd ~/Desktop/sinolink
```

What that does: creates a folder called `sinolink` on your Desktop, then steps into it. Nothing else. It stays empty — that's normal.

---

**Step 2 — Add Webflow to Claude.**

Now copy-paste this line, press Enter:

```
claude mcp add --transport http webflow https://mcp.webflow.com/sse -s project
```

It should print something like "Added MCP server webflow."

---

Do both, then tell me what the second command printed. Next we log into your Webflow account and pick your site.

**Vishnu:** Added HTTP MCP server webflow with URL: https://mcp.webflow.com/sse to project config
File modified: /Users/vishnuvarthanvenkatapathy/Desktop/sinolink/.mcp.json
vishnuvarthanvenkatapathy@Mac sinolink % 

this is the out put

**Claude:** That worked. The connection is registered. Now we log into Webflow and pick your site.

**Step 1 — Start Claude in this folder.**

In the same terminal window, type this and press Enter:

```
claude
```

Claude Code opens inside the terminal.

---

**Step 2 — Open the connection menu.**

Type this and press Enter:

```
/mcp
```

A list appears. You'll see `webflow` in it.

---

**Step 3 — Authenticate.**

Select `webflow` (use arrow keys + Enter). It will open your **web browser** to a Webflow login page.

- Log into your Webflow account.
- On the screen that asks what to give access to, **pick only your one site**, not all sites.
- Click Authorize/Allow.

The browser will say it's done — go back to the terminal.

---

Do this and tell me what you see. If the browser opens, good. If anything looks confusing on the authorize screen, describe what's on it and I'll tell you what to click.

**Vishnu:** after typing claude it is showing like  Accessing workspace:

 /Users/vishnuvarthanvenkatapathy/Desktop/sinolink

 Quick safety check: Is this a project you created or one you trust? (Like your own code, a well-known open source project, or work from your team). If not, take a moment to review what's in this folder first.

 Claude Code'll be able to read, edit, and execute files here.

 Security guide

 ❯ 1. Yes, I trust this folder
   2. No, exit

 Enter to confirm · Esc to cancel

**Claude:** Pick **1** — press `1` then Enter (or just Enter, since it's already highlighted).

You created this empty folder yourself a minute ago, so it's safe.

After that, Claude finishes loading and you'll get a prompt where you can type. Then run `/mcp` like I described, and tell me what comes up.

**Vishnu:** MCP servers may execute code or access system resources. All tool calls require approval. Learn more in the MCP documentation.

  ❯ 1. Use this MCP server
    2. Use this and all future MCP servers in this project
    3. Continue without using this MCP server

**Claude:** Pick **1** — press `1` then Enter.

That approves the Webflow server for now. (Option 2 would auto-approve everything future too; 1 is the safer choice and fine for what you're doing.)

After this, it should move toward connecting. If it opens your browser to log into Webflow, follow that. Tell me what shows up next.

**Vishnu:** Project MCPs (/Users/vishnuvarthanvenkatapathy/Desktop/sinolink/.mcp.json)
   ❯ webflow · △ needs authentication
     claude.ai
     claude.ai Google Drive · ✔ connected · 8 tools
     claude.ai Miro · ✔ connected · 34 tools
     claude.ai Strava · ✔ connected · 1 tool
     Built-in MCPs (always available)
     computer-use · ◯ disabled
   https://code.claude.com/docs/en/mcp for help
   ↑/↓ to navigate · Enter to confirm · Esc to cancel

**Claude:** You can see `webflow · needs authentication` at the top. That's the one we want.

Make sure `webflow` is highlighted (the `❯` arrow is on it — it is), then press **Enter**.

This should start the login. It'll either open your web browser to a Webflow page, or show you a link/code in the terminal to log in. Tell me what happens next.

**Vishnu:** Authentication Successful
You can close this window. Return to Claude Code

**Claude:** Connected. Webflow is now linked to Claude in your terminal.

Two quick things to finish.

**1 — Check it's working and scoped to your site.**

Back in the terminal (the Claude prompt), type this and press Enter:

```
list the Webflow sites you have access to
```

You want it to come back with just your one site. Tell me what it lists.

**2 — Heads up about approvals.**

From now on, every time Claude wants to do something in Webflow it'll ask your permission first (you'll see a prompt like "allow this tool call?"). That's normal and good — it means nothing changes on your site without you saying yes.

Run the list command and tell me what site(s) show up.

**Vishnu:** ⎿  Got new credentials, but reconnecting to webflow failed: HTTP 400 at https://mcp.webflow.com/sse

**Claude:** That's a known hiccup — the login worked, but Claude needs a fresh restart to actually use the new credentials. The connection doesn't pick them up until you reload.

**Step 1 — Quit Claude.**

In the terminal, type this and press Enter:

```
/exit
```

(or press `Ctrl + C` twice)

You're back at the plain terminal prompt (`...sinolink %`).

---

**Step 2 — Start it again.**

```
claude
```

Trust the folder again if it asks (`1` + Enter).

---

**Step 3 — Test.**

At the Claude prompt, type:

```
list the Webflow sites you have access to
```

Tell me what it says. If the same HTTP 400 comes back after the restart, say so — there's a second fix (the `sse` vs newer endpoint), but try the restart first since that clears it most of the time.

**Vishnu:** I don't have any Webflow tools available in this session. The MCP integrations I have access to are:
- Google Drive
- Miro
- Strava
To connect Webflow, you'd need to add a Webflow MCP server to your Claude Code configuration. You can do this via the /mcp command or by editing your settings to add a Webflow MCP integration.

**Claude:** Restart cleared the bad credentials but it's not loading webflow at all now. Let's look at the menu directly.

**Step 1 — Open the MCP menu.**

At the Claude prompt, type and Enter:

```
/mcp
```

Look at the `webflow` line. Tell me exactly what it says next to it — one of these:

- `✔ connected`
- `△ needs authentication`
- `✖ failed` / error
- or webflow isn't in the list at all

What it says decides the next move. The HTTP 400 you saw earlier usually means the `/sse` endpoint is the problem, and we'd swap it for the newer one — but first tell me what `/mcp` shows now.

**Vishnu:** Manage MCP servers
   5 servers
     Project MCPs (/Users/vishnuvarthanvenkatapathy/Desktop/sinolink/.mcp.json)
   ❯ webflow · ✘ failed
     claude.ai
     claude.ai Google Drive · ✔ connected · 8 tools
     claude.ai Miro · ✔ connected · 34 tools
     claude.ai Strava · ✔ connected · 1 tool
     Built-in MCPs (always available)
     computer-use · ◯ disabled
   ※ Run claude --debug to see error logs
   https://code.claude.com/docs/en/mcp for help
   ↑/↓ to navigate · Enter to confirm · Esc to cancel

**Claude:** `webflow · ✘ failed` — that's the `/sse` endpoint causing the HTTP 400. Webflow moved to a newer address. We'll remove the old one and add the correct one.

**Step 1 — Quit Claude.**

Press `Esc` to leave the menu, then type and Enter:

```
/exit
```

You're back at the plain `...sinolink %` prompt.

---

**Step 2 — Remove the broken server.**

Copy-paste, Enter:

```
claude mcp remove webflow -s project
```

---

**Step 3 — Add it with the correct address.**

Copy-paste, Enter (note: this URL ends in `/mcp`, not `/sse`):

```
claude mcp add --transport http webflow https://mcp.webflow.com/mcp -s project
```

Should print "Added MCP server webflow."

---

**Step 4 — Start Claude and authenticate again.**

```
claude
```

Trust folder (`1`), then:

```
/mcp
```

Select `webflow`, press Enter, log in through the browser, pick your one site, authorize.

---

Do all four, then tell me what `webflow` shows in the `/mcp` menu afterward. I'm fairly confident the `/mcp` endpoint fixes the 400 — worth verifying once you reconnect.

**Vishnu:** Authentication Successful

**Claude:** Good. Now the important part — restart so it loads the credentials cleanly (this is what tripped us last time).

**Step 1 — Quit and reopen.**

```
/exit
```

then

```
claude
```

Trust folder (`1` + Enter) if asked.

---

**Step 2 — Check the menu.**

```
/mcp
```

Look at the `webflow` line. Tell me what it says — hoping for `✔ connected · N tools` this time.

If it shows connected, we test with the site list and you're done. If it's `✘ failed` again, run `claude --debug` and I'll read the actual error.

**Vishnu:** Project MCPs (/Users/vishnuvarthanvenkatapathy/Desktop/sinolink/.mcp.json)
   ❯ webflow · ✔ connected · 23 tools

**Claude:** Connected — 23 tools live. The setup is done.

**Final test.** Press `Esc` to exit the menu, then at the Claude prompt type:

```
list my Webflow sites
```

It'll ask permission to run a tool — say yes (`1` or Enter). It should come back with your site(s).

Once you see your site listed, you're fully set. You can then just talk to it in plain English, like:

- "show me the pages on my SinoLink site"
- "list the CMS collections"
- "what's the SEO title on the homepage"

Every change it wants to make will ask your approval first, so nothing happens to your live site without your yes.

Tell me what the site list shows.

**Vishnu:** Let me first load the Webflow guide to understand available capabilities.
⏺ Calling webflow…
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 Tool use
   webflow - webflow_guide_tool(context: "User wants to list all Webflow sites associated with their account to see available projects and resources.") (MCP)
   Provides essential guidelines and best practices for effectively using the Webflow tools. Call this tool to understand recommended workflows and important considerations before performing actions. ALWAYS CALL THIS TOOL FIRST BEFORE CALLING ANY OTHER TOOLS.  ALWAYS CALL THIS TOOL FIRST BEFORE CALLING ANY OTHER TOOLS.
 Do you want to proceed?
 ❯ 1. Yes
   2. Yes, and don't ask again for webflow - webflow_guide_tool commands in /Users/vishnuvarthanvenkatapathy/Desktop/sinolink
   3. No
 Esc to cancel · Tab to amend

**Claude:** Press **1** then Enter.

This is exactly the approval prompt I mentioned. Claude is asking to run a Webflow tool — a harmless read-only one that just loads usage guidelines. Saying yes is safe.

You'll keep seeing these prompts before each Webflow action. For now, approve them one at a time with `1` so you stay in control. Once you trust a routine read action, option 2 saves you the repeated clicks — but `1` is the cautious default.

Go ahead, press `1`. It'll then move on to actually listing your sites (and likely ask permission again for that step — say yes there too).

**Vishnu:** ┌────────────────┬──────────────────────────┐
│     Field      │          Value           │
├────────────────┼──────────────────────────┤
│ Name           │ SinoLink                 │
├────────────────┼──────────────────────────┤
│ Short Name     │ sinolink-dev             │
├────────────────┼──────────────────────────┤
│ Site ID        │ 6a1988f9e12be13152632d3e │
├────────────────┼──────────────────────────┤
│ Workspace ID   │ 69c28e9c5b0c8f75de08ef92 │
├────────────────┼──────────────────────────┤
│ Time Zone      │ Asia/Calcutta            │
├────────────────┼──────────────────────────┤
│ Created        │ May 29, 2026             │
├────────────────┼──────────────────────────┤
│ Last Updated   │ June 7, 2026             │
├────────────────┼──────────────────────────┤
│ Last Published │ June 4, 2026             │
├────────────────┼──────────────────────────┤
│ Custom Domains │ None                     │
└────────────────┴──────────────────────────┘

**Claude:** Done. Everything works.

Claude in your terminal is now connected to your **SinoLink** site (and only that site). It can read pages, CMS collections, SEO data, and make changes — each one asking your permission first.

You're set. Just talk to it in plain English from here:

- "show me all the pages on the site"
- "list the CMS collections"
- "check the homepage meta title and description"
- "what's published vs draft right now"

If you ever close the terminal and come back: `cd ~/Desktop/sinolink`, then `claude`, and it reconnects to this same site automatically.

**Vishnu:** perfect it is wokin well and it is change the text work that i gave in the hero section ok now i need to change a set fo thisn lets do that 

i will give you the changes you make as a proerm promt to give to the terimnala webfliw

**Claude:** Give me the list of changes you want. I'll turn them into a clean prompt you can paste straight into the terminal.

One thing that makes the prompt work better — for each change, tell me **where** and **what**, like:

- which page (homepage, about, etc.)
- which section or element (hero, nav, footer…)
- the exact old text/thing → the exact new text/thing

Drop your changes in whatever rough form you have them. I'll tighten them into the prompt.

**Vishnu:** ↳ Consulting Company for China Sourcing and inter-cultural competence.
to 
↳ We design and source your new, whether BOM optimization or end-to-end supply chain full outsourced production.

We provide specialized engineering, technology transfer, and sourcing expertise, helping businesses connect with the right materials, manufacturing services for outsourcing, and quality control incl. certification services across global markets.to We provide specialized engineering, technology transfer, and sourcing expertise, helping businesses connect with the right materials, manufacturing services for outsourcing, and quality control incl. certification services across global markets.


* German Material Scientist and Engineering Expert
* Technology Transfer Expertise
* As Expat, over 10 years to many places in China
* Over 10 years substantial Experience as SinoLink Consultant
* Sourcing Specialist for all kinds of materials, outsourced mfg services and machinery for semiconductor, electronics  and automotive applications
to 

* Strategic Component Sourcing & BOM Optimization
* Full-Outsourced Production & Onsite Auditing
* Strong Partner Network supporting CAD, DFM, Compliance, FW…
* We secure cost optimization, source change and contracting in your supply chain

helping businesses connect with the right materials,  global markets.
to 

With deep root access to top manufacturing ecosystems in China, Southeast Asia, and Europe. Specializing in component and material sourcing, PCB/PCBA manufacturing, precision injection molding, from prototyping to NPI (New Product Introduction) or scaling to mass production – we do cover your needs.



Comprehensive Logistics Solutions

to 


Comprehensive Sourcing Solutions



+ YOUR INFO 
to 

+ Analyze My Supply Chain


Get In Touch
to 
Speak with our Sourcing Specialist

and in fomr section i need these input filesds 

Full Name
Company
Phone Number
Email Address
Message

and after that we need to give a message 

We are open for your NDA and will use strict protection protocols to protect your data and actively vet potential sources against international compliance along with your project necessities.


this are the changes and mention that thate shoud not be any desing chneg please staty wiht the deing themem 

no extra change

**Claude:** Here's your prompt — copy-paste this whole block into the terminal:

```
Make the following text changes on the SinoLink site. Do NOT change any design, layout, styling, colors, fonts, or structure. Only swap the text content exactly as specified. Keep the existing theme intact. No extra changes beyond what is listed.

1. Subheadline under hero:
FROM: "Consulting Company for China Sourcing and inter-cultural competence."
TO: "We design and source your new, whether BOM optimization or end-to-end supply chain full outsourced production."

2. Intro paragraph:
SET TO: "We provide specialized engineering, technology transfer, and sourcing expertise, helping businesses connect with the right materials, manufacturing services for outsourcing, and quality control incl. certification services across global markets."

3. Replace the bullet list:
FROM:
- German Material Scientist and Engineering Expert
- Technology Transfer Expertise
- As Expat, over 10 years to many places in China
- Over 10 years substantial Experience as SinoLink Consultant
- Sourcing Specialist for all kinds of materials, outsourced mfg services and machinery for semiconductor, electronics and automotive applications
TO:
- Strategic Component Sourcing & BOM Optimization
- Full-Outsourced Production & Onsite Auditing
- Strong Partner Network supporting CAD, DFM, Compliance, FW…
- We secure cost optimization, source change and contracting in your supply chain

4. Paragraph:
FROM: "helping businesses connect with the right materials, global markets."
TO: "With deep root access to top manufacturing ecosystems in China, Southeast Asia, and Europe. Specializing in component and material sourcing, PCB/PCBA manufacturing, precision injection molding, from prototyping to NPI (New Product Introduction) or scaling to mass production – we do cover your needs."

5. Heading:
FROM: "Comprehensive Logistics Solutions"
TO: "Comprehensive Sourcing Solutions"

6. Button text:
FROM: "+ YOUR INFO"
TO: "+ Analyze My Supply Chain"

7. Heading:
FROM: "Get In Touch"
TO: "Speak with our Sourcing Specialist"

8. Contact form fields — set the form to have exactly these fields, in this order:
- Full Name
- Company
- Phone Number
- Email Address
- Message

9. Add this text below the form:
"We are open for your NDA and will use strict protection protocols to protect your data and actively vet potential sources against international compliance along with your project necessities."

After making the changes, list what you changed. Do not publish — leave it in draft for me to review.
```

Two notes:

- I added "do not publish — leave it in draft" at the end so nothing goes live until you check it. Remove that line if you'd rather it publish directly.
- For change #8 (form fields), the MCP may or may not be able to add/edit form input fields via the Designer API — form editing is more limited than text editing. If Claude says it can't modify the form fields, tell me and I'll give you the manual steps to do that part inside Webflow Designer.

**Vishnu:** ok if i pase this in the temaial and the claus make chanehg whihc token will be used claud token or webflow ai tolen

**Claude:** Neither uses an "AI token." Two separate things are happening:

- **Claude Code usage** runs on your **Anthropic/Claude subscription** (the plan tied to the account you're logged into in the terminal). That's what pays for Claude doing the thinking and writing.
- **Webflow access** uses the **OAuth connection** you authorized earlier — not a paid AI token, just permission to read/write your site. Webflow doesn't charge "AI tokens" for MCP edits; it's normal API access under your existing Webflow plan.

So: Claude's work = your Claude plan. Touching the site = your Webflow OAuth permission. No Webflow "AI credits" get consumed by these text edits.

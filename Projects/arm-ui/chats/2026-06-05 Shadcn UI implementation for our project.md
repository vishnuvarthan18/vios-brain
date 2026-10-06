---
tags: chat
date: 2026-06-05
source: Claude personal account
uuid: 1c3b66ab-5c88-4aba-bed1-c338be03ed69
---
# Shadcn UI implementation for our project

## Summary
**Conversation Overview**

The person is exploring UI component libraries for a project they are building and had questions about shadcn/ui specifically. The conversation covered three main topics: which companies use shadcn/ui, what "blocks" are within the shadcn ecosystem, and whether a shadcn-equivalent exists for UnoCSS.

Claude explained that shadcn/ui operates on a copy-paste model rather than a traditional npm dependency, making comprehensive adoption tracking difficult. Detection tools like Wappalyzer estimate hundreds of thousands of sites using it, while smaller databases list named companies. Claude clarified that shadcn's "blocks" are full pre-built page sections (dashboards, auth flows, sidebars) composed from primitive components. The recommended workflow discussed was to initialize via CLI, set design tokens first, and add components on demand rather than bulk-copying everything.

Regarding UnoCSS, the person asked whether a shadcn-equivalent exists built on UnoCSS rather than Tailwind v3. Claude identified **una-ui** as the closest direct analog — a Nuxt/Vue component library built on UnoCSS using the copy-paste philosophy and Radix-Vue primitives. Claude also noted that UnoCSS's `preset-wind` makes shadcn components largely compatible with UnoCSS's engine, but recommended staying on Tailwind unless there was a specific reason to switch. The person's frontend stack was not confirmed during the conversation.

## Chat

**Vishnu:** what are the company using https://ui.shadcn.com/, what are the blokes and what are the plae we need to use somethign like this for oru project what do you suggest can we make this whole cp

**Claude:** Your

**Vishnu:** what are the company using https://ui.shadcn.com/, what are the blokes and what are the plae we need to use somethign like this for oru project what do you suggest can we make this whole companets copya and create a new ui for us easliy

**Claude:** Your question has a few distinct parts. Let me address each.

**Companies using shadcn/ui**

Shadcn/ui is widely adopted because it's not a dependency — you copy the components into your codebase. Known adopters and contexts:

- **Vercel** — the creator (shadcn works there); used across their products and v0.
- **OpenAI, Sonos, Adobe, Mistral** — cited as users in shadcn's own materials.
- Thousands of startups and indie projects, since it's the default for most Next.js + Tailwind stacks today.

The exact roster shifts, so treat any "logo wall" as indicative, not authoritative.

**What "blocks" are**

Blocks (at `ui.shadcn.com/blocks`) are pre-built, full sections composed from primitive components — not single buttons but complete patterns: dashboards, login/signup screens, sidebars, auth flows, settings pages, calendars. You copy a whole block, then strip it down.

**The model itself**

Shadcn/ui is not a component library you install and import. It's a CLI that drops the source code (Radix primitives + Tailwind classes) directly into your repo. You own the code, so you restyle freely. This is the core reason it fits your "make a new UI for us easily" goal.

**Can you copy all components and build your own UI?**

Yes — that's the intended workflow, not a hack.

1. Install the CLI: `npx shadcn@latest init`
2. Add components as needed: `npx shadcn@latest add button card dialog` (or `add` a block).
3. Customize via your `globals.css` design tokens (CSS variables for color, radius, spacing) — change these once and every component reskins automatically.
4. Edit individual component files for structural changes.

**My recommendation**

Don't copy everything upfront. Pull components on demand as your screens need them — bulk-copying bloats your repo with unused code. Set your design tokens first (`--primary`, `--radius`, font stack), then add components; they'll inherit your brand immediately without per-component edits.

**One thing worth verifying for your project:** is your stack React + Tailwind? Shadcn/ui assumes both. If you're on Vue, Svelte, or Angular, you need a port (e.g. `shadcn-vue`, `shadcn-svelte`) — same philosophy, different ecosystem.

What's your current frontend stack?

**Vishnu:** list me all the compsny used shadcn

**Claude:** I'll search for current information on this rather than rely on memory.

_[tool: web_search]_

No definitive public roster of "all companies using shadcn/ui" exists — adoption is detected by third-party scanners, and the counts vary wildly by source.

**Scale of adoption (by detection source)**
- Wappalyzer tracks 580,000 live websites using shadcn/ui.
- Prospeo lists 4,698+ companies.
- TheirStack has data on 43 companies.
- Hunter.io and Reo.dev detect far fewer (71 and 25 respectively).

These differ because each scanner only detects sites it crawls, and shadcn's copy-paste model leaves no npm dependency fingerprint — so detection undercounts heavily.

**Named companies confirmed in these sources**
From the shadcn/ui homepage and detection lists: Vercel (creator), plus detected domains including `adobe.io`, `arag.com` (insurance), `adomik.com`, `aliz.ai`, `bejamas.io`. The shadcn-svelte port (different framework) shows recognizable names: Appwrite, Zipline, Decathlon Türkiye, dbrand, CometChat, Goldcast.

**The accurate answer:** there is no authoritative, complete list. shadcn's own marketing cites large brands (OpenAI, Sonos, Adobe, Mistral) as users, but because the components are copied into codebases rather than imported, no tool can reliably enumerate every adopter. The "4,698" and "580,000" figures are lead-generation databases, not verified rosters — and most are gated behind paid signups.

If you need this for a specific reason (competitive research, credibility for a pitch), tell me the use case and I'll point you to the most reliable single source.

**Vishnu:** like shadcn is bulid tw3 
is somethign there build on https://unocss.dev/

**Claude:** Yes. The shadcn equivalent for UnoCSS exists, though the ecosystem is smaller.

**Closest direct analog**
- **shadcn-vue / shadcn-svelte ports don't use UnoCSS by default** — they're Tailwind-based. But several community projects bring the shadcn copy-paste model to UnoCSS.

**What's actually built on UnoCSS**
- **`@unocss/preset-wind` (and `preset-wind3`/`preset-wind4`)** — this is the core piece. It's UnoCSS's Tailwind-compatible preset, replicating Tailwind's utility API. This is why a shadcn-style setup is even possible on UnoCSS: the class names match, so shadcn components mostly work after swapping the engine.
- **`una-ui`** (una-ui.netlify.app) — the most direct answer. A Nuxt/Vue component library explicitly built on UnoCSS, modeled on shadcn's philosophy (copy-paste, you own the code, Radix-Vue primitives). This is the one to look at.

**The mechanism worth understanding**

UnoCSS isn't a UI library — it's an atomic CSS *engine* (like Tailwind, but faster and more configurable, on-demand). Tailwind v3 is the layer shadcn assumes. So "is there a shadcn for UnoCSS" really means "can shadcn's Tailwind classes run on UnoCSS's engine" — and yes, via `preset-wind`, often with minor config.

**My recommendation**

If you want shadcn specifically, stay on Tailwind — that's the supported path, and you avoid fighting subtle preset mismatches. Choose UnoCSS only if you have a concrete reason: build speed, multi-framework setup, or custom utility generation. For that case, use **una-ui** rather than trying to retrofit shadcn onto UnoCSS yourself.

What's driving the UnoCSS interest — performance, a non-React framework, or something else? That determines whether the switch is worth it.

---
tags: chat
date: 2026-06-23
source: Claude personal account
uuid: 72ecd35e-2b3e-428d-9801-f24a493d09e3
---
# Customizing Webflow template for brand

## Summary
**Conversation Overview**

The person is working on a Webflow website build for a brand called DreamSpace. They purchased a template from the Cambridge education/ecommerce template (https://az-cambridge.webflow.io/home), duplicated it, and have been customizing it to match their brand. Their in-progress site is at https://dsa-dev.webflow.io/. The person writes with informal shorthand and spelling variations, which is their natural communication style.

The core issue they brought to Claude was that image placement and layout in their hero section were not matching the original template correctly. They shared a screenshot comparing their version to the template original. Claude identified three specific problems: the DreamSpace hexagon logo was incorrectly placed as a center overlay element (instead of a decorative shape accent), the three image cards had inconsistent sizing and uneven stagger/overlap, and the concentric ring background element was missing from the parent wrapper div. Claude provided actionable Webflow-specific fixes for each issue, including equalizing card dimensions, correcting the hexagon logo placement, and restoring the background SVG/PNG rings layer. Claude also offered to write a Claude Code MCP prompt to inspect and fix the layout directly, referencing the likely section name as `Home Multi Image` block.

**Tool Knowledge**

Claude attempted to use the `web_fetch` tool on both URLs. Fetching the original template (https://az-cambridge.webflow.io/home) succeeded and returned enough structural information to describe the layout. Fetching the staging site (https://dsa-dev.webflow.io/) failed because Webflow staging sites block automated access, requiring the person to share screenshots directly instead.

## Chat

**Vishnu:** hey i am workin on the webflow 

i buy the temaplte and duplicateed that and do the needed cheng to macth our banrd can  will share images first

**Claude:** Go ahead and share the images.

**Vishnu:** https://az-cambridge.webflow.io/home this is the temaplarte that i buy

**Claude:** I can't see the template from just the URL. Let me look at it.

_[tool: web_fetch]_

Got it — Cambridge education/ecommerce template. Hero with circle image, trust logos, benefits cards, course grid, blog, testimonials, FAQ, newsletter.

Share your images and tell me what changes you need to match your brand.

**Vishnu:** https://dsa-dev.webflow.io/  this is the theinge i ahve buidl 

but now the porble in the iamge are nore coorect placing and more ussues

**Claude:** _[tool: web_fetch]_

Can't fetch that one — the staging site blocks automated access.

Share the screenshots of the image placement issues. I can see them directly and tell you the fixes.

**Claude:** Image 1 (yours) vs Image 2 (template original) — your image cluster is broken in three ways:

1. **Center decorative element wrong.** Template has a clean orange leaf/circle motif sitting behind/between the three image cards. Yours has the DreamSpace hexagon logo dumped in the center overlapping the photos — it reads as a misplaced asset, not a design element.

2. **Card sizing/overlap off.** Template's three cards are equal-sized, evenly staggered with consistent gaps and overlap rhythm. Yours are unequal sizes (top tall, middle small, bottom medium) and the overlap looks accidental rather than staggered.

3. **Concentric ring guides missing.** Template has faint circular rings centered on the cluster giving it structure. Yours lost them, so the cluster floats with no anchor.

**Fixes in Webflow:**

- **Hexagon logo** — that's the `Home Multi Image` slot or a stacked image div. Either remove the hexagon from center, or move it out as a small accent. Don't let it overlap two photos. Replace with the template's leaf/circle shape (or your brand's circular accent) as the center anchor.
- **Equalize cards** — select all three image cards, set identical `width`/`height` (or `aspect-ratio: 1`), same `border-radius`. Re-apply the stagger via consistent negative margins or absolute offsets.
- **Restore rings** — the concentric circles are a background SVG/PNG on the parent wrapper. Check if you deleted that image layer; re-add it as background, centered.

To get exact class names and fix directly, open this section in Webflow and tell me — or use Claude Code MCP on the site. Which section is this? (`Home Multi Image` block based on the template.)

Want me to write the Claude Code MCP prompt to inspect and fix the layout?

---
tags: chat
date: 2026-07-01
source: Claude personal account
uuid: 5a405e36-d3b9-4cee-a88f-3999487873d0
---
# Webflow AI not showing on some sites

## Summary
**Conversation Overview**

The person asked about inconsistent Webflow AI feature availability across multiple sites in their workspace. Specifically, they observed that some sites showed AI features while others did not, and sought to understand why. The conversation used Webflow-specific terminology including site plans, workspace plans, Starter tier, and CMS Basic plan.

Claude walked through several possible explanations for the discrepancy, including site plan tier restrictions, workspace seat permissions, regional rollout differences, and grandfathered access. The person confirmed that at least one affected site was on the Starter plan, which Claude identified as the free tier that typically excludes AI features. However, the person then noted that another site also on the Starter plan did have AI access, creating an unexplained inconsistency. Claude offered speculative reasons such as grandfathered access, workspace-level billing credits, or A/B testing rollouts, but acknowledged these were unconfirmed. The person also asked whether there was a toggle to enable AI on a specific site, and Claude confirmed no such option exists — access is plan-determined, not switch-controlled. Claude ultimately recommended the person contact Webflow support directly, as the inconsistency between two identically-tiered sites appeared to be an account-specific backend entitlement issue beyond what could be diagnosed or resolved from the user side.

## Chat

**Vishnu:** i have multiple slites in the webfliw but for the few sites the webflow ai is showing but for other site it not showing why

**Claude:** Webflow AI (the Workspace-level AI features) is gated per-site by a few things:

- **Site plan tier** — AI features require the site to be on a paid plan (CMS/Business/Ecommerce) in some workspace configurations; free/Starter sites often don't show it.
- **Workspace seat permissions** — AI access can be tied to your role on that specific site, not just the workspace.
- **Site created via different workspace/account** — if a site was transferred or is under a different workspace than others, its AI entitlement may not carry over.
- **Regional/staged rollout** — Webflow has rolled AI features gradually; not all sites in an account get it simultaneously even with the same plan.

Check: open the site missing AI → Site Settings → General → confirm plan tier, then compare against a site where it works.

Worth verifying directly with Webflow support since AI rollout status isn't something I can confirm live — want me to pull up their current docs on this?

**Vishnu:** * This site is currently on the Starte!
site plan. this is what it is showing

**Claude:** That's your answer — AI features require a paid site plan (CMS Basic or higher). Starter is Webflow's free tier and excludes them.

Fix: upgrade that specific site's plan (Site Settings → Billing/Upgrade), not the workspace plan — Webflow bills AI access per-site, so having other sites on paid plans doesn't extend it.

**Vishnu:** but anothet site is aslo in the starter plan how that alon showing ai

**Claude:** Possible reasons for the mismatch:

- **Grandfathered access** — that site may have been upgraded to a paid plan previously (even briefly) and retained AI access after downgrading to Starter, or was created during a promo/beta window.
- **Workspace-level AI credits** — some Webflow workspace plans (not site plans) include AI features that apply across all sites in that workspace, regardless of individual site plan — check Workspace Settings → Billing, not just the site's plan.
- **Feature flag rollout** — Webflow sometimes A/B tests or gradually enables AI per-site rather than strictly by plan, so two Starter sites can differ.

Check Workspace Settings → Billing first — if your workspace itself is on a paid plan (Core/Growth/Agency), that could be granting AI access site-wide independent of each site's individual tier.

**Vishnu:** is ther any option to on ai in the perticular sit

**Claude:** No direct per-site toggle for AI in site settings — access is determined by plan (site or workspace), not a switch you flip.

Only path: upgrade that specific site to a paid plan (CMS Basic or higher) via Site Settings → Billing/Upgrade, or confirm your workspace plan already grants it and troubleshoot why it isn't applying (could need a page refresh, re-login, or site re-sync with the workspace).

Worth verifying: contact Webflow support directly — this behavior (same plan tier, different AI access) sounds like an account-specific bug or entitlement issue they'd need to check on their end, not something fixable from your side.

**Vishnu:** heyy one sie has ai and one not habve but both are in the staret pack how

**Claude:** I don't have a confirmed answer for this — my earlier guesses (grandfathering, workspace credits, feature flags) are speculation, not verified Webflow behavior.

Worth verifying: open Webflow's live chat/support and ask directly why two Starter-plan sites show different AI access. That's an account-specific entitlement question only they can see on their backend.

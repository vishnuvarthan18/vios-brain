---
tags: chat
date: 2026-07-14
source: Claude personal account
uuid: e6cca2f3-abc0-46b5-9a62-17a200b22f25
---
# Starvaa connection setup

## Summary
**Conversation Overview**

The person attempted to connect their Strava account (with a minor typo in their initial request — "starvaa" — which Claude correctly interpreted as Strava). Claude confirmed the connection attempt but found the account ineligible for the Strava MCP integration, as it requires an active Strava subscription. The person confirmed this was indeed the service they meant. No tasks were completed due to the eligibility barrier; the resolution was that the person would need to subscribe to Strava directly before the integration can be enabled.

**Tool Knowledge**

Strava's MCP connector requires an active Strava subscription on the user's account to be eligible; without it, the tool returns an ineligibility status that cannot be bypassed from Claude's side. When a user attempts to connect Strava and the eligibility check fails, the issue lies with their Strava subscription status, not with the connection process itself.

## Chat

**Vishnu:** connect my starvaa

**Claude:** I don't have a connector matching "starvaa" — but you have Strava connected. Is that what you meant?

_[tool: tool_search]_

_[tool: Strava:eligibility]_

Your account isn't eligible — Strava's MCP requires a Strava subscription. Not something I can bypass; you'd need to subscribe on Strava's side first.

**Vishnu:** yes

**Claude:** Confirmed — your Strava account isn't eligible for the MCP toolset. It requires a Strava subscription; without one, I can't pull your activity data through this connector. Subscribe on Strava's end if you want that access.

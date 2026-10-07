---
type: agent
zone: ai
date: {{date}}
name: {{name}}
status: active        # idea | active | paused | retired
runner:               # opencode | goose | hermes | claude-code | cursor | n8n-free | custom
model:                # e.g. claude-opus, qwen3, gpt-oss
schedule:             # none | cron expression | on-demand
projects: []
tools_allowed: []     # e.g. [read, write:ai/, backlog, qmd]
tools_denied: [delete, network-send, core-write, private]
tags: [agent]
---
## For future agent
Registry card for the AI agent [[{{name}}]]: what it does, where it runs, what it may touch. Runs are logged in ai/logs/runs/.

# Agent — {{name}}

## Job (one line)
- 

## Inputs → Outputs
- In: 
- Out: 

## Prompt / instructions
- Location: 

## Limits (Rule of Two)
- Untrusted input? yes/no · Private data? no · Can send out? yes/no  → max two "yes"

## Last runs
- 

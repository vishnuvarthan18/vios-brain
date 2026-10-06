# Engineering Thinking

## Purpose
This is not a 5th subject to finish — it's the reasoning layer underneath all of them. You said it directly: you don't just want facts in each domain, you want to think the way an engineer thinks. This doc holds the core mental models; each domain roadmap (software, electronics, mechanical) should apply these explicitly as it's worked through, not just teach facts.

## Core mental models

### First-principles thinking
Break a problem down to its fundamental truths, not the nearest analogy or existing solution. Ask "what do I actually know to be true here, from the ground up" instead of "how do others usually solve this." This is the opposite of vibe-coding/tutorial-following — it's exactly the gap identified in your software and electronics diagnostics.

### Systems thinking
Nothing in engineering exists in isolation. A change in one part affects others (a circuit's load affects voltage elsewhere; a material choice affects manufacturing cost; a code change affects performance elsewhere). Always ask: what does this connect to, and what breaks if I change it?

### Constraints define the problem
Real engineering is never "build the best possible thing" — it's "build the best thing given cost, time, materials, physics." Learning to see the constraints clearly is often more valuable than knowing more facts.

### Debugging mindset
Assume the system is telling you the truth and you're missing something, not that the system is broken for no reason. Isolate variables, change one thing at a time, verify your assumptions rather than guessing. This applies identically to a bug in code, a circuit that doesn't work, or a mechanism that binds.

### Tradeoff analysis
Every design decision is a tradeoff (speed vs. cost, strength vs. weight, simplicity vs. flexibility). An engineer's job is rarely "find the perfect answer" — it's "make the tradeoff explicit and choose deliberately." Being able to name the tradeoff you're making, out loud, is itself a skill.

### Reading before building
Datasheets, documentation, existing code/schematics — real engineers read carefully before acting. This is a direct antidote to "connect wires as the tutorial says" and "vibe code it."

## How this gets applied
- Software: apply first-principles + debugging mindset from Phase 1 onward — don't just learn syntax, learn to reason about why code behaves as it does.
- Electronics: apply systems thinking to every circuit (what does this component affect elsewhere) and reading-before-building to datasheets from Phase 1.
- Mechanical: apply constraints + tradeoff analysis heavily in Phases 2–4 (materials, manufacturing) — this is where tradeoffs are most concrete and visible.
- Strategy: tradeoff analysis and constraints thinking directly support the "hype vs. substance" evaluation gap already identified.

## Progress log
- 2026-09-01: Doc created as the cross-cutting reasoning layer. Reinforce it explicitly whenever working through a phase in any domain doc — don't treat it as a one-time read.

---
tags: chat
date: 2026-06-12
source: Claude personal account
uuid: 2d79abb3-66f7-4e86-a768-85bb9f584578
---
# Generate separate dev and testing documentation

## Summary
**Conversation Overview**

The person was working on creating structured technical documentation and sought help refining a prompt for generating two sub-documents: one developer specification and one QA/test plan. The core goal of the documents is rebuilding existing functionality, with a clear separation between existing behavior and mandatory new requirements beyond existing behavior. The person uses IDs like F-XX and NR-XX for traceability across both documents.

The person's communication style is brief and informal, and they explicitly corrected Claude's first attempt for being too long and padded with unnecessary context. Their stated preference is for content that is small, neat, and free of extra explanation. Claude revised the prompt accordingly, producing a compact version covering both documents with rules, structure, and traceability requirements in a condensed format.

## Chat

**Vishnu:** this is sooo good using this 

next genertee two sub documets for one dev team and one for testing team 


dev team docunt shoube have proper list of featues and funcianlyt and mention spereate Mandatory New Requirements (beyond existing behavior) becasue thee main toint is to rebiuld what is exsiingnt so make the docimet as deveoper frenly and it need to be int he wat theat they dont miss a single thigns 

adn for test make. it like a poepr arest an things need to chce for that alos handel seprer what is now and what is Mandatory New Requirements (beyond existing behavior)


= just ocrrec this and  make rthis as a poepr highteffect promt

**Claude:** Generate two separate sub-documents from the source material: one for the **development team** and one for the **QA/testing team**. The core goal is a faithful rebuild of existing functionality, so both documents must cleanly separate existing behavior from new requirements.

```
Using the source material provided, produce TWO standalone documents. The primary objective is to REBUILD existing functionality exactly as-is — treat existing behavior as the source of truth. Both documents must keep existing behavior and new requirements in clearly separated sections.

═══════════════════════════════════════
DOCUMENT 1 — DEVELOPER SPECIFICATION
═══════════════════════════════════════
Audience: engineers rebuilding the product. Goal: zero ambiguity, nothing missed.

Structure:
1. Overview — what is being rebuilt, in 2–3 sentences.
2. Existing Features & Functionality — exhaustive list of every feature, behavior, state, edge case, and interaction in the current system. Each item must be specific enough to implement without seeing the original. Include: trigger, expected behavior, data involved, UI/interaction states, dependencies.
3. Mandatory New Requirements (beyond existing behavior) — separate section. Each new requirement listed discretely, with rationale and acceptance criteria. Never mix these with existing behavior.
4. Data Model / API / Logic — where applicable, the structures and rules needed to build.
5. Out of Scope — explicitly state what NOT to build, to prevent scope creep.

Rules:
- Be exhaustive. Assume the developer has never seen the original — if it's not written, it won't get built.
- Every feature gets a unique ID for traceability (e.g., F-01, NR-01).
- No vague verbs ("handle", "manage"). State the exact behavior.

═══════════════════════════════════════
DOCUMENT 2 — QA / TEST PLAN
═══════════════════════════════════════
Audience: testers verifying the rebuild. Goal: complete coverage, traceable to Document 1.

Structure:
1. Scope & Approach — what is being tested, test types (functional, regression, edge, UI).
2. Test Cases — Existing Behavior — one section. Each case: ID, linked feature ID (F-XX), preconditions, steps, expected result, pass/fail. Cover happy paths, edge cases, and error states.
3. Test Cases — Mandatory New Requirements (beyond existing behavior) — separate section, linked to NR-XX IDs.
4. Regression Checklist — confirm rebuilt behavior matches original exactly.
5. Traceability Matrix — map every feature/requirement ID to its test case ID(s) so nothing is untested.

Rules:
- Every feature and requirement in Document 1 must have at least one test case.
- Keep existing-behavior tests and new-requirement tests in separate sections — never blended.
- Each test case must be executable by someone unfamiliar with the product.
```

Two structural choices I'd flag: cross-document ID traceability (F-XX/NR-XX shared between both docs) is the single highest-leverage addition — it's what guarantees nothing slips. And the explicit **Out of Scope** section in the dev doc is what prevents a "rebuild" from quietly becoming a redesign.

Want this tuned for a specific format (Word doc, Markdown, Figma-ready), or is the prompt itself the deliverable?

**Vishnu:** no this muhc big needed make it small and neat no need extra contct

**Claude:** ```
From the source material, produce TWO documents. Goal: faithfully REBUILD existing functionality. In both docs, keep "Existing Behavior" and "Mandatory New Requirements (beyond existing behavior)" in separate sections.

DOC 1 — DEVELOPER SPEC
- Existing Features & Functionality: exhaustive list. Each item = trigger, behavior, data, states, dependencies. Assume the dev has never seen the original — if unwritten, it won't get built. ID each (F-01...).
- Mandatory New Requirements (beyond existing behavior): separate section, each with acceptance criteria. ID each (NR-01...).
- Out of Scope: what NOT to build.

DOC 2 — QA / TEST PLAN
- Test Cases (Existing Behavior): each linked to F-XX. ID, preconditions, steps, expected result. Cover happy path, edge, error states.
- Test Cases (Mandatory New Requirements): separate section, linked to NR-XX.
- Traceability: every F/NR ID maps to ≥1 test case.

Rules: no vague verbs; every feature/requirement must have a test case; never blend existing and new.
```

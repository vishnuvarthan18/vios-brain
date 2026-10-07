---
tags: chat
date: 2026-07-08
source: Claude personal account
uuid: f416700d-48fc-4755-99ae-4d6435c1a9d4
---
# Project knowledge migration to Cursor IDE

## Summary
**Conversation Overview**

Vishnu is a solo founder based in Vellore, Tamil Nadu, India, building a B2B mobile marketplace called Wedding2day (W2D) for the wedding decoration industry. He has over 10 years of experience in wedding stage decoration manufacturing. The project is a React Native app built with Expo (managed workflow), Expo Router, NativeWind, and Firebase (Auth, Firestore, Storage, FCM), with EAS for builds. The Firebase project ID is `wedding2day-a99ea`, the EAS account is `vishnu18`, and the app package name is `com.w2d.app`. The active project folder on his Mac is `~/Desktop/w2d`.

In this conversation, Vishnu asked Claude to produce an exhaustive brain-dump of all project knowledge into six structured files for migration to Cursor IDE: `.cursorrules`, and five docs covering product context, architecture and data, integrations and APIs, core logic and constraints, and an execution roadmap. Claude generated all six files using the project's uploaded v1 scope document and stored knowledge, then presented them for download. The files covered the full locked tech stack, Firestore data model (collections: `users`, `listings`, `interests`), 38 hardcoded Tamil Nadu districts, Firebase emulator configuration, known bugs and their solutions (including the critical `google-services.json` reference in `app.json`, emulator host IP issues on physical devices, and wrong directory errors), Firestore and Storage security rules, and a granular phase-by-phase development roadmap.

Vishnu then clarified that the Cursor migration plan has been dropped entirely, and asked whether the six documents were still worth keeping. Claude confirmed all six remain useful as project reference docs inside `~/Desktop/w2d/docs/` — particularly for pasting into new Cursor chat sessions at the start of each development phase. When Vishnu asked Claude to save the files directly to his Mac, Claude clarified it cannot access his local file system and provided simple Terminal commands to move the downloaded files into place. The conversation ended with Vishnu closing all terminals and stepping away until the next day, with Phase 3 (Authentication — Phone OTP and Google Sign-In) identified as the next development step.

## Chat

**Vishnu:** I am migrating this entire project to Cursor IDE. I need an exhaustive, uncompressed brain-dump of everything stored in this Project's knowledge, uploaded files, and our chat history.
Do not summarize, skip technical details, or leave out nuances. Output the following files in Markdown code blocks so I can save them locally:
1. .cursorrules The strict coding standards, tech stack, library versions, linting rules, and naming conventions the AI agent must follow.
2. docs/1_product_context.md The business strategy, target audience, core user journeys, UI/UX design system (colors, typography, component rules), and positioning.
3. docs/2_architecture_and_data.md The complete system design, folder structure, database schemas, data models, state management strategy, and how different services communicate.
4. docs/3_integrations_and_apis.md Every third-party service, API, authentication method, and external dependency required, including how they should be implemented.
5. docs/4_core_logic_and_constraints.md The complex business logic, specific edge cases we discussed, security constraints, and any technical hurdles or bugs we already solved in this chat.
6. docs/5_execution_roadmap.md A granular, step-by-step development roadmap categorized into small, testable milestones.

**Claude:** _[tool: view]_

_[tool: view]_

All knowledge sourced. Generating all six files now from userMemories, project file, and the v1 scope document.

_[tool: create_file]_

_[tool: bash_tool]_

_[tool: create_file]_

_[tool: create_file]_

_[tool: create_file]_

_[tool: create_file]_

_[tool: create_file]_

_[tool: bash_tool]_

_[tool: bash_tool]_

_[tool: present_files]_

All 6 files generated. Here's exactly where each goes in your Cursor project:

| File | Save location in `~/Desktop/w2d/` |
|------|----------------------------------|
| `.cursorrules` | Root: `~/Desktop/w2d/.cursorrules` |
| `1_product_context.md` | `~/Desktop/w2d/docs/1_product_context.md` |
| `2_architecture_and_data.md` | `~/Desktop/w2d/docs/2_architecture_and_data.md` |
| `3_integrations_and_apis.md` | `~/Desktop/w2d/docs/3_integrations_and_apis.md` |
| `4_core_logic_and_constraints.md` | `~/Desktop/w2d/docs/4_core_logic_and_constraints.md` |
| `5_execution_roadmap.md` | `~/Desktop/w2d/docs/5_execution_roadmap.md` |

**One thing to do after placing these files:** In Cursor, open Settings → Rules for AI (or the `.cursorrules` file is auto-detected in the project root) — Cursor will pick it up automatically as the agent ruleset for every prompt in that project.

**One correction vs. your original system prompt:** Your system prompt listed FlutterFlow + Supabase + Twilio as the stack, but your actual memory shows those were fully abandoned. The docs I generated reflect the real locked stack (Expo + Firebase) and explicitly mark the old stack as abandoned. Do not let Cursor's agent see that original system prompt — it will confuse it.

**Vishnu:** no i have change the plan see if this docuem will help for use in future if yes save that

**Claude:** The document you're referring to — can you clarify which one? I see the v1 scope file in the project, but you mentioned "this document" — did you mean to upload a file? I don't see a new attachment in your message.

**Vishnu:** i am saying about the docuemts that you have create the 6 docmuet moving to the curose plan has dropped

**Claude:** Got it. You've dropped the Cursor migration plan entirely — the 6 documents are no longer needed for that purpose.

The real question: are those 6 docs still useful as **project reference docs** inside `~/Desktop/w2d/docs/` for your ongoing Expo + Firebase build?

Yes — they are. Here's why each one is still valuable:

| Doc | Still useful? | Why |
|-----|--------------|-----|
| `.cursorrules` | ✅ Yes | Cursor Pro is still your coding AI inside the IDE — these rules govern how it behaves |
| `1_product_context.md` | ✅ Yes | Business context, user journeys, v1 scope — paste into any new Cursor chat as context |
| `2_architecture_and_data.md` | ✅ Yes | Folder structure, Firestore schema, TypeScript types — reference during every phase |
| `3_integrations_and_apis.md` | ✅ Yes | Firebase setup, EAS commands, security rules — you'll consult this repeatedly |
| `4_core_logic_and_constraints.md` | ✅ Yes | Known bugs + fixes already solved — prevents re-hitting the same walls |
| `5_execution_roadmap.md` | ✅ Yes | Phase-by-phase checklist — your build plan |

**Recommendation:** Keep all 6. Place them in `~/Desktop/w2d/docs/`. They serve as your project memory that you paste into each new Cursor chat at the start of a phase — which is exactly the workflow you're already using.

Do you want to proceed with Phase 3 (Auth)?

**Vishnu:** you it self save it there

**Claude:** I can't do that — I don't have access to your Mac's file system. I can only create files inside this chat's temporary container, which you've already downloaded above.

To save them to `~/Desktop/w2d/docs/`, you do this:

1. Download each file from the links above
2. Open Terminal on your Mac
3. Run:
```bash
mkdir -p ~/Desktop/w2d/docs
```
4. Move the downloaded files into that folder

That's it — 2 minutes of work.

**Vishnu:** ok lete cloase all  i havr closed all the termainals and all lets see tomotrreo

**Claude:** Ok. See you tomorrow.

---
tags: chat
date: 2026-10-02
source: Claude personal account
uuid: 71647b51-2bf0-4514-9338-8f67ea257fe2
---
# Moving project to viOS

## Summary
**Conversation overview**

The person asked Claude to help move a project into their personal knowledge management system called viOS, following a workflow documented in RULES.md. The person's name is Vishnu. The request outlined a detailed multi-step process: first reading all chats and files within the project, then checking viOS for existing pages on People, Companies, Tools, and Projects to avoid duplicates. The workflow then called for writing a structured project summary with specific frontmatter and sections covering project overview, status, next steps, decisions, timeline, key facts, files, open questions, and a list of all chats in the project. The person specified that all people, companies, tools, and other projects mentioned should be converted into wiki-style links (e.g., [[People/Name]], [[Tools/Name]]) and that linked pages should be created or updated accordingly. The process also included saving work in incremental parts to avoid timeouts, creating STATE.md and LOG.md files, appending to a daily log, and logging any unclear items to a dedicated questions file for the person to review later. Claude acknowledged the request and indicated it would begin by reading all chats and files, checking viOS for existing pages, and then proceed through the summary, linked pages, state, log, and questions steps. The actual content of the project being summarized was not yet discussed in this excerpt.

## Chat

**Vishnu:** You are helping me move this project into my personal second brain (viOS). Follow RULES.md in viOS.

STEP 1 — READ
Search and read ALL chats in this project, from the very first to the latest. Also read the project instructions and project files. Then use list_notes on People, Companies, Tools and Projects in viOS so you reuse pages that already exist (same spelling).

STEP 2 — WRITE the summary in this format (simple English, short points, nothing important skipped, "(not sure)" if unsure):

---
tags: project
status: <active / paused / done>
owner: "[[People/Vishnu]]"
---
# PROJECT: <project name>
## 1. What this project is
## 2. Status now (as of <today's date>)
## 3. Next steps
## 4. Decisions
- <date> — <what> — <why> #decision   (every decision, oldest first)
## 5. Timeline
- <date> — <what happened>
## 6. Key facts (people, companies, tools, links, GitHub repos, servers, where work was done e.g. Claude Code — NO passwords or keys)
## 7. Files and documents
## 8. Open questions and problems
## 9. All chats in this project
- <chat title> — <date>   (EVERY chat)

LINKS: turn every person, company, tool and other project into a link:
[[People/Name]], [[Companies/Name]], [[Tools/Name]], [[Projects/project-name/SUMMARY]].

STEP 3 — SAVE in small parts (so it does not time out). Folder: Projects/<project-name-lowercase-with-dashes>/
a) write_note SUMMARY.md with frontmatter, title and sections 1–3
b) append_note section 4
c) append_note sections 5–6
d) append_note sections 7–9
Say "Part saved" after each. If a save fails, wait and retry that part.

STEP 4 — LINKED PAGES
For each person, company and tool linked: if the page does NOT exist, create it with write_note:
---
tags: person / company / tool
---
# Name
- One line about who/what it is
- Used in: [[Projects/<project-name>/SUMMARY]]
If the page ALREADY exists, use append_note to add: - Used in: [[Projects/<project-name>/SUMMARY]]

STEP 5 — STATE and LOG
- write_note STATE.md (same frontmatter as SUMMARY): where it is now, next steps, blockers, key links. Link to [[Projects/<project-name>/SUMMARY]] and [[Projects/<project-name>/LOG]].
- write_note LOG.md: the timeline, newest on top. Link to SUMMARY and STATE.
- append_note to Daily log/<today>.md: "- Moved [[Projects/<project-name>/SUMMARY]] into viOS"

STEP 6 — REPORT
Tell me: project name, status you chose, pages created, pages updated, anything unclear.

STEP 7 — QUESTIONS
Add every unclear item to Inbox/Questions for Vishnu.md using append_note, under a heading "## <project name> — [[Projects/<project-name>/SUMMARY]]". Do not wait for my answers.

**Claude:** I'll read all chats and files in this project, check viOS for existing pages, then write the summary, linked pages, state, log and quest

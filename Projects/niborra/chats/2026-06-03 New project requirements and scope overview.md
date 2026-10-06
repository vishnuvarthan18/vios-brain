---
tags: chat
date: 2026-06-03
source: Claude personal account
uuid: 8697144d-b074-4211-9775-4561e9359f0c
---
# New project requirements and scope overview

## Summary
**Conversation Overview**

The person is working on a product development project for InkWave (previously called Niborra), a desktop robot that writes letters by hand using real pen on paper, sold to businesses like hotels and luxury retailers for automated personalized outreach. The conversation involved two uploaded documents: a software partner brief and an HTML presentation file created by Ahmad Taleb. The first document established the product concept, competitive landscape, target verticals (hotels, luxury retail, real estate, e-commerce), hardware tiers, and software requirements. The second document revealed a rebrand to InkWave, a shift in tech stack from Python/Qt to Electron, detailed software architecture across InkWave Studio and a web portal, 63 numbered features, and concrete pricing tiers ranging from a free tier through Starter (€49/mo), Pro (€149/mo), and Enterprise (custom).

The person asked Claude to explain both documents clearly and simply, explicitly noting they found the initial technical presentation confusing. They requested plain-language explanations suited to both a UI/UX designer and a product manager. The conversation then focused on competitor analysis covering Signascript (France/Switzerland) and UUNA TEK (China), mapping what works and fails for users with each competitor before identifying InkWave's market gap. The person then asked for a full end-to-end process flow covering all 13 steps from competitor analysis through launch metrics, with enough detail at each step to guide both roles practically. After receiving the written steps, the person requested the flow be placed visually into a Miro board rather than presented as text.

The person communicates informally with frequent typos and shorthand, prefers simple plain-language explanations over technical jargon, and becomes disoriented when presented with overly complex or visually dense outputs. They work iteratively, pausing topics to focus on one thing at a time, and prefer to hold certain work aside before returning to it. The competitor analysis section was explicitly set aside to focus on the process flow steps. The Miro board was the final requested output, using the layout tool to create a color-coded 13-step vertical flow organized across six phases: Discovery, Strategy, Architecture, Design, Build, and Launch.

**Tool Knowledge**

Miro board creation used `Miro:board_create` with name and description parameters, which succeeded and returned a board URL. The `Miro:diagram_create` tool with `diagram_type=flowchart` and DSL-based node definitions was attempted multiple times but encountered repeated failures, likely due to label length constraints or DSL syntax sensitivity around special characters and long node text strings. Shorter, simplified node labels in the third attempt also did not resolve the issue. The successful fallback was `Miro:layout_create` using the DSL layout format with explicit SHAPE, TEXT, and positioning parameters. This tool reliably accepted HTML content inside quoted strings using `&amp;` and `&lt;` entities, supported `type=round_rectangle` and `type=rectangle` shapes with fill, border, font, size, align, valign, and color parameters, and rendered multi-paragraph content using `<p>` tags within the content string. Absolute x/y coordinates with w/h dimensions were required for all items. Color-coded phases were achieved by varying fill and border_color hex values per phase grouping, with arrow indicators placed as TEXT items between SHAPE items using the `↓` character.

## Chat

**Vishnu:** we are starting a new project so please this requirment and tell the full contpet of the projec to get ready in te terms of both ui ux desinger nad the product manager

**Claude:** It looks like you forgot to paste or attach the requirements! Could you share them — as text, a document, or a file upload — and I'll break down the full project concept for both the UI/UX designer and the product manager.

**Claude:** _[tool: visualize:read_me]_

_[tool: visualize:show_widget]_

Here's the full project concept broken down for both roles. Use the three tabs — **Overview**, **UX / Designer brief**, and **Product Manager brief** — to navigate.

---

## The big idea in plain terms

Niborra is entering a market where the two existing players have divided the spoils: one is reliable but can't grow, the other can grow but has a terrible product. **Neither has ever built real software.** That gap is the entire bet.

The hardware is the hook, but the software is the business. Every design and product decision needs to be made with that in mind.

---

## For the UI/UX Designer — what to hold onto

The single most important brief line is this: **the person clicking your buttons is a hotel manager or a marketing exec, not an engineer.** They expect the same ease as opening Spotify for the first time.

Four things follow from that:

1. **45-second setup.** Unbox, pair, write. Any friction in the onboarding is a product defect.
2. **Plain-language AI creation.** The user types an intent — the AI handles card layout, copy, and handwriting style. No forms, no configurations.
3. **Guided digitisation wizard.** Capturing a person's handwriting style takes 20–30 minutes but must feel like a friendly step-by-step, not a calibration process.
4. **Zero silent failure.** The machine is always doing something, and the user must always know what — and what to do if something goes wrong. Plain English, always.

The platform is a **desktop Qt app (PySide6)** on Mac and Windows — not a web app, so design within native windowing constraints.

---

## For the Product Manager — the architectural decisions that cannot be deferred

Three things are explicitly flagged as **day-one assumptions**, not future roadmap items:

- **Multi-machine orchestration.** The platform must run one desktop unit and a farm of many machines on the same codebase from the start. This is an architectural constraint, not a feature.
- **SaaS + usage metering built in.** The entire commercial model — subscription tiers, usage gates, in-product unlocks — lives inside the software. It cannot be bolted on later.
- **Handwriting digitisation owned in-platform.** Competitors outsource this at $100–120 per person. Niborra brings it in-house as a guided wizard. This is a direct cost and retention advantage.

The competitive moat is simple: Signascript proved customers pay for quality, UUNA proved customers buy at scale. **No one has combined both — and no one has built software.** That is Niborra's entire opening.

**Vishnu:** no i cant underst anythign whis is this prdocut who are the comptete from where do i need to start

**Claude:** No worries at all — let me explain it simply, like I'm telling a friend.

---

## What is Niborra?

Imagine a **robot that sits on a desk and writes letters by hand** — like a real pen on real paper, with natural handwriting. Businesses use it to send personal-feeling notes to customers at scale.

A hotel might send every VIP guest a handwritten welcome card. A luxury brand might send a thank-you note after every big purchase. Instead of hiring someone to write thousands of letters, the robot does it automatically.

**Niborra is building that robot — and the software that controls it.**

---

## Why does the software matter so much?

The robot is just hardware. The real product is the **app** that:
- lets a hotel manager set up campaigns ("send a welcome card to every guest who books a suite")
- connects to tools like Shopify or HubSpot to trigger those campaigns automatically
- controls one robot or a whole room full of them
- handles subscriptions, billing, usage

Right now, **no competitor has built good software.** That's the entire business opportunity.

---

## Who are the two competitors?

| | Signascript | UUNA TEK |
|---|---|---|
| Where | France/Switzerland | China |
| Strength | Beautiful, reliable product | Cheap, can make lots of machines |
| Weakness | Only 6 people, can't grow, no software | Terrible product and software |
| Machines sold | ~4,000–5,000 over 20 years | ~1,000 last year |

Neither has an app worth talking about. Niborra's entire bet is: **build the software they never did.**

---

## Where do you start?

Think of the project in three layers, in this order:

**1. Understand the user first**
The person using this is a hotel manager or marketer — not a tech person. They expect to unbox it, open an app, pair the robot, and start writing in about 45 seconds. No manual. No technical setup. Your whole design starts from that person.

**2. The core flows to build**
- Pair the robot with the app (like connecting AirPods)
- Set up a campaign ("thank new customers")
- The AI writes the message and picks a style
- The robot executes the job
- The user sees status / progress

**3. The business model wired in from day one**
Subscriptions, usage limits, billing — these aren't added later. They're built into the app from the start.

---

Want me to go deeper on any one piece — the user flows, the competitor gaps, or what to actually build first?

**Vishnu:** what are the things mentioned anout the softwere here

**Claude:** Here's everything the document says about the software, in plain language:

---

## What the software IS

- It runs on **desktop (Mac + Windows)** — built with Python + Qt (PySide6)
- There's also a **mobile app** mentioned alongside the desktop app
- It talks to the robot over **Wi-Fi or Bluetooth**
- Jobs are sent to the robot as **file packages**
- It stores job history locally using **SQLite** (a simple database)

---

## What the software must DO

**Control the robot**
- Pair with the machine
- Send writing jobs to it
- Show live status — what is it doing right now
- Handle firmware updates safely (so a bad update doesn't brick the machine)
- Predict when maintenance is needed

**Run campaigns**
- Let users create campaigns in plain language ("thank new customers")
- AI suggests the message, card layout, and handwriting style
- Connect to Shopify, HubSpot, Salesforce, Klaviyo to trigger campaigns automatically

**Handle the business model**
- Subscription tiers and billing built inside the app
- Usage metering (track how many cards have been written)
- In-app plan upgrades and unlocks

**Digitise handwriting**
- A guided 20–30 minute wizard to capture a real person's handwriting style
- Competitors outsource this for $100–120 per person — Niborra owns it in-platform

**Scale to many machines**
- Must control one robot or a whole farm of many robots
- Same platform, same interface for both

---

## What the software must FEEL like

- Setup in **45 seconds** — no manual, no technical steps
- Non-technical users only — hotel managers, marketers, agency operators
- **Zero silent failure** — always tell the user what's happening and what to do next
- Premium and simple on the surface, complex technology hidden underneath

---

That's everything. Want me to turn any of these into actual screen flows or feature specs?

**Vishnu:** i will share the next docuem then we can disciss

**Claude:** Got it! Go ahead and share the next document whenever you're ready.

**Vishnu:** this is the new file

**Claude:** _[tool: view]_

_[tool: view]_

_[tool: view]_

Got it — I've read the whole file. Here's a clear breakdown of what this presentation is and how it compares to the original Niborra brief.

---

## What this file is

This is a **13-slide interactive React presentation** created by Ahmad Taleb, dated 08.05.2026. It's a development partner kickoff deck — the first detailed spec document translating the Niborra brief into actual product decisions.

---

## The big change: Niborra → InkWave

The product has been **rebranded**. Everything in this deck uses the new name:

| Niborra brief | InkWave presentation |
|---|---|
| Niborra (brand) | InkWave |
| Desktop app (Python + Qt) | **InkWave Studio** (Electron) |
| The machine | **InkWave Core** |
| Web portal | **app.inkwave.com** |

The tech stack also changed — the brief said **Python + Qt (PySide6)**, but the presentation specifies **Electron** (JavaScript-based desktop app).

---

## What's new and more detailed

**Software (63 numbered features):**
- A persistent global frame — top bar + left rail — always visible
- Home dashboard with library (recent campaigns, templates, uploads tabs)
- Campaign Designer: Setup → Data → Content → Preview → Compile & Assign
- Machine view: live status, controls, job queue
- Settings: maintenance, firmware OTA, language, help

**Pricing (now defined):**
- Free — €0, 1 machine, 5 handwriting styles, 1 user
- Starter — €49/mo, 2 machines, full library, up to 3 users, 2 pens/month
- Pro — €149/mo *(recommended)*, 4 machines, full+, unlimited users, 5 pens/month, REST API + webhooks
- Enterprise — Custom pricing, unlimited machines, custom integrations, account manager

**Hardware (now specified):**
- ESP32-S3 — host controller, Wi-Fi/LAN/USB, LVGL touchscreen, NFC pen reader
- STM32F407 — runs Marlin firmware, owns XY + Z writing axes
- STM32G0B1 — feeder controller (card pickup, separation, jam detection)
- Triple-OTA via ST AN3155 protocol — one firmware bundle, one toolchain

---

## What this means for both roles

**For the UX/UI designer**, this deck adds critical structure: the navigation is now locked in (left rail: Home, New Campaign, Machine, Settings, Help), the Home screen layout is defined (hero tiles + machine cards + library tabs), and the Campaign Designer has a clear 5-step flow.

**For the Product Manager**, pricing is now concrete and you have 63 features scoped across the two surfaces (Studio + web portal). The Electron decision matters — it changes what's possible with offline capability and native OS integration compared to Qt.

---

Want to go through any specific slide or section in more detail?

**Vishnu:** ok now let do one thing take let us keep the 2 documes aside and lets do one thing lets take the comperts anaysis first and lets see what work for users and what wont then we can finalise the the thiss and before that i need a good explain and steps need to follow from the beginning to the end in the terms of all ui ux and product managemt

**Claude:** Good call — starting with the competition first is exactly the right way to approach this. It tells you what users already accept, what frustrates them, and where the gap is.

Let me lay this out in two parts — the competitor analysis, then the full step-by-step process.

---

## Part 1 — Competitor analysis: what works and what doesn't

### Signascript (France/Switzerland)

**What works for users:**
- The machine is beautiful and feels premium — users trust it
- It's reliable. Hotels and luxury brands have used it for 20 years without problems
- The leasing model (€3,500–3,800/year) means low upfront cost — easy to say yes to
- Replacements every 6 years means they maintain quality

**What doesn't work for users:**
- There is no app. You get the machine, that's it
- Everything is manual — calibration, setup, adjustments all need a technician
- You can't connect it to your CRM or Shopify — it's completely closed
- You can only have one or two machines — it cannot scale at all
- If you want to grow, Signascript cannot help you

**The user feeling:** *"I trust this machine but I feel stuck. I can't automate anything."*

---

### UUNA TEK (China)

**What works for users:**
- It's affordable — the entry price is much lower than Signascript
- You can buy many machines — some customers already run small farms of them
- Ships quickly, available globally

**What doesn't work for users:**
- The software is genuinely bad — basic interface, no real features
- They have admitted themselves they won't build proper software for at least 5 years
- If you want your handwriting digitised, they send you to a third party who charges €100–120 per style — every single time
- The quality of the actual handwriting output is weaker
- No subscription, no usage tracking, no billing logic — it's all manual and messy

**The user feeling:** *"It's cheap and I can buy more of them, but using it feels like 2010. Nothing connects to anything."*

---

### The gap both miss — this is InkWave's opening

| What users need | Signascript | UUNA TEK | InkWave |
|---|---|---|---|
| Beautiful, reliable machine | ✓ | Weak | To build |
| Can scale to many machines | ✗ | Limited | Day one |
| Real software with features | ✗ | Very basic | Core product |
| CRM / Shopify connection | ✗ | ✗ | Built in |
| Handwriting digitisation in-app | ✗ | Outsourced | Own it |
| Subscription / billing built in | ✗ | ✗ | Core |
| Non-technical user can use it | ✗ | ✗ | Must be yes |

The one sentence that summarises the gap: **both competitors validated that customers will pay, but neither built software — and that is the entire opportunity.**

---

## Part 2 — The full process, beginning to end

### For the Product Manager

**Step 1 — Define what you're building and why**
Write a one-page product brief before touching anything else. Answer: what problem does this solve, who is the exact user, what does success look like in 6 months, and what are you explicitly not building in v1.

**Step 2 — Map the user journey**
Write out exactly what a hotel manager does from the moment they unbox the machine to the moment their first card is printed. Every single step. This becomes your feature list.

**Step 3 — Prioritise ruthlessly**
Split everything into three buckets: must have for launch, nice to have later, never do. Use the competitor gap as your guide — anything that neither Signascript nor UUNA TEK does well goes in the must-have bucket.

**Step 4 — Define the business model in the product**
Pricing tiers, usage metering, plan limits — these must be designed into the architecture from day one, not added later. The InkWave presentation already has this with Free / Starter / Pro / Enterprise.

**Step 5 — Write specs for each feature**
For every feature, write: what it does in one sentence, who uses it, what triggers it, what happens after, and what happens if it fails. No design starts without a spec.

**Step 6 — Set milestones**
Break the work into phases. Phase 1 is always: machine pairs, one campaign runs, one card prints. Everything else is phase 2 and beyond.

**Step 7 — Define success metrics**
Before launch, agree on what numbers tell you if it's working. For InkWave this would be: time from unbox to first card printed, campaign creation time, error rate, upgrade conversion from Free to Starter.

**Step 8 — Feedback loop**
After launch, talk to real users every two weeks. What confused them, what they skipped, what they wished existed.

---

### For the UI/UX Designer

**Step 1 — Understand the user deeply**
Before drawing anything, interview or research the actual users. For InkWave that's hotel managers, marketing managers, agency operators — not engineers. Write a one-page persona: who they are, what they're doing all day, what frustrates them, what tools they already use.

**Step 2 — Map the user journey in words**
Same journey the PM mapped, but now you add the emotional layer. Where does the user feel confident? Where do they feel confused or anxious? Those anxious moments are where you will do your best design work.

**Step 3 — Information architecture**
Decide the structure of the app before any visuals. What are the main sections? How do you get from one to another? For InkWave this is already decided: Home, New Campaign, Machine, Settings, Help. Draw this as a simple map.

**Step 4 — User flows**
For every key task, draw the path the user takes — screen by screen, click by click. Start with the three most important flows: first-time setup, creating a campaign, monitoring a machine.

**Step 5 — Wireframes (low fidelity first)**
Draw rough boxes and labels. No colours, no fonts, no icons. Just layout and content. Show these to someone who has never seen the product and ask them to talk out loud while they try to complete a task.

**Step 6 — Refine based on testing**
Fix what confused people. Simplify anything that needed explanation. The goal is that a hotel manager can complete every task without ever reading a manual.

**Step 7 — Visual design**
Now add colour, typography, spacing, icons. The InkWave deck already defines the design language — Inter font, the purple/teal/amber colour system, the clean flat aesthetic.

**Step 8 — Prototype and test again**
Build a clickable prototype and test it with real users again. One round of testing at this stage saves weeks of development time.

**Step 9 — Handoff to development**
Document every component, every state, every error message. The developer should never have to guess what something looks like or what it does.

**Step 10 — Review in development**
Check the built product against your designs before launch. Things always drift — catch it early.

---

## The order everything should happen in

```
Competitor analysis (done above)
      ↓
User research & persona
      ↓
Product brief & feature prioritisation (PM)
      ↓
User journey mapping (PM + UX together)
      ↓
Information architecture (UX)
      ↓
User flows (UX)
      ↓
Wireframes + usability testing (UX)
      ↓
Feature specs (PM)
      ↓
Visual design (UX)
      ↓
Prototype + test again (UX)
      ↓
Development specs & handoff (UX + PM)
      ↓
Build & milestone reviews (PM)
      ↓
Launch metrics & feedback loop (PM)
```

---

Where do you want to go next — the user research and persona, or jumping straight into the feature prioritisation based on the competitor gaps?

**Vishnu:** lets hold the compaters analys now lets focus on the steps 

the flow is super good now we need to jsut brive the flow now it is just headding now i need a bit detail what are the steps will done in the each step

**Claude:** Great — let's flesh out each step properly. I'll keep it practical and clear, not too long, but enough detail that both the PM and the designer know exactly what they're doing at each stage.

---

## The Full Process — Step by Step

---

### Step 1 — Competitor Analysis
*Already done — but here's what it produces*

**What you do:**
- Study every competitor's product hands-on if possible
- List what users love, what frustrates them, and what's completely missing
- Map the gaps into a table — this becomes your product's reason to exist

**What you produce:**
- A clear gap map showing exactly where InkWave wins
- A list of "never again" problems that users face with competitors — these become your design mandates

**Who does it:** PM leads, UX observes

---

### Step 2 — User Research & Persona

**What you do:**
- Interview 5–8 real people who match your target user — hotel managers, marketing managers, agency operators
- Ask them about their current workflow, not about your product. What tools do they use? Where do things break down? What do they wish was easier?
- Watch how they use existing tools if possible — don't just listen, observe
- Look for patterns across all interviews — what frustrations come up repeatedly

**What you produce:**
- One primary persona document. Not a fake marketing persona — a real one based on what you heard. It includes: their job, their daily routine, the tools they already use, their biggest frustrations, their comfort level with technology, and the one thing that would make their life easier
- A list of user quotes — real words real people said. These become your design compass throughout the project

**Who does it:** UX leads, PM attends all interviews

---

### Step 3 — Product Brief & Feature Prioritisation

**What you do:**
- PM writes a one-page brief: what problem we're solving, who we're solving it for, what success looks like in 6 months, and what we are explicitly not building in v1
- List every possible feature from the competitor gaps, the user research, and the brief documents
- Run every feature through three questions: Does it solve the core user problem? Can we build it for launch? Does it create revenue or enable the business model?
- Sort everything into three buckets — must have now, build later, never

**What you produce:**
- A prioritised feature list with clear reasoning for each decision
- A v1 scope document — one page that everyone agrees on before any design starts
- A "parking lot" list of good ideas that are explicitly saved for later, not forgotten

**Who does it:** PM leads, UX contributes

---

### Step 4 — User Journey Mapping

**What you do:**
- Take the most important user tasks and write them out step by step in plain words — no screens yet, just actions
- For InkWave the three critical journeys are: first-time setup (unbox to first card printed), creating and running a campaign, and monitoring a machine mid-job
- Add the emotional layer to each step — at this point is the user confident, confused, anxious, or relieved? Mark the moments of anxiety because those are your design opportunities
- Identify every decision point — moments where the user could go wrong or get lost

**What you produce:**
- A journey map for each key flow — typically a table with columns for: the step, what the user is doing, what they're thinking, how they're feeling, and the design opportunity
- A list of "moments that matter" — the 4–5 points in the journey where good design makes the biggest difference to the user's experience

**Who does it:** PM and UX together — this is the most important collaborative session in the whole process

---

### Step 5 — Information Architecture

**What you do:**
- Decide the skeleton of the app — what are the main sections, how do you navigate between them, what lives where
- Draw it as a simple tree or map — no visuals, just labels and connections
- For InkWave this means defining: what's in the left rail, what's on the Home screen, what's inside Campaign Designer, what's in Machine view, what's in Settings
- Test the structure by asking: can a new user find everything they need without hunting? Does the grouping make sense to someone who has never seen the app?

**What you produce:**
- An IA map — a simple diagram showing every section and how they connect
- A navigation decision document — why things are grouped the way they are, so developers and stakeholders don't second-guess it later

**Who does it:** UX leads, PM reviews and approves

---

### Step 6 — User Flows

**What you do:**
- Take each key journey from Step 4 and turn it into a detailed screen-by-screen flow
- Draw every path the user can take — the happy path (everything goes right) and the error paths (machine not connected, CSV has wrong columns, pen is low)
- Every decision point gets two branches — what happens if yes, what happens if no
- Keep it in boxes and arrows — no design yet, just logic

**What you produce:**
- A user flow diagram for each key task — typically 3–5 flows for v1
- An edge case list — all the things that can go wrong and what the app should do about each one
- This becomes the developer's logic map and the designer's content map

**Who does it:** UX leads, PM reviews for completeness

---

### Step 7 — Wireframes & First Usability Test

**What you do:**
- Draw every screen as rough boxes and labels — black and white, no colour, no icons, no fonts
- Focus entirely on: what information is on this screen, where is it positioned, what can the user do here
- Print them out or put them in a simple prototype tool and test with 3–5 real users
- Ask each person to complete a specific task while talking out loud — "create a campaign for new hotel guests"
- Watch where they hesitate, where they click the wrong thing, where they say "I'm not sure what this means"

**What you produce:**
- A set of wireframes covering every screen in v1
- A usability test report — what worked, what confused people, what needs to change
- A revised wireframe set after fixing the problems found in testing

**Who does it:** UX leads and runs the tests, PM observes

---

### Step 8 — Feature Specs

**What you do:**
- For every feature in v1, PM writes a spec. Each spec answers: what does this feature do in one sentence, who uses it, what triggers it, what happens step by step, what are the success and error states, and what does the user see when something goes wrong
- Pay special attention to the business logic — plan limits, usage metering, upgrade prompts, billing events
- Every spec gets reviewed by UX to make sure it's designable, and by a developer to make sure it's buildable

**What you produce:**
- A spec document for every v1 feature — typically stored in Notion, Confluence, or a shared doc
- An acceptance criteria list for each feature — the exact conditions that must be true for the feature to be considered done

**Who does it:** PM writes, UX and dev review

---

### Step 9 — Visual Design

**What you do:**
- Take the approved wireframes and apply the visual language — colour, typography, spacing, icons, motion
- For InkWave the visual language is already defined: Inter font, purple/teal/amber colours, flat clean aesthetic, no gradients or shadows
- Design every state of every component — default, hover, active, disabled, loading, error, empty
- Build a component library — a set of reusable buttons, inputs, cards, modals that stay consistent across the whole app
- Design the responsive behaviour — how does each screen adapt on a smaller window

**What you produce:**
- High-fidelity designs for every screen in every state
- A component library that developers can reference
- A design system document — the rules for colour, spacing, typography, and component usage

**Who does it:** UX/designer leads, PM reviews for alignment with specs

---

### Step 10 — Prototype & Second Usability Test

**What you do:**
- Build a clickable prototype using the high-fidelity designs — Figma, Framer, or similar
- Run another round of usability testing with 3–5 users — same format as Step 7 but now with the real visual design
- This round catches problems that wireframes hide — things that look clear in black and white but confuse people when colour and hierarchy are added
- Also test the emotional feel: does it feel premium? Does it feel simple? Does it feel trustworthy?

**What you produce:**
- A tested, approved prototype ready for developer handoff
- A final list of design changes based on test findings
- Sign-off from PM and stakeholders that this is what gets built

**Who does it:** UX leads and runs tests, PM and stakeholders attend

---

### Step 11 — Development Handoff

**What you do:**
- Export every design with precise measurements, colour codes, font sizes, and spacing values
- Write annotations on every screen explaining interactions, animations, and edge case behaviour
- Sit with developers and walk through every screen — don't just send files, explain intent
- Set up a shared reference channel where developers can ask questions during build

**What you produce:**
- A fully annotated design file with every measurement and interaction documented
- A handoff document covering: component names, spacing system, colour tokens, font scale, icon library
- A shared Q&A log so every design decision made during build is recorded

**Who does it:** UX hands off, PM supports, both stay available during build

---

### Step 12 — Build & Milestone Reviews

**What you do:**
- PM tracks progress against the v1 scope agreed in Step 3
- At each milestone, UX reviews the built feature against the design — look for drift, broken states, missing interactions
- PM runs a logic review — does the feature behave exactly as specced? Are the plan limits working? Does the billing logic trigger correctly?
- Any deviation gets flagged immediately — small drift becomes big drift if left unchecked

**What you produce:**
- A milestone review log — what was checked, what passed, what needs fixing
- A running list of bugs separated by severity: blocks launch, fixes before launch, can fix after launch

**Who does it:** PM tracks milestones, UX reviews design fidelity, both review behaviour

---

### Step 13 — Launch Metrics & Feedback Loop

**What you do:**
- Before launch, agree on the 4–5 numbers that tell you if the product is working
- For InkWave those would be: time from unbox to first card printed, campaign creation completion rate, error rate during machine pairing, upgrade conversion from Free to Starter, support ticket volume in first 30 days
- After launch, talk to real users every two weeks — structured interviews, not just support tickets
- Every user session reveals something. Feed it back into the next cycle of improvements

**What you produce:**
- A launch metrics dashboard
- A regular feedback report — what users are saying, what the numbers show, what gets prioritised next
- A v2 backlog built from real evidence, not assumptions

**Who does it:** PM owns metrics, UX owns user feedback sessions, both feed into the next cycle

---

## The full flow in one view

```
1. Competitor analysis → gap map
         ↓
2. User research → persona + real quotes
         ↓
3. Product brief → prioritised feature list + v1 scope
         ↓
4. Journey mapping → emotional map + moments that matter
         ↓
5. Information architecture → app structure map
         ↓
6. User flows → screen logic + edge cases
         ↓
7. Wireframes → test → fix → retest
         ↓
8. Feature specs → acceptance criteria
         ↓
9. Visual design → component library
         ↓
10. Prototype → test → sign off
         ↓
11. Handoff → annotated files + walkthrough
         ↓
12. Build → milestone reviews → bug log
         ↓
13. Launch → metrics → feedback → v2
```

---

Which step do you want to go into first and start actually doing the work on?

**Vishnu:** no i need in the flow it self can you put this in the miro morad

**Claude:** _[tool: tool_search]_

_[tool: Miro:diagram_get_dsl]_

_[tool: tool_search]_

Creating a new Miro board for this now — one moment.

_[tool: Miro:board_create]_

_[tool: Miro:diagram_get_dsl]_

Board created. Now getting the diagram spec and building the full flow.

_[tool: Miro:diagram_create]_

_[tool: Miro:diagram_create]_

_[tool: Miro:diagram_create]_

Let me try using the layout tool instead.

_[tool: Miro:layout_get_dsl]_

_[tool: Miro:layout_create]_

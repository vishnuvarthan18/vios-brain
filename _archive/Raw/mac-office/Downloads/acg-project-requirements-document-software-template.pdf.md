---
source: office Mac ~/Downloads/acg-project-requirements-document-software-template.pdf
---

NIBORRA PRD
DOCUMENT

Document Title:       NIBORRA PRD

Document ID:          AC-NOA-PRD

Document Type:        PROJECT REQUIREMENTS DOCUMENT

Document Version:     v0.1

Document Date:        14.06.2026

Requirement ID:       NOT AVAILABLE

Authors:              Shyam Sathish Kumar <shyam@aracreate.group>

PROJECT

Project:              NIBORRA

Version:              v1

Phase:                PLAN

Supplier              araCreate GmbH, Hubertusstr. 5, 12163 Berlin, Germany

Client:

APPROVAL

Supplier
                      Shyam Sathish Kumar
                      Project Manager

                      Approver                         Signature | Date

Client
                      [PROJECT-OWNER]
                      Project Owner

                      Approver                         Signature | Date




www.aracreate.group                                                [PROJECT-NAME] PRD   1 / 18
1 GENERAL

1.1 INDEX
1 GENERAL......................................................................................................................... 2
   1.1 INDEX.................................................................................................................................. 2
   1.2 REVISION HISTORY............................................................................................................3
   1.3 TERMS & ACRONYMS........................................................................................................4
   1.4 SUMMARY.......................................................................................................................... 5
   1.5 NOTES......................................................................................................................... 5
2 OVERVIEW........................................................................................................................... 6
       2.1 PROJECT...............................................................................................................6
       2.2 PRODUCT..............................................................................................................6
       2.3 MARKET................................................................................................................ 6
       2.4 CONSTRAINTS......................................................................................................7
3 ENGINEERING + MANUFACTURING SERVICES.............................................................. 8
   3.1 HARDWARE.................................................................................................................8
       3.1.1 DESIGN & USER EXPERIENCE........................................................................ 8
       3.1.2 ENGINEERING & DEVELOPMENT..................................................................10
       3.1.3 MANUFACTURING & SUPPLY CHAIN.............................................................11
       3.1.4 COMPLIANCE & LIFECYCLE...........................................................................13
4 REFERENCES................................................................................................................. 16
   4.1 LINKS................................................................................................................................ 16
   4.2 FIGURES............................................................................................................................16
   4.3 TABLE........................................................................................................................ 16




 www.aracreate.group                                                                                          [PROJECT-NAME] PRD            2 / 18
1.2 REVISION HISTORY
VERSION         MATURITY   DESCRIPTION OF CHANGES         DATE

v0.1            Draft      Initial Document               01.01.2025




www.aracreate.group                                 [PROJECT-NAME] PRD   3 / 18
1.3 TERMS & ACRONYMS
TERM                  ACRONYMS DESCRIPTION

PROJECT               PRD       A document that outlines the needs,
REQUIREMENTS                    expectations, and constraints of the project.
DOCUMENT




www.aracreate.group                                      [PROJECT-NAME] PRD   4 / 18
1.4 SUMMARY
This Project Requirements Document (PRD) defines project expectations, guides
development, and ensures objectives are met. It creates shared understanding among
stakeholders about goals, scope, and deliverables, serving as the project's blueprint.

Key components include:

    ●​   Project Overview: Purpose and goals
    ●​   Scope: Inclusions and exclusions
    ●​   Functional Requirements: Specific features and capabilities
    ●​   Non-Functional Requirements: Performance, security, and quality standards
    ●​   User Stories: End-user interactions with the system
    ●​   Constraints: Budget, timeline, and technical limitations
    ●​   Acceptance Criteria: Conditions for deliverable approval



1.5 NOTES
Please adhere to the following instructions:

    ●​ Review and answer the questions below that are relevant and important to your
       product.
    ●​ If a question is relevant, but you do not currently have an answer, kindly mark it for
       clarification.
    ●​ All questions will be revisited and reviewed during the project's exploration phase.
    ●​ Include any relevant images or external links to websites or cloud storage as needed.
    ●​ Maintain the existing formatting and styling of this document to ensure consistency
       across all deliverables.




 www.aracreate.group                                                   [PROJECT-NAME] PRD   5 / 18
2 OVERVIEW

2.1 PROJECT
   1.​ What does success look like for this product 6 months after launch? (e.g. number of
       customers, revenue target, number of machines deployed)
       Answer: [NEEDS INPUT]

   2.​ Are there any hard deadlines the team must be aware of? (e.g. a trade show, a pilot
       customer commitment, a funding milestone)
       Answer: [NEEDS INPUT]

   3.​ Is there a specific target launch date or quarter
       Answer: [NEEDS INPUT]


2.2 PRODUCT

   1.​ In one or two sentences, how would you describe Niborra to a potential customer
       who has never heard of it?
       Answer: [FROM BRIEF] Niborra is a handwriting automation system that turns
       pre-printed cards, postcards, and stationery into personalised handwritten messages
       — automatically, consistently, at scale. Built for businesses that want the warmth of a
       handwritten note without the cost of staff doing it manually.

       Please confirm or revise:

   2.​ What problem does Niborra solve that no other product solves today?
       [FROM BRIEF] Businesses that want to send personalised handwritten cards at
       scale currently have to choose between hiring staff to write them (slow, expensive,
       inconsistent) or using printed text (impersonal, lower response rates). Niborra
       eliminates that trade-off — every card is written with a real pen, with stroke variation,
       pressure changes, and natural irregularities designed to look indistinguishable from
       human handwriting.

       Please confirm or revise:

   3.​ What are the three components of the Niborra product?
       Answer:[FROM BRIEF]
         Component            Description

         Niborra Studio       Electron desktop app. Campaign managers design cards,



www.aracreate.group                                                      [PROJECT-NAME] PRD   6 / 18
                             import recipient data, send jobs to the machine, and monitor
                             output. Available on Mac, Windows, and Linux.

         Niborra Core        Physical desktop card-writing machine. Card feeder, writing
                             head, custom pen system, touchscreen. Connects via Wi-Fi,
                             LAN, or USB.

         app.niborra.com     Browser-based web portal for account management, billing,
                             fleet view, and support. Used by admins and managers only.


       Please confirm or revise:


2.3 MARKET
   1.​ Who is the primary daily user of Niborra Studio?
       Answer:[FROM BRIEF] The primary user is a Campaign Manager or operator. They
       design the card, import recipient data, send jobs to the machine, and monitor output.
       They use Niborra Studio exclusively and do not use the web portal.

       Please confirm or revise:

   2.​ Who buys and manages the Niborra subscription?
       Answer:[FROM BRIEF] The account admin or manager is a different person from the
       daily operator. They use app.niborra.com for billing, fleet management, and team
       management. The web portal is described as "used by managers and admins, not
       operators."

       Please confirm or revise:

   3.​ What industries or business types are you targeting first?
       Answer:[NEEDS INPUT] The brief describes the customer type but does not name
       specific industries.

   4.​ What is the typical size of the businesses you are targeting?
       Answer:[NEEDS INPUT]

   5.​ How technical are your target users? Are they comfortable with design tools like
       Canva or Adobe, or are they non-technical users who need everything to be very
       simple?
       Answer:[NEEDS INPUT]

   6.​ Who are the main competitors to Niborra and what must Niborra do better than each
       of them?
       Answer:[NEEDS INPUT]




www.aracreate.group                                                    [PROJECT-NAME] PRD   7 / 18
2.4 CONSTRAINTS
       1.​ Which operating systems must Niborra Studio support at launch?
           Answer:[FROM BRIEF] Mac, Windows, and Linux — all three confirmed.

       2.​ What is the minimum macOS version required?
           Answer:[NEEDS INPUT]

       3.​ What is the minimum Windows version required?
           Answer:[NEEDS INPUT]

       4.​ What languages must the app support at launch?

           Answer:[FROM BRIEF — partial] German and English confirmed.
           Please confirm or add further languages:

       5.​ Must Niborra Studio work without an internet connection?
           Answer:[FROM BRIEF] Yes — partial offline support is confirmed and is a core
           architectural principle:
           — Studio works fully offline for design and job compilation and dispatch.
           — The machine is autonomous — it completes a job even if Wi-Fi drops after upload.
           — The web portal requires an internet connection and never connects to machines
           directly.

           Please confirm:


3 SOFTWARE REQUIREMENT

3.1 NIBORRA STUDIO — DESKTOP APPLICATION
       1.​ What are the main screens of Niborra Studio?
           [FROM BRIEF] Five screens: Home, Campaign Designer, Machine Monitor, Settings.

           Please confirm the screen list or note any changes:
​
       2.​ Should there be different permission levels within one account?
           [FROM BRIEF] Yes — the brief confirms a role-aware UI with three roles: Admin,
           Operator, and Viewer.

           Please define what each role can and cannot do. Also confirm whether the Viewer
           role is required at v1 launch.
           Is the Viewer role required at v1? [ ] Yes [ ] No
           Admin       [NEEDS INPUT]
           Operator [NEEDS INPUT]


    www.aracreate.group                                                [PROJECT-NAME] PRD   8 / 18
       Viewer         [NEEDS INPUT — confirm if needed at v1]

   3.​ What card formats must be supported at launch?
       [FROM BRIEF] Confirmed from the UI spec: A4, A5, Postcard (standard), and
       Custom size (user-defined).

       Please confirm or add any missing formats:

   4.​ What file types can users upload as backgrounds, logos, and other assets?
       [FROM BRIEF — partial] The UI spec references SVG, PDF (first page as image),
       and PNG. JPG status is unconfirmed.

       FORMAT SUPPORTED
       PNG    Yes — confirmed
       SVG    Yes — confirmed
       PDF   Yes — first page used as image
       JPG   [NEEDS INPUT — please confirm]

       Please confirm:

   5.​ Does Niborra Studio need to support variable data printing at launch? This means
       each card in a batch can have different content — for example, each card has a
       different recipient name.
       [FROM BRIEF] Yes — required at v1. The brief confirms "imports recipient data" as a
       core operator task.​
       ​
       Please confirm:

   6.​ How do users provide the variable data?
       Answer: [NEEDS INPUT]
       [ ] Upload a CSV file
       [ ] Upload an Excel file
       [ ] Type or paste data manually
       [ ] Connect to an external data source (CRM, database)
       [ ] Other: _______________
       Answer:

   7.​ What text formatting options are required in the card designer?
       Answer:[NEEDS INPUT]
       [ ] Font family selection
       [ ] Font size
       [ ] Bold / italic / underline
       [ ] Text colour
       [ ] Text alignment (left / centre / right)
       [ ] Line spacing
       [ ] Other: _______________​


www.aracreate.group                                                      [PROJECT-NAME] PRD   9 / 18
           Answer:

       8.​ Should users be able to save card designs as reusable templates?
           [FROM BRIEF] Yes — the UI mockup includes a Templates tab with "My Templates"
           and "Team Templates" with import/export functionality. Templates are shareable
           across the team account.

           Please confirm:

       9.​ How many campaigns or designs would a typical user manage at one time?
           Answer:[NEEDS INPUT]

       10.​How does Niborra Studio connect to Niborra Core?
           [FROM BRIEF] Three connection methods are supported: Wi-Fi (local network), LAN
           (wired network), and USB cable.

           Please confirm:

       11.​What protocol does Niborra Core use to communicate with Studio?
           [FROM BRIEF — partial] The ESP32-S3 host controller talks to Studio over Wi-Fi,
           LAN, and USB. The OTA update protocol is ST AN3155. The application-level
           protocol for job dispatch is not yet defined in the brief.

           Please describe the current protocol or confirm it is yet to be defined:

       12.​What information does Niborra Core send back to the app in real time?
           [FROM BRIEF — partial]
​
            DATA FIELD                              STATUS

            Machine status (idle/busy/error)        Confirmed

            Ink / pen level                         Confirmed

            Job progress (card n of N)              [NEEDS INPUT — please confirm if
                                                    firmware supports this]​
                                                    Answer:

            Estimated time to completion            [NEEDS INPUT — please confirm]


       13.​Can a user control a job that is already running?
           [FROM BRIEF — partial] The machine is autonomous once a job is uploaded. The UI
           mockup shows queue reorder and remove controls, but firmware support for pause
           and cancel mid-job is unconfirmed.

           Please confirm what the firmware supports:


    www.aracreate.group                                                      [PROJECT-NAME] PRD   10 / 18
       [ ] Pause a running job
       [ ] Cancel / abort a running job
       [ ] Resume a paused job
       [ ] None — jobs run to completion once started

   14.​What happens when the machine is already busy and a new job is sent?
       [FROM BRIEF] New jobs are queued automatically and dispatched when the
       machine is free.

       Please confirm:

   15.​What should happen if the machine disconnects or goes offline mid-job?
       [FROM BRIEF] The machine continues running autonomously even if the Wi-Fi
       connection drops after the job has been uploaded. Studio should detect the
       disconnection and update the UI status accordingly, but the job itself is not
       interrupted.

       Please confirm this is the expected behaviour or describe any differences:

       Answer:

   16.​Can one user account manage multiple machines?
       [FROM BRIEF] Yes — machine limits are enforced per subscription tier:

       Free tier users are limited to 1 machine. Starter tier allows up to 2, and Pro tier up to
       4. Enterprise subscribers have no machine limit.

       Please confirm:

   17.​Should users be able to rename their machines from within the app?
       [FROM BRIEF] Yes — inline machine renaming is included in the UI mockup.

       Please confirm:

   18.​What machine settings are available to users?
       [FROM BRIEF — partial] The UI spec includes: inline rename, auto-update channel
       (stable/beta), self-test trigger, and remove machine (Admin only).

       Please confirm or list any additional machine settings required:


3.2 WEB PORTAL — APP.NIBORRA.COM
   1.​ Who can access the web portal?
       [FROM BRIEF] The portal is for managers and admins only. Operators use only the
       desktop app and do not access the web portal.



www.aracreate.group                                                       [PROJECT-NAME] PRD   11 / 18
       Please confirm:

   2.​ Must the web portal work on phones and tablets?
       Answer:[NEEDS INPUT]

       [ ] Yes — must be mobile-responsive
       [ ] Desktop-only is acceptable for v1

   3.​ What does the portal structure look like?
       [FROM BRIEF] Three sections: Account / Team, Billing, and Fleet / Support. 12
       features total. There is no separate home dashboard — the portal is a settings and
       admin area.

       Please confirm or revise:

   4.​ What must be manageable from the web portal at launch?
       [FROM BRIEF]
         Section                   Features

         Account / Team            Profile management, team member management, user
                                   roles

         Billing                   Subscription plan, upgrade/downgrade, billing history,
                                   invoice download, cancel subscription

         Fleet / Support           View all machines, machine status, firmware version,
                                   add/remove machines


       Please confirm and list any additional features required:


3.3 SUBSCRIPTION & PRICING
   1.​ What subscription tiers will Niborra offer at launch?

       [FROM BRIEF] Four tiers confirmed:
         Tier         Price        Notes

         Free         €0    /      Forever, no credit card required
                      month

         Starter      €49   / —
                      month

         Pro          €149 / Recommended
                      month




www.aracreate.group                                                   [PROJECT-NAME] PRD   12 / 18
         Enterprise    Custom       Annual contract


       Please confirm pricing or note any changes:

   2.​ What are the inclusions and limits for each tier?
       [FROM BRIEF]
        Feature             Free        Starter        Pro               Enterprise

        Machines            1           2              4                 Unlimited

        Characters     per 50           80             100               150
        minute

        Users               —           Up to 3        Unlimited         Unlimited

        Handwriting         5           Full library   Full library      Full library
        styles

        Humanization        Basic       Full           Full + advanced   Full + advanced

        Pen                 —           2 pens         5 pens            Custom volume
        consumables
        (monthly)

        Support             KB only     Email/tick     [NEEDS INPUT]     Named          account
                                        et                               manager

        SLA                 —           24 hours       [NEEDS INPUT]     Custom

        REST   API       / No           No             Yes               Yes
        Webhooks

        Analytics           No          No             Yes               Yes

        Custom              No          No             No                Yes
        integrations

        SSO                 No          No             No                Yes

        Audit          log No           No             No                Yes
        retention


       Please confirm all details and provide the Pro support level and SLA:

   3.​ Will annual billing be offered at a discount?

       [FROM BRIEF] Yes — annual billing saves 17% across all paid tiers.




www.aracreate.group                                                       [PROJECT-NAME] PRD   13 / 18
       Please confirm:

   4.​ Will there be a free trial?
       [FROM BRIEF] Yes — all paid tiers (Starter, Pro, Enterprise) include a one-month
       free trial. The Free tier has no trial — it is free indefinitely.

       Please confirm:

   5.​ Is the software subscription separate from the hardware cost?
       [FROM BRIEF] Yes — software subscription and machine leasing are billed
       separately. Pen consumables are included in Starter and above as a recurring
       monthly shipment.

       Please confirm:

   6.​ Which payment processor should be used?
       Answer:[NEEDS INPUT]

       [ ] Stripe (recommended)
       [ ] PayPal
       [ ] Other: _______________

   7.​ What currencies must be supported at launch?
       [FROM BRIEF — partial] Pricing is defined in Euros (€). Please confirm whether
       additional currencies are required.

       Answer:


3.4 BRANDING & DESIGN
   1.​ Does Niborra have an existing brand identity such as a logo, colour palette, and
       fonts?
       Answer: [NEEDS INPUT] The product was recently rebranded from InkWave to
       Niborra. Please confirm the status of the new brand identity.

       [ ] Yes — brand guidelines exist and will be shared with the team
       [ ] Partially — a logo exists but there are no full guidelines
       [ ] No — the design team will create the brand identity from scratch

   2.​ Are there any existing products, apps, or websites whose design you admire and
       would like Niborra to draw inspiration from?
       Answer:[NEEDS INPUT]

   3.​ How would you describe the desired look and feel of the Niborra product?
       Answer:[NEEDS INPUT]



www.aracreate.group                                                    [PROJECT-NAME] PRD   14 / 18
   4.​ Are there any colours, styles, or design directions that must be avoided?
       Answer:[NEEDS INPUT]


3.5 TECHNICAL CONSTRAINTS & INTEGRATIONS
   1.​ Are there any third-party tools or services that Niborra must integrate with at launch?
       [FROM BRIEF — partial] The Pro tier offers a public REST API and webhooks for
       outbound customer integrations. Enterprise includes custom integrations. Please
       confirm whether any specific inbound integrations (e.g. Salesforce, HubSpot, Zapier)
       are required at v1 launch.

       Answer:

   2.​ Should the REST API and webhooks be available at v1 launch as part of the Pro tier,
       or is this a post-launch addition?
       Answer:[NEEDS INPUT]
       [ ] Included in v1 at Pro tier launch
       [ ] Post-launch addition

   3.​ Does the analytics feature in the Pro tier refer to an in-app analytics dashboard,
       analytics data available via API only, or both?
       Answer:[NEEDS INPUT]

   4.​ Where should user data and account data be hosted?
       [FROM BRIEF] Multi-region hosting is confirmed: EU and US data residency
       required.

       Please confirm or specify further:

   5.​ What data privacy regulations must the product comply with?
       [FROM BRIEF — inferred] EU data residency implies GDPR compliance is required.

       [ ] GDPR (EU)
       [ ] CCPA (US)
       [ ] Other: _______________

   6.​ Is there an existing backend or API that Niborra must connect to, or is this a
       greenfield project built from scratch?
       [NEEDS INPUT]

   7.​ What is the current readiness of the Niborra Core firmware and is there API
       documentation available?
       [FROM BRIEF] The firmware architecture is confirmed:




www.aracreate.group                                                    [PROJECT-NAME] PRD   15 / 18
        Controller    Chip            Role

        Host          ESP32-S3        Runs LVGL touchscreen UI. Orchestrates jobs and
                                      OTA. Communicates with Studio over Wi-Fi / LAN /
                                      USB. Reads NFC pen tag.

        Writing       STM32F407       Runs Marlin firmware. Owns XY axes and Z writing
                                      head. 168 MHz, hardware FPU, 192 KB SRAM.

        Feeder        STM32G0B1       Custom FreeRTOS state machine. Handles card
                                      pickup, separation, transport, and jam detection.


       OTA Protocol: ST AN3155. One firmware bundle pushed from the desktop app.
       ESP32-S3 updates first, then STM32F407 and STM32G0B1 sequentially.

       Please confirm firmware readiness and share API or protocol documentation:


3.6 LAUNCH & SUPPORT
   1.​ How will users download Niborra Studio at launch?
       [NEEDS INPUT]

       [ ] Direct download from the Niborra website
       [ ] Mac App Store
       [ ] Microsoft Store
       [ ] Bundled with the physical machine in the box
       [ ] Other: _______________

   2.​ How should Niborra Studio be updated after launch?
       [NEEDS INPUT]

       [ ] Automatic background updates (silent, no user action needed)
       [ ] Prompted updates — user approves before installing
       [ ] Manual download from the website
       [ ] Other: _______________

   3.​ What level of customer support will be offered at launch for each tier?
       [FROM BRIEF — partial]
        Tier             Support

        Free             Knowledge base only

        Starter          Email and ticket support, 24-hour SLA

        Pro              [NEEDS INPUT]



www.aracreate.group                                                     [PROJECT-NAME] PRD   16 / 18
        Enterprise      Named account manager, custom SLA


   4.​ Will there be a beta or pilot phase before the public launch?
       Answer:[NEEDS INPUT]
       [ ] Yes — we have specific beta users in mind
       [ ] Yes — the development team should help organise this
       [ ] No — go straight to public launch

   5.​ Who will be responsible for ongoing maintenance and hosting after launch?
       Answer:[NEEDS INPUT]


3.7 OUT OF SCOPE
   1.​ The following features are proposed as out of scope for v1. Please confirm or
       challenge each one.
       Feature                     Out of Scope for V1    Actually Needed at V1

       Real-time multi-user        [ ] Confirmed          [ ] Needed — reason:
       design collaboration                               _______________

       Mobile app (iOS /           [ ] Confirmed          [ ] Needed — reason:
       Android)                                           _______________

       White-labelling or          [ ] Confirmed          [ ] Needed — reason:
       reseller programme                                 _______________

       Third-party API             [ ] Post-launch        [ ] V1 launch feature
       (outbound, Pro tier)

       Advanced in-app             [ ] Confirmed          [ ] Needed — reason:
       analytics dashboard                                _______________


   2.​ Are there any other features you are expecting in v1 that have not been mentioned
       anywhere in this document?
       Answer:[NEED INPUT]




www.aracreate.group                                                     [PROJECT-NAME] PRD   17 / 18
4 REFERENCES

4.1 LINKS

4.2 FIGURES

4.3 TABLE




www.aracreate.group   [PROJECT-NAME] PRD   18 / 18

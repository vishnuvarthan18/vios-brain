# EDE Portal on Liferay DXP — Developer Delivery Plan

Development work only. Liferay DXP 2026.Q1 LTS, self-hosted in the UAE. Go-live at the end of W8; hypercare W9–W12.

## Summary

| Group | Tasks | Templates | Dev hours | Share of hours |
| --- | --- | --- | --- | --- |
| 1. Initial setup | 7 | 0 | 48 | 9.6% |
| 2. Website page development | 20 | 63 | 135 | 27.0% |
| 3. Platform features and integrations | 31 | 8 | 188 | 37.6% |
| 4. Cross-page activities | 4 | 0 | 23 | 4.6% |
| 5. Developer testing, deployment and go-live | 13 | 0 | 106 | 21.2% |
| Total | 75 | 71 | 500 | 100.0% |

- Hypercare hours (W9–W12): **16**
- Hours to go-live: **484**
- Developer testing (manual: self-testing, EN/AR end-to-end check of key journeys, pre-UAT regression pass): **25 h**, inside section 5

### Assumptions

- Inputs: the sitemap and the EDE Figma file are the scope basis. EDE's feature and task list is a draft; rows marked "(draft list, to confirm)" stay in scope only once EDE confirms them.
- Development work only. EDE sign-offs are gates; testing rounds and content work by EDE or other teams are not included.
- Design: remaining Figma screens (EN and AR) are final before their page build starts; a screen changed after build is a change request.
- Integrations: by end of W2 EDE provides API docs and a test environment for the Drugs Registry, Open Data feeds, GIS, UAE PASS, happiness meter, e-consultation and newsletter platform. Until then development uses mock data.
- Approvals: EDE signs off each gate — BRD/HLD and hosting (W2), UAT test cases (W5), UAT (W8). The sign-off turnaround is an open question.
- Scope: 71 page templates (desktop + mobile each): 68 from the EDE Figma file plus 3 it implies (generic content page, HTML sitemap, error page). No signed-in user area: UAE PASS uses its own hosted sign-in page. AI assistant, dossier workspace and forum are MVPs. Track & Trace links out to the Tatmeen portal (no API).
- Delivery standards: acceptance against the Figma screen (EN/AR, desktop/mobile) and BRD criteria; changes after sign-off are change requests. Browser support, performance target, meeting rhythm, sign-off turnaround and hypercare length are open questions for EDE.
- Exclusions: licences and subscriptions paid by EDE (Liferay DXP, Azure OpenAI usage, Hotjar, live chat vendor, map tiles, newsletter platform); support after hypercare needs a separate warranty or SLA.
- Source: hours and weeks are araCreate estimates in the EDE Developer Delivery Plan; edit them on the Dev Plan sheet.

## Dev Plan

| ID | Group | Task / Page | Start week | End week | Templates | Dev hours | Dev work / How in Liferay |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T01 | 1. Initial setup | Local environment / module setup | 1 | 1 |  | 4 | Set up the Liferay Workspace, Git repository and CI pipeline, plus a local Docker development stack |
| T02 | 1. Initial setup | Environment access | 1 | 1 |  | 2 | Get developer access to EDE's hosting, DNS and integration sandboxes |
| T03 | 1. Initial setup | Development server provisioning and access | 1 | 1 |  | 8 | Build the shared dev environment (DXP, PostgreSQL, Elasticsearch). UAT and production are in section 5 |
| T04 | 1. Initial setup | BRD and HLD | 1 | 2 |  | 10 | Write the technical HLD: architecture, Objects data model, integration contracts. Dev input to the BRD |
| T05 | 1. Initial setup | Information security design review | 2 | 2 |  | 4 | Prepare security inputs (data flows, authentication, hardening plan) and fix review findings |
| T06 | 1. Initial setup | Design system import and adaptation in Liferay | 1 | 2 |  | 12 | Export Figma variables as JSON and convert them into the theme CSS client extension (CSS variables and Style Book tokens). Hand-build the base fragments from the Figma components, with RTL variants |
| T07 | 1. Initial setup | Header, footer, navigation and master pages | 2 | 2 |  | 8 | Build master page templates, the mega menu and "More" menu, breadcrumbs and the mobile accordion footer |
| T08 | 2. Website page development | Homepage (hero carousel, KPI figures, Alerts Hub, services tabs, impact counters, news tabs, Open Data search, partner carousel) | 2 | 3 | 1 | 10 | Content page, fragments, collections |
| T09 | 2. Website page development | Search results | 5 | 6 | 1 | 6 | Search Experiences (Blueprints) with Arabic analyzer |
| T10 | 2. Website page development | Services: listing, detail Overview, detail Requirements, detail FAQs | 2 | 3 | 4 | 11 | Structure, categories as filters, display page template, tabs fragment |
| T11 | 2. Website page development | Legislation & Regulations: Legislation & Circulars landing, legislation listing, circulars listing, subscribe to circulars, Alerts Hub listing, alert detail | 3 | 3 | 6 | 9 | Structures, collections with filters, subscription Object |
| T12 | 2. Website page development | About EDE (mission, leadership messages, awards, partners, interactive org chart) | 3 | 3 | 1 | 6 | Content page, fragments, org chart custom element |
| T13 | 2. Website page development | Contact us: landing (channels, Services for Seniors & People of Determination, office map), contact form, book appointment, contact leadership | 3 | 3 | 4 | 8 | Objects + form container fragments, MapLibre map |
| T14 | 2. Website page development | Media Centre: landing, news listing, news detail, events listing, events detail, media library (with lightbox) | 3 | 4 | 6 | 9 | Structures, display pages, Documents and Media |
| T15 | 2. Website page development | FAQs (category sidebar + accordion) | 4 | 4 | 1 | 3 | Structure and collection |
| T16 | 2. Website page development | Projects & initiatives: listing, detail | 4 | 4 | 2 | 4 | Structure, categories, display page |
| T17 | 2. Website page development | Careers (job listing) | 4 | 4 | 1 | 3 | Job posting Object + collection |
| T18 | 2. Website page development | Investment portal: landing (incl. success-stories logo carousel and Download Investment Guide buttons), enquiry form, live Drug Data Panel | 4 | 5 | 3 | 10 | Objects, ECharts dashboard custom element |
| T19 | 2. Website page development | Track & Trace (Tatmeen) information page | 5 | 5 | 1 | 2 | Content page with links to the Tatmeen portal |
| T20 | 2. Website page development | Digital participation: landing, blogs listing and detail, policies listing and detail, social media feed landing + X, Facebook, LinkedIn, Instagram, YouTube | 4 | 5 | 11 | 10 | Blogs, structures, feed custom elements |
| T21 | 2. Website page development | Open Data content: landing, guidelines, policies listing and detail, research, publications plan listing and detail | 4 | 5 | 7 | 10 | Structures, collections, Documents and Media |
| T22 | 2. Website page development | Open Data dashboards: real-time data (2), reports (2), budget (2), statistics (1) | 5 | 6 | 7 | 11 | Shared ECharts dashboard custom element, data via integration proxy |
| T23 | 2. Website page development | Open Data geospatial data | 5 | 5 | 1 | 5 | MapLibre map custom element |
| T24 | 2. Website page development | Open Data drugs registry: listing, detail | 4 | 5 | 2 | 10 | Custom element over the registry API |
| T25 | 2. Website page development | Open Data request data form | 5 | 5 | 1 | 2 | Object + form container |
| T26 | 2. Website page development | Static content pages: Sitemap (HTML), Copyright, Disclaimer, Privacy policy, Terms and conditions, Customer happiness charter, UAE Government charter, Awards, Media kit, Regulations, Legal references, Service user guide | 5 | 5 | 2 | 4 | One generic content-page template and an HTML sitemap template, both editable in the CMS |
| T27 | 2. Website page development | Error and system pages: 404, 500, maintenance | 5 | 5 | 1 | 2 | One shared error template, EN/AR |
| T28 | 3. Platform features and integrations | CMS workflow and configuration | 2 | 2 |  | 6 | Kaleo workflow and Publications, plus roles and permissions for EDE's content team (author, reviewer, approver, site admin) on pages, web content and Objects |
| T29 | 3. Platform features and integrations | SMTP email configuration | 2 | 2 |  | 1 | Instance mail settings |
| T30 | 3. Platform features and integrations | Cookie consent banner | 2 | 2 |  | 2 | Built-in cookie banner and consent panel |
| T31 | 3. Platform features and integrations | CAPTCHA and integration | 2 | 2 |  | 1 | Google reCAPTCHA, native instance setting |
| T32 | 3. Platform features and integrations | Site-wide alert banner | 3 | 3 |  | 2 | Fragment on the master page plus a CMS toggle |
| T33 | 3. Platform features and integrations | Social media links (Facebook, Instagram, LinkedIn, X, YouTube, TikTok) | 3 | 3 |  | 1 | Footer fragment |
| T34 | 3. Platform features and integrations | Footer – last updated date | 3 | 3 |  | 1 | Fragment reading the page modified date |
| T35 | 3. Platform features and integrations | Alerts Hub homepage widget | 3 | 3 |  | 2 | Collection display fragment |
| T36 | 3. Platform features and integrations | Accessibility settings panel (custom, as designed) | 3 | 3 |  | 8 | Global JS client extension: text size, contrast, spacing, saved preferences |
| T37 | 3. Platform features and integrations | "Need Help?" block | 3 | 3 |  | 2 | Fragment: live chat, user guide, call centre, enquiry |
| T38 | 3. Platform features and integrations | Service share, print, "On this page" and "Was this useful?" | 4 | 4 |  | 4 | Fragments on display page templates |
| T39 | 3. Platform features and integrations | Service save (bookmark icon) | 4 | 4 |  | 4 | Browser-only bookmark stored on the visitor's device; no login needed |
| T40 | 3. Platform features and integrations | Mobile app download block | 4 | 4 |  | 1 | Fragment |
| T41 | 3. Platform features and integrations | Feedback and complaints form | 4 | 4 |  | 3 | Object + form container, workflow routing |
| T42 | 3. Platform features and integrations | Report a side effect CTA and form | 4 | 4 |  | 5 | Object + form container, webhook to pharmacovigilance |
| T43 | 3. Platform features and integrations | Accessibility pages | 5 | 5 |  | 2 | Content pages |
| T44 | 3. Platform features and integrations | Web analytics and event tracking (Google Analytics 4, Hotjar): page views plus events for form submits, searches, downloads and ratings (draft list, to confirm) | 5 | 5 |  | 4 | Consent-aware global JS client extension |
| T45 | 3. Platform features and integrations | Newsletter subscription and integration (block with success and error states) | 5 | 5 |  | 4 | Object + external email platform API |
| T46 | 3. Platform features and integrations | Customer happiness rating ("Rate your experience" on every page) and integration | 5 | 5 |  | 4 | Fragment + custom element over the happiness meter API |
| T47 | 3. Platform features and integrations | Live chat | 5 | 5 |  | 1 | Click to Chat provider configuration |
| T48 | 3. Platform features and integrations | Surveys and polls (4 templates: listing and detail each, open/closed) | 4 | 5 | 4 | 7 | Objects + form container, results custom element |
| T49 | 3. Platform features and integrations | Forum and integration (2 templates: listing, topic detail) | 4 | 6 | 2 | 20 | Custom on Objects (Message Boards is deprecated). Signed-in posting screens must be supplied by EDE |
| T50 | 3. Platform features and integrations | E-consultation and integration (2 templates: listing, consultation detail) | 4 | 6 | 2 | 9 | Objects + Kaleo workflow + custom elements |
| T51 | 3. Platform features and integrations | UAE PASS login integration (draft list, to confirm) | 3 | 6 |  | 13 | OpenID Connect relying party, claim mapping |
| T52 | 3. Platform features and integrations | AI assistant (draft list, to confirm) | 3 | 6 |  | 30 | Microservice client extension, RAG over published content |
| T53 | 3. Platform features and integrations | Agentic dossier workspace (draft list, to confirm) | 3 | 6 |  | 35 | Custom element + Objects + agent microservice, UAE PASS gated. Signed-in screens are not in Figma and must be supplied by EDE |
| T54 | 3. Platform features and integrations | Back-office handling of submissions (feedback, side effects, appointments, data requests, investment enquiries, consultation responses) | 5 | 6 |  | 6 | Object views and filters for EDE staff, assignment and status workflow, CSV export |
| T55 | 3. Platform features and integrations | Email notifications in EN and AR (form confirmations, subscription double opt-in, appointment, circular and alert notices) | 5 | 5 |  | 4 | Object action and workflow notification templates over SMTP |
| T56 | 3. Platform features and integrations | SEO: friendly URLs, meta and Open Graph tags, hreflang EN/AR, sitemap.xml, robots.txt | 5 | 5 |  | 3 | Page SEO settings, display page mappings, sitemap configuration |
| T57 | 3. Platform features and integrations | Print stylesheet for detail pages | 5 | 5 |  | 1 | Print CSS in the theme client extension |
| T58 | 3. Platform features and integrations | Data protection: consent records and audit logging for personal and side-effect data | 5 | 5 |  | 2 | Object permissions, audit fields, restricted roles |
| T59 | 4. Cross-page activities | Arabic translation assessment | 1 | 2 |  | 6 | RTL audit of fragments; set up localized fields and the XLIFF translation flow |
| T60 | 4. Cross-page activities | Arabic labels for custom components | 4 | 6 |  | 3 | EN/AR language keys for all interface text in our fragments and custom elements (buttons, form errors, empty states, dashboard labels) |
| T61 | 4. Cross-page activities | Content migration (loading EDE-supplied content) | 4 | 7 |  | 10 | Scripted load of EDE-supplied content via headless APIs and batch client extensions |
| T62 | 4. Cross-page activities | Content migration validation | 6 | 7 |  | 4 | Fix import defects, re-run batches |
| T63 | 5. Developer testing, deployment and go-live | Developer self-testing | 3 | 6 |  | 12 | Each developer manually checks every page and feature they build against its Figma screen, in EN and AR, on desktop and mobile. Forms and integrations are checked against the mocked EDE APIs |
| T64 | 5. Developer testing, deployment and go-live | Manual end-to-end check of key journeys (EN and AR) | 6 | 7 |  | 9 | Walk through 10 key journeys in both languages: service detail, contact and feedback forms, report a side effect, UAE PASS login, Drugs Registry search, Open Data dashboard, survey/poll, forum post, AI assistant, search |
| T65 | 5. Developer testing, deployment and go-live | Pre-UAT manual regression pass | 6 | 6 |  | 4 | Manual regression pass on the UAT build, plus an axe accessibility browser check and an OWASP ZAP scan; fix failures before EDE's UAT starts |
| T66 | 5. Developer testing, deployment and go-live | UAT and production environments | 4 | 5 |  | 12 | Provision clusters, CI/CD to UAT and production, monitoring, backups |
| T67 | 5. Developer testing, deployment and go-live | Security / VAPT clearance | 6 | 7 |  | 10 | Hardening, fix VAPT findings, retest |
| T68 | 5. Developer testing, deployment and go-live | Accessibility compliance | 7 | 7 |  | 8 | Fix accessibility issues against EDE's required standard across fragments and custom elements |
| T69 | 5. Developer testing, deployment and go-live | Performance testing | 7 | 7 |  | 6 | Caching, CDN rules, query and search tuning |
| T70 | 5. Developer testing, deployment and go-live | UAT defect resolution | 6 | 7 |  | 18 | Fix and redeploy UAT defects |
| T71 | 5. Developer testing, deployment and go-live | Production readiness user training (draft list, to confirm) | 7 | 7 |  | 3 | Hands-on CMS sessions for EDE's content team and admins: editing pages, workflow approvals, forms back-office, publishing in EN and AR |
| T72 | 5. Developer testing, deployment and go-live | Go-live | 8 | 8 |  | 4 | Production deployment, DNS cutover, smoke tests |
| T73 | 5. Developer testing, deployment and go-live | Go-live rehearsal and rollback plan | 7 | 7 |  | 1 | Dry-run deployment to UAT and documented rollback steps |
| T74 | 5. Developer testing, deployment and go-live | Handover documentation | 8 | 8 |  | 3 | Architecture document, deployment runbook, CMS admin guide for EDE IT |
| T75 | 5. Developer testing, deployment and go-live | Hypercare | 9 | 12 |  | 16 | Post-launch support for 4 weeks after go-live, budgeted at about 4 h a week. Covers fixing defects found on the live EDE portal, watching error logs and uptime alerts, and small fixes to existing templates. New features and change requests are excluded and handled separately |
|  |  | Total |  |  | 71 | 500 |  |

## Timeline (W1–W12)

■ = task active that week. Go-live: end of W8.

| ID | Task / Page | Dev hours | W1 | W2 | W3 | W4 | W5 | W6 | W7 | W8 | W9 | W10 | W11 | W12 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T01 | Local environment / module setup | 4 | ■ |  |  |  |  |  |  |  |  |  |  |  |
| T02 | Environment access | 2 | ■ |  |  |  |  |  |  |  |  |  |  |  |
| T03 | Development server provisioning and access | 8 | ■ |  |  |  |  |  |  |  |  |  |  |  |
| T04 | BRD and HLD | 10 | ■ | ■ |  |  |  |  |  |  |  |  |  |  |
| T05 | Information security design review | 4 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T06 | Design system import and adaptation in Liferay | 12 | ■ | ■ |  |  |  |  |  |  |  |  |  |  |
| T07 | Header, footer, navigation and master pages | 8 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T08 | Homepage (hero carousel, KPI figures, Alerts Hub, services tabs, impact counters, news tabs, Open Data search, partner carousel) | 10 |  | ■ | ■ |  |  |  |  |  |  |  |  |  |
| T09 | Search results | 6 |  |  |  |  | ■ | ■ |  |  |  |  |  |  |
| T10 | Services: listing, detail Overview, detail Requirements, detail FAQs | 11 |  | ■ | ■ |  |  |  |  |  |  |  |  |  |
| T11 | Legislation & Regulations: Legislation & Circulars landing, legislation listing, circulars listing, subscribe to circulars, Alerts Hub listing, alert detail | 9 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T12 | About EDE (mission, leadership messages, awards, partners, interactive org chart) | 6 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T13 | Contact us: landing (channels, Services for Seniors & People of Determination, office map), contact form, book appointment, contact leadership | 8 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T14 | Media Centre: landing, news listing, news detail, events listing, events detail, media library (with lightbox) | 9 |  |  | ■ | ■ |  |  |  |  |  |  |  |  |
| T15 | FAQs (category sidebar + accordion) | 3 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T16 | Projects & initiatives: listing, detail | 4 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T17 | Careers (job listing) | 3 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T18 | Investment portal: landing (incl. success-stories logo carousel and Download Investment Guide buttons), enquiry form, live Drug Data Panel | 10 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T19 | Track & Trace (Tatmeen) information page | 2 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T20 | Digital participation: landing, blogs listing and detail, policies listing and detail, social media feed landing + X, Facebook, LinkedIn, Instagram, YouTube | 10 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T21 | Open Data content: landing, guidelines, policies listing and detail, research, publications plan listing and detail | 10 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T22 | Open Data dashboards: real-time data (2), reports (2), budget (2), statistics (1) | 11 |  |  |  |  | ■ | ■ |  |  |  |  |  |  |
| T23 | Open Data geospatial data | 5 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T24 | Open Data drugs registry: listing, detail | 10 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T25 | Open Data request data form | 2 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T26 | Static content pages: Sitemap (HTML), Copyright, Disclaimer, Privacy policy, Terms and conditions, Customer happiness charter, UAE Government charter, Awards, Media kit, Regulations, Legal references, Service user guide | 4 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T27 | Error and system pages: 404, 500, maintenance | 2 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T28 | CMS workflow and configuration | 6 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T29 | SMTP email configuration | 1 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T30 | Cookie consent banner | 2 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T31 | CAPTCHA and integration | 1 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T32 | Site-wide alert banner | 2 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T33 | Social media links (Facebook, Instagram, LinkedIn, X, YouTube, TikTok) | 1 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T34 | Footer – last updated date | 1 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T35 | Alerts Hub homepage widget | 2 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T36 | Accessibility settings panel (custom, as designed) | 8 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T37 | "Need Help?" block | 2 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T38 | Service share, print, "On this page" and "Was this useful?" | 4 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T39 | Service save (bookmark icon) | 4 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T40 | Mobile app download block | 1 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T41 | Feedback and complaints form | 3 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T42 | Report a side effect CTA and form | 5 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T43 | Accessibility pages | 2 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T44 | Web analytics and event tracking (Google Analytics 4, Hotjar): page views plus events for form submits, searches, downloads and ratings (draft list, to confirm) | 4 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T45 | Newsletter subscription and integration (block with success and error states) | 4 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T46 | Customer happiness rating ("Rate your experience" on every page) and integration | 4 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T47 | Live chat | 1 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T48 | Surveys and polls (4 templates: listing and detail each, open/closed) | 7 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T49 | Forum and integration (2 templates: listing, topic detail) | 20 |  |  |  | ■ | ■ | ■ |  |  |  |  |  |  |
| T50 | E-consultation and integration (2 templates: listing, consultation detail) | 9 |  |  |  | ■ | ■ | ■ |  |  |  |  |  |  |
| T51 | UAE PASS login integration (draft list, to confirm) | 13 |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |  |
| T52 | AI assistant (draft list, to confirm) | 30 |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |  |
| T53 | Agentic dossier workspace (draft list, to confirm) | 35 |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |  |
| T54 | Back-office handling of submissions (feedback, side effects, appointments, data requests, investment enquiries, consultation responses) | 6 |  |  |  |  | ■ | ■ |  |  |  |  |  |  |
| T55 | Email notifications in EN and AR (form confirmations, subscription double opt-in, appointment, circular and alert notices) | 4 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T56 | SEO: friendly URLs, meta and Open Graph tags, hreflang EN/AR, sitemap.xml, robots.txt | 3 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T57 | Print stylesheet for detail pages | 1 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T58 | Data protection: consent records and audit logging for personal and side-effect data | 2 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T59 | Arabic translation assessment | 6 | ■ | ■ |  |  |  |  |  |  |  |  |  |  |
| T60 | Arabic labels for custom components | 3 |  |  |  | ■ | ■ | ■ |  |  |  |  |  |  |
| T61 | Content migration (loading EDE-supplied content) | 10 |  |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |
| T62 | Content migration validation | 4 |  |  |  |  |  | ■ | ■ |  |  |  |  |  |
| T63 | Developer self-testing | 12 |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |  |
| T64 | Manual end-to-end check of key journeys (EN and AR) | 9 |  |  |  |  |  | ■ | ■ |  |  |  |  |  |
| T65 | Pre-UAT manual regression pass | 4 |  |  |  |  |  | ■ |  |  |  |  |  |  |
| T66 | UAT and production environments | 12 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T67 | Security / VAPT clearance | 10 |  |  |  |  |  | ■ | ■ |  |  |  |  |  |
| T68 | Accessibility compliance | 8 |  |  |  |  |  |  | ■ |  |  |  |  |  |
| T69 | Performance testing | 6 |  |  |  |  |  |  | ■ |  |  |  |  |  |
| T70 | UAT defect resolution | 18 |  |  |  |  |  | ■ | ■ |  |  |  |  |  |
| T71 | Production readiness user training (draft list, to confirm) | 3 |  |  |  |  |  |  | ■ |  |  |  |  |  |
| T72 | Go-live | 4 |  |  |  |  |  |  |  | ■ |  |  |  |  |
| T73 | Go-live rehearsal and rollback plan | 1 |  |  |  |  |  |  | ■ |  |  |  |  |  |
| T74 | Handover documentation | 3 |  |  |  |  |  |  |  | ■ |  |  |  |  |
| T75 | Hypercare | 16 |  |  |  |  |  |  |  |  | ■ | ■ | ■ | ■ |

## Tech Stack

One choice per layer. Versions follow the Liferay DXP 2026.Q1 LTS compatibility matrix (learn.liferay.com).

| Layer | Locked choice | Not used |
| --- | --- | --- |
| CMS platform | Liferay DXP 2026.Q1 LTS (latest patch at kickoff), self-hosted | SaaS, PaaS, Portal CE |
| Runtime | JDK 21 and Tomcat 10.1, from the official liferay/dxp Docker image | JDK 17 (deprecated) |
| Database | PostgreSQL 16.x, primary + streaming replica | MySQL, MariaDB |
| Search engine | Elasticsearch 8.19.x, 3-node cluster, Arabic analyzer + ICU plugin | OpenSearch (not yet in the 2026.Q1 matrix), semantic search |
| File store | Advanced File System Store on a shared persistent volume | DB store |
| Clustering | 2+ DXP nodes with Liferay cluster link | Single node in production |
| Container platform | Kubernetes 1.30+ (CNCF-conformant, OpenShift accepted), Helm 3 charts | VMs, manual deploys |
| Extension model | Client extensions + fragments only (client extensions) | OSGi modules, Ext plugins, Themes Toolkit |
| Theme | Theme CSS client extension (SCSS on Liferay Clay), Figma tokens as CSS custom properties, Style Book | Classic theme overrides |
| Page building | Content pages, master page templates, custom fragment library, display page templates, Collections | Widget pages, Asset Publisher |
| Custom UI | React 18.3 + TypeScript 5 + Vite, packaged as customElement client extensions; React shared once via a JS import map client extension | React 19, per-element React bundles, Angular, Vue |
| Frontend tooling | Node.js 22 LTS + pnpm | npm bundler (deprecated) |
| Styling in custom elements | Theme CSS variables + CSS Modules, logical properties for RTL | Tailwind, a second CSS framework |
| Charts | Apache ECharts 5 (RTL, Arabic labels) | Chart.js, D3 |
| Maps | MapLibre GL JS 4 | Leaflet, Google Maps |
| Data model and forms | Liferay Objects + form container fragments, object actions | Liferay Forms (maintenance mode) |
| Forum | Custom on Objects (Topic, Reply, Report) + React custom element | Message Boards (deprecated) |
| Surveys and polls | Objects + form container + results custom element | Forms poll settings |
| E-consultation, appointments | Objects + Kaleo workflow + custom elements | External scheduler |
| Workflow and publishing | Kaleo workflow + Publications | Staging |
| Localization | EN + AR locales; human translation via Liferay XLIFF export/import | Machine translation (data residency) |
| Public login | UAE PASS via Liferay OpenID Connect (authorization code + PKCE) | SAML for UAE PASS |
| EDE staff login | Pending EDE's answer: single sign-on through EDE's identity provider (OpenID Connect), or Liferay accounts | — |
| CAPTCHA | Google reCAPTCHA, native instance setting | hCaptcha, Turnstile |
| Cookie consent | Liferay built-in banner and consent panel | Third-party CMP |
| Analytics | GA4 + Hotjar loaded by a consent-aware global JS client extension | Site analytics settings field (ignores consent) |
| Live chat | Liferay Click to Chat with a supported provider | Custom chat build |
| Microservices | Spring Boot 3.x on JDK 21, deployed as microservice client extensions; OAuth 2 via Liferay (headless server + user agent apps) | Node/Python services |
| Integration cache | Redis 7 | Caching in the browser |
| AI service | Spring AI on Spring Boot 3; RAG with pgvector (separate PostgreSQL 16 DB); tool calling over Liferay headless APIs for the dossier agent | Liferay AI Hub (SaaS beta), Liferay MCP server (feature-flagged), LangChain/Python |
| LLM and embeddings | Azure OpenAI Service, UAE North region; confirm model availability in W1 | Any model hosted outside the UAE |
| Content migration | Java 21 CLI over Liferay headless REST + batch client extensions | Manual copy-paste |
| Source control and CI/CD | GitLab + GitLab CI: build, scan and deploy to dev, UAT and prod; mirrored to EDE at handover | Manual builds |
| Testing | Manual developer testing against Figma (EN/AR, desktop/mobile); axe browser extension for accessibility checks; OWASP ZAP scan before UAT; k6 for the performance round | Automated test suites in CI (not in scope) |
| Observability and backup | Prometheus + Grafana + Loki; pgBackRest for PostgreSQL, Elasticsearch snapshots, volume snapshots | — |

## Feature Fit

Native = out of the box · Configure = settings only · Build = custom fragment or client extension · Integrate = depends on an external system

| Feature | Fit | How |
| --- | --- | --- |
| Pages, navigation, master pages, header/footer | Native | Content pages, master page templates, navigation menus |
| Design system from Figma | Build | Theme CSS client extension + custom fragment library |
| News, events, circulars, legislation, FAQs, projects (careers use a job posting Object) | Native | Web content structures, display page templates, collections |
| Services with tabs (overview, requirements, FAQs) | Native + Build | Structure + display page + tabs fragment |
| Media library | Native | Documents and Media |
| CMS workflow and configuration | Native | Kaleo workflow + Publications (recommended over Staging) |
| Site-wide alert banner, Alerts Hub widget, last updated date | Build | Fragments on the master page, CMS-controlled |
| Hero KPI figures, customisable shortcuts | Build | Fragments. Hero KPIs are the homepage figure strip (registered products, licensed facilities and similar); customisable shortcuts are site-wide quick links that EDE manages in the CMS (no login; from EDE's draft list, not in Figma, to confirm) |
| Social media links, service share | Build | Fragments |
| Service save | Build | Browser-only bookmark on the visitor's device, no login |
| Blog | Native | Blogs |
| Forum | Build | Objects (topic, reply), custom element UI, workflow moderation — Message Boards is deprecated |
| Surveys and polls (active/closed) | Build | Objects + form container; results via custom element. Forms has poll settings (docs) but is in maintenance mode |
| E-consultation | Build | Objects + Kaleo workflow + custom elements |
| Feedback/complaints, report a side effect, contact, request data, investment form | Build (low-code) | Objects + form container fragments, object actions for email/webhook |
| Book appointment | Build | Object + slot calendar custom element |
| Newsletter subscription | Integrate | Object for opt-in + external email platform (no native newsletter) |
| Circulars and alerts subscription | Build | Object + object action email on publish |
| SMTP email | Configure | Instance mail settings |
| CAPTCHA | Configure | Google reCAPTCHA, native instance setting |
| Cookie consent banner | Native | Built-in banner + consent panel, four categories |
| Web analytics (GA, Hotjar) | Configure | Consent-aware global JS client extension loading GA4 and Hotjar |
| Live chat | Configure | Click to Chat (Zendesk, LivePerson, Intercom, HubSpot and others) |
| Accessibility settings panel, pages, compliance | Build | Custom accessibility settings panel (global JS client extension), as designed; WCAG built into the fragments |
| Customer happiness rating | Integrate | Custom element over the UAE government happiness service API |
| UAE PASS login | Configure + Integrate | Liferay OpenID Connect as relying party + UAE PASS onboarding |
| Social media feeds | Integrate | Custom element; platform API tokens held in the integration proxy |
| Drug registry, live dashboards, Open Data (Tatmeen is an information page only) | Integrate + Build | Microservice client extension (API proxy, cache) + custom elements |
| Search | Native + Configure | Search Experiences / Blueprints; Arabic analyzer on the search engine |
| Arabic / RTL | Native + Build | Arabic locale native; RTL styles in our theme and fragments |
| AI assistant | Build | Microservice + chat custom element (RAG over published content) |
| Agentic dossier workspace | Build | Custom app on Objects + microservice, UAE PASS gated |
| Content migration | Native tooling | Headless APIs + batch client extensions |

## Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| The 8-week plan has no slack | Any late sign-off or late input moves go-live directly | Agree a sign-off turnaround with EDE (see Open questions); weekly demo with the EDE PO |
| Figma design not fully complete | Pages wait on screens, or get rework late | Agree a screen-by-screen delivery schedule in W1, matched to the page build order; a screen changed after it is built becomes a change request |
| Third-party APIs or sandboxes not ready by W2 (Drugs Registry, Open Data feeds, Investment Drug Data Panel, UAE PASS, happiness meter, e-consultation) | API-dependent rows slip | Build against mocks; an item whose API is not ready by W6 moves to a post-launch release |
| UAE PASS onboarding (staging, then production approval) usually takes longer than 8 weeks | Login and the dossier workspace are blocked at go-live | Submit the onboarding request on day 1 (EDE IT owns it) |
| AI assistant and agentic dossier scope | Effort beyond the MVP will not fit | MVP fixed in the BRD (W2); the rest goes to a post-launch release |
| Arabic content volume and translation quality | Migration and UAT slip | Arabic setup in W1–W2; EDE's content team owns translation |
| VAPT findings in W6–W7 leave little time to fix | Go-live moves | Security review at the HLD stage; internal OWASP ZAP scan in W6 before UAT |
| Custom forum build on Objects (Message Boards is deprecated) | Effort overrun | Forum MVP (topics, replies, moderation, reporting) fixed in the BRD |
| UAE hosting environment not ready by W2 | Environments and the AI model choice blocked | Sign-off gate in W2 |

## Open Questions

Fill in the Answer and Answered on columns as EDE replies; set Status to Answered.

| Area | Question | Status | Answer from EDE | Answered on |
| --- | --- | --- | --- | --- |
| Hosting and platform | Which UAE data centre or government cloud hosts the portal, and on which platform (Kubernetes cluster, virtual machines or a managed service)? | Open |  |  |
| Hosting and platform | Is the Liferay DXP subscription in place for 2+ production nodes? | Open |  |  |
| Hosting and platform | How will EDE staff sign in to the CMS: single sign-on through EDE's identity provider, or Liferay accounts? | Open |  |  |
| Hosting and platform | What uptime, backup and disaster-recovery (RPO/RTO) and expected traffic does EDE require? | Open |  |  |
| EDE data and integrations | Drugs Registry: when are the API spec and sandbox ready, and which fields are public? | Open |  |  |
| EDE data and integrations | Open Data feeds (Real-time data, Reports, Budget, Statistics) and the Investment Drug Data Panel: source system, format and refresh rate? Is any of it restricted? | Open |  |  |
| EDE data and integrations | Geospatial data: which GIS service, and GeoJSON or WMS? | Open |  |  |
| EDE data and integrations | "Rate your experience": which government happiness service and API does EDE report to? | Open |  |  |
| EDE data and integrations | E-consultation is planned on Liferay. Must responses also go to a federal e-participation platform? | Open |  |  |
| EDE data and integrations | UAE PASS: who owns onboarding at EDE, and when are staging credentials issued? | Open |  |  |
| EDE data and integrations | Report a side effect: which pharmacovigilance system receives the submissions? | Open |  |  |
| EDE data and integrations | Newsletter: which email platform? | Open |  |  |
| EDE data and integrations | Social media feeds: who provides the X, Facebook, LinkedIn, Instagram and YouTube API tokens? | Open |  |  |
| EDE data and integrations | Request data form: who fulfils requests, and through which workflow? | Open |  |  |
| AI | AI assistant: does EDE approve Azure OpenAI (UAE North), and which content may it answer from? | Open |  |  |
| AI | Dossier workspace: which dossier type is the MVP, who uses it, and which documents does it hold? | Open |  |  |
| AI | EDE's feature and task list is a draft. Which of these items, which have no Figma screens, stay in scope: UAE PASS sign-in, AI assistant, agentic dossier workspace, web analytics (GA4, Hotjar), customisable shortcuts, user training? | Open |  |  |
| Content and design | Which EN and AR content will EDE supply, in which format, and by when? | Open |  |  |
| Content and design | Who translates the Arabic content, and who signs it off? | Open |  |  |
| Content and design | Which Figma screens are still incomplete (job detail, signed-in forum and consultation states), and when will each arrive? | Open |  |  |
| Content and design | Placeholder copy to replace: the footer copyright ("Ministry of Human Resources & Emiratisation"), the TDRA questions on the FAQ page, "Ireland" in the Investment success stories, and "Singapore Government" in the contact form consent text. | Open |  |  |
| Content and design | Careers: does "View job" open a detail page on the portal or an external HR system? | Open |  |  |
| Content and design | Hero KPIs and Platform Impact counters: entered manually or fed live? | Open |  |  |
| Delivery | Who runs SIT and regression QA before UAT, and who runs VAPT? | Open |  |  |
| Delivery | Which accessibility standard and level must the portal meet (for example WCAG 2.1 AA)? | Open |  |  |
| Delivery | What warranty and SLA apply after hypercare? | Open |  |  |
| Delivery | Who writes the UAT test cases? | Open |  |  |
| Delivery | Which browsers and devices must the portal support? Proposed: latest Chrome, Edge, Firefox and Safari on desktop, Safari on iOS and Chrome on Android. | Open |  |  |
| Delivery | Does EDE have a performance target? Proposed: Lighthouse mobile score of 80 or higher on the homepage, service detail and Open Data dashboard templates. | Open |  |  |
| Delivery | Which meeting rhythm does EDE want? Proposed: a weekly sprint demo from W2. | Open |  |  |
| Delivery | What sign-off turnaround can EDE commit to for each gate? Proposed: 2 working days. | Open |  |  |
| Delivery | Is 4 weeks of hypercare (W9–W12, 16 h) acceptable? | Open |  |  |
| Compliance and links | Must the portal follow the UAE government website standards (TDRA digital government guidelines), and which version? | Open |  |  |
| Compliance and links | Which UAE PDPL requirements apply to the forms and the side-effect data (retention period, consent wording)? | Open |  |  |
| Compliance and links | Download Mobile App block: does EDE have published App Store and Google Play apps, and what are the store links? | Open |  |  |
| Compliance and links | Do the Report a side effect form or the dossier workspace need file uploads? No upload field is designed. If uploads are needed, virus scanning is required and is estimated as a change request. | Open |  |  |

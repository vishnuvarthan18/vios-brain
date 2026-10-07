# EDE Portal on Liferay DXP — Developer Delivery Plan

Development work only. Liferay DXP 2026.Q1 LTS, self-hosted in the UAE. Go-live at the end of W8; hypercare W9–W12.

## Summary

| Group | Tasks | Templates | Dev hours | Share of hours |
| --- | --- | --- | --- | --- |
| 1. Initial setup | 7 | 0 | 50 | 10.0% |
| 2. Website page development | 18 | 60 | 143 | 28.6% |
| 3. Platform features and integrations | 27 | 8 | 203 | 40.6% |
| 4. Cross-page activities | 3 | 0 | 24 | 4.8% |
| 5. Deployment and go-live | 7 | 0 | 80 | 16.0% |
| Total | 62 | 68 | 500 | 100.0% |

- Hypercare hours (W9–W12): **16**
- Hours to go-live: **484**

### Assumptions

- Development work only. EDE sign-offs are gates; testing rounds and content work by EDE or other teams are not included.
- Design: remaining Figma screens (EN and AR) are final before their page build starts; a screen changed after build is a change request.
- Integrations: by end of W2 EDE provides API docs and a test environment for the Drugs Registry, Open Data feeds, GIS, UAE PASS, happiness meter, e-consultation and newsletter platform. Until then development uses mock data.
- Approvals: EDE signs off each gate within 2 working days — BRD/HLD and hosting (W2), UAT test cases (W5), UAT (W8).
- Scope: 68 unique page templates from the EDE Figma file (desktop + mobile each). AI assistant, dossier workspace and forum are MVPs. Track & Trace links out to the Tatmeen portal (no API).
- Source: hours and weeks are araCreate estimates in the EDE Developer Delivery Plan; edit them on the Dev Plan sheet.

## Dev Plan

| ID | Group | Task / Page | Start week | End week | Templates | Dev hours | Dev work / How in Liferay |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T01 | 1. Initial setup | Local environment / module setup | 1 | 1 |  | 4 | Set up the Liferay Workspace, Git repository and CI pipeline, plus a local Docker development stack |
| T02 | 1. Initial setup | Environment access | 1 | 1 |  | 2 | Get developer access to EDE's hosting, DNS and integration sandboxes |
| T03 | 1. Initial setup | Development server provisioning and access | 1 | 1 |  | 8 | Build the shared dev environment (DXP, PostgreSQL, Elasticsearch). UAT and production are in section 5 |
| T04 | 1. Initial setup | BRD and HLD | 1 | 2 |  | 10 | Write the technical HLD: architecture, Objects data model, integration contracts. Dev input to the BRD |
| T05 | 1. Initial setup | Information security design review | 2 | 2 |  | 4 | Prepare security inputs (data flows, authentication, hardening plan) and fix review findings |
| T06 | 1. Initial setup | Design system import and adaptation in Liferay | 1 | 2 |  | 14 | Export Figma variables as JSON and convert them into the theme CSS client extension (CSS variables and Style Book tokens). Hand-build the base fragments from the Figma components, with RTL variants |
| T07 | 1. Initial setup | Header, footer, navigation and master pages | 2 | 2 |  | 8 | Build master page templates, the mega menu and "More" menu, breadcrumbs and the mobile accordion footer |
| T08 | 2. Website page development | Homepage (hero carousel, KPI figures, Alerts Hub, services tabs, impact counters, news tabs, Open Data search, partner carousel) | 2 | 3 | 1 | 12 | Content page, fragments, collections |
| T09 | 2. Website page development | Search results | 5 | 6 | 1 | 6 | Search Experiences (Blueprints) with Arabic analyzer |
| T10 | 2. Website page development | Services: listing, detail Overview, detail Requirements, detail FAQs | 2 | 3 | 4 | 12 | Structure, categories as filters, display page template, tabs fragment |
| T11 | 2. Website page development | Legislation & Regulations: Legislation & Circulars landing, legislation listing, circulars listing, subscribe to circulars, Alerts Hub listing, alert detail | 3 | 3 | 6 | 10 | Structures, collections with filters, subscription Object |
| T12 | 2. Website page development | About EDE (mission, leadership messages, awards, partners, interactive org chart) | 3 | 3 | 1 | 6 | Content page, fragments, org chart custom element |
| T13 | 2. Website page development | Contact us: landing (channels, office map), contact form, book appointment, contact leadership | 3 | 3 | 4 | 8 | Objects + form container fragments, MapLibre map |
| T14 | 2. Website page development | Media Centre: landing, news listing, news detail, events listing, events detail, media library (with lightbox) | 3 | 4 | 6 | 10 | Structures, display pages, Documents and Media |
| T15 | 2. Website page development | FAQs (category sidebar + accordion) | 4 | 4 | 1 | 3 | Structure and collection |
| T16 | 2. Website page development | Projects & initiatives: listing, detail | 4 | 4 | 2 | 4 | Structure, categories, display page |
| T17 | 2. Website page development | Careers (job listing) | 4 | 4 | 1 | 3 | Job posting Object + collection |
| T18 | 2. Website page development | Investment portal: landing, enquiry form, live Drug Data Panel | 4 | 5 | 3 | 12 | Objects, ECharts dashboard custom element |
| T19 | 2. Website page development | Track & Trace (Tatmeen) information page | 5 | 5 | 1 | 2 | Content page with links to the Tatmeen portal |
| T20 | 2. Website page development | Digital participation: landing, blogs listing and detail, policies listing and detail, social media feed landing + X, Facebook, LinkedIn, Instagram, YouTube | 4 | 5 | 11 | 12 | Blogs, structures, feed custom elements |
| T21 | 2. Website page development | Open Data content: landing, guidelines, policies listing and detail, research, publications plan listing and detail | 4 | 5 | 7 | 10 | Structures, collections, Documents and Media |
| T22 | 2. Website page development | Open Data dashboards: real-time data (2), reports (2), budget (2), statistics (1) | 5 | 6 | 7 | 14 | Shared ECharts dashboard custom element, data via integration proxy |
| T23 | 2. Website page development | Open Data geospatial data | 5 | 5 | 1 | 5 | MapLibre map custom element |
| T24 | 2. Website page development | Open Data drugs registry: listing, detail | 4 | 5 | 2 | 12 | Custom element over the registry API |
| T25 | 2. Website page development | Open Data request data form | 5 | 5 | 1 | 2 | Object + form container |
| T26 | 3. Platform features and integrations | CMS workflow and configuration | 2 | 2 |  | 5 | Kaleo workflow, roles, Publications |
| T27 | 3. Platform features and integrations | SMTP email configuration | 2 | 2 |  | 1 | Instance mail settings |
| T28 | 3. Platform features and integrations | Cookie consent banner | 2 | 2 |  | 2 | Built-in cookie banner and consent panel |
| T29 | 3. Platform features and integrations | CAPTCHA and integration | 2 | 2 |  | 1 | Google reCAPTCHA, native instance setting |
| T30 | 3. Platform features and integrations | Site-wide alert banner | 3 | 3 |  | 2 | Fragment on the master page plus a CMS toggle |
| T31 | 3. Platform features and integrations | Social media links | 3 | 3 |  | 1 | Footer fragment |
| T32 | 3. Platform features and integrations | Footer – last updated date | 3 | 3 |  | 1 | Fragment reading the page modified date |
| T33 | 3. Platform features and integrations | Sticky quick-action sidebar | 3 | 3 |  | 3 | Global JS/CSS client extension |
| T34 | 3. Platform features and integrations | Alerts Hub homepage widget | 3 | 3 |  | 2 | Collection display fragment |
| T35 | 3. Platform features and integrations | Accessibility settings panel (custom, as designed) | 3 | 3 |  | 8 | Global JS client extension: text size, contrast, spacing, saved preferences |
| T36 | 3. Platform features and integrations | "Need Help?" block | 3 | 3 |  | 2 | Fragment: live chat, user guide, call centre, enquiry |
| T37 | 3. Platform features and integrations | Service share, print, "On this page" and "Was this useful?" | 4 | 4 |  | 4 | Fragments on display page templates |
| T38 | 3. Platform features and integrations | Service save (bookmark icon) | 4 | 4 |  | 4 | Object plus custom element |
| T39 | 3. Platform features and integrations | Mobile app download block | 4 | 4 |  | 1 | Fragment |
| T40 | 3. Platform features and integrations | Feedback and complaints form | 4 | 4 |  | 3 | Object + form container, workflow routing |
| T41 | 3. Platform features and integrations | Report a side effect CTA and form | 4 | 4 |  | 5 | Object + form container, webhook to pharmacovigilance |
| T42 | 3. Platform features and integrations | Accessibility pages | 5 | 5 |  | 2 | Content pages |
| T43 | 3. Platform features and integrations | Web analytics (Google Analytics, Hotjar) | 5 | 5 |  | 3 | Consent-aware global JS client extension |
| T44 | 3. Platform features and integrations | Newsletter subscription and integration (block with success and error states) | 5 | 5 |  | 4 | Object + external email platform API |
| T45 | 3. Platform features and integrations | Customer happiness rating ("Rate your experience" on every page) and integration | 5 | 5 |  | 4 | Fragment + custom element over the happiness meter API |
| T46 | 3. Platform features and integrations | Live chat | 5 | 5 |  | 1 | Click to Chat provider configuration |
| T47 | 3. Platform features and integrations | Surveys and polls (4 templates: listing and detail each, open/closed) | 4 | 5 | 4 | 8 | Objects + form container, results custom element |
| T48 | 3. Platform features and integrations | Forum and integration (2 templates: listing, topic detail) | 4 | 6 | 2 | 24 | Custom on Objects (Message Boards is deprecated) |
| T49 | 3. Platform features and integrations | E-consultation and integration (2 templates: listing, consultation detail) | 4 | 6 | 2 | 12 | Objects + Kaleo workflow + custom elements |
| T50 | 3. Platform features and integrations | UAE PASS login integration | 3 | 6 |  | 14 | OpenID Connect relying party, claim mapping |
| T51 | 3. Platform features and integrations | AI assistant | 3 | 6 |  | 38 | Microservice client extension, RAG over published content |
| T52 | 3. Platform features and integrations | Agentic dossier workspace | 3 | 6 |  | 48 | Custom element + Objects + agent microservice, UAE PASS gated |
| T53 | 4. Cross-page activities | Arabic translation assessment | 1 | 2 |  | 6 | RTL audit of fragments; set up localized fields and the XLIFF translation flow |
| T54 | 4. Cross-page activities | Content migration (loading EDE-supplied content) | 4 | 7 |  | 14 | Scripted load of EDE-supplied content via headless APIs and batch client extensions |
| T55 | 4. Cross-page activities | Content migration validation | 6 | 7 |  | 4 | Fix import defects, re-run batches |
| T56 | 5. Deployment and go-live | UAT and production environments | 4 | 5 |  | 12 | Provision clusters, CI/CD to UAT and production, monitoring, backups |
| T57 | 5. Deployment and go-live | Security / VAPT clearance | 6 | 7 |  | 10 | Hardening, fix VAPT findings, retest |
| T58 | 5. Deployment and go-live | Accessibility compliance | 7 | 7 |  | 8 | Fix WCAG 2.1 AA issues across fragments and custom elements |
| T59 | 5. Deployment and go-live | Performance testing | 7 | 7 |  | 6 | Caching, CDN rules, query and search tuning |
| T60 | 5. Deployment and go-live | UAT defect resolution | 6 | 7 |  | 24 | Fix and redeploy UAT defects |
| T61 | 5. Deployment and go-live | Go-live | 8 | 8 |  | 4 | Production deployment, DNS cutover, smoke tests |
| T62 | 5. Deployment and go-live | Hypercare | 9 | 12 |  | 16 | Post-launch support for 4 weeks after go-live, budgeted at about 4 h a week. Covers fixing defects found on the live EDE portal, watching error logs and uptime alerts, and small fixes to existing templates. New features and change requests are excluded and handled separately |
|  |  | Total |  |  | 68 | 500 |  |

## Timeline (W1–W12)

■ = task active that week. Go-live: end of W8.

| ID | Task / Page | Dev hours | W1 | W2 | W3 | W4 | W5 | W6 | W7 | W8 | W9 | W10 | W11 | W12 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T01 | Local environment / module setup | 4 | ■ |  |  |  |  |  |  |  |  |  |  |  |
| T02 | Environment access | 2 | ■ |  |  |  |  |  |  |  |  |  |  |  |
| T03 | Development server provisioning and access | 8 | ■ |  |  |  |  |  |  |  |  |  |  |  |
| T04 | BRD and HLD | 10 | ■ | ■ |  |  |  |  |  |  |  |  |  |  |
| T05 | Information security design review | 4 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T06 | Design system import and adaptation in Liferay | 14 | ■ | ■ |  |  |  |  |  |  |  |  |  |  |
| T07 | Header, footer, navigation and master pages | 8 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T08 | Homepage (hero carousel, KPI figures, Alerts Hub, services tabs, impact counters, news tabs, Open Data search, partner carousel) | 12 |  | ■ | ■ |  |  |  |  |  |  |  |  |  |
| T09 | Search results | 6 |  |  |  |  | ■ | ■ |  |  |  |  |  |  |
| T10 | Services: listing, detail Overview, detail Requirements, detail FAQs | 12 |  | ■ | ■ |  |  |  |  |  |  |  |  |  |
| T11 | Legislation & Regulations: Legislation & Circulars landing, legislation listing, circulars listing, subscribe to circulars, Alerts Hub listing, alert detail | 10 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T12 | About EDE (mission, leadership messages, awards, partners, interactive org chart) | 6 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T13 | Contact us: landing (channels, office map), contact form, book appointment, contact leadership | 8 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T14 | Media Centre: landing, news listing, news detail, events listing, events detail, media library (with lightbox) | 10 |  |  | ■ | ■ |  |  |  |  |  |  |  |  |
| T15 | FAQs (category sidebar + accordion) | 3 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T16 | Projects & initiatives: listing, detail | 4 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T17 | Careers (job listing) | 3 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T18 | Investment portal: landing, enquiry form, live Drug Data Panel | 12 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T19 | Track & Trace (Tatmeen) information page | 2 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T20 | Digital participation: landing, blogs listing and detail, policies listing and detail, social media feed landing + X, Facebook, LinkedIn, Instagram, YouTube | 12 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T21 | Open Data content: landing, guidelines, policies listing and detail, research, publications plan listing and detail | 10 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T22 | Open Data dashboards: real-time data (2), reports (2), budget (2), statistics (1) | 14 |  |  |  |  | ■ | ■ |  |  |  |  |  |  |
| T23 | Open Data geospatial data | 5 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T24 | Open Data drugs registry: listing, detail | 12 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T25 | Open Data request data form | 2 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T26 | CMS workflow and configuration | 5 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T27 | SMTP email configuration | 1 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T28 | Cookie consent banner | 2 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T29 | CAPTCHA and integration | 1 |  | ■ |  |  |  |  |  |  |  |  |  |  |
| T30 | Site-wide alert banner | 2 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T31 | Social media links | 1 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T32 | Footer – last updated date | 1 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T33 | Sticky quick-action sidebar | 3 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T34 | Alerts Hub homepage widget | 2 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T35 | Accessibility settings panel (custom, as designed) | 8 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T36 | "Need Help?" block | 2 |  |  | ■ |  |  |  |  |  |  |  |  |  |
| T37 | Service share, print, "On this page" and "Was this useful?" | 4 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T38 | Service save (bookmark icon) | 4 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T39 | Mobile app download block | 1 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T40 | Feedback and complaints form | 3 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T41 | Report a side effect CTA and form | 5 |  |  |  | ■ |  |  |  |  |  |  |  |  |
| T42 | Accessibility pages | 2 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T43 | Web analytics (Google Analytics, Hotjar) | 3 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T44 | Newsletter subscription and integration (block with success and error states) | 4 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T45 | Customer happiness rating ("Rate your experience" on every page) and integration | 4 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T46 | Live chat | 1 |  |  |  |  | ■ |  |  |  |  |  |  |  |
| T47 | Surveys and polls (4 templates: listing and detail each, open/closed) | 8 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T48 | Forum and integration (2 templates: listing, topic detail) | 24 |  |  |  | ■ | ■ | ■ |  |  |  |  |  |  |
| T49 | E-consultation and integration (2 templates: listing, consultation detail) | 12 |  |  |  | ■ | ■ | ■ |  |  |  |  |  |  |
| T50 | UAE PASS login integration | 14 |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |  |
| T51 | AI assistant | 38 |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |  |
| T52 | Agentic dossier workspace | 48 |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |  |
| T53 | Arabic translation assessment | 6 | ■ | ■ |  |  |  |  |  |  |  |  |  |  |
| T54 | Content migration (loading EDE-supplied content) | 14 |  |  |  | ■ | ■ | ■ | ■ |  |  |  |  |  |
| T55 | Content migration validation | 4 |  |  |  |  |  | ■ | ■ |  |  |  |  |  |
| T56 | UAT and production environments | 12 |  |  |  | ■ | ■ |  |  |  |  |  |  |  |
| T57 | Security / VAPT clearance | 10 |  |  |  |  |  | ■ | ■ |  |  |  |  |  |
| T58 | Accessibility compliance | 8 |  |  |  |  |  |  | ■ |  |  |  |  |  |
| T59 | Performance testing | 6 |  |  |  |  |  |  | ■ |  |  |  |  |  |
| T60 | UAT defect resolution | 24 |  |  |  |  |  | ■ | ■ |  |  |  |  |  |
| T61 | Go-live | 4 |  |  |  |  |  |  |  | ■ |  |  |  |  |
| T62 | Hypercare | 16 |  |  |  |  |  |  |  |  | ■ | ■ | ■ | ■ |

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
| EDE staff login | OpenID Connect to EDE's identity provider | Local passwords for staff |
| CAPTCHA | Google reCAPTCHA, native instance setting | hCaptcha, Turnstile |
| Cookie consent | Liferay built-in banner and consent panel | Third-party CMP |
| Analytics | GA4 + Hotjar loaded by a consent-aware global JS client extension | Site analytics settings field (ignores consent) |
| Live chat | Liferay Click to Chat with a supported provider | Custom chat build |
| Microservices | Spring Boot 3.x on JDK 21, deployed as microservice client extensions; OAuth 2 via Liferay (headless server + user agent apps) | Node/Python services |
| Integration cache | Redis 7 | Caching in the browser |
| AI service | Spring AI on Spring Boot 3; RAG with pgvector (separate PostgreSQL 16 DB); tool calling over Liferay headless APIs for the dossier agent | Liferay AI Hub (SaaS beta), Liferay MCP server (feature-flagged), LangChain/Python |
| LLM and embeddings | Azure OpenAI Service, UAE North region; confirm model availability in W1 | Any model hosted outside the UAE |
| Content migration | Java 21 CLI over Liferay headless REST + batch client extensions | Manual copy-paste |
| Source control and CI/CD | GitLab + GitLab CI: build, test, scan, deploy to dev, UAT and prod; mirrored to EDE at handover | Manual builds |
| Testing | Vitest + React Testing Library, Playwright E2E, axe-core for accessibility, k6 for load, OWASP ZAP baseline in CI | Manual-only QA |
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
| Hero KPI figures, customisable shortcuts, sticky quick-action sidebar | Build | Fragments; per-user shortcuts need login + Objects |
| Social media links, service share | Build | Fragments |
| Service save | Build | Object + custom element (per user) |
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
| The 8-week plan has no slack | Any late sign-off or late input moves go-live directly | A 2-day sign-off SLA in the contract; daily stand-up with the EDE PO |
| Figma design not fully complete | Pages wait on screens, or get rework late | Agree a screen-by-screen delivery schedule in W1, matched to the page build order; a screen changed after it is built becomes a change request |
| Third-party APIs or sandboxes not ready by W2 (Drugs Registry, Open Data feeds, Investment Drug Data Panel, UAE PASS, happiness meter, e-consultation) | API-dependent rows slip | Build against mocks; feature flags so go-live can happen with an item off |
| UAE PASS onboarding (staging, then production approval) usually takes longer than 8 weeks | Login and the dossier workspace are blocked at go-live | Submit the onboarding request on day 1 (EDE IT owns it); fall back to launching without login |
| AI assistant and agentic dossier scope | Effort beyond the MVP will not fit | MVP fixed in the BRD (W2); the rest goes to a post-launch release |
| Arabic content volume and translation quality | Migration and UAT slip | Arabic setup in W1–W2; EDE's content team owns translation |
| VAPT findings in W6–W7 leave little time to fix | Go-live moves | Security review at the HLD stage; internal scans from W4 |
| Custom forum build on Objects (Message Boards is deprecated) | Effort overrun | Forum MVP (topics, replies, moderation, reporting) fixed in the BRD |
| UAE hosting environment not ready by W2 | Environments and the AI model choice blocked | Sign-off gate in W2 |

## Open Questions

Fill in the Answer and Answered on columns as EDE replies; set Status to Answered.

| Area | Question | Status | Answer from EDE | Answered on |
| --- | --- | --- | --- | --- |
| Hosting and platform | Which UAE data centre or government cloud hosts the portal, and who provides the Kubernetes cluster? | Open |  |  |
| Hosting and platform | Is the Liferay DXP subscription in place for 2+ production nodes? | Open |  |  |
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
| Content and design | Which EN and AR content will EDE supply, in which format, and by when? | Open |  |  |
| Content and design | Who translates the Arabic content, and who signs it off? | Open |  |  |
| Content and design | Which Figma screens are still incomplete (job detail, signed-in forum and consultation states), and when will each arrive? | Open |  |  |
| Content and design | Placeholder copy to replace: the footer copyright ("Ministry of Human Resources & Emiratisation"), the TDRA questions on the FAQ page, "Ireland" in the Investment success stories, and "Singapore Government" in the contact form consent text. | Open |  |  |
| Content and design | Careers: does "View job" open a detail page on the portal or an external HR system? | Open |  |  |
| Content and design | Hero KPIs and Platform Impact counters: entered manually or fed live? | Open |  |  |
| Delivery | Who runs SIT and regression QA before UAT, and who runs VAPT? | Open |  |  |
| Delivery | The plan assumes WCAG 2.1 AA. Is that EDE's accessibility target? | Open |  |  |
| Delivery | What warranty and SLA apply after hypercare? | Open |  |  |

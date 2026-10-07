# Vercel Dashboard — UI/UX Flow & Design Research

Source: live capture of vishnu-aracreategr's projects workspace (Hobby plan), dark theme, 34 screenshots in `vercel-screens-final/`. Cross-checked against the original capture plan; every top-level sidebar section, project/deployment detail, settings (project + team), command menu, and account overlay is represented. Confirmed coverage below.

## Coverage verification

All 34 files are present and match the index. Full breadth achieved: Projects, Deployments, Logs, Analytics, Speed Insights, Observability (+Query drill-in), Firewall, CDN, Environment Variables, Domains, Connect, Integrations, Storage, Flags, Agent, AI Gateway, Sandboxes, Workflows, Images, Usage, project detail, project Deployments tab, deployment detail, project Settings (General, Deployment Protection), team Settings (General, Billing, Members), command menu, account overlay menu, and two zoomed micro-detail shots. Not captured (noted as a scope tradeoff last round): every nested sub-tab inside Observability, AI Gateway, and Team Settings beyond the ones listed — those are single-level drill-downs of sections already documented at their landing screen, so the core flow and pattern language is fully covered even though line-item leaves aren't each individually shot.

---

## 1. Information architecture

Vercel's dashboard uses a two-tier navigation model: a **workspace switcher** at the very top (team logo + name + plan badge), and a **persistent left sidebar** beneath it that changes contents depending on scope.

- At the **team/all-projects scope**, the sidebar lists workspace-wide tools: Projects, Deployments, Logs, Analytics, Speed Insights, Observability, Firewall, CDN, Environment Variables, Domains, Connect, Integrations, Storage, Flags, Agent, AI Gateway, Sandboxes, Workflows, Images, Usage.
- At the **project scope** (after clicking into v0-login-02), the sidebar swaps to project-specific items: Overview, Deployments, Logs, Analytics, Speed Insights, Observability, Firewall, CDN, Environment Variables, Domains, Connect, Integrations, Storage, Flags, Agent, AI Gateway, Sandboxes, Workflows, Images, Usage — an almost identical list, which is a deliberate consistency choice: the same mental model applies whether you're looking at "all my projects" or "this one project," just scoped differently.
- Several sidebar items (Observability, Agent, AI Gateway, Sandboxes) are **expandable chevron groups** that push a nested sub-nav into the same sidebar real estate, replacing the top-level list and adding a "back" chevron plus a breadcrumb crumb at the top (e.g. `Observability` in the header, plus `< Observability` in the sidebar itself). This is a drill-down-in-place pattern rather than opening a new panel — it keeps the transition spatially predictable (same column, same width) at the cost of losing the top-level list until you back out.

**UX read:** this is a flat-feeling but actually quite deep IA. A first-time user sees ~20 sidebar items at the top level, which is already a lot, and several of those hide 6-18 more items one level down (Observability alone has 18). The breadth is justified by Vercel's platform having genuinely grown from "static hosting" into a full compute/AI/observability platform, but it does mean the sidebar no longer fits on one screen at 1568×764 without scrolling — "Usage" and below are cut off and need a scroll, which is a real discoverability cost for less-used features.

## 2. Global navigation & wayfinding

- **Top bar** is minimal and constant: workspace switcher (top-left, with plan badge chip), breadcrumb-style page title (center), and a persistent "Agent" quick-access button (top-right) that's present on literally every screen we captured — Vercel is clearly pushing the AI agent as a first-class, always-available action rather than burying it in a menu.
- **Breadcrumbs** appear centered in the top bar rather than left-aligned under the logo (e.g. `Observability / Query`, `Project Settings / Deployment Protection`). Centering is unusual — most dashboards left-align breadcrumbs — and works here because the top bar is otherwise sparse, but it does mean your eye has to jump from the far-left workspace switcher to the center for hierarchy context, then back to the left sidebar for the actual click targets. Three separate horizontal anchor points for "where am I."
- **Account menu** (bottom-left avatar) opens a compact dropdown: identity card with settings gear, Feedback, Theme (inline three-way segmented control: system/light/dark — nice, no extra click needed), Home Page, Changelog, Help, Docs, Log Out, plus an "Upgrade to Pro" CTA baked into the menu itself. Putting a monetization CTA inside the account dropdown is a deliberate low-friction upsell placement — it's the one menu almost every active user opens repeatedly.

## 3. The "continue to X" project-picker pattern

A distinctive, recurring interaction: several team-scoped tools (Logs, Analytics, Speed Insights, Firewall, CDN, Images) don't have team-wide aggregate views — clicking them from the sidebar at the "All Projects" scope shows an interstitial **"Continue to [Tool]"** screen: an icon, one line of copy, a search box, and a list of your projects to pick from before the actual tool loads.

**UX assessment:** this is a pragmatic solution to a real constraint (these tools are inherently per-project), but it adds a mandatory extra click and a context switch for every single one of six+ sidebar items, and the screen is visually identical each time (same icon-in-a-box treatment, same copy pattern "Continue to X / Choose a project to continue"). For a workspace with only 1-2 projects (like this one), the picker screen is almost pure overhead — Vercel could reasonably auto-select the sole project or remember the last choice. Right now every visit resets to the picker.

## 4. Status communication & color language

- **Status dots** are the primary state-communication device across the whole product: a small colored circle precedes almost every list row (deployments, sandboxes, storage). Green = Ready, and other colors presumably map to Building/Error/Queued, though only "Ready" appeared in this workspace's data. The dot is consistently ~6-8px, positioned left of the label, and doubles up with a text label ("Ready") rather than relying on color alone — good baseline accessibility practice.
- **Badges** (blue "Production" pill, gray outlined "Beta"/"Pro" chips, orange team-avatar circles) are used liberally to tag scope, plan-gating, and environment. The visual weight of a solid blue "Production" pill next to a plain outlined "Preview" pill on the deployments list makes production deployments pop appropriately — an important safety signal in a tool where the wrong click can affect a live site.
- **Checkmarks in blue circles** mark completed build steps on the deployment detail page (Build Logs, Deployment Summary, Assigning Custom Domains all get a checkmark; Deployment Checks gets a neutral clock icon while pending). This is a clear, linear "build pipeline" visualization — you can scan top-to-bottom and see exactly which stage you're at.
- **Skeleton loading states** (pulsing gray blocks) appear consistently across nearly every screen during data fetch — Deployments, Environment Variables, Domains, Members all showed this mid-load in our captures. The skeletons mirror the final layout's shape closely (same row heights, same column widths), which is good practice for perceived-performance and avoiding layout shift.

## 5. Empty states & upsell integration

Vercel's empty states are unusually consistent and unusually salesy. The pattern repeats almost verbatim across Domains, Connect, Integrations, Flags, Drains, Agent, Passport, Networking: a centered icon in a rounded box, a bold one-line headline, a short description, and a primary button — frequently "Upgrade to Pro" or "Upgrade to Enterprise" rather than a neutral "Get Started."

This is a deliberate monetization-through-UX strategy: instead of hiding paid features, Vercel shows every feature's full UI shell (Passport SSO config, Access Groups, Static IPs, Protected Git Scopes) fully rendered with real controls, then gates the actual save/create action behind an upgrade wall. The user sees exactly what they'd get, which is a stronger upsell than a locked padlock icon, but it does mean a Hobby-plan user encounters "Upgrade to Pro" language extremely often — nearly every third screen in this workspace showed some paywall messaging (Billing, Networking, Passport, Deployment Protection's password option, Team Members role assignment, Access Groups, Drains, Alerts, Agent). For a first-time user exploring the product, the sheer repetition risks feeling naggy even though each individual instance is well-designed.

## 6. Settings architecture

Settings exist at two scopes — **Project Settings** and **Team Settings** — each with its own long vertical sub-nav (14 items for project, 18 for team) rendered the same way as the Observability drill-down: sidebar swaps in-place with a back chevron.

Notable groupings:
- Project Settings orders items roughly by frequency-of-use-then-risk: General → Build and Deployment → Environments → Git → Deployment Protection → Passport → Functions → Cron Jobs → Microfrontends → Project Members → Drains → Security → Networking → Activity → Advanced. "Advanced" as a catch-all last item (Directory Listing, Skew Protection, External Rewrite Caching, Bulk Redirects) is a sensible dumping ground for power-user toggles that don't fit elsewhere.
- Team Settings duplicates several names that also exist at project scope (Deployment Protection, Passport, Microfrontends, Networking, Activity) but with team-wide defaults instead of per-project overrides — the relationship between "team default" and "project override" is implied by proximity/naming rather than explicitly cross-linked in the UI text, except in one place (Deployment Protection team page explicitly says "Configure the default... settings that will be applied to newly created projects").
- Every settings card follows the same micro-pattern: bold heading, one-line description, the control itself, then a "Learn more" link + Save button pinned to the bottom-right of the card. Extremely consistent — once you've used one settings card you know exactly where to look on every other one.

## 7. Command menu (⌘K)

The command palette is contextually intelligent: triggered from a deployment logs page, its default suggestions were "Settings," "Observability," "Deployments," "Analytics," "Speed Insights" — all scoped to the *current project* — plus a "Navigation Assistant" AI suggestion synthesized from context ("How do I add another domain"). Typing "domain" surfaces three tiers of results (project-scoped Domains, team-scoped Domains, Account) ranked with the most locally-relevant result first. This is a well-executed fuzzy/contextual search that reduces the need to ever use the sidebar for a known destination — a meaningful accessibility and efficiency win for power users, though it's only discoverable if you already know to press ⌘K (there's no visible search icon inviting the shortcut in the top bar itself).

## 8. Deployment detail page — the core artifact

The single deployment page (`24_deployment-detail.jpg`) is arguably the most information-dense and well-composed screen in the product: a preview thumbnail, a compact metadata table (Created/Status/Duration/Environment side-by-side), domain list, and then a vertically stacked, collapsible pipeline (Deployment Settings → Build Logs → Deployment Summary → Deployment Checks → Assigning Custom Domains), each row showing a status icon on the right edge. Below that, four equal-width cards link out to Runtime Logs, Observability, Speed Insights, and Web Analytics — the latter two explicitly labeled "Not Enabled" as a soft upsell/onboarding nudge rather than being hidden.

This page succeeds because it answers "did my deploy work, and what do I do next" in one un-scrolled view, with progressive disclosure (collapsed sections) for anyone who wants build log detail without it cluttering the default view.

## 9. Friction points & recommendations

1. **Project-picker interstitials for single/dual-project workspaces are pure friction.** Auto-selecting the only project (or remembering the last-viewed one) for Logs/Analytics/Firewall/CDN/Images/Speed Insights would remove a click on every visit for small teams — likely the majority of Hobby-plan users.
2. **Sidebar depth without a persistent "you are here" trail.** Drilling into Observability → Query, the only way back to the top-level Projects/Deployments list is the small `<` chevron at the very top of the sidebar; there's no breadcrumb *inside* the sidebar itself showing the full path, so users can lose track of how many "back" clicks stand between them and the main list.
3. **Upsell density.** The repetition of "Upgrade to Pro/Enterprise" CTAs across a dozen+ screens is consistent brand-wise but risks fatigue; consolidating messaging (e.g., a single persistent "You're on Hobby — see what's available on Pro" banner) instead of a full-card CTA on every empty state could reduce repetition while keeping the same conversion surface.
4. **Sidebar list length.** ~20 top-level items plus expandable groups means the sidebar itself requires scrolling on a standard laptop viewport before reaching Usage/Support at the bottom. Grouping related items under collapsible section headers (the way Observability's *own* sub-nav already groups Compute/CDN/Services) at the top level too would shorten the visible list and mirror a pattern the product already uses successfully one level deeper.
5. **Centered breadcrumb placement** is visually clean but adds a third focal point (workspace switcher far-left, breadcrumb center, sidebar left) to "where am I" wayfinding — a left-aligned breadcrumb directly under/beside the workspace switcher would consolidate hierarchy information into one visual zone.

## 10. What's working well (patterns worth reusing elsewhere)

- Inline three-way theme toggle in the account menu — zero extra navigation for a common preference change.
- Deployment pipeline visualization with per-stage status icons — scannable, linear, honest about what's pending vs. done.
- Empty states that render the *full* feature UI with real controls before gating the action, rather than hiding functionality behind a locked icon — shows value before asking for payment.
- Skeleton loaders shaped like the real content, minimizing layout shift.
- Contextual, scope-aware command palette with AI-assisted fallback suggestions when no exact match exists.
- Consistent settings-card anatomy (heading → description → control → learn-more/save) trains the user once, pays off across dozens of pages.

## 11. Feedback & interaction states (validation, confirmations, menus)

This section covers triggered states beyond the static default view of each screen: inline validation errors, confirmation modals, and dropdown/kebab menus. Files: `32` through `37` in `vercel-screens-final/`.

**Inline validation.** Renaming a project to an invalid string ("Invalid Name!!") produces a red, two-line error directly beneath the input (`33_project-settings-name-validation-error.jpg`): "Project names can be up to 100 characters long and must be lowercase. They can include letters, digits, and the following characters: '.', '_', '-'. However, they cannot contain the sequence '---'." The error is specific and rule-complete in one message rather than a generic "invalid input," so the user does not need to guess-and-check. The Save button stays visible and clickable rather than disabling, which is a reasonable choice since it lets the user attempt-and-correct without a dead-end state, though it means an inattentive user could click Save repeatedly against a still-invalid value.

**Confirmation modals.** Both destructive-leaning actions tested (Instant Rollback, Redeploy) use the same modal anatomy: title, one-sentence description of the consequence, a data card showing exactly what will change (current vs. target deployment), and a clearly differentiated primary/secondary button pair (Cancel plain, action button solid). The Instant Rollback modal (`34_instant-rollback-confirmation-modal.jpg`) goes further by adding an optional "Share a reason for this rollback" textarea, which is a good incident-review pattern, it captures institutional knowledge about *why* a rollback happened at the moment someone has full context, not after. It also embeds a Pro upsell ("Upgrade to Pro to roll back to an earlier deployment") directly inside the confirmation flow rather than blocking it, consistent with the empty-state upsell pattern noted in section 6. The Redeploy modal (`36_redeploy-confirmation-modal.jpg`) surfaces a build-cache checkbox with a "Learn about Build Cache" link inline, so a technical decision that affects build time is made available at the point of action instead of buried in settings.

**Kebab / context menus.** The deployment-row kebab (`35_deployment-kebab-menu.jpg`) and project-card kebab (`37_project-card-kebab-menu.jpg`) both group actions in a consistent order: primary state-changing actions first (Instant Rollback, Promote — greyed out here since this deployment is already Production), then common utility actions (Redeploy, Inspect, View Source, Copy URL), then navigation-out actions (Assign Domain, Visit) last. Disabled items remain visible rather than being hidden, which preserves muscle memory (the option is always in the same position) at the cost of some visual noise for actions that will never apply to certain rows. The project-card kebab folds in workspace-organization actions (Add Favorite, Transfer Project) that don't exist on the deployment-row menu, showing the menu content is scoped correctly to what the entity actually supports rather than reusing one generic menu everywhere.

**Gaps in this capture pass.** A few interaction states could not be captured despite attempts and are worth flagging rather than silently omitting: toast/success notifications (e.g. after a clipboard copy or settings save) were too timing-sensitive to catch in a scripted browser session, since they appear and auto-dismiss faster than a screenshot round-trip; no destructive delete confirmation (delete project, remove domain) was triggered because no visible delete control was found in Project Settings > Advanced within the viewport; the invite-member email field on Team Settings > Members is disabled on the Hobby plan (Pro-gated), so its validation behavior is untested; and right-click context menus and hover tooltips were not attempted. If any of these matter for your purposes, they're the natural next pass.

---

*Screenshots referenced by filename correspond 1:1 to `vercel-screens-final/INDEX.md`.*

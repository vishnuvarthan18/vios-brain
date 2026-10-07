# Vercel Dashboard Screenshot Library — Index

Account: vishnu-aracreategr's projects (Hobby plan). Captured live from the logged-in workspace via Claude in Chrome, dark theme, 1568x764 viewport.

Files are organized into subfolders by category; paths below are relative to `vercel-screens-final/`.

## 01_Projects-Overview
| File | Section | Screen / State |
|---|---|---|
| 00_projects-overview-first-pass.jpg | Projects | Overview — usage panel, project cards, alerts |
| 01_overview.jpg | Projects | Overview (full re-capture) |

## 02_Deployments
| File | Section | Screen / State |
|---|---|---|
| 02_deployments-list.jpg | Deployments | Team-level deployments list, all projects |

## 03_Analytics-Insights
| File | Section | Screen / State |
|---|---|---|
| 03_logs-project-picker.jpg | Logs | "Continue to Logs" project picker state |
| 04_analytics-project-picker.jpg | Analytics | Project picker state |
| 05_speed-insights-project-picker.jpg | Speed Insights | Project picker state |
| 06_firewall-project-picker.jpg | Firewall | Project picker state |
| 07_cdn-project-picker.jpg | CDN | Project picker state |

## 04_Env-Domains-Integrations
| File | Section | Screen / State |
|---|---|---|
| 08_environment-variables.jpg | Environment Variables | Team-level, empty state |
| 09_domains.jpg | Domains | Empty state, Buy/Transfer/Connect actions |
| 10_connect.jpg | Connect (Beta) | No connectors yet |
| 11_integrations-marketplace.jpg | Integrations | Marketplace sidebar with featured integrations |

## 05_Storage-Flags-AI
| File | Section | Screen / State |
|---|---|---|
| 12_storage-create-database.jpg | Storage | Create-a-database picker (Postgres, Redis, Blob, etc.) |
| 13_flags-experimentation.jpg | Flags | No flags found + marketplace experimentation providers |
| 14_observability-overview.jpg | Observability | Overview charts (Edge Requests, Fast Data Transfer, Functions, Middleware) |
| 15_observability-query.jpg | Observability | Query builder with live chart |
| 16_ai-gateway-overview.jpg | AI Gateway | Get Started panel, code sample, usage charts |
| 17_agent-tasks-upsell.jpg | Vercel Agent | Tasks tab, Pro upsell modal with code review example |
| 18_sandboxes-overview.jpg | Sandboxes | Get Started panel, Node.js/Python code sample |
| 19_workflows-overview.jpg | Workflows | Get Started panel, workflow.ts code sample |
| 20_images-project-picker.jpg | Images (Beta) | Project picker state |

## 06_Usage
| File | Section | Screen / State |
|---|---|---|
| 21_usage-overview.jpg | Usage | Full usage breakdown sidebar (Networking, ISR, Data Cache, Functions) |

## 07_Project-Detail
| File | Section | Screen / State |
|---|---|---|
| 22_project-detail-overview.jpg | Project detail | v0-login-02 overview, Ready status, Firewall/Observability/Analytics widgets |
| 23_project-deployments-list.jpg | Project detail | Deployments tab, 2 entries with status badges |
| 24_deployment-detail.jpg | Deployment detail | Full deployment page — build logs, checks, domains, runtime logs/observability/speed insights cards |

## 08_Settings
| File | Section | Screen / State |
|---|---|---|
| 25_project-settings-general.jpg | Project Settings | General — name, ID, Vercel Toolbar |
| 26_project-settings-deployment-protection.jpg | Project Settings | Deployment Protection — auth, password protection, trusted sources |
| 28_team-settings-general.jpg | Team Settings | General — team name, URL, avatar |
| 29_team-settings-billing.jpg | Team Settings | Billing — Hobby plan features, payment method |
| 30_team-settings-members.jpg | Team Settings | Members — invite form, owner row with 2FA badge |
| 32_project-settings-general-project-id.jpg | Interaction | Project ID field with copy-icon affordance |

## 09_Overlays-Menus
| File | Section | Screen / State |
|---|---|---|
| 27_command-menu-cmdk.jpg | Overlay | Cmd+K command menu with contextual suggestions |
| 31_account-overlay-menu-theme-toggle.jpg | Overlay | Account dropdown — profile, theme toggle (system/light/dark), nav links |
| 35_deployment-kebab-menu.jpg | Interaction / Menu | Deployment row kebab (...) menu — Instant Rollback/Promote (disabled), Redeploy, Inspect, View Source, Copy URL, Assign Domain, Visit |
| 37_project-card-kebab-menu.jpg | Interaction / Menu | Project card kebab (...) menu — Add Favorite, Open in v0, Visit with Toolbar, View Logs, Manage Domains, Transfer Project, Settings |

## 10_Interactions-Feedback
| File | Section | Screen / State |
|---|---|---|
| 33_project-settings-name-validation-error.jpg | Interaction / Error | Invalid project name — inline red validation message under the rename field |
| 34_instant-rollback-confirmation-modal.jpg | Interaction / Confirmation | Instant Rollback modal — current vs. previous deployment cards, Pro upsell banner, reason textarea, Cancel/Continue |
| 36_redeploy-confirmation-modal.jpg | Interaction / Confirmation | Redeploy modal — environment dropdown, deployment info card, build-cache checkbox, Cancel/Redeploy |

## 11_Zoom-Details
| File | Section | Screen / State |
|---|---|---|
| zoom_deployment-status-dot.png | Micro-detail | Close-up of "Ready" status dot + timestamp |
| zoom_deployment-checkmarks.png | Micro-detail | Close-up of build-step checkmark icons |

## Coverage notes
- Every top-level sidebar section of the team/project dashboard is represented (Projects, Deployments, Logs, Analytics, Speed Insights, Observability, Firewall, CDN, Environment Variables, Domains, Connect, Integrations, Storage, Flags, Agent, AI Gateway, Sandboxes, Workflows, Images, Usage).
- Project-level and deployment-level detail pages, plus two representative Settings sub-pages (General, Deployment Protection) at both project and team scope, are included.
- Overlay/micro-interaction states captured: command palette, account dropdown with theme toggle, and two zoomed status-indicator details.
- Interaction/feedback states captured: one inline form-validation error, two confirmation modals (Instant Rollback, Redeploy), and two kebab context menus (deployment row, project card).
- Not captured despite attempts: toast/success notifications (copy-to-clipboard, save actions), a genuine delete confirmation (no visible danger-zone delete control found in Project Settings > Advanced), right-click context menus, hover tooltips, and the Pro-gated invite-member field (non-interactive on Hobby plan).
- Given time constraints, this is a representative one-screenshot-per-screen pass rather than every nested sub-tab (e.g. Observability's Functions/Sandboxes/Blob sub-pages, AI Gateway's Model List/API Keys, all Team Settings sub-pages). Ask if you want any specific area drilled deeper, in light theme, or the still-missing interaction states pursued further.

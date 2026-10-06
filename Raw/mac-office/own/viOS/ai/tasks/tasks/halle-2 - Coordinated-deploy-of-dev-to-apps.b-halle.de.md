---
id: HALLE-2
title: Coordinated deploy of dev to apps.b-halle.de
status: To Do
assignee: []
created_date: '2026-10-03 09:00'
labels:
  - halle-feedback-widget
dependencies: []
priority: high
ordinal: 2000
---

## Description
When no testers are active: DB backup, then deploy server code + widget bundle (WIDGET_API_ORIGIN=https://apps.b-halle.de) + `npm run db:migrate` (0013, 0014, 0015) together — the old widget and new server do not understand each other. Then `make user-admin EMAIL=vishnu@aracreate.group` on the box, and a test report from the live site. Deploy steps: [[halle-feedback-widget]] STATE.

---
id: HALLE-5
title: Fix open redirect in login next param
status: To Do
assignee: []
created_date: '2026-10-03 09:00'
labels:
  - halle-feedback-widget
dependencies: []
priority: medium
ordinal: 5000
---

## Description
src/web/app/login/actions.ts redirects to `next` when it starts with '/', so `//other-site.com` is accepted. Allow only a single leading slash.

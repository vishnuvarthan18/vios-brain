---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_users-admin.txt
---

feat(admin): manage logins from admin-only users screen

Logins could only be managed from a terminal. Admins now add, edit,
disable and enable logins, reset passwords and make other admins from
the dashboard. Every login can change its own password on My account.

- One flag, users.is_admin; checked on the server by the page and every
  action, not only by hiding the link. Non-admins get a 404
- Passwords are generated (16 chars, no look-alikes) and shown once
- You cannot disable yourself or remove your own admin, and the last
  active admin cannot be removed; admin changes in one org take turns,
  so two admins removing each other cannot both succeed
- A reset signs the person out everywhere: sessions now carry iat, and
  middleware refuses one older than users.password_changed_at.
  make user-password does the same
- make user-admin EMAIL=... makes the first admin on each server
- Migration 0014; agent-rules §5 records the partial override of
  admin-v2-spec §1.4

Spec: docs/users-admin-spec.md

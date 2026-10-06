# Users admin — spec

Agreed with Vishnu, 2 October 2026.

## What it is

Logins could only be managed from a terminal (`make user-create`,
`user-disable`, `user-enable`, `user-password`). This adds a **Users** screen
for that, and a **Settings** page (from the account menu at the foot of the
sidebar) where anyone changes their own name and password.

## Decisions

| Question | Decision |
| --- | --- |
| Who manages logins? | **Admins only.** One `is_admin` flag on `users`. Everything else stays the same for every login. |
| What can an admin do? | Add a login, edit name and email, disable and enable, reset a password, make or remove an admin. Never delete. |
| How does a password reach someone? | The app **generates** one and shows it **once**, to copy and send. No email. |
| Own password? | Yes, in **Settings**, from the account menu at the foot of the sidebar. Your own name too; your email is your login, so an admin changes it. |

This partly sets aside admin-v2-spec.md §1.4 ("one login for everyone, no
roles"). It is on record in agent-rules.md §5.

## Rules

- The admin check is on the server, on the page and in every action. The
  nav link only hides the screen; it does not protect it.
- You cannot disable yourself or remove your own admin, so you can never
  lock yourself out from this screen.
- There is always at least one active admin. The last one cannot be
  disabled or made a normal login.
- A reset password signs that person out everywhere at once
  (`users.password_changed_at`, checked by middleware against the
  session's `iat`). Changing your own password keeps you signed in on this
  device and signs you out everywhere else.
- Generated passwords: 16 characters from a 31-character alphabet with no
  look-alikes (no 0/O, 1/l/I), about 79 bits. Shown once, never stored in
  plain text.
- Emails stay unique across the whole app (the existing unique index), and
  lower-case.

## First admin

The migration makes nobody an admin. On each server, once:

```sh
make user-admin EMAIL=you@example.com
```

## Not in this change

- Roles beyond admin / not admin.
- Invitation emails, or a "forgot password" link on the login page.
- An audit log of user changes.

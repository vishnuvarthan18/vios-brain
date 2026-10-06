# LOCAL TEST PLAN

**For Vishnu to run by hand, on his own Mac, before anything goes to
deploy.** Not automated tests — those already run separately (`make lint`,
`make test`, `make test-e2e`, `make test-widget`) and were all green when this
plan was last written. This is you being a tester, then an ordinary logged-in
user, then an admin, and seeing whether the thing actually works end to end —
UI and functionality both.

Nothing here touches B. Halle's real site. Everything runs on your machine.

**The role model, so the rest of this makes sense:** there is no
client/developer/staff split in the current app. One login works everywhere.
The only distinction is a per-person Admin flag, and it gates exactly one
screen — Users — plus a few guard rails on your own account (see §8). Every
other screen below is reachable by anyone logged in.

Set aside about 75 minutes. Do it in order. Write PASS or FAIL next to each
numbered item as you go — [§11](#11-results-sheet) is the sheet.

| Part | What you are checking |
| --- | --- |
| [0 Start it up](#0-start-it-up) | One command, everything running |
| [1 Be a tester](#1-be-a-tester) | The widget, as an elderly tester would meet it |
| [2 The screenshot](#2-the-screenshot) | Including the privacy check, by eye |
| [3 Overview](#3-overview) | The dashboard home |
| [4 Queue, Tracked, Classified](#4-queue-tracked-classified) | The three report lists, and the report viewer |
| [5 The report viewer, in detail](#5-the-report-viewer-in-detail) | Where almost every action actually lives |
| [6 Pages](#6-pages) | Single add and bulk import |
| [7 Testers](#7-testers) | Create, copy link, revoke |
| [8 Classifications](#8-classifications) | Custom report types |
| [9 Wording](#9-wording) | Change the widget's text live |
| [10 Users (admin only)](#10-users-admin-only) | Accounts, and the guard rails |
| [11 Results sheet](#11-results-sheet) | What to send back |
| [12 Break it on purpose](#12-break-it-on-purpose) | The failure behaviour |
| [13 What this plan cannot tell you](#13-what-this-plan-cannot-tell-you) | Only the live round answers these |

---

## 0 Start it up

```sh
cd ~/araCreate/HLE/testing_widget/halle-feedback-widget
make demo
```

**You should see:** URLs for the app and the test page, a working tester
link, and three logins (all password `demo-pass-123` unless you changed it):
`staff@demo.test`, `developer@demo.test`, `client@demo.test`. (Those account
names are historical — today they are three ordinary logins, nothing more;
none of them has special powers over the others except whichever one you
make an admin.)

If anything errors, stop and note the error — do not work around it.

---

## 1 Be a tester

Open the test page URL **with a tester token on the end** (`?t=...`), exactly
as a real tester would receive it.

1. **The launcher appears.** Bottom right. Readable. Not covering anything
   important.
2. **Open the page WITHOUT the token.** The launcher must NOT appear. This is
   what stops a real site visitor from seeing a feedback button.
3. Back with the token. **Click the launcher.** The bar appears with the
   framing sentence.
4. **Move the mouse around.** A box outlines whatever you hover over. It
   follows you without flickering or lagging badly.
5. **Click a link in the page.** It must SELECT the link, not navigate away.
   If the page changes, that is a serious FAIL.
6. **The question screen.** Read the options.
   - Big enough to read comfortably? Each at least a finger-width tall?
   - **Close the widget and start again three times.** The options should
     appear in a different order each time — if always the same, that is a
     FAIL, it biases every result.
7. **Pick one option.** It advances immediately, no "next" button.
8. **The detail screen.** Type a sentence, then Send.
9. **The thank-you screen.** Offers "something else on this page" — click it,
   you should return to pointing, same page.
10. **Keyboard only**, from the question screen onward: Tab, arrows, Enter.
    - Is the focused thing always clearly outlined?
    - Can you finish and send?
    - (Picking the element itself still needs a mouse/touch — a deliberate,
      known limit. Confirm you're still fine with that.)
11. **"It was the whole page."** Should go straight to the question.
12. **The escape.** Escape, and separately "Stop" — both return to the
    launcher with nothing sent.

**Items 6 and 10 matter most here.** Everything else is mechanics.

---

## 2 The screenshot

Still as a tester.

13. **File a report and watch for the picture.** After Send, you should be
    shown the screenshot before it goes anywhere.
14. **Time it.** More than ~3 seconds staring at nothing is a FAIL — it
    should show the picture quickly or move on without one.
15. **"Don't include it."** Choose it, the report must still send. Check
    later (§5) that it arrived with no picture and no error.
16. **THE PRIVACY CHECK — do this one carefully.**
    - The test page has a form. Type something memorable into a text field
      and a bigger text box — something you'd notice, like a phone number.
    - File a report and look hard at the screenshot you're shown.
    - **Your typed text must not appear anywhere in that picture.** This is
      the one thing here that could embarrass you with real people's data.

---

## 3 Overview

Log in with any account at `http://localhost:3000`.

17. **The four tiles** — Waiting for a look, Fixed bugs waiting for your
    check, Testers reporting, Reports in total — should roughly match what
    you just did in §1–§2 plus whatever the demo fixture seeded.
18. **Click "Triage N waiting."** Lands on Queue. (Only shown when something
    is actually waiting.)
19. **Click "Export CSV."** A file downloads (see §4 of the old plan's
    checks, now folded into §6 of the sheet below — German characters, a
    `=1+1` note).
20. **The "Where the problems are" bars.** Click one. Lands on Queue
    filtered to that template.
21. **A brand-new install with nothing set up** (skip if you've already
    seeded data): Overview's empty state should point you at Pages, then
    Testers — not just show a blank chart.

---

## 4 Queue, Tracked, Classified

These three lists share the same filter bar and the same report viewer —
check the filter bar once, thoroughly, then move faster through the other two.

22. **Queue** (`/app/queue`): every report with no status yet, grouped by
    page template (Home, Product Category, Product Detail, Contact,
    404/Not&nbsp;Found), newest first in each group.
23. **The filter bar.** Template, Tester, Mode, From, To, Search comments.
    - **The boxes should line up in a clean row, not a ragged/crowded one.**
      This was a recent fix — check it actually looks right on your screen,
      not just at one window size. Try the browser narrower (down toward
      tablet width) and confirm the row still looks sane, then full-width.
    - **Each select's dropdown arrow should sit clearly inside its box**, not
      jammed against the right edge. Also recently fixed.
    - **The date fields' calendar icon** should have a visible gap from the
      border too.
    - Filter by template, then by tester, then combine both — the count
      should update and make sense. Clear filters, confirm it resets.
    - Type into Search comments — it should apply after you stop typing
      (no extra button), not on every keystroke.
24. **Click "View" on a row.** It should open as a popup OVER the list
    (not a full page navigation) — same filters, same scroll position
    waiting behind it.
25. **Close the popup** (the × button, Escape, or click the backdrop). You
    should land back exactly where you were on the list.
26. **Refresh the page while the popup is open.** This was just changed: you
    should land back on the list (Queue/Tracked/Classified, whichever opened
    it), **not** on a full standalone report page. If you see a full report
    page with "← Back to Queue" instead of the list, that's a regression —
    flag it.
27. **Open a report link directly in a new tab** (paste a `/app/reports/...`
    URL with no `?src=` on it, if you have one, or note that every link the
    app itself generates does carry `src=` — see the note at the end of this
    section). It should show the full report page on its own, not redirect.
28. **Tracked items** (`/app/tracked`): tabs for Bug/Fixed/Closed/Deleted.
    Confirm each tab's count matches what you'd expect from §5's actions.
29. **Classified** (`/app/classified`): tabs per custom type, then
    Open/Done within a type. If you haven't made any custom types yet, it
    should show an empty state pointing at Classifications (§8), not an
    error.

**Known, deliberate trade-off (confirm you're still fine with it):** because
every link the app generates to a report carries `?src=...`, a URL you copy
from the address bar *while the popup is open* also redirects to the list
if you paste it into a new tab, rather than opening the report standalone.
Only a link with no `src=` at all opens the full page.

---

## 5 The report viewer, in detail

Open any report (from any of the three lists above). This is where nearly
every action actually lives — the lists themselves are view-only.

30. **The step tracker** (bug or custom-type reports only — not shown for an
    unsorted Queue report). Numbered circles and a connecting line.
    - **The line should run only between the first and last circle** — no
      stray line poking out before circle 1 or after the last circle. This
      was a recent fix; look closely at both ends.
    - The circles and numbers should read clearly, not feel cramped —
      recently made bigger on purpose.
31. **A Queue report's actions** ("Sort as" row): Bug (solid, clearly the
    main option), your custom types (lighter, clearly secondary — recently
    restyled so they don't visually compete with Bug), and Delete over on
    the right, separated from the rest by a visible divider line.
    - Click Bug. The report should leave Queue and land in Tracked's Bug
      tab, status Processing.
32. **A Processing (bug) report:** only "Mark fixed" is offered. Click it —
    moves to Fixed.
33. **A Fixed report:** "Confirm fix" (→ Closed) and "Not fixed, reopen"
    (→ back to Processing) are both offered — no restriction against
    re-opening something already fixed.
34. **A Closed report:** "Reopen" only.
35. **A custom-type report, Open:** "Done" only. Done: "Reopen" only.
36. **Change type**, on any classified or bug report: opens a short list,
    current type marked "Now", one click changes it and closes the list.
37. **Delete** a Queue report, then find it in Tracked's Deleted tab — the
    row itself should still be there with "Restore" offered, not gone for
    good.
38. **Previous / Next**, inside the popup. Should stay within whichever
    list's current filters opened the viewer — if you filtered Queue to one
    template first, Next should only walk that filtered set.
39. **Comments and Activity.** Add a comment. Confirm it appears in the
    Activity timeline with your name and the time, alongside every status
    change you just made.
40. **"What else was sent with this"** (collapsed panel). Open it — should
    show the page URL, device/browser info, and any console errors captured
    at the time.

---

## 6 Pages

41. **Add one page by hand** (path, label, page type).
42. **Bulk import.** Paste five URLs, one per line — include one duplicate
    of something already in the list, and one duplicate repeated twice
    within the paste itself, and one blank line. It should report how many
    it added and how many it skipped (both kinds of duplicate, plus the
    blank, should all be skipped, not just DB duplicates).
43. **Edit a page** you just added — change its label or type, save, confirm
    it shows the new value in the list.

---

## 7 Testers

44. **Create three testers** (name only — two can share a name, that's
    allowed).
45. **Copy a link**, open it in a private/incognito window — the widget
    should appear and know who you are without you doing anything else.
46. **Revoke a tester.** Confirm the confirmation step itself works (it
    should ask before doing it). After revoking: the old link must no
    longer work, but any reports that tester already filed must still be
    there, fully readable, in Queue/Tracked/Classified.
47. **Revoke the same tester a second time** (if the UI lets you reach the
    action again) — should be harmless, not an error.
48. **A removed tester's row** should show their name but no link at all —
    not even a greyed-out one.

---

## 8 Classifications

49. **Add two custom types** (name + colour). Try to name one "Bug" — it
    should be refused, Bug is built in and reserved.
50. **Try a duplicate name** of a type you just made — refused.
51. **Hide a type** that already has reports classified under it. Confirm
    it disappears from the "Sort as" row on new Queue reports, but its tab
    still shows in Classified as long as it still has reports.
52. Confirm the built-in Bug row at the top cannot be edited or hidden.

---

## 9 Wording

53. Find the launcher text field.
54. **Change it to something obvious** — "TEST TEST TEST" — and save.
55. **Reload the tester page** (§1's test page). Within about a minute the
    launcher should show your new words — no rebuild, no restart.
56. **Roll it back**, from the History list. The old wording should return,
    and the history should still show both versions (rollback adds a new
    entry, it doesn't erase anything).
57. **Try to save something broken** — clear a required field entirely. It
    should refuse and change nothing (check the live wording is unchanged
    after the refused save, not just that an error appeared).

**If item 55 fails, flag it immediately** — it's the difference between
changing wording yourself and needing a rebuild for every comma.

---

## 10 Users (admin only)

This screen is the one actual role gate in the app. Log in as whichever
account is currently an admin (or make one via `make user-admin EMAIL=...`).

58. **A non-admin login has no "Users" link in the sidebar**, and typing
    `/app/admin/users` directly gets a plain "not found" — not a login
    prompt, not a permission-denied message. (This is deliberate: the screen
    shouldn't even reveal it exists to a non-admin.)
59. **Add a user.** The generated password is shown exactly once — note it,
    then confirm you can log in with it.
60. **Reset another user's password** — again, shown once; confirm the new
    one works and the old one doesn't.
61. **Toggle another user's Admin flag on, then off.**
62. **Disable another user**, then try to log in as them — refused at once,
    even with a previously-valid, already-open session in another browser
    tab (not just blocked at next login).
63. **Re-enable them** — login works again.
64. **Try to disable your own login.** Must be refused with a clear message.
65. **Try to remove your own Admin flag** while you're the only admin. Must
    be refused ("ask another admin" / "make someone else admin first").
    If you have a second admin, removing your own admin flag should then be
    allowed.
66. **Edit your own name.** It should update in the sidebar immediately,
    without needing to log out and back in.

---

## 11 Results sheet

Send this back filled in. Items 2, 9, 10, 16, 24–27, 30, 46, 55, 58, 62, 64,
65 are the ones worth the closest look — the rest is mechanics.

```
1  __   10 __   19 __   28 __   37 __   46 __   55 __   64 __
2  __   11 __   20 __   29 __   38 __   47 __   56 __   65 __
3  __   12 __   21 __   30 __   39 __   48 __   57 __   66 __
4  __   13 __   22 __   31 __   40 __   49 __   58 __
5  __   14 __   23 __   32 __   41 __   50 __   59 __
6  __   15 __   24 __   33 __   42 __   51 __   60 __
7  __   16 __   25 __   34 __   43 __   52 __   61 __
8  __   17 __   26 __   35 __   44 __   53 __   62 __
9  __   18 __   27 __   36 __   45 __   54 __   63 __
```

For anything that fails, write down: what you did, what you expected, what
happened. Do not fix it and do not ask the agent to fix it before you've
seen it written down — a failure here sometimes means the plan or the spec
was wrong, not the code.

---

## 12 Break it on purpose

67. **Stop the app** (Ctrl-C in the terminal, or `make demo-stop`). Reload
    the tester page. It must look completely normal — no error, no broken
    box, no console noise, and no launcher.
68. **Start the app again.** Load the tester page, click the launcher, get
    to the detail screen, and THEN stop the app. Press Send.
    - You should still see the thank-you screen.
    - The tester must never see an error — a lost report is our problem,
      not theirs.
69. **A wrong key.** Open the test page with a made-up key in place of the
    real one. Nothing at all should appear.
70. **The export, under stress.**
    - Put a note through the widget containing
      `Grüße, größer, Straße, Öl`, export, open in Numbers/Excel — those
      characters must look right, not as `GrÃ¼ÃŸe`.
    - File a report whose note is exactly `=1+1`. Export and open — it must
      show as the text `=1+1`, NOT the number 2. If your spreadsheet
      calculates it, that's a FAIL.

---

## 13 What this plan cannot tell you

This plan runs on your Mac, in your browser. It says nothing about Webflow's
published site, CMS product pages, the real 404 page, Safari, an iPad, or
whether taking a screenshot disturbs anything on the real page. All of that
is `docs/live-test-plan.md` — the live-site round, after this one passes
clean. Do not skip straight to deploy from this plan alone.

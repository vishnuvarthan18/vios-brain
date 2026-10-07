# Going live — the morning sheet

Written overnight, 20 Sep. Everything below was tested; nothing was deployed,
because the bridge from this session has no route to the server (port 22 is
unreachable and the domain does not resolve from it).

**Read section 1. Run section 2. Section 3 is if it goes wrong.**

---

## 1. What you are shipping, in one breath

- The **new front end at `/`**. Students open the site and get the React app.
- **Scoring v3** — nobody marks anything by hand. This has been in the repo
  for two days and has never been on the server.
- The **daily survey**, which has also never been on the server.
- **Track 5** — the team page, the completion matrix, and a name or a number
  you can click to get the list of people and their phone numbers.
- **Track 6** — five groups of navigation for you, five tabs for a student.

Three bugs were found by looking at screens, and fixed. They are in section 4.

---

## 2. Do this

**One command at a time. Wait for each to finish.**

### 2.1 Build the front end (on the Mac)

```sh
cd ~/araCreate/bootcamp-dashboard/web && npm run build
```

Expect: `✓ built in …`. The build is not in git, so this must happen before
the copy or the server gets the old screens.

### 2.2 Commit (on the Mac)

```sh
cd ~/araCreate/bootcamp-dashboard
git rm -q web/src/pages/Marking.jsx web/src/pages/Board.jsx
rm -f .xfer-tmp.tgz
git add -A && git commit -m "scoring v3, track 5, track 6, and the / cutover" && git push
```

`.xfer-tmp.tgz` is a 14 MB copy of the repo used to move it into the test
machine overnight. Deleting it is the line above; it must not reach the server.

### 2.3 Copy the code up

```sh
rsync -az --delete --exclude node_modules --exclude .git --exclude .env \
  ~/araCreate/bootcamp-dashboard/ hetzner:/tmp/bootcamp-src/

ssh hetzner 'sudo rsync -a --delete --exclude .env /tmp/bootcamp-src/ /opt/bootcamp-dashboard/'
```

### 2.4 Go live

```sh
ssh hetzner 'sudo bash /opt/bootcamp-dashboard/scripts/go-live.sh'
```

That one script does all of it, and stops at the first thing that is not right:

1. takes a full backup **and counts the students inside the backup file** — if
   the count does not match the database it refuses to go on
2. applies only the migrations the server is missing, **in dependency order**,
   checking after each one that it really landed
3. checks the student and team counts are unchanged
4. restarts, then asks the running site whether `/`, `/v3/` and `/old/` answer

It is safe to run twice. Rehearsed three times overnight: on a database with
nothing but the base schema (16 migrations applied), on the same database
again (0 applied), and on a full 209-student database (0 applied, backup
verified).

### 2.5 Look at it yourself

Open **https://vcet.aracreate.academy** on your phone and sign in as a
student. You should get five tabs: Today · My work · Board · Posts · You.

Then sign in as yourself and check **Reports → Completion**.

---

## 3. If the morning goes wrong

**Two levels. Try the first one. It fixes the look without touching data.**

### 3.1 The site looks wrong → put the old screens back

```sh
ssh hetzner 'sudo bash /opt/bootcamp-dashboard/scripts/ui.sh old'
```

To go back to the new one, the same command with `v3`. To ask which is
running, the same command with nothing after it.

Ten seconds. Students get exactly the screens they had yesterday. **No data
changes at all**, and everything built this week is still there at `/v3/`.

### 3.2 Something is actually broken → put the database back

```sh
ssh hetzner 'ls -t /var/backups/bootcamp/pre-golive-*'
ssh -t hetzner 'sudo bash /opt/bootcamp-dashboard/scripts/rollback-live.sh <the newest one>'
```

It asks you to type `yes`, and it takes its own backup of the current state
first, so even the rollback is undoable.

**This loses everything students did after the backup was taken** — attendance,
quiz answers, work handed in. If the problem is only how the site looks, use
3.1 instead. Rehearsed overnight: a row added after the dump was gone after
the restore, and the count came back to exactly what the dump held.

---

## 4. What was found by looking, and fixed

None of these threw an error. None would have been caught by a test that only
reads a response.

**1. Every student's home said "0 points".**
`/api/my-team` was still reading `teams.total_points`, a column the scoring
cutover pins at 0 on purpose. So a team's own screen said 0 while the board
two taps away showed the real number. On the screen students look at most.
Now both read the same view, and a test asserts they agree.

**2. The completion matrix made every page scroll sideways on a phone.**
The hidden text that tells a screen reader what each coloured block means was
positioned against the page instead of against its own cell, so it sat 193px
off the right edge — outside the table's own scroll box. Section 7 forbids a
page that scrolls sideways. One word of CSS.

**3. A mentor's first screen was a dead end.**
Every panel on Home reads an admin-only endpoint, so a mentor signing in
landed on "Admin only / Try again". It had been true since before the v3
migration, and no test saw it because every test signs in as an admin.
Mentors now start on the Leaderboard. (There are no mentor accounts on the
live site, so this affected nobody — it would have, the day you made one.)

Also fixed while in there: table headings that read "DayWhat" and
"PointsNote" because the heading row had no horizontal padding while the
cells below it did.

---

## 4b. Two things that will happen the moment you run it

**1. The board stops being zeros.** Every team is on 0 today because nothing
has ever been marked. The instant the scoring migrations land, points appear
from attendance and hand-ins. On the real dump last night that was **0 →
1082, every team between 16 and 29, nobody went down and nobody stayed on
zero.** That is the intended effect, but it is visible to all 209 students at
once, so run it when you are ready to answer "why do we have 24?"

The answer, if anyone asks: 1 point per person per day present, 5 per team per
hand-in. Nothing else scores yet, because there are no quiz or survey
questions.

**2. Completion will look nearly empty, and that is correct.** Nothing has
ever been opened for a venue, so the matrix says "nothing was asked" rather
than branding 209 students as behind. It fills in as you open things. Checked
against a copy shaped exactly like the live site: **0 are behind**, which is
the true answer.

---

## 4c. If you roll the screens back, tell staff one thing

The old screens still have a **Marking** tab. On a scoring-v3 database it will
say "215 waiting" and let someone mark all of them.

**Every save comes back OK, the counter goes down, and the points do not
move.** Checked: 200 OK, counter 215 → 214, leaderboard byte-identical.

It is not dangerous — nothing is corrupted, it just achieves nothing. But an
afternoon of it would be an afternoon wasted, so if you ever run
`scripts/ui.sh old`, **staff should work at `/v3/`**, where there is no
Marking tab. The script prints this reminder itself when you run it.

Students are unaffected either way: all seven of their old screens were driven
on a scoring-v3 database and every one works, with the points agreeing with
the board.

---

## 5. Still yours. None of it is code

| | |
| --- | --- |
| **Rotate the staff password** | It is in an old chat transcript. You said leave it overnight; it is still the oldest open item |
| **Rotate the Google key** | Cloud Console → the service account → Keys → new key → both `.env` files → **delete the old one** |
| **Write quiz questions** | Zero exist. Quiz points are structurally zero until they do |
| **Write survey questions** | Zero exist |
| **Open the pre-assessment for EEE** | 55 students have never had it |

---

## 6. What was NOT deployed or decided

- **No quiz or survey questions were written.** You said not to.
- **Progress and Quiz results were not deleted**, though the matrix now covers
  most of what they showed. Deleting a screen is a decision, not a tidy-up.
- The four failing assertions in `tests/tasks.js` are the same four that were
  failing before any of this, for the same two reasons, written up in
  `known-issues.md`. They were proved unrelated by running the whole suite
  with tonight's server change removed and getting an identical result.

---

## 7. What is still NOT tested. Read this before you decide

Everything above was tested against a **209-student synthetic fixture** and a
second copy shaped like the live site. Not against your actual data, which I
cannot reach. These remain open:

| Not tested | Why it matters |
| --- | --- |
| **Google Drive hand-ins** | No credentials in the test machine. It is the thing students do most. The code was not touched — but it was not proved either |
| **The deploy mechanics** | `go-live.sh` was rehearsed with systemd and sudo switched off. The real `.env`, Caddy, the service, and the server's actual migration state are unverified. The script checks itself at every step and stops rather than guessing, which is the mitigation |
| **The server's migration state** | Unknown from here. The script detects and applies only what is missing — but that detection has never run against the real thing |
| **Real phones** | Headless Chromium at 390px, not an Android on college wifi |
| **A real student** | The React front end has never been in front of one |

**209 at once WAS tested**: all 209 signed in in under a second, then all of
them loaded their home screen at the same instant — 1,463 requests, median
2.3s, slowest 3.2s, **zero errors**. 77 requests were told to wait and came
back on their own, which is the queue doing exactly what it was built to do.
That was on this machine, not on the Hetzner box, so treat it as "the code
holds up", not "the server will".

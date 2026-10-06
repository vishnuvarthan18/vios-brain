---
tags: chat
project: "[[Projects/ac-training/SUMMARY]]"
date: 2026-09-26
source: Claude office account (Cowork)
session: cse_014L5QHqrmD5XCartbRjVaqB
---
# Team photo and project form

- Project: [[Projects/ac-training/SUMMARY]]
- Date: 2026-09-26
- Full chat, word for word (tool steps shortened).

---

**Vishnu:** i need to create a google fomr to get all the team photot and poject anme 

first they need to select tem propject name and photot

**Claude:**   _[tool: Projects — team names list project names]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: Projects — claude/ece-student-logins.csv]_

**Claude:**   _[tool: Projects — claude/eee-student-logins.csv]_

**Claude:** Here are the steps. It takes about 5 minutes.

**1. Make the form**

- Go to **forms.google.com** → click **Blank form**
- Title: `Team Photo & Project Name`
- Description: `One form per team. Team lead fills it.`

**2. Question 1: Team (dropdown)**

- Click the question type → choose **Dropdown**
- Question: `Select your team`
- Click in the "Option 1" box, then paste the whole list below. Google Forms makes one option per line.
- Turn on **Required**

```
EEE-T01-CIRCUITCREW
EEE-T02-COREX
EEE-T03-NEXORA
EEE-T04-ELECTROVERSE
EEE-T05-CORECREW
EEE-T06-TECHSPARK
EEE-T07-POWERPULSE
EEE-T08-RENEWTECH
EEE-T09-SPARKX
EEE-T10-THEVOLT
EEE-T11-ENGINOVA
EEE-T12-ELECTROEMPIRE
EEE-T13-SPARKSHIFT
EEE-T14-WATTMINDS
ECE-T01-VOLTSQUAD
ECE-T02-LIVEWIRE
ECE-T03-OHMFORCE
ECE-T04-HIGHVOLTAGE
ECE-T05-BITCREW
ECE-T06-BYTEFORCE
ECE-T07-WAVERIDERS
ECE-T08-PULSETEAM
ECE-T09-CHIPSQUAD
ECE-T10-DATACREW
ECE-T11-DIODESQUAD
ECE-T12-RELAYTEAM
ECE-T13-FUSEFORCE
ECE-T14-COILCREW
ECE-T15-WIREWORKS
ECE-T16-LASERSQUAD
ECE-T17-RADARTEAM
ECE-T18-SONARCREW
ECE-T19-RADIOWAVE
ECE-T20-ANTENNACREW
ECE-T21-SENSORSQUAD
ECE-T22-MOTORFORCE
ECE-T23-POWERGRID
ECE-T24-LOGICCREW
ECE-T25-GATEFORCE
ECE-T26-SIGNALTEAM
ECE-T27-NODESQUAD
ECE-T28-LINKFORCE
ECE-T29-ECHOCREW
ECE-T30-BEAMTEAM
ECE-T31-PIXELSQUAD
ECE-T32-ROBOTCREW
ECE-T33-DRONEFORCE
ECE-T34-CODETEAM
ECE-T35-CLOCKWORKS
ECE-T36-SWITCHSQUAD
ECE-T37-OPENLOOP
ECE-T38-SILICONCREW
```

**3. Question 2: Project name**

- Click **+** → type **Short answer**
- Question: `Project name`
- Turn on **Required**

**4. Question 3: Team photo**

- Click **+** → type **File upload** → click **Continue**
- Question: `Upload team photo (all members)`
- Turn on **Allow only specific file types** → tick **Image**
- Max number of files: **1** (or 3 if you want more than one photo)
- Max file size: **10 MB**
- Turn on **Required**

**5. Send it**

- Click **Send** → link icon → tick **Shorten URL** → copy the link
- Share the link in the team groups

**Good to know**

- For the photo upload, students must **sign in to Google**. The form will not open without sign-in.
- Photos are saved in **your Google Drive**, in a folder named after the form.
- To see answers: open the **Responses** tab → click **Link to Sheets** to get a sheet with team, project name and photo link.
- The list has all **52 teams** (14 EEE + 38 ECE), taken from the student login sheets in the project.

**Vishnu:** all memes plus prohect infromt of

**Claude:** Update **Question 3** so students know what the photo must show.

**Question 3: Team photo**

- Question: `Upload team photo`
- Click the **3 dots** (bottom right of the question) → **Description** → paste this:

```
Photo rules:
- All team members must be in the photo
- Keep your project in front of the team, clearly visible
- Take it in good light, not blurry
- Landscape (sideways) photo is best
```

- Keep the other settings the same: **Image** only, **1 file**, **10 MB**, **Required**

**Form description (top of the form):** you can also change it to:

```
One form per team. Team lead fills it.
Photo must show all members with the project in front.
```

**Vishnu:** need to add both depattem so first lets select depatn then team name

**Claude:** Use **sections**. Students pick the department first, then they see only that department's teams.

**Final layout**

- **Section 1:** Department (EEE / ECE)
- **Section 2:** EEE team list
- **Section 3:** ECE team list
- **Section 4:** Project name + team photo

**Step 1: Section 1 – Department**

- Delete the old team dropdown that has all 52 teams
- Add a question → type **Multiple choice**
- Question: `Select your department`
- Options: `EEE` and `ECE`
- Turn on **Required**
- Click the **3 dots** on this question → tick **Go to section based on answer** (you will set this in Step 5)

**Step 2: Section 2 – EEE teams**

- Click **Add section** (the icon with two bars on the right side toolbar)
- Section title: `EEE Team`
- Add question → **Dropdown** → `Select your team`
- Paste this list, turn on **Required**:

```
EEE-T01-CIRCUITCREW
EEE-T02-COREX
EEE-T03-NEXORA
EEE-T04-ELECTROVERSE
EEE-T05-CORECREW
EEE-T06-TECHSPARK
EEE-T07-POWERPULSE
EEE-T08-RENEWTECH
EEE-T09-SPARKX
EEE-T10-THEVOLT
EEE-T11-ENGINOVA
EEE-T12-ELECTROEMPIRE
EEE-T13-SPARKSHIFT
EEE-T14-WATTMINDS
```

**Step 3: Section 3 – ECE teams**

- Click **Add section** again
- Section title: `ECE Team`
- Add question → **Dropdown** → `Select your team`
- Paste this list, turn on **Required**:

```
ECE-T01-VOLTSQUAD
ECE-T02-LIVEWIRE
ECE-T03-OHMFORCE
ECE-T04-HIGHVOLTAGE
ECE-T05-BITCREW
ECE-T06-BYTEFORCE
ECE-T07-WAVERIDERS
ECE-T08-PULSETEAM
ECE-T09-CHIPSQUAD
ECE-T10-DATACREW
ECE-T11-DIODESQUAD
ECE-T12-RELAYTEAM
ECE-T13-FUSEFORCE
ECE-T14-COILCREW
ECE-T15-WIREWORKS
ECE-T16-LASERSQUAD
ECE-T17-RADARTEAM
ECE-T18-SONARCREW
ECE-T19-RADIOWAVE
ECE-T20-ANTENNACREW
ECE-T21-SENSORSQUAD
ECE-T22-MOTORFORCE
ECE-T23-POWERGRID
ECE-T24-LOGICCREW
ECE-T25-GATEFORCE
ECE-T26-SIGNALTEAM
ECE-T27-NODESQUAD
ECE-T28-LINKFORCE
ECE-T29-ECHOCREW
ECE-T30-BEAMTEAM
ECE-T31-PIXELSQUAD
ECE-T32-ROBOTCREW
ECE-T33-DRONEFORCE
ECE-T34-CODETEAM
ECE-T35-CLOCKWORKS
ECE-T36-SWITCHSQUAD
ECE-T37-OPENLOOP
ECE-T38-SILICONCREW
```

**Step 4: Section 4 – Project and photo**

- Click **Add section** again
- Section title: `Project & Photo`
- Move (or make again) the **Project name** and **Team photo** questions into this section. Drag them down using the dots at the top of each question.

**Step 5: Set the jumps (most important)**

- Go back to the **Department** question in Section 1:
  - Next to **EEE** → choose **Go to section 2 (EEE Team)**
  - Next to **ECE** → choose **Go to section 3 (ECE Team)**
- At the bottom of **Section 2 (EEE)**, find **"After section 2"** → choose **Go to section 4 (Project & Photo)**
  - Without this, EEE students will also see the ECE list
- Leave **Section 3 (ECE)** as **Continue to next section**

**Step 6: Test it**

- Click the **eye icon** (Preview)
- Pick EEE → you should see only EEE teams → then project and photo
- Pick ECE → you should see only ECE teams → then project and photo

**In the response sheet:** EEE and ECE teams show in **two different columns**. Each row fills only one of them, so this is normal.

**Vishnu:** only this showing

**Claude:** The dropdown only shows **Section 1** because there is only one section right now. "EEE Team" was added as a **text title block**, not a **section**, and a section was also deleted (see the "Section deleted" message at the bottom).

**First, fix small typos**

- Form title: remove "Untitled form" → keep `Team Photo & Project Name`
- Question: change to `Select your department`

**Step 1: Delete the "EEE Team" text block**

- Click the "EEE Team" box → click the **bin icon**
- A list inside a title block cannot be selected by students. It must be a **Dropdown** question.

**Step 2: Add real sections**

- Click the **Department** question
- In the right side toolbar, click the **last icon** (two bars, at the bottom). This is **Add section**.
- You will see a new purple tab that says **"Section 2 of 2"** at the top. That is a real section.
- Do this **3 times** so you have: **Section 2**, **Section 3**, **Section 4**

**Step 3: Fill each section**

- **Section 2:** title `EEE Team` → click **+** → **Dropdown** → `Select your team` → paste the EEE list → Required
- **Section 3:** title `ECE Team` → **+** → **Dropdown** → `Select your team` → paste the ECE list → Required
- **Section 4:** title `Project & Photo` → put the **Project name** and **Team photo** questions here

**Step 4: Set the jumps**

- Go back to the Department question → click the dropdown next to each option:
  - **EEE** → **Go to section 2 (EEE Team)**
  - **ECE** → **Go to section 3 (ECE Team)**
- At the bottom of Section 2 → **After section 2** → **Go to section 4 (Project & Photo)**

The dropdown now lists all 4 sections, because they exist.

**Note:** your form already says **Published**, so students with the link can see it now. Finish the changes before you share the link.

**Vishnu:** One form per team. Team lead fills it.
Photo must show all members with the project in front.

chae this to only one subittion per team

**Claude:** **New description (paste this):**

```
Only one submission per team.
Photo must show all members with the project in front.
```

**Good to know**

- Google Forms **cannot block a second submission from the same team**. It can only allow **1 submission per Google account**.
- To turn that on: **Settings** → **Responses** → turn on **Limit to 1 response**
- If two people from the same team submit, you will see two rows for that team in the sheet. Keep the latest one.

**Vishnu:** https://forms.gle/Dee3w9cCoSMpzkoW6

chcek one in your browers

**Claude:**   _[tool: ToolSearch — select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome]_

**Claude:**   _[tool: mcp__claude-in-chrome__navigate — https://forms.gle/Dee3w9cCoSMpzkoW6]_

**Claude:**   _[tool: mcp__claude-in-chrome__tabs_context_mcp]_

**Claude:** Chrome isn't reachable, so I'll try the built-in browser.

**Claude:**   _[tool: ToolSearch — mcp__remote-devices__Claude_Browser__]_

**Claude:**   _[tool: mcp__remote-devices__Claude_Browser__preview_start — https://forms.gle/Dee3w9cCoSMpzkoW6]_

**Claude:**   _[tool: mcp__remote-devices__Claude_Browser__preview_start — https://docs.google.com/forms/d/1FfsSU4Dj6J7J3eS1w4-kGxvHt8TYM87ESoOELAkCxco/viewform]_

**Claude:** I couldn't open the form:

- **Your Chrome:** the Claude extension isn't connected right now. Chrome may be closed, or the extension may be off or not signed in.
- **Built-in browser:** your organization's policy blocks Google Forms links there.

**How we can still check it**

- **Option 1:** open Chrome, make sure the Claude extension is on and signed in to this account, then tell me "try again"
- **Option 2:** open the form link yourself, click through it, and send me screenshots:
  - The first page (department question)
  - The page after picking **EEE**
  - The page after picking **ECE**
  - The last page (project name + photo)

**What I will check**

- EEE shows only EEE teams, and ECE shows only ECE teams
- After the EEE page it jumps straight to project and photo, not to the ECE page
- The title and description have no typos
- The photo question accepts images only and is required

**Vishnu:** ok next feed back fomr no need to colled name it newd to. be anonymus

**Claude:** Here is a ready **anonymous feedback form** for the bootcamp. Just copy each part into Google Forms.

**Step 1: Make it anonymous (most important)**

- Make a new blank form
- Go to **Settings** → **Responses**:
  - **Collect email addresses** → **Do not collect**
  - **Limit to 1 response** → **OFF** (turning it on makes students sign in, which is not anonymous)
- Do **not** add any **File upload** question. It also makes students sign in.
- Do **not** ask for name, reg no, phone or team

**Step 2: Title and description**

```
Bootcamp Feedback – Basic Electronics
```

```
This form is anonymous. We do not collect your name or email.
Please give honest feedback. It helps us make the next bootcamp better.
```

**Step 3: Questions**

**Q1. Department**, Multiple choice, Required

- `EEE`
- `ECE`

**Q2. Overall, how was the bootcamp?**, Linear scale 1 to 5, Required

- 1 = `Poor`, 5 = `Excellent`

**Q3. Rate each part**, Multiple choice grid, Required

- Rows:

```
Hands-on projects
Daily quizzes
Tinkercad
Mentors / trainers
Bootcamp app / website
Time given for each task
```

- Columns:

```
Poor
Average
Good
Very good
Excellent
```

**Q4. How much did you learn?**, Multiple choice, Required

- `A lot`
- `Some`
- `A little`
- `Nothing new`

**Q5. Was the pace right?**, Multiple choice, Required

- `Too fast`
- `Just right`
- `Too slow`

**Q6. Would you tell a friend to join this bootcamp?**, Linear scale 0 to 10, Required

- 0 = `No`, 10 = `Yes, for sure`

**Q7. What did you like most?**, Paragraph, Optional

**Q8. What should we change or improve?**, Paragraph, Optional

**Q9. Any other comments?**, Paragraph, Optional

**Step 4: Test and share**

- Open it in a **private / incognito window**. It should open **without asking you to sign in**. If it asks, check Step 1 again.
- **Send** → link → **Shorten URL** → share it

**Good to know**

- Because there is no sign-in, one student can submit more than once. That is the trade-off for being fully anonymous.

**Vishnu:** jsut it need to be a paragraha and Overall, how was the bootcamp

**Claude:** Here is the short version with just **2 questions**.

**Settings (keeps it anonymous)**

- **Settings** → **Responses** → **Collect email addresses** → **Do not collect**
- **Limit to 1 response** → **OFF**

**Title**

```
Bootcamp Feedback
```

**Description**

```
This form is anonymous. We do not collect your name or email.
```

**Q1. Overall, how was the bootcamp?**

- Type: **Linear scale**, 1 to 5
- 1 = `Poor`, 5 = `Excellent`
- **Required**

**Q2. Share your feedback**

- Type: **Paragraph**
- Description (optional): `What did you like? What should we improve?`
- **Required**

If you want only **one** question, delete Q1 and rename Q2 to `Overall, how was the bootcamp?` (still a **Paragraph**).

**Vishnu:** we have a change as plan lets make it full star rating and one otpanl tesxt

**Claude:** Here is the new plan: one **star rating** and one **optional text box**.

**Q1. Overall, how was the bootcamp?**

- Click the question type → choose **Rating** (the star icon)
- Levels: **5**
- Icon: **Star** ⭐
- Turn on **Required**

**Q2. Any feedback?**

- Type: **Paragraph**
- Description: `What did you like? What should we improve?`
- Keep **Required** turned **OFF**, so it stays optional

**Keep it anonymous**

- **Settings** → **Responses** → **Collect email addresses** → **Do not collect**
- **Limit to 1 response** → **OFF**

**Note:** if you don't see **Rating** in the list, use **Linear scale** 1 to 5 instead. It works the same way, just without stars.

**Vishnu:** lets ask 5 qursion likehow was the trainer like that

**Claude:** Here are **5 star-rating questions** and **1 optional text box**.

**For all 5 rating questions:** type **Rating**, **5** levels, icon **Star**, **Required ON**

**Q1.** `How was the bootcamp overall?`

**Q2.** `How were the trainers?`

**Q3.** `How were the hands-on projects?`

**Q4.** `How much did you learn?`

**Q5.** `How was the bootcamp app / website?`

**Q6.** `Any feedback? (optional)`

- Type: **Paragraph**
- Description: `What did you like? What should we improve?`
- **Required OFF**

**Quick tip:** after you make Q1, click the **Duplicate** icon (two squares) 4 times and change only the question text. That keeps the star settings the same.

**Keep it anonymous**

- **Settings** → **Responses** → **Collect email addresses** → **Do not collect**
- **Limit to 1 response** → **OFF**

**Claude:** The form looks good. Fix these 3 things before you share it:

**1. The last question is still required**

- "Any feedback? (optional)" has a red **\*** next to it, so students must fill it
- Open that question → turn **Required OFF**
- Also check its type is **Paragraph**, not Short answer, so students get a big text box

**2. The form is not published yet**

- At the top it says **Not published** and **This form isn't accepting responses**
- Go back to edit → click **Publish** (top right) → confirm

**3. Let students outside your company open it**

- The bottom says "created inside araCreate Group". Students use Gmail accounts, so they may be blocked.
- **Settings** → **Responses** → turn **OFF** "Restrict to users in araCreate Group"
- Or in **Publish** → **Responders** → set it to **Anyone with the link**

**Final check**

- Copy the responder link → open it in an **incognito window**
- It should open **without sign-in**, show all 5 star questions, and let you submit without typing feedback

Everything else is fine: all 5 rating questions have stars, are required, and the description says it is anonymous.

**Claude:** Good, it's **Published** now and **Limit to 1 response** is OFF. A few small changes on this page:

**On this screen**

- **Show link to submit another response** → turn **OFF**. This stops students from easily sending feedback twice.
- **Confirmation message** → click **Edit** → paste:

```
Thank you for your feedback! 🙏
```

- Leave everything else as it is

**Other checks**

- **File name:** top left still says **Untitled form**. Click it and rename it to `Bootcamp Feedback` so you can find it in Drive.
- **"Section deleted" message:** you deleted something by mistake. Go to **Questions** and check all 6 questions are still there. If something is missing, click **UNDO** now.
- **Who can open it:** click the **Published** button (top right) → **Responders** → set to **Anyone with the link**. If it says only araCreate Group, students cannot open it.

**Last test:** copy the link → open it in **incognito** → it should open with no sign-in.

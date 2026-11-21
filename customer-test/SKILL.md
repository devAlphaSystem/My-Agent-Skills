---
name: customer-test
description: Use a web project's running system the way its end customers would, from a new chat. A scout learns the product through the browser, then many customer delegates, each with a persona and one purpose, use it without reading the code and write first-person diaries with screenshots. An aggregator turns the diaries into a findings report written for the agent that will fix them. The user starts the system. The run never starts or stops it and never fixes anything. For testing the server directly, with the code and the database in view, use server-test. Use only when the user says 'customer-test'.
---

# Customer web

Use this skill to learn what it is like to be a customer of a web system. It runs only when the user asks for it, in a new chat that knows nothing about the project. Never start it automatically. The user has already started the system. A scout learns what the product is through the browser, the coordinator derives a cast of customers from what the scout saw, and each customer uses the system with one purpose and writes a first-person diary of the experience. Nobody reads the source code, because a customer never sees it. An aggregator turns the diaries into one report.

The user does not read the results. Another agent receives the report and fixes what it describes. Write every file for that agent. It has not seen the run, it can't ask the customers anything, and it acts on what the report says. A finding belongs in the report only if a customer saw it on the screen and wrote down how to see it again.

A run can start dozens of delegates and take hours. Invoking this skill is the explicit request for that scale. The delegates report problems and never fix them. For testing the server directly, with the code and the database in view, use server-test.

## Roles

The coordinator is the agent in the conversation. It talks to the user, finds the running system, builds the roster, starts each delegate, and reports back. The coordinator does not use the system on its own, apart from the session check in step 2. It does not read or edit source files, and it never starts or stops the system.

Use these exact models and reasoning efforts:

| Role       | Model                   |
| ---------- | ----------------------- |
| Scout      | Claude Sonnet 5.5 xhigh |
| Customer   | Claude Sonnet 5.5 xhigh |
| Rechecker  | Claude Sonnet 5.5 high  |
| Aggregator | GPT 6.2 Sol xhigh       |

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh and knows only its prompt. Start every delegate with the prompt for its role from the Prompts section, and never with a brief the coordinator wrote.

## Run directory and roster

Files on disk are the source of truth, because a run is longer than one conversation can hold and its reader is another agent. Create one run directory inside the project, at `customer-test/<date and time of the start>/` under the workspace root, or in the place the user names. It holds:

- `brief.md`, the scout's account of the product
- `roster.md`, with one row per customer
- `diaries/`, with one file per customer
- `evidence/`, with one subdirectory of screenshots per customer
- `report.md`, the findings for the agent that will fix them

The roster starts with the facts of the run: the address, the date and time of the start, the focus, the lanes, and whether video is on. The facts also name the content that was in the system before the run, and any limit of an account that customers share, with how it was divided. When the workspace is a Git repository, the facts include the current commit and whether the working tree has uncommitted changes, so that the agent that fixes the findings can tell what changed since the run. Below the facts, the roster has one row per customer, like this:

| ID  | Persona                                           | Purpose  | Territory             | Device | Lane             | Wave | Status  | Notes |
| --- | ------------------------------------------------- | -------- | --------------------- | ------ | ---------------- | ---- | ------- | ----- |
| C01 | Owner of a small pet shop, rarely uses a computer | Newcomer | Home page and sign-up | Phone  | Incognito, alone | 1    | pending |       |

The statuses are pending, running, done, and blocked. A blocked status always has a reason in Notes. Re-read the roster before every wave and after any pause or summary of the conversation. If the user gives the path of an earlier run directory, resume from its roster instead of starting over, after checking that the system still answers.

The run writes nothing else in the project. It does not edit `.gitignore` or any other project file. The run directory stays in place and untracked, because it is the deliverable and the user decides what to commit.

## Workflow

The run asks the user only what the steps below name. After that it goes on without waiting.

### 1. Find the running system

Take the system's address from the request. If the request has none, read `PROJECT.md` if it exists, then the README, for the documented development address. Read them only for the address and for a development inbox, such as a local mail catcher. Read nothing else in the project.

Take two more things from the request, without asking for them. The focus is the part of the product the user wants used, with any address the request gives for it. Without one, the focus is the whole product. The offered session is a browser profile that the request says is already signed in for the run.

Check that the address answers with one request. If it does not answer, or no address can be found, ask the user to start the system or to give its address, and wait. Never start the system, never run the project's scripts, and never stop or restart a process.

The user starts and configures the system, so the run does not inspect its configuration or its database. Invoking this skill states that the running system holds test data that customers may create and change through the interface. Customers reach the data only through the interface.

Tell the user that the run can take hours and start dozens of delegates. Do not wait for a reply.

### 2. Scout

Start the scout with the Scout prompt. It visits the system as a first-time visitor, tests which tabs share a session, and writes `brief.md` with the contents its prompt lists. When the run has a focus, the scout spends most of its visit there. When the user offers a session, the scout also reads the signed-in areas in that profile and changes nothing there. The coordinator reads `brief.md` in full, because the roster comes from it.

The scout can only test tabs that it opened itself, so it leaves a marker in the local storage of each free profile. The coordinator then checks from its own session whether one delegate's browser state reaches another. For each free profile, it opens a background tab at the address with `preview_open`, runs `localStorage.getItem('customer-test-probe')` and then `localStorage.removeItem('customer-test-probe')` with `preview_evaluate`, and closes the tab. Sessions and lanes says what each result means. This is the only time the coordinator opens the system.

If the scout can't get in because sign-up needs something outside the browser, the coordinator asks the user once how customers should get in. The user can say where the confirmation message appears, or sign in to a browser profile themselves and name it. When the user signs in to a profile after the scout has returned, start the scout again with that profile as the offered session, so that the brief covers the signed-in areas before customers are sent there. Customers use a profile the user signed in with the session text Shared session, and no prompt ever carries a password. While waiting, run the customers that need no account. Without an answer, the run covers only what a visitor without an account can reach, and the report says so.

### 3. Build the roster

The coordinator derives the customers from `brief.md`. Each customer has:

- A persona, which is one of the kinds of people in the brief, with that person's familiarity with technology and patience
- One purpose from the Purposes section
- A territory, which is the part of the product that customer uses, named by what the screen calls it
- A device, desktop or phone width, and an appearance when the product offers more than one
- A lane, from Sessions and lanes

There is no cap on the number of customers. Size the roster by the product:

- One newcomer for each persona
- One customer for each job in the brief
- Each other purpose that the product gives something to act on, split across several customers when the product has more areas than one customer can use with care
- Purposes the product calls for that the section does not name

A customer with one purpose and a small territory writes a more careful diary than one that must cover everything, so prefer more customers to wider territories. Vary the personas and devices across the roster, so that the customers together resemble the people in the brief.

When the run has a focus, every territory lies inside it. Cover each side of the focus that the brief describes, such as the pages a reader sees and the screens where an owner manages them.

When customers share an account that has a limit, such as a number of projects in a plan, divide what is left of it before the first wave. Give the limited records to the customers whose purpose needs to create them, and send the others to a record that already exists. At most one customer reaches the limit on purpose, to see what the product says there. Each customer's part goes into its prompt as the `<LIMITS>` field.

Write the facts of the run at the top of `roster.md`. They include the content that was in the system before the run, from the brief and from what the request says about it, and how a limit was divided. Then write the rows, group the customers into waves by the lane rules, and tell the user the number of customers and waves. Do not wait for a reply.

### 4. Run the customers

Start the customers of a wave in parallel, each with the Customer prompt and the fields from its roster row. Each customer writes its diary and returns a short summary instead of the details. The coordinator updates the roster from the summaries and does not read the diaries in full. Reading them is the aggregator's work.

Managing the run:

- Update the roster after every wave, and send the user one line with the wave, the customers done, and the findings so far.
- There is no cap on how many customers run at once. Lanes limit a wave, and so does the load on the system. A development server slowed by many tabs produces false findings about speed, so a finding about slowness or a timeout counts only after the recheck in step 6.
- If a customer returns with part of its territory unused, add a row for one new customer with the same persona and purpose and the remaining territory. If that one also falls short, mark its row blocked with the reason.
- If a customer could not get past a point in the product, that is a finding. Do not run it again to get past it.
- If the system stops answering, stop starting customers, ask the user to check it, and wait. Never restart it.
- If a delegate reports signs that the system is live, stop the run and ask the user before going on.
- If a customer's reply shows that the profile's session was not what its prompt said, follow the end of Sessions and lanes.
- If delegates fail to start, or the provider limits them, make the waves smaller and go on.

### 5. Aggregate

Start the aggregator with the Aggregator prompt, with every diary as its sources and `report.md` as its output. It merges the findings of all diaries, groups the findings about the product into work packages, lists the findings about content apart from them, and returns the IDs of the findings that need a recheck.

With more than about twenty diaries, start several aggregators in parallel. Each takes the diaries of one group of territories as its sources and writes one file in `report-parts/`. One more aggregator then takes the part reports as its sources and writes `report.md`.

### 6. Recheck

The agent that fixes the findings acts on them without anyone reviewing them, so a serious finding that only one customer saw gets a second look. Start one rechecker with the Rechecker prompt for each of these findings, under the lane rules:

- Every blocker and major finding with the status seen once
- Every finding that the run itself may have caused, whatever its severity and status. These are findings about slowness or a timeout, a page that stopped answering or an action that had no effect, an unexpected sign-out, and data that changed or disappeared in an account that customers shared. Run each of these rechecks with no other delegate on the system.

Findings of the kind content are never rechecked, because the text of sample content always shows up again.

The coordinator updates the status of each rechecked finding in `report.md`. The new status replaces the old one, including the remark that the finding may come from the run:

- Reproduced: confirmed, with the rechecker's screenshot added to the evidence
- Not reproduced: not reproduced, with what the rechecker saw instead. The finding stays in the report.
- Unclear: unclear, with what the rechecker saw and how it repeated the step. A recheck that could not tell the product from the browser tool never confirms a finding, even when two customers reported it.
- Could not follow the steps: the status stays seen once, with the rechecker's reason added to the finding

### 7. Close and reply

Each delegate closes its tabs before it returns. The coordinator closes any tab the run left open, and deletes the delegates' temporary files outside the run directory. Leave the system running, because the run did not start it. Keep the run directory.

The final reply is short, because the user passes the report on instead of reading it. Include:

- The path of `report.md`, and the path and size of the run directory
- A table of the findings by kind and severity, with the number confirmed. Count the findings of the kind content apart, because no code changes for them.
- The number of customers by purpose, the delegate runs by role with model and effort, and the rows that ended blocked
- What no customer could use and why
- The accounts and records the customers left in the system, by identifier
- Whether the report has security findings. The run directory is inside the project, so committing it to a shared repository would publish them.
- The fix brief below, ready to paste and filled in for the first work package. After it, the IDs and titles of the other packages in order, and one line that says to send the brief once for each package, with the next ID.

The report is long, and an agent that reads all of it has little room left to work. So each work package is one fix request. The fix brief is this text, with `<RUN_DIR>` and `<PACKAGE>` replaced and nothing else changed:

```text
Fix work package <PACKAGE> of the customer-test report at <RUN_DIR>/report.md. In the run behind that report, delegates used the running system as its customers and wrote down what went wrong.

Read part 1 of the report in full. It is the note to the agent that fixes the findings. Then read only the findings of <PACKAGE>, from the heading that starts with "### <PACKAGE>" to the next heading of that level. Do not read the rest of the report. It is long, and each other package is a separate request. Open a diary entry or a screenshot only when a finding cites it and is unclear without it.

No finding names a cause, because nobody on the run read the code. Find the cause in the code before you change anything. Reproduce a finding first when its status is seen once, unclear, or not reproduced. A gap is a proposal, so say what you would build and do not build it without a decision. When you finish, list each finding of the package with what you changed, or why you changed nothing.
```

Never report a finding that no customer saw, and never state the cause of a problem. Nobody on the run saw the code.

## Sessions and lanes

Tabs in a browser can share cookies and local storage. Two customers who sign in to different accounts in tabs that share them sign each other out, and they also share a cart and saved preferences. A lane is browser state that tabs share. A lane holds one account at a time.

`brief.md` gives the scout's result for each browser profile, and the coordinator's check in step 2 adds whether state passes from one delegate to another:

- A profile that was already signed in holds a session the run did not create. The run leaves that profile alone, never uses its account, and never signs out of it. The only exception is a profile the user offers for the run. If no profile is free, ask the user to sign out of the system in one profile, or to say which signed-in profile the run may use.
- A profile the user offers is one lane for the whole run. The scout only reads in it. Every customer in it uses the session text Shared session, and nobody signs out of it. Purposes that need an account of their own, such as Newcomer and Snooper, can't run in it.
- If the scout's second tab was not signed in, or the coordinator did not find the scout's marker, every customer is its own lane in that profile. Every customer creates its own account, with the session text Own account, and the lanes do not limit a wave.
- If the scout's second tab was signed in and the coordinator found the marker, the profile is one lane for the whole run.

When a profile is one lane, the coordinator plans its waves like this:

- A customer needs the lane to itself when its purpose depends on how an account starts, or on data that other customers would change. This covers newcomers, jobs, returning customers, rule benders, and snoopers. It runs alone in the lane, with the session text Own account.
- Customers that judge screens, wording, and behavior that does not depend on the rest of the account share one account. Start one of them alone, with the session text First in a shared session, and wait for it to return. Then start the others at once in the same lane, with the session text Shared session.
- Where the product needs no account, customers who only read share a lane, with the session text No account. Customers who change saved state, such as a cart, take turns.

A customer's reply can show that the lanes differ from the plan:

- A customer with the text Own account says the profile was signed in when it arrived. If the plan had every customer as its own lane, state is shared after all. Treat the profile as one lane from then on, and run again the customers that were in it at the same time, because their sessions collided.
- A customer with the text Shared session says the profile was signed out. Start one customer of that group alone with the text First in a shared session, and then the rest.

## Purposes

A purpose is what one customer pays attention to while using the system for real. The list is a starting point and not a checklist. The coordinator keeps the purposes that the product gives something to act on, and adds others the product calls for. The text after each name goes into the Customer prompt unchanged, so a purpose the coordinator adds is written in the same form.

- Newcomer: You arrive knowing nothing about the product. Find out what it is and whether it is for you, sign up, and take the first steps a new customer takes.
- Job: Carry the job named in your territory from start to finish, the way this person would. Check that lists, totals, and later pages agree with what you did.
- Returning customer: Create an account and some data, then come back to it as a customer who returns. Sign out and in, recover your password, find your earlier data, edit and delete your own data, and change your account settings.
- Looks: Judge how each screen looks: alignment, spacing, contrast, images, text that is cut off or overflows, long values, empty lists, and whether the product looks trustworthy. Try each width and appearance the product offers.
- Consistency: Compare the same element across pages: buttons, forms, tables, dialogs, messages, icons, and terms. Look for the same action behaving differently in two places.
- Wording and formats: Read everything: labels, messages, spelling, mixed languages, and tone. Check the formats of dates, currency, numbers, and documents, and whether a value shown matches the value you typed.
- Phone: Do this person's jobs at phone width. Watch menus, tables, forms, dialogs, and controls that are hard to reach or fall off the screen.
- Hurried: You are in a hurry. Skim, click fast, submit twice, go back, reload in the middle of a flow, and open the same page in two tabs.
- Mistakes: You make mistakes. Fill fields wrong, leave them empty, and use wrong formats and boundary values. Judge whether each message says what to fix and whether what you typed survives the error.
- Keyboard and reading aids: Use the keyboard only, and zoom the page. Check tab order, visible focus, Enter, and Escape. You may read the accessible names in the accessibility tree.
- Features: Compare what the product promises with what it delivers. Look for missing, half-working, duplicated, and needless features, dead ends, and jobs that take more steps than they should.
- Under the hood: Visit every page of your territory and perform its main actions while reading the diagnostics of each snapshot. Look for console errors, failed requests, slow responses, and broken images. You may use `preview_evaluate` to read page state.
- Rule bender: Try to get more than the system should give. Skip a required step, pass a limit, reuse a one-time benefit, change a price or a quantity, and reach a paid feature without paying. You may edit addresses, and you may use `preview_evaluate` the way a person uses the browser's developer tools.
- Snooper: Try to see or change what is not yours. Open signed-in pages without signing in, change an identifier in an address to reach another account's records, enter script text in fields and look for where it is rendered back, and look for data the pages expose. You may edit addresses, and you may use `preview_evaluate` the way a person uses the browser's developer tools. Create two accounts for this and use only those, never an account that existed before.

## Prompts

Every delegate gets the prompt for its role word for word, so that every run briefs its delegates the same way. To build a prompt, the coordinator:

- Replaces each `<FIELD>` with its value from the Fields table
- Replaces each `[[Block: name]]` line with the full text of that block
- Changes, adds, removes, summarizes, and translates nothing else

What is particular to a run goes into the fields. If a prompt does not fit a case, send it unchanged and say in the final reply what did not fit.

The Browser block names T3 Code's preview tools. In another host, replace only that block with the instructions for the host's browser tools.

### Fields

| Field                   | Value                                                                                                                                                                                                                                                   |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `<ADDRESS>`             | The system's address from step 1                                                                                                                                                                                                                        |
| `<RUN_DIR>`             | The absolute path of the run directory                                                                                                                                                                                                                  |
| `<INBOX>`               | The address of the development inbox, or `none`                                                                                                                                                                                                         |
| `<FOCUS>`               | The part of the product the request names, in the user's words, with any address the request gives for it. Without one: `the whole product`                                                                                                             |
| `<OFFERED>`             | What the request says about a browser profile the user signed in for the run, in the user's words. Without one: `none`                                                                                                                                  |
| `<CONTENT>`             | The content that was in the system before the run, from the facts of the roster, by its titles and where it appears. Without any: `none`                                                                                                                |
| `<LIMITS>`              | This customer's part of a limit of the account it uses, from the roster, such as `The plan allows 3 projects and 1 exists. Create at most one project.` Without a limit: `none`                                                                         |
| `<PERSONA>`             | Two or three sentences in the second person, from the roster row: who this person is, what they do, how familiar they are with technology, and how patient they are                                                                                     |
| `<KNOWS>`               | At most three sentences with what this person knows before opening the product, without paths or steps. It may give one address this person was sent, such as the link to a page of the focus. For a newcomer: `Nothing. Someone gave you the address.` |
| `<PURPOSE>`             | The purpose's name                                                                                                                                                                                                                                      |
| `<PURPOSE_DESCRIPTION>` | The text after the purpose's name in the Purposes section                                                                                                                                                                                               |
| `<TERRITORY>`           | The part of the product this customer uses, named by what the screen calls it. For the purpose Job, the job itself.                                                                                                                                     |
| `<PROFILE>`             | The browser profile of the lane, by the name `preview_status` gives it                                                                                                                                                                                  |
| `<SESSION>`             | One of the session texts below                                                                                                                                                                                                                          |
| `<DEVICE>`              | `Desktop` or `Phone`, the width in px, and the appearance, such as `Desktop, 1280 px wide, light` or `Phone, 360 px wide, light`. The appearance is `light` unless the roster gives this customer `dark`.                                               |
| `<VIDEO>`               | One of the video texts below                                                                                                                                                                                                                            |
| `<DIARY_PATH>`          | `<RUN_DIR>/diaries/<ID>-<purpose>.md`, such as `diaries/C07-phone.md`                                                                                                                                                                                   |
| `<EVIDENCE_DIR>`        | `<RUN_DIR>/evidence/<ID>/` for a customer, and `<RUN_DIR>/evidence/recheck/` for a rechecker                                                                                                                                                            |
| `<FINDING_ID>`          | The ID of the finding to recheck                                                                                                                                                                                                                        |
| `<FINDING>`             | That finding, copied from `report.md` from its ID to its last field                                                                                                                                                                                     |
| `<SOURCES>`             | `every file in <RUN_DIR>/diaries/`, or a list of diary paths for a part report, or `the part reports in <RUN_DIR>/report-parts/`                                                                                                                        |
| `<OUTPUT>`              | `<RUN_DIR>/report.md`, or `<RUN_DIR>/report-parts/<number>.md` for a part report                                                                                                                                                                        |
| `<PACKAGE>`             | The ID of a work package, such as `P01`, in the fix brief of step 7                                                                                                                                                                                     |

The session texts:

- Own account: `Create your own account, the way this person would. If the profile is already signed in when you arrive, sign out first and say so in your reply. Sign out before you finish.`
- First in a shared session: `Create an account, the way this person would, and stay signed in when you finish. Other customers will use this session after you, so do not change the password or the email, and do not close the account. If the profile is already signed in when you arrive, sign out first and say so in your reply.`
- Shared session: `This browser profile is already signed in to an account that other customers are using at the same time. Use that session and do not sign up. Records you did not create will appear, and lists, totals, and counts change because of what the others do, so judge only what your own actions explain. Do not edit or delete records you did not create. Do not sign out, change the password or the email, or close the account. If you find the profile signed out, stop and return that.`
- No account: `Use the product without an account, and do not sign up.`

The video texts. Use the second one only when the user's request asked for video:

- `Do not record video.`
- `When screenshots can't show a problem, record a short clip with preview_recording_start and preview_recording_stop, and move the file into your evidence directory. A recording holds at most 50 MiB.`

### Scout prompt

```text
You are the scout of a customer-test run. The run finds out what a web system is like for the people who use it. After you, many delegates will each use the system as one of its customers, and the coordinator chooses those customers from what you write. Nobody on the run reads the source code, because a customer never sees it.

The system runs at <ADDRESS>. The user started it for this run, and it holds test data. Development inbox: <INBOX>.

Focus of the run: <FOCUS>

Session the user offers: <OFFERED>

Visit the system as someone who has never seen it. Follow the menus and links and read the pages. Submit nothing except what it takes to create one account and sign in. Do not carry out the product's jobs, because the customers do that. Then write what you learned in <RUN_DIR>/brief.md.

When the focus is a part of the product, spend most of your visit there. Open every section and page of that part, on each side of it that you can reach, such as the pages a reader sees and the screens where an owner manages them. Give the rest of the product a few lines.

Explore in the first free profile of the session test below, at a width of 1280 px and in the light appearance. If the user offers a session, also read the signed-in areas in the offered profile. Create, change, and delete nothing there.

[[Block: Browser]]

## Session test

Tabs can share cookies and local storage. Customers in tabs that share them would sign each other out, so the coordinator needs to know which tabs share a session. Do this test before you explore.

1. Call `preview_status` and note the browser profiles it lists.
2. For each profile, open the system in a new tab of that profile and see whether somebody is already signed in. A profile that is signed in holds a session this run did not create. Record it as taken, close your tab, and do not use that profile again. The exception is the session the user offers. Record that profile as offered, with the identifier of its account, and open a second tab in it to see whether its tabs share the session.
3. In the first free profile, create your account, the way a visitor would. Then open a second tab in the same profile and see whether it is signed in too.
4. In each other free profile, sign in to your account and repeat the check with a second tab.
5. If the product has no accounts, check instead whether two tabs of a profile share saved state, such as a cart.

Record for each profile whether it is free, taken, or offered, and whether its tabs share a session. If sign-up needs something outside the browser, such as a message in an inbox you do not have, a code on a phone, a captcha, or a card, record the exact step and what it asked for, skip the steps that need an account, and explore what a visitor without an account can reach.

## The brief

Write <RUN_DIR>/brief.md in the language of the product's interface, and quote the text on the screen exactly. Cover:

- What the product says it is and who it is for
- The kinds of people who would use it, and what each comes to do. Describe each in a few lines: who they are, what they want from the product, and how familiar with technology they are likely to be.
- The areas and features you reached, with the path of each, the name the screen gives it, the pages or sections inside it, and whether it needs an account
- The jobs a customer can carry from start to finish, such as finding a product, ordering it, and following the order
- How a customer gets in: the sign-up steps, the data they ask for, and any step that needs something outside the browser
- Actions with an effect outside the system, such as payments, tax documents, emails, and messages, and whether the screen shows a test mode
- The content that was already in the system when you arrived. Content is what the product lets an account holder enter, such as documents, products, or posts. It is data and not the product. List it by its titles and where it appears, and quote any text that shows it is sample or test data.
- The limits the product states for an account or a plan, such as a number of projects, with the exact text and where it appears. For an account that customers will share, also write how much of each limit is used.
- The widths, appearances, and languages the product offers
- The result of the session test, profile by profile
- The account you created, by identifier
- What you could not reach, and why

Report what the screen shows. Do not judge the product and do not list problems, because the customers do that.

[[Block: Ground rules]]

## When you finish

Sign out in every profile you signed in to yourself, and never in an offered profile. Then, as the last thing you do in each free profile, run `localStorage.setItem('customer-test-probe', '1')` with `preview_evaluate` in a tab that shows a page of the system, and leave the value there. The coordinator looks for it from another session, to learn whether delegates share browser state. This is the only use of `preview_evaluate` you may make. Close your tabs.

Return a short summary: the path of the brief, the product in two sentences, the number of kinds of people and of jobs, the result of the session test, the profiles where you left the marker, whether sign-up worked, the content and the limits you found, what you could not reach, and your tool notes.
```

### Customer prompt

```text
You are a customer of a web product, in a customer-test run. The run finds out what the product is like for the people who use it. You are one of many customers, and each has a different purpose. Use the product for real, the way the person below would, and write what you see, expect, do, and think as you go.

Another agent will read your diary and fix the problems you describe. It has not seen your screen and can't ask you anything, so it has only what you write. Nobody on the run reads the source code, because a customer never sees it.

## Who you are

<PERSONA>

What you know before you open the product: <KNOWS>

## What you came to do

Purpose: <PURPOSE>

<PURPOSE_DESCRIPTION>

Territory: <TERRITORY>

Stay in your territory and keep to your purpose, because other customers cover the rest. Use your territory with care instead of rushing through it.

You are done when you have used every part of your territory for your purpose, or given up on what you could not do. If you can't go on, finish the ending parts of the diary first and list what is left under Not reached.

## Your setup

- Address: <ADDRESS>
- Browser profile: <PROFILE>
- Account: <SESSION>
- Limits of the account: <LIMITS>
- Width and appearance: <DEVICE>
- Development inbox: <INBOX>
- Video: <VIDEO>
- Diary file: <DIARY_PATH>
- Evidence directory: <EVIDENCE_DIR>

Set the width and the appearance before you begin. Create no more than the limits of the account allow. If the product refuses an action because of a limit, write what the screen says and go on with a record that already exists.

[[Block: Browser]]

## How to be this customer

You know what this person would know before opening the product, and nothing more. Do not use what you know about how such systems are built to find a page or a feature. Reach everything through what the screen offers: menus, links, buttons, search, and help. Do not type an address that this person could not know, unless your purpose says you may.

Before each action, look at the page and write what you want and what you expect the action to do. Act. Look again and write what happened and what you make of it.

A customer has limited patience and nobody to ask. When something does not work or you can't find it, try what this person would try: read the page again, look in the menus, use the search or the help. Then give up on that goal, the way a customer would. Write where you stopped, what you tried, and what you would do in real life, such as leaving or calling support, and go on to your next goal. Do not work around the problem with a method this person does not have.

A step that needs something outside the browser ends the goal the same way. This covers an inbox other than the development inbox above, a phone, a captcha, and a real card.

## Product and content

Content that was in the system before the run: <CONTENT>

The product is what its makers built: the pages, the menus, the controls, the messages, and the way it shows, searches, and handles what is put into it. Content is what an account holder enters through the product, such as documents, products, or posts. The content named above is sample data that its owner can rewrite at any time, and so is anything you enter yourself.

Judge how the product shows and handles content, and not what the content says. A spelling slip, a missing description, a placeholder, or a contradiction inside the content is not a problem of the product. If this person would stumble over one, write it as a finding of the kind content. When you can't tell whether a text comes from the product or from the content, say so in the finding.

## The diary

Write the diary file as you go, in the first person and in the language of the product's interface. Quote the text on the screen exactly, because the agent that fixes a problem finds the code by searching for that text.

Start with who you are, what you came to do, your width and appearance, the identifier of the account you use, and the goals this person has in your territory. Then write one numbered entry for each step:

- Where: the path and the page title
- What I see
- What I want, and what I expect to happen
- What I do
- What happened
- What I think of it
- Evidence: the file name of the screenshot, if you saved one

Screenshots are the evidence, because the agent that reads the diary can open an image and can't watch a video. Save one when something is wrong, surprising, or worth showing. Take the snapshot with `save` set to true and move the file into your evidence directory, under a name that starts with the entry number, such as 012-checkout-error.png. If you can't move it, write the path the tool returned. To show something that moves, save one before and one after.

End the diary with these parts:

- Gave up: each goal you abandoned, with the entry number
- Findings: one block for each, in the format below
- Technical annex: the console errors, and the requests that failed or returned a status of 400 or more, from the diagnostics of your snapshots. Read the diagnostics at every step. Give each one line with the entry number and the exact text, and for a request its method, address, and status. Write a repeated error once, with a count. A customer does not see these, so keep them out of the entries. End the annex with Tool notes: one line for each failure of the browser tool, with the entry number. The browser section says what counts as one.
- Created: the accounts and records you left in the system, by identifier
- Not reached: the parts of your territory you did not use, and why

[[Block: Findings]]

[[Block: Ground rules]]

## When you finish

Close your tabs. Return a short summary and not the diary: the diary path, the number of findings by kind and severity, the goals you gave up on, and what you did not reach and why.
```

### Rechecker prompt

```text
You are rechecking one finding of a customer-test run. In the run, delegates used a web system as its customers and reported what went wrong. An agent will fix the reported problems, and nobody reviews them first, so a serious problem that one customer saw is checked a second time. Follow the steps below exactly and say whether the same thing happens. Do not explore beyond the steps, and do not look for other problems.

## Your setup

- Address: <ADDRESS>
- Browser profile: <PROFILE>
- Account: <SESSION>
- Development inbox: <INBOX>
- Evidence directory: <EVIDENCE_DIR>

## The finding

<FINDING>

[[Block: Browser]]

## What to do

1. Set the width and the appearance that the finding gives under Before.
2. Bring the system to the state under Before. Create the data it names, the way a customer would.
3. Follow the Steps one at a time, and look at the page image after each one. You may edit an address or use `preview_evaluate` only where a step says so.
4. Compare what you see with Actual.
5. Take a snapshot with `save` set to true and move the file into your evidence directory as <FINDING_ID>-recheck.png.
6. If the problem did not happen, follow the steps once more from the start. Then stop.
7. If the problem is a page or a tab that stopped answering, an action that had no effect, or keys that did nothing, the browser tool may be the cause. Repeat the failing step once in a new tab, with a locator in place of coordinates. If it works there, or if you can't tell the product from the tool, the result is unclear.

For a finding about slowness or a timeout, also write how long each step took.

[[Block: Ground rules]]

## When you finish

Close your tab. Return:

- Result: reproduced, not reproduced, unclear, or could not follow the steps
- What you saw at the last step, quoting the screen exactly
- For not reproduced, what happened instead, and at which step
- For unclear, what failed, how you repeated it, and what happened then
- For could not follow the steps, which step and why
- The file name of the screenshot
- Your tool notes, if the browser tool failed
```

### Aggregator prompt

```text
You are the aggregator of a customer-test run. In the run, many delegates used a web system as its customers, each with a different purpose, and each wrote a diary that ends with its findings. Turn them into one report.

The reader of the report is an agent that will fix the findings. It has not seen the run, it can't ask the customers anything, and nobody reviews the report before it acts. Nobody on the run read the source code, so no finding may name a cause.

## What to read and write

- The scout's account of the product: <RUN_DIR>/brief.md
- The facts of the run and the customers: <RUN_DIR>/roster.md. Its facts name the content that was in the system before the run, and its Lane column shows which customers shared an account.
- Sources: <SOURCES>
- The screenshots a finding depends on, under <RUN_DIR>/evidence/
- Write only: <OUTPUT>

Read every source in full. Do not open the system. Read nothing outside the run directory, run no Git command, and change no other file.

## How to merge

- Reports of the same behavior in the same place become one finding that keeps every customer who reported it. The same behavior in two places stays two findings in the same work package.
- Add no finding that no customer reported, and drop none. A report that is too vague to act on still goes in, with what is missing.
- Take each field from the diaries. Quote the screen text exactly as the diaries have it. When reports differ on a detail, give both. When another customer's diary shows the same steps working, say so in the finding.
- Set the kind and the severity by the definitions below. When a customer chose differently, keep the customer's choice in parentheses.
- A report about what the content says, and not about how the product shows or handles it, becomes a finding of the kind content, whatever kind the customer chose. The content is what the facts of the roster name, and what customers entered during the run. When a report covers both, split it into one finding about the product and one about the content. When a customer could not tell which of the two a text comes from, keep the finding with the product and state the doubt in it.
- A line of a technical annex that no finding uses becomes a defect of its own, once for each distinct error, with the steps of the diary entries that led to it. Its severity is minor unless a diary shows that it affected the customer. Lines under Tool notes never become findings.
- When the sources are part reports, their findings are already merged within each part. Merge across the parts and number the findings again.
- Write every path relative to the run directory, such as evidence/C07/012-checkout-error.png and diaries/C07-phone.md, so that the report still works when the directory moves. A diary names a screenshot by its file name, and the file is in that customer's evidence directory.

The statuses:

- Confirmed: two or more customers reported it independently, or a recheck reproduced it. Set it now when two or more customers reported the finding.
- Seen once: one customer reported it and no recheck has run. Set it now for every other finding.
- Unclear: a recheck saw it too and could not tell the product from the browser tool. The coordinator sets it after the rechecks.
- Not reproduced: a recheck followed the steps and the problem did not happen. The coordinator sets it after the rechecks.

If the run itself may have caused a finding, add `may come from the run` to its status. This covers slowness or a timeout, a page that stopped answering or an action that had no effect, an unexpected sign-out, and data that changed or disappeared in an account that customers shared.

[[Block: Findings]]

## The report

Write the report in the language of the diaries. It has these parts, in this order:

1. A note to the agent that will fix the findings. It says:
   - Each finding describes what customers saw on the screen. Nobody on the run read the code, so no finding names a cause. Find the cause in the code before changing anything.
   - Text in quotes is copied from the screen and can be searched for in the code.
   - What each kind, severity, and status means.
   - A gap is an opinion about what the product should offer. Nothing that exists is broken. Treat it as a proposal.
   - A finding of the kind content is about sample content and not about the code. These findings are listed apart, in part 5, and no code changes for them.
   - Each work package is one fix request. For one package, read this note and the findings of that package, and not the whole report.
   - A finding with the status seen once, unclear, or not reproduced needs to be reproduced before it is fixed.
   - To see a finding, bring the system to the state under Before and follow the Steps. The run kept no passwords, so sign up again where a finding needs an account.
   - The diary entry cited under Reported by has the full context of a finding. Open it when a finding is unclear.
   - The paths in the report are relative to the run directory, which holds the report.
   - The facts below say which commit the run saw. If the code changed since, check that a finding still happens before fixing it.
2. The run: the product in a few lines, the facts from the top of the roster, and the number of customers. Add the failures of the browser tool from the Tool notes of the diaries, with one line for each kind of failure and the customers that had it.
3. Work packages. Each groups the findings about the product that touch the same page or flow and is small enough for one fix request. Findings of the kind content are in no package. Give each package an ID such as P01, a title, its finding IDs, and its worst severity. Order the packages by worst severity, then by the number of customers who reported their findings.
4. Findings, ordered by package. Give each package a heading of the form `### P01 - title` and each finding a heading of the form `#### F001 - title`, so that a reader can find one package without reading the others. Each finding has an ID such as F001, the fields of the findings format, and:
   - Reported by: the customer IDs, with the diary path and entry number of each
   - Status
   - Done when: what a customer would see on the screen once it is fixed
5. Content findings: a table with one row for each finding of the kind content. The row has an ID that continues the numbering of part 4, the title, where it is, and the customers who reported it with their diary entries. Editing the content removes these findings, so they have no steps and no status.
6. Not covered: the areas, purposes, and actions that no customer used, and why. Take them from the brief, the blocked rows of the roster, and the Not reached parts of the diaries. An area that nobody used has no findings, and that says nothing about its quality.
7. Created: the accounts and records that the scout and the customers left in the system.

## When you finish

Return a short summary: the path you wrote, the number of findings by kind, severity, and status, the IDs of the blocker and major findings with the status seen once, the IDs of the findings that may come from the run, and any source you could not use and why.
```

### Block: Browser

```text
## The browser

Use the T3 Code preview tools and no other automation system. If a tool is not loaded yet, load it with the tool search and call it again.

- Call `preview_status` first. Then open your own tab with `preview_open`, with `reuseExistingTab` set to false and `profileId` set to the browser profile you are using. For a system on a local port, pass `target` with `kind` set to `environment-port` and the port. For any other address, pass `url`. Use the `tabId` it returns in every later call. Status can report unavailable only because no tab exists yet.
- `preview_snapshot` with `includeImage` set to true shows the part of the page that is on the screen, as a person sees it. It also returns the accessibility tree, with `aria-ref` locators, and the diagnostics of the page, which are its `consoleEntries` and `networkEntries`. With `save` set to true it writes the screenshot to a file and returns its path.
- `preview_act` clicks, types, presses keys, hovers, selects, uploads, and scrolls, on a locator from the last snapshot. Locators expire after navigation or a new snapshot, so take a new snapshot first.
- `preview_navigate` goes to an address in your tab, the way a person types one in the address bar.
- `preview_wait_for` waits for text, an element, or a URL. `preview_resize` sets the width. `preview_set_appearance` switches light and dark. `preview_dialog` answers a browser dialog.
- `preview_evaluate` runs JavaScript in the page. Use it only where this prompt says you may.
- Close every tab you opened with `t3_preview_close` before you return.

Judge each page by its image, because a customer sees the screen and not the markup. The image shows only what fits on the screen, so scroll and look again before you judge a whole page. The accessibility tree is for finding the element to act on. Something that appears only in the tree is not on the customer's screen.

The preview tools can fail without any fault of the product. A tab that is not visible can stop answering, a snapshot can fail with a message about screenshot frames, an action can time out, a click or a tap sent by coordinates can do nothing, and keys can fail to reach the page. When an action fails or has no effect, take a new snapshot and repeat it once with a locator. If the tab still does not answer, open the same address in a new tab and repeat the step there. If it works then, the tool failed and not the product. Write it down as a tool note and never as a problem of the product. It is a problem of the product only when it happens again in a new tab, with a locator, at the same step.

If the preview tools are not available, return and say so.
```

### Block: Ground rules

```text
## Ground rules

- Everything you know about the system comes from the browser. Never read the project's source files, configuration, or database, and never run its scripts.
- Write only in the paths this prompt gives you. Besides the browser, run commands only to create those directories and to move the screenshots and recordings you saved into them. Run no Git command.
- Type invented but realistic data, the kind this person would type. For email addresses use a reserved domain such as example.com, so that no real person receives a message. Use invented documents and phone numbers in a valid format. Never use a real person's data.
- Never write a password in a file, a screenshot, or your reply.
- Never complete an action with an effect outside the system, such as a real payment, a tax document, or an email or message to a real person. Go as far as the last step before it. When the screen shows a test mode, use the test values it documents.
- If you see signs that the system is live, such as a payment provider in live mode or records that look like real people's data, stop and return what you saw.
- Do everything by hand, in your tab. Send no requests from outside the browser, and run no scripts that repeat requests, no load or stress tests, no brute-force attempts, and no automated fuzzing.
- Never use or sign out of a session this run did not create, unless this prompt tells you to.
- Never delete in bulk, reset data, or import over existing data.
- To upload a file, create an invented one in a path this prompt gives you and upload that. Never upload a file that was already on the computer.
- If a link leads outside the product, such as to a payment provider or a social network, note where it went and come back. Do not use the other site.
- Never start, stop, or restart the system or any other process. If the system stops answering, wait a little and try once more, and if it still does not answer, stop and return that.
- Never install anything. Never deploy, publish, or push anything.
- If you are not sure an action is safe, skip it and write what you would have done and why.
```

### Block: Findings

```text
## Findings format

A finding is something a customer would complain about, or something a customer can't see that the technical annex shows. Write each one so that an agent that never saw the screen can see the same thing:

- Title, in one line
- Kind: defect, friction, inconsistency, gap, security, or content
- Severity: blocker, major, or minor
- Where: the path, the page title, and the element: what it is, such as a button or a field, and its visible label
- Before: the account state and the data it needs, the width, and the appearance
- Steps, numbered, starting from the address
- Expected, and what on the screen led the customer to expect it
- Actual, with the exact text of any message
- Evidence: the screenshot files and the annex lines
- In the customer's words: one or two sentences from the diary, with the entry number

The kinds:

- Defect: the system failed or did something wrong, such as an error, lost data, a broken layout, a dead link, or an error that only the annex shows.
- Friction: it works, but the experience is poor. The customer was confused, slowed down, misled, or put off by how it looks or reads.
- Inconsistency: the same thing looks or behaves differently in two places. Name both places.
- Gap: something the product promises or the job needs is missing, or something is there that gets in the way and serves nothing.
- Security: the customer saw or did something that the system should not allow.
- Content: the problem is in what an account holder entered through the product, such as the text of a document, a product, or a post, and not in how the product shows or handles it. Editing that content removes the problem, and no code changes. Its severity is always minor.

The severities:

- Blocker: the customer can't finish the job, loses data, or reaches what belongs to someone else.
- Major: the customer finishes with real difficulty, gets a wrong result, or would likely leave.
- Minor: the customer notices it and goes on.

Describe what happened and never guess its cause. Nobody on the run has seen the code.
```

## Delegate permissions

The prompts carry the delegates' permissions, and the coordinator grants nothing beyond them. The scout, the customers, and the recheckers work in the browser tabs they open and follow the Ground rules block. The scout also leaves one marker in the browser's local storage for the session check, and in a profile the user offers it only reads. They write only inside the run directory: the scout writes `brief.md`, each customer writes its diary and its evidence directory, and each rechecker writes its screenshot. The aggregator reads only the run directory and writes only its report file. No delegate reads the project's source files, starts or stops the system, or completes an action with an effect outside the system.

## Git restriction

The delegates run no Git command at all. The coordinator may run only these read-only Git commands:

- `git --no-pager --no-optional-locks status`
- `git --no-pager diff --no-ext-diff --no-textconv`
- `git --no-pager show --no-ext-diff --no-textconv`
- `git --no-pager log`

Arguments that only filter or format the output are allowed, such as paths, commit ranges, `--stat`, `--oneline`, `-p`, and `-n`. Never use `--output` or any option that writes files.

Never run any other Git command or version-control operation. This includes commit, add, checkout, switch, restore, reset, stash, merge, rebase, pull, push, fetch, branch, tag, and clean. This restriction applies to the coordinator and all delegates, including their tools and subprocesses.

## Scope

The run covers what a person who arrives at the address can reach, with the accounts a customer can create. Areas that need an account only the system's owner can create are listed as not covered, unless the user signs in to a profile for them. Pages hosted by third parties are out of scope. A customer notes that it left the product and comes back.

Nobody fixes anything, and nobody reads the code to explain a finding. The user decides how the findings are handled, for example by passing the fix brief of one work package at a time to squad, or to debug when a cause is unknown.

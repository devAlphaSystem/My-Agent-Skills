---
name: server-test
description: Test the server side of a web project in any language from a cold start in a new chat. Map its endpoints, data, background work, and external services from the code, start the system on a disposable database with local stand-ins for external services, and run contract, access, data, background, security, concurrency, resilience, and load passes through testers that call the server directly. It reports problems with their likely place in the code, never fixes them, and stops and deletes everything it started. For a customer's view of the system through the browser use customer-test. Use only when the user says 'server-test'.
---

# Test backend full

Use this skill to test what the server of a web project does, starting from a chat that knows nothing about the project. It runs only when the user asks for it. Never start it automatically. Testers call the server directly, without the pages, and look where the pages can't: the data that was written, background work, messages and calls to other services, many requests at once, failing dependencies, and load. The run starts the system on a disposable database that it creates, so nothing touches the database the project is configured to use.

The skill works for any language and framework. Every command comes from the project itself or from a tool the run downloads.

The user does not read the results. Another agent receives the report and fixes what it describes. Write every file for that agent. It has not seen the run, it can't ask the testers anything, and it acts on what the report says.

A run can take hours and start many delegates. Invoking this skill is the explicit request for that depth, and for load and security tests against the local system. The testers report problems and never fix them. For a customer's view of the system through the browser, use customer-test.

## Roles

The coordinator is the agent in the conversation. It talks to the user, prepares and stops the environment, owns every process the run starts, keeps the ledger, starts each delegate, and reports back. The coordinator does not test on its own and does not edit source files.

Use these exact models and reasoning efforts:

| Role       | Model                   |
| ---------- | ----------------------- |
| Mapper     | GPT 6.2 Sol xhigh       |
| Tester     | Claude Sonnet 5.5 xhigh |
| Rechecker  | Claude Sonnet 5.5 high  |
| Aggregator | GPT 6.2 Sol xhigh       |

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh and knows only its prompt. Start every delegate with the prompt for its role from the Prompts section, and never with a brief the coordinator wrote.

## Run directory and ledger

Files on disk are the source of truth, because a run is longer than one conversation can hold and its reader is another agent. Create one run directory inside the project, at `server-test/<date and time of the start>/` under the workspace root, or in the place the user names. It holds:

- `map.md`, the mapper's account of the server
- `environment.md`, how the coordinator started the system, with no secret values
- `ledger.md`, with one row per unit and its status in each pass
- `findings/`, with one file per tester run
- `evidence/`, with request and response captures, query results, log excerpts, and load results, in one subdirectory per tester run
- `logs/`, with the output of every process the run started
- `report.md`, the findings for the agent that will fix them

The run writes nothing else in the project. It does not edit `.gitignore`, `.env`, or any other project file. The run directory stays in place and untracked, because it is the deliverable and the user decides what to commit.

Everything else goes in one tools directory outside the project, in the scratchpad or the system temporary directory: downloaded binaries, the data of the disposable database, the stand-in servers, the mail catcher, and the testers' scripts and tools. The run deletes it at the end.

A unit is one endpoint (a method and a path), one inbound webhook, one background job, one scheduled task, or one outbound call or message. The ledger starts with the facts of the run: the start time, the stack, the database engine and version, the disposable database, the address, the stand-ins, and, when the workspace is a Git repository, the current commit and whether the working tree has uncommitted changes. Below the facts, it has one row per unit:

| ID  | Unit             | Roles         | Contract | Access  | Data    | Background | Security | Concurrency | Resilience | Load | Notes |
| --- | ---------------- | ------------- | -------- | ------- | ------- | ---------- | -------- | ----------- | ---------- | ---- | ----- |
| U01 | POST /api/orders | member, admin | pass     | pending | pending | n/a        | pending  | pending     | n/a        | n/a  |       |

The statuses are pending, pass, fail, blocked, skipped, and n/a. A pass is n/a when the Passes section does not apply it to that kind of unit. A blocked or skipped status always has a reason in Notes. Re-read the ledger before every batch and after any pause or summary of the conversation. If the user gives the path of an earlier run directory, resume from its ledger instead of starting over, after starting the environment again.

## Workflow

The run asks the user only what the steps below name. After that it goes on without waiting.

### 1. Understand the project and ask

Find the project to test. In a multi-project workspace, ask which one. Read `PROJECT.md` if it exists, as a hint only, and read the README, the project's instruction files, and its manifests and lockfiles. Tell the user that the run can take hours.

Ask the user once, in a single message, only what no file can answer:

- Features or areas to leave out
- Where to put the run directory, if not the default

The run creates its own test accounts on the disposable database, so it does not ask for any. Start the mapper while waiting for the answers.

### 2. Map

Start the mapper with the Mapper prompt. It reads the project and writes `map.md`. The coordinator reads `map.md` in full, because the environment and the ledger come from it.

### 3. Prepare the environment

Invoking this skill is the permission to create, migrate, seed, read, reset, and delete one disposable database that the run creates, and nothing more. The run never connects to the database the project is configured to use. It reads only the names of variables and configuration keys, never their values.

Create the disposable database of the engine and version the map names, in the first way that works:

1. The project's documented way to create a test database, when the result is disposable
2. A temporary file in the tools directory, for an embedded engine that runs inside the system
3. The engine's portable binary, in the version the project uses, downloaded from the engine's official source into the tools directory, with its data in the tools directory. Choose the build for the computer's operating system and processor, and check the published checksum when there is one.
4. A container, when a container runtime is already installed and running

If none of these works without installing something on the computer, such as a runtime, a container engine, or a database server, stop and ask the user. Say exactly what would be installed, from where, why, and how to remove it. Install it and go on if the user agrees, and stop the run if the user does not. Install nothing outside the tools directory without that answer.

Point the system to the disposable database through environment variables or command-line options when you start it. Never edit a project file to do it. If the system can only be pointed there by editing a file, or it always connects to the configured database, stop and ask the user.

Apply the project's own migrations or schema command to the disposable database, and run the project's seeds against it when it has them. Create two test accounts for each role, the way the project does it, such as through its sign-up endpoint, its admin command, or its seeds. Give them invented passwords, and keep the passwords only in the prompts. Never write them in a file.

Give each external service in the map a local stand-in:

- A stand-in is a small HTTP server that the coordinator writes in the tools directory. It records every request it receives in a file and answers by a mode that it reads from a file before each request: `ok`, `error`, `slow`, or `down`. It starts in `ok`.
- Email goes to a local mail catcher, downloaded as a portable binary, or to a stand-in that writes each message to a file.
- Point each service's address variable to its stand-in, and give its keys and secrets invented values. Secrets the system needs to start, such as a session or signing key, also get invented values. Never reuse a value from the project's configuration.
- When the system calls a service at an address that can't be changed, mark the units that call it skipped, and tell the testers in their units not to trigger them. The run never causes a payment, a message, an email, or any other effect outside the computer.

Start the system with the project's own start command. Identify the project's dependency tool from its manifest and lock files and do not mix tools. If dependencies are missing, install them with that tool and report the changes to the dependency and lock files. Removing packages, major version upgrades, global installs, and deleting dependency folders or lockfiles need the user's permission. Start the web process, and every worker and scheduler the map lists, on free ports bound to localhost only, in a way you can interrupt. Send the output of each process to `logs/`, record the process ids, and read the real address from the output. Do not use or stop a process you did not start.

Check that the system answers, and that it writes to the disposable database: create one record through the system and find it in the disposable database. If it is not there, stop everything and ask the user.

Write `environment.md` and `ledger.md`. Mark the units of the user's exclusions skipped. Tell the user the number of units and the stand-ins, and do not wait for a reply.

### 4. Baseline

Start one tester with the Tester prompt and the Baseline pass, with the whole project as its units. It runs the project's own build, type checks, linters, and test suites, and records each failure under a `baseline` line in the ledger.

### 5. Run the passes

Run the passes in the order of the Passes section, each over the units it applies to. Group the units into batches by area, with about five to eight units each, and keep units that share data together. Start each tester with the Tester prompt, the checks of its pass, and the entries of its units from `map.md`.

- Contract, Access, Data, Background, and Security: start several testers at once, one batch each. Each tester creates its own records, labeled with its tester ID, and changes only those.
- Concurrency, Resilience, and Load: one tester at a time, with no other delegate on the system, because their results depend on having the system to themselves.

Each tester writes its findings file and returns a short summary instead of the details. The coordinator updates the ledger from the summaries and does not read the findings files in full. Reading them is the aggregator's work.

Managing the run:

- Update the ledger after every batch, and send the user one line with the pass, the units done, and the findings so far.
- If a tester returns with units unfinished, put them in one new batch. If they are unfinished again, mark them blocked with the reason.
- Do not retest a unit because it failed. The finding goes to the report.
- If the system stops answering or a process exits, the tester reports it as a finding with the log lines. The coordinator checks the process and its log, and restarts what it started, at most twice in a row. After that, stop testing, mark the remaining units blocked, and go to step 6.
- If a pass leaves the disposable database unusable, recreate it as in step 3 and note it in the ledger.
- Before the rechecks, restart the system and recreate the disposable database, so that the load does not affect them.
- Do not end a pass early because few problems appeared, and do not add passes beyond these.

### 6. Aggregate

Start the aggregator with the Aggregator prompt, with every findings file as its sources and `report.md` as its output. It merges the findings, groups them into work packages, and returns the IDs of the findings that need a recheck.

With more than about thirty findings files, start several aggregators in parallel. Each takes the findings of one group of passes as its sources and writes one file in `report-parts/`. One more aggregator then takes the part reports as its sources and writes `report.md`.

### 7. Recheck

The agent that fixes the findings acts on them without anyone reviewing them, so a serious finding gets a second look. Start one rechecker with the Rechecker prompt for each of these findings, with no other delegate on the system:

- Every blocker and major finding with the status seen once
- Every finding that the environment may have caused, whatever its severity and status

The coordinator updates the status of each rechecked finding in `report.md`. The new status replaces the old one, including the remark that the finding may come from the environment:

- Reproduced: confirmed, with the rechecker's evidence added
- Not reproduced: not reproduced, with what the rechecker saw instead. The finding stays in the report.
- Unclear: unclear, with what the rechecker saw. A recheck that could not tell the system from the environment never confirms a finding.
- Could not follow the steps: the status stays seen once, with the rechecker's reason added to the finding

### 8. Stop and clean up

Stop everything the run started, with no exception: the system's processes, the workers and schedulers, the disposable database, the stand-ins, the mail catcher, and any container. Confirm that none of the recorded processes runs and that their ports are free. Do not stop processes you did not start.

Delete the tools directory, with the downloaded binaries and the data of the disposable database. Remove the containers and the images the run created. Leave in place anything the user agreed to install in step 3, and list it in the reply with how to remove it. Keep the run directory.

### 9. Reply

The final reply is short, because the user passes the report on instead of reading it. Include:

- The path of `report.md`, and the path and size of the run directory
- A table of the findings by kind and severity, with the number confirmed
- The ledger totals by pass: units passed, failed, blocked, skipped
- The delegate runs by role with model and effort
- What was not covered and why, such as units that call a service at a fixed address
- The load results in one line per level
- What the run started, downloaded, and installed, and that everything it started was stopped. Give the changes to dependency and lock files, if any.
- Whether the report has security findings. The run directory is inside the project, so committing it to a shared repository would publish them.
- The fix brief below, ready to paste and filled in for the first work package. After it, the IDs and titles of the other packages in order, and one line that says to send the brief once for each package, with the next ID.

The report is long, and an agent that reads all of it has little room left to work. So each work package is one fix request. The fix brief is this text, with `<RUN_DIR>` and `<PACKAGE>` replaced and nothing else changed:

```text
Fix work package <PACKAGE> of the server-test report at <RUN_DIR>/report.md. In the run behind that report, testers called the server of this project directly, on a disposable database, and wrote down what went wrong.

Read part 1 of the report in full. It is the note to the agent that fixes the findings. Then read only the findings of <PACKAGE>, from the heading that starts with "### <PACKAGE>" to the next heading of that level. Do not read the rest of the report. It is long, and each other package is a separate request. Open an evidence file only when a finding cites it and is unclear without it.

The place in the code that a finding gives is the tester's best reading, not a proven cause. Confirm the cause in the code before you change anything. Reproduce a finding first when its status is seen once, unclear, or not reproduced. A gap is a proposal, so say what you would build and do not build it without a decision. When you finish, list each finding of the package with what you changed, or why you changed nothing.
```

Never report a finding that no tester saw, and never report a unit as passed if no tester ran it.

## Passes

Each pass applies to the kinds of units named after its title, and its text goes into the Tester prompt unchanged as `<PASS_CHECKS>`.

- Baseline (the whole project): Run the project's build, type checks, linters in check-only mode, and test suites, for those the project has, with the commands from the map. Run a suite only when it uses no database, or when the variables in your setup point it to the disposable database. Skip a suite that would reach any other database, and write why. Record each failure with the command and the relevant output. Never change a test to make it pass.
- Contract (endpoints and inbound webhooks): Call each unit with the accounts its roles allow. Send valid requests, then each validation rule from the map at its boundary and just past it, missing, empty, null, and wrong-type fields, unknown fields, very long values, the wrong method, and the wrong content type. Compare the status code, the shape of the response, and the error messages with what the code says. Look for a status of 500, stack traces, internal paths, query text, and secrets in responses and in the logs. When a unit takes dates, money, numbers, documents, or phone numbers, use the project's own formats and try values where day and month could swap, cents, large values with thousands separators, zero, and month, year, and time zone boundaries.
- Access (endpoints and inbound webhooks): Call each unit with no session, with an expired and a tampered session, as each role, and as the second account of the same role. Check that each call is allowed or refused as the code intends. Change identifiers in the path, the query, and the body to reach the other account's records, in lists, reads, changes, and deletions. Send fields that the role may not set, such as an owner, a role, a price, a plan, or a status. When the product separates organizations or workspaces, check that one can't reach another. Check that signing out and changing a password end the sessions the code says they end.
- Data (endpoints that write): After each write, read the disposable database and compare what was stored with what was sent and with what the code does to it: every field, types, defaults, time zones, money units, and timestamps. Check what a rejected or failed request leaves behind, such as half-written records, orphaned rows or files, and counters that moved. Delete records and check what happens to the records and files that depend on them. Read the database only. Change it only through the system.
- Background (jobs, scheduled tasks, outbound calls and messages, inbound webhooks): Trigger each unit the way the system does, through the endpoint that starts it, the project's own command for a scheduled task, or by waiting its interval when it is short. Check that it runs, what it writes, and what it sends to the stand-ins and the mail catcher: recipients, subject, content, links, language, and payload. Trigger it twice and check that nothing is done or sent twice. For an inbound webhook, send a valid signature, a wrong one, none, the same event twice, and events out of order. Leave the modes of the stand-ins at `ok`, because the Resilience pass changes them.
- Security (endpoints and inbound webhooks): Send injection attempts to every input that the map says reaches a query, a command, a template, a file path, or an outbound address: SQL and NoSQL injection, command injection, template injection, path traversal, and server-side requests to the run's own stand-ins and to local addresses. Upload files with the wrong type, a disguised type, a large size, odd names, and archives, where the unit takes uploads. Fuzz each input with an automated fuzzer in the tools directory. Repeat sign-in, password reset, and code checks quickly to see whether rate limits and lockouts hold. Check response headers, cross-origin rules, cookie flags, and the protection of state-changing requests. Run the dependency audit command of the project's tooling in read-only form, if it has one.
- Concurrency (units the map marks with a race risk): Send the same request about twenty times at once, and two different requests on the same record at once. Afterwards, read the database and check limits, quotas, counters, balances, stock, unique values, and states. Repeat each case three times, because a race does not show every time.
- Resilience (units that call an external service or depend on a worker): Set the mode of each stand-in to `error`, then `slow`, then `down`, and use the units that depend on it in each mode. Check what the caller is told, that no request hangs without a time limit, that no data is left half-written, what the system retries and how often, and that nothing is sent twice. Set the mode back to `ok` and check that the system recovers without a restart. Always leave every stand-in at `ok` when you finish.
- Load (the busiest read routes, the main write route, search, sign-in, and any route the map marks as heavy): Use a load tool in the tools directory, aimed only at the system's local address. Run three levels, and wait for the system to settle between them. Light: about 10 simultaneous users for 2 minutes. Medium: about 50 for 5 minutes. Heavy: start at 50 users and add 50 every minute until the system breaks or a stop condition is met: errors above 5 percent of requests, a 95th percentile response time above ten times the light level, or 15 minutes. Stop at once if the computer has less than 10 percent of its memory free or the load tool can't keep its rate. For each level, record the requests per second, the 50th, 95th, and 99th percentile response times, the error rate and the kinds of errors, the memory and processor use of the system's processes, and the errors in the logs. After the heavy level, check that the system answers normally again within a minute at light load, and read the database for records written under load that are wrong or duplicated.

## Prompts

Every delegate gets the prompt for its role word for word, so that every run briefs its delegates the same way. To build a prompt, the coordinator:

- Replaces each `<FIELD>` with its value from the Fields table
- Replaces each `[[Block: name]]` line with the full text of that block
- Changes, adds, removes, summarizes, and translates nothing else

What is particular to a run goes into the fields. If a prompt does not fit a case, send it unchanged and say in the final reply what did not fit.

### Fields

| Field             | Value                                                                                                                                                                                                                                |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `<PROJECT_DIR>`   | The absolute path of the project                                                                                                                                                                                                     |
| `<RUN_DIR>`       | The absolute path of the run directory                                                                                                                                                                                               |
| `<TOOLS_DIR>`     | The absolute path of the tools directory, and for a tester its own subdirectory, such as `<TOOLS_DIR>/T07/`                                                                                                                          |
| `<HINTS>`         | What the user said about the project and the exclusions, in the user's words, or `none`                                                                                                                                              |
| `<ADDRESS>`       | The local address of the running system                                                                                                                                                                                              |
| `<PROCESSES>`     | The processes the run started, with their names, process ids, ports, and log files under `logs/`                                                                                                                                     |
| `<DATABASE>`      | The engine and version of the disposable database, and how to read it: the address or file and the command or tool to query it. No secret of the configured database.                                                                |
| `<STAND_INS>`     | For each external service: the name, the address of its stand-in, the file where it records requests, and the path of its mode file. The address of the mail catcher and where to read its messages. Or `none`.                      |
| `<ACCOUNTS>`      | The test accounts by role, as account A and account B of each role, with their identifiers and invented passwords                                                                                                                    |
| `<VARIABLES>`     | The environment variables that point the project's commands to the disposable database and the stand-ins, by name and value. Only invented values and local addresses.                                                               |
| `<ID>`            | The tester ID, such as `T07`, numbered in the order the testers start                                                                                                                                                                |
| `<PASS>`          | The name of the pass                                                                                                                                                                                                                 |
| `<PASS_CHECKS>`   | The text after the name of the pass in the Passes section                                                                                                                                                                            |
| `<UNITS>`         | The entries of the batch's units, copied from `map.md`, with the ledger ID of each. For Baseline: `the whole project`. For a unit marked skipped because it calls a service at a fixed address, the line `Do not trigger this unit.` |
| `<ALONE>`         | `Other testers are working on the system at the same time, with their own records.` or `You have the system to yourself.`                                                                                                            |
| `<FINDINGS_PATH>` | `<RUN_DIR>/findings/<ID>-<pass>.md`, such as `findings/T07-access.md`                                                                                                                                                                |
| `<EVIDENCE_DIR>`  | `<RUN_DIR>/evidence/<ID>/` for a tester, and `<RUN_DIR>/evidence/recheck/` for a rechecker                                                                                                                                           |
| `<FINDING_ID>`    | The ID of the finding to recheck                                                                                                                                                                                                     |
| `<FINDING>`       | That finding, copied from `report.md` from its ID to its last field                                                                                                                                                                  |
| `<SOURCES>`       | `every file in <RUN_DIR>/findings/`, or a list of findings files for a part report, or `the part reports in <RUN_DIR>/report-parts/`                                                                                                 |
| `<OUTPUT>`        | `<RUN_DIR>/report.md`, or `<RUN_DIR>/report-parts/<number>.md` for a part report                                                                                                                                                     |
| `<PACKAGE>`       | The ID of a work package, such as `P01`, in the fix brief of step 9                                                                                                                                                                  |

### Mapper prompt

```text
You are the mapper of a server-test run. The run tests the server side of a web project: testers will call the server directly on a disposable database, with local stand-ins for the services it calls, and look at the data, background work, security, concurrency, failures of dependencies, and load. Everything they test comes from your map, so the map must be complete and point to the code.

Project: <PROJECT_DIR>
Hints from the user: <HINTS>

Read the project and write <RUN_DIR>/map.md. Do not start the system, do not connect to any database, and send no requests to it.

## What the map holds

- Stack and commands: the languages, frameworks, and package or build tool, and the commands to install, build, start, test, migrate, and seed, with the file that defines each. The ports and the address the system listens on.
- Configuration: every environment variable and configuration key the server reads, by name only, with what it controls: the database, each external service, secrets, ports, flags, and limits. Say which can be set from the environment at start and which are read only from a file.
- Database: the engine and its version, where the version is fixed, how the project creates the schema (the migrations and the command), its seeds, and any documented way to create a test database. Whether the system can be pointed to another database without editing a file.
- Processes: the web process, workers, schedulers, and queues, how each starts, and what each needs.
- Roles and permissions: the roles, how a session is created and checked, how organizations or workspaces are separated, and the files that enforce each rule.
- Units: one entry for each endpoint, inbound webhook, background job, scheduled task, and outbound call or message. Give each entry:
  - The kind, and the method and path or the name
  - The handler, as file:line
  - The roles allowed
  - The inputs, with the validation rules and the file:line of each
  - What it reads and writes in the database, by table or collection
  - What it starts: jobs, messages, emails, outbound calls, files in storage
  - An effect outside the computer, such as a payment, an email, a message, or a call to another company's service, and whether its address can be changed from the configuration
  - A race risk: a limit, quota, counter, balance, stock, unique value, or state that two requests at once could break
  - Whether it is heavy, such as a search, an export, a report, or a large list
  - What it does with dates, money, numbers, documents, and phone numbers, and the project's formats for them
- External services: for each, its name, its purpose, the variable names of its address and its keys, and the units that call it.
- Email: how the system sends it and the variables that control it.
- Files and storage: where uploads go, their limits, and the units that touch them.
- Limits and business rules: plan limits, quotas, rate limits, and lockouts, with the file:line of each.
- Logs: where each process writes, and at which levels.
- Existing tests: the commands, and which database each suite uses.
- Not determined: anything the code alone could not answer, such as values that only exist in the deployed configuration.

[[Block: Code reading]]

## When you finish

Return a short summary: the path of the map, the stack, the database engine and version and how it can be pointed elsewhere, the number of units by kind, the external services and whether each address can be changed, the processes to start, and what you could not determine.
```

### Tester prompt

```text
You are a tester in a server-test run. The run tests the server side of a web project. The system is running on a disposable database that the run created, with local stand-ins for the services it calls, so nothing you do reaches real data or real services. Test your units with the checks of your pass, and write down every problem you find.

Another agent will read your findings and fix the problems. It has not seen your work and can't ask you anything, so it has only what you write.

## Your setup

- Tester ID: <ID>
- Project: <PROJECT_DIR>
- System: <ADDRESS>
- Processes and logs: <PROCESSES>
- Database: <DATABASE>
- Stand-ins and mail: <STAND_INS>
- Accounts: <ACCOUNTS>
- Variables for the project's commands: <VARIABLES>
- On the system: <ALONE>
- Map: <RUN_DIR>/map.md
- Findings file: <FINDINGS_PATH>
- Evidence directory: <EVIDENCE_DIR>
- Your tools directory: <TOOLS_DIR>

## Your pass

Pass: <PASS>

<PASS_CHECKS>

## Your units

<UNITS>

## How to work

Read the code of each unit before you test it, so that you know what it is meant to do. A problem is a difference between what the system does and what its code, its messages, or the rules it states mean it to do, or a behavior that loses data, exposes data, or breaks under use.

Label every record you create with your tester ID, such as in a name or a title, and change or delete only records you created. Use the accounts above, and create no others unless a check needs one.

When a check fails, run it once more from the start before you write it down, and write how many times you tried and how many times it failed.

Save the evidence of each problem in your evidence directory under a name that starts with the unit ID, such as U07-order-total.txt: the request and the response, the query and its result, the log lines, or the output of the tool. Replace every token, cookie, and password in them with a description such as <session of account A of member>.

Write the findings file as you go. Start it with your ID, your pass, and your units. Then write one block for each problem, in the format below. End it with:

- Passed: the units where every check passed
- Not tested: the units or checks you could not run, and why
- Created: the records you left in the database, by identifier
- Environment notes: anything that went wrong with the environment and not with the system, such as a stand-in that did not answer or a tool that failed

[[Block: Findings]]

[[Block: Code reading]]

[[Block: Ground rules]]

## When you finish

Stop every tool you started and leave every stand-in at `ok`. Return a short summary and not the findings: the findings path, a verdict for each unit (pass, fail, blocked, or skipped), the number of findings by kind and severity, and what you could not test and why.
```

### Rechecker prompt

```text
You are rechecking one finding of a server-test run. In the run, testers called the server of a web project directly, on a disposable database, and reported what went wrong. An agent will fix the reported problems, and nobody reviews them first, so a serious finding is checked a second time. Follow the steps below exactly and say whether the same thing happens. Do not look for other problems.

## Your setup

- Project: <PROJECT_DIR>
- System: <ADDRESS>
- Processes and logs: <PROCESSES>
- Database: <DATABASE>
- Stand-ins and mail: <STAND_INS>
- Accounts: <ACCOUNTS>
- Variables for the project's commands: <VARIABLES>
- On the system: You have the system to yourself.
- Evidence directory: <EVIDENCE_DIR>
- Your tools directory: <TOOLS_DIR>

## The finding

<FINDING>

## What to do

1. Bring the system to the state under Before. Create the data it names through the system.
2. Follow the Steps one at a time, and compare what happens with Actual.
3. Save the evidence in your evidence directory under names that start with <FINDING_ID>-recheck.
4. If the problem did not happen, follow the steps once more from the start. Then stop.
5. If the problem may come from the environment, such as a stand-in, the load of another test, or the limits of the computer, look at the logs and the processes for a sign of it. If you can't tell the system from the environment, the result is unclear.

For a finding about speed, load, or a race, also write the numbers of each try.

[[Block: Ground rules]]

## When you finish

Leave every stand-in at `ok`. Return:

- Result: reproduced, not reproduced, unclear, or could not follow the steps
- What you saw at the last step, with the response, the query result, or the log line
- For not reproduced, what happened instead, and at which step
- For unclear, what you saw and why you can't tell the system from the environment
- For could not follow the steps, which step and why
- The names of the evidence files
```

### Aggregator prompt

```text
You are the aggregator of a server-test run. In the run, testers called the server of a web project directly, on a disposable database, each with one pass over a group of units, and each wrote a findings file. Turn them into one report.

The reader of the report is an agent that will fix the findings. It has not seen the run, it can't ask the testers anything, and nobody reviews the report before it acts.

## What to read and write

- The map of the server: <RUN_DIR>/map.md
- The facts of the run and the units: <RUN_DIR>/ledger.md and <RUN_DIR>/environment.md
- Sources: <SOURCES>
- The evidence a finding depends on, under <RUN_DIR>/evidence/
- Write only: <OUTPUT>

Read every source in full. Do not call the system, and do not connect to the database. You may read the project's code to check the place a finding gives, under the rules below, and change no other file.

[[Block: Code reading]]

## How to merge

- Reports of the same behavior in the same unit become one finding that keeps every tester who reported it. The same behavior in two units stays two findings in the same work package, unless the code shows one place causes both. Then merge them and name every unit.
- Add no finding that no tester reported, and drop none. A report that is too vague to act on still goes in, with what is missing.
- Take each field from the findings files. When reports differ on a detail, give both.
- Set the kind and the severity by the definitions below. When a tester chose differently, keep the tester's choice in parentheses.
- Lines under Environment notes never become findings. Summarize them in part 2.
- When the sources are part reports, their findings are already merged within each part. Merge across the parts and number the findings again.
- Write every path relative to the run directory, such as evidence/T07/U07-order-total.txt and findings/T07-access.md, and every place in the code relative to the project, as file:line.

The statuses:

- Confirmed: it happened every time it was tried, at least twice, or two testers reported it, or a recheck reproduced it. Set it now when the findings file shows two failing tries or two testers.
- Seen once: it happened once, and the second try passed or was not made. Set it now for every other finding.
- Unclear: a recheck saw it too and could not tell the system from the environment. The coordinator sets it after the rechecks.
- Not reproduced: a recheck followed the steps and the problem did not happen. The coordinator sets it after the rechecks.

If the environment may have caused a finding, add `may come from the environment` to its status. This covers a stand-in, the records or load of another tester, the limits of the computer, and a difference between the disposable database and the engine or settings the project uses in production.

[[Block: Findings]]

## The report

Write the report in the language of the findings files. It has these parts, in this order:

1. A note to the agent that will fix the findings. It says:
   - The testers read the code and give the likely place of each problem as file:line. That is their best reading and not a proven cause. Confirm the cause in the code before changing anything.
   - What each kind, severity, and status means.
   - A gap is an opinion about what the system should do. Nothing that exists is broken. Treat it as a proposal.
   - Each work package is one fix request. For one package, read this note and the findings of that package, and not the whole report.
   - A finding with the status seen once, unclear, or not reproduced needs to be reproduced before it is fixed.
   - To see a finding, start the system on a disposable database as `environment.md` describes, bring it to the state under Before, and follow the Steps. The run kept no passwords, so create the accounts again.
   - The findings file cited under Reported by has the full context of a finding.
   - The paths in the report are relative to the run directory, and the places in the code relative to the project.
   - The facts below say which commit the run saw. If the code changed since, check that a finding still happens before fixing it.
2. The run: the system in a few lines, the facts from the top of the ledger, the ledger totals by pass, the load results by level, and a summary of the environment notes.
3. Work packages. Each groups the findings that touch the same unit, the same part of the code, or the same flow, and is small enough for one fix request. Give each an ID such as P01, a title, its finding IDs, and its worst severity. Order the packages by worst severity, then by the number of findings.
4. Findings, ordered by package. Give each package a heading of the form `### P01 - title` and each finding a heading of the form `#### F001 - title`, so that a reader can find one package without reading the others. Each finding has the fields of the findings format, and:
   - Reported by: the tester IDs, with the findings file of each
   - Status
   - Done when: what the request, the database, or the logs show once it is fixed
5. Not covered: the units marked blocked or skipped and why, the checks the testers could not run and why, and what the environment could not reproduce, such as a service at a fixed address. A unit nobody tested has no findings, and that says nothing about its quality.
6. Created: the records the testers left in the disposable database. The database was deleted at the end of the run, so this is for reading the evidence.

## When you finish

Return a short summary: the path you wrote, the number of findings by kind, severity, and status, the IDs of the blocker and major findings with the status seen once, the IDs of the findings that may come from the environment, and any source you could not use and why.
```

### Block: Code reading

```text
## Reading the project

- Read any source file, migration, schema, test, and documentation file of the project.
- Never read the values in `.env` files, secret files, or credentials. To learn which variables a file defines, read only the names, with a command that prints the text before each `=` and nothing after it.
- Never edit a project file.
- You may run these read-only Git commands, with arguments that only filter or format the output: `git --no-pager --no-optional-locks status`, `git --no-pager diff --no-ext-diff --no-textconv`, `git --no-pager show --no-ext-diff --no-textconv`, and `git --no-pager log`. Run no other Git command.
```

### Block: Ground rules

```text
## Ground rules

- The only database you may connect to is the disposable one in your setup. Never connect to the database the project is configured to use, and never use a connection string from the project's configuration.
- Read the disposable database freely. Change it only through the system.
- Send requests only to the system's local address and to the run's stand-ins and mail catcher. Never send a request to an outside service, except to download a tool into your tools directory from its official source.
- Keep your scripts, tools, and downloads in your tools directory. Install nothing anywhere else, and install no project dependency.
- Write only in your findings file, your evidence directory, and your tools directory.
- Never start, stop, or restart the system, its workers, the database, the stand-ins, or the mail catcher. You may run tools that end on their own, such as a load tool or a fuzzer, and stop them before you return.
- Load, fuzzing, and repeated attempts are allowed only against the system's local address, and only within your pass.
- Never write a password, a token, a cookie, or a secret in a file or in your reply.
- Change or delete only records you created. Never delete in bulk, reset data, or import over existing data.
- If something shows that the system reaches real data or a real service, such as a request to an outside address in the logs or a record that looks like a real person's data, stop and return what you saw.
- Never deploy, publish, or push anything.
- If you are not sure an action is safe, skip it and write what you would have done and why.
```

### Block: Findings

```text
## Findings format

Write each finding so that an agent that never saw your work can see the same thing:

- Title, in one line
- Kind: defect, security, data, performance, resilience, or gap
- Severity: blocker, major, or minor
- Unit: the ledger ID and the method and path or the name
- Where in the code: the likely place, as file:line, and one sentence on why you think so. Write `not found` when you could not find it.
- Before: the account by role and letter, such as account A of member, the data it needs, and the mode of each stand-in involved
- Steps, numbered: each request with its method, path, the headers that matter, and the body, or each command. Replace tokens and passwords with a description.
- Expected, and what in the code or in the system's messages led to expect it
- Actual: the status, the response, the database state, or the log line, quoted exactly
- Tries: how many times you ran it and how many times it failed
- Evidence: the files in the evidence directory

The kinds:

- Defect: the system fails or returns a wrong result, such as an error status, a crash, a wrong response, or a validation that does not hold.
- Security: someone can see, change, or do what the system should not allow, or the system exposes secrets, internals, or other people's data.
- Data: what the database holds after an action is wrong, incomplete, duplicated, or orphaned.
- Performance: the system is slow, uses too much memory or processor, or fails under load.
- Resilience: the system handles the failure of a dependency badly, such as hanging, losing data, or sending something twice.
- Gap: something the system needs is missing, such as a limit, a time limit, a check, or a log line. Nothing that exists is broken.

The severities:

- Blocker: data is lost or corrupted, one account reaches another's data, the system crashes or stops answering, or a security problem can be used without an account.
- Major: a wrong result, bad data let in, a security problem that needs an account, a failure under the medium load level or below, or a dependency failure that leaves data half-written.
- Minor: everything else.
```

## Delegate permissions

The prompts carry the delegates' permissions, and the coordinator grants nothing beyond them. The mapper reads the project and writes only `map.md`. The testers and the rechecker read the project, call the system the coordinator started, read the disposable database, and write only their findings file, their evidence directory, and their tools directory. The aggregator reads the run directory and the project and writes only its report file. No delegate edits a project file, connects to any database but the disposable one, starts or stops a long-running process, installs anything outside its tools directory, or causes an effect outside the computer.

## Git restriction

The coordinator and all delegates may run only these read-only Git commands:

- `git --no-pager --no-optional-locks status`
- `git --no-pager diff --no-ext-diff --no-textconv`
- `git --no-pager show --no-ext-diff --no-textconv`
- `git --no-pager log`

Arguments that only filter or format the output are allowed, such as paths, commit ranges, `--stat`, `--oneline`, `-p`, and `-n`. Never use `--output` or any option that writes files.

Never run any other Git command or version-control operation. This includes commit, add, checkout, switch, restore, reset, stash, merge, rebase, pull, push, fetch, branch, tag, and clean. This restriction applies to the coordinator and all delegates, including their tools and subprocesses.

## Scope

The run covers the server side of the project that the map lists, minus the user's exclusions: endpoints, inbound webhooks, background jobs, scheduled tasks, and outbound calls and messages, as seen from outside the process and in the disposable database. Pages and how they look are out of scope, and so are other companies' services, which the stand-ins replace. Load, fuzzing, and repeated attempts run only against the local system the run started.

Nobody fixes anything. The user decides how the findings are handled, for example by passing the fix brief of one work package at a time to squad, or to debug when a cause is unclear.

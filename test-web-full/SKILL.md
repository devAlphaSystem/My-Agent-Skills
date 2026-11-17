---
name: test-web-full
description: Test an entire web project from a cold start in a new chat. Map its routes, pages, forms, roles, and APIs, check the map against the running app, run functional, presentation, and access passes over every mapped unit through testers, and report coverage and problems. It can run for hours, reports problems and never fixes them, and stops everything it started. For one change or flow use test-web-app. Use only when the user says 'test-web-full'.
---

# Test web full

Use this skill to test a whole web project, starting from a chat that knows nothing about it. It runs only when the user asks for it. Never start it automatically. It is exhaustive by design. Delegates map the project, the coordinator checks the map against the running app, and testers go through every mapped page, form, and endpoint in three passes. A run can take hours and start many delegates. Invoking this skill is the explicit request for that depth. The testers report problems and never fix them. For one change or flow, use test-web-app. For a loop that also fixes problems and retests, use qa.

## Roles

The coordinator is the agent in the conversation. It talks to the user, prepares and stops the environment, keeps the ledger, briefs each delegate, filters their findings, and reports back. The coordinator does not map or test on its own and does not edit source files. It is the only agent that starts or stops the app.

Use these exact models and reasoning efforts:

| Role               | Model                   |
| ------------------ | ----------------------- |
| Mapper             | GPT 6.2 Sol xhigh       |
| Verifier           | Claude Sonnet 5.5 high  |
| Automated tester   | GPT 6.2 Sol xhigh       |
| Exploratory tester | Claude Sonnet 5.5 xhigh |

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh, so every brief must include all the context it needs.

## Run directory and ledger

A run is longer than one conversation can hold, so files on disk are the source of truth. Create one run directory outside the project's source tree, in the place the user names or else in the scratchpad or the system temporary directory. It holds:

- `map.md`, with one entry per unit
- `ledger.md`, with one row per unit and its status in each pass
- `findings/`, with one file per delegate run
- `evidence/`, with screenshots and logs

A unit is one page, form, endpoint, error page, or journey. The ledger starts with a line for the status of the baseline run, then has one row per unit, like this:

| ID  | Unit       | Roles     | Pass 1 | Pass 2  | Pass 3  | Notes |
| --- | ---------- | --------- | ------ | ------- | ------- | ----- |
| U01 | GET /login | anonymous | pass   | pending | pending |       |

The statuses are pending, pass, fail, blocked, and skipped. A blocked or skipped status always has a reason in Notes. Re-read the ledger before every batch and after any pause or summary of the conversation. If the user gives the path of an earlier run directory, resume from its ledger instead of starting over, after checking that the environment is ready.

## Workflow

### 1. Understand the project and ask

Find the project to test. In a multi-project workspace, ask which one. Read `PROJECT.md` if it exists, as a hint only, and read the README, the package scripts, the configuration, and the project's instruction files. Tell the user that the run can take hours.

Ask the user once, in a single message, only what no file can answer. After that, the run goes on without waiting, unless a step says to stop.

- The permission described in step 3, if the database is real or shared
- A test account for each role, or where the project documents them
- Features or areas to leave out
- Where to put the run directory, if not the default

If the coordinator can't tell how to start the app, it asks that too. The mapper does not need the answers, so start it while waiting.

### 2. Map

Brief the mapper with the workspace path, the project, and any hints. The mapper reads files and runs read-only commands. It does not edit files. It writes `map.md` in the run directory and returns a summary with:

- The stack, how to build and start the app, the ports, and the variable names, by name only
- The existing test commands and which database they use
- The roles and the permission model, with the files that enforce it
- Every unit, with its URL pattern and method, the handler and template files, the roles allowed, the inputs with their validation rules from the code, the states it can show, the data it needs, and how to reach it from the menus
- Units to cover: pages, forms, API endpoints, login, logout, registration, password reset, error pages, uploads, downloads, and exports
- Units that send emails, take payments, issue tax documents, call third-party services, or delete or overwrite data in bulk, each marked with that effect
- Journeys that cross several units, such as sign up, create, edit, and report
- Dead code, such as routes without templates, and anything it could not determine from the code alone, such as client-side routing, dynamic menus, and feature flags

### 3. Prepare the environment

Wait for the user's answers. Run the app in a development or test configuration. Never use production or real customer data.

Check which database the configuration points to, without printing credentials. A disposable database needs no question, such as an in-memory database, a temporary file, or a container the test creates. Prefer one when the project documents a way to create it. If the configuration points to a real or shared database, stop and ask the user for permission before starting the app. The request explains:

- Which database and environment, and exactly what will run
- Which tables or records it affects, and roughly how many, since a full run creates and edits many records through the app
- Why it is needed, and whether an alternative avoids the database
- Whether it can be reversed, and the expected result

The permission covers only that action. Migrations, seeds, resets, and direct writes need the same permission. Test suites that reach such a database need it too. Delegates reach the database only through the app.

Use sandbox or test modes, or local stubs, for external services. Never trigger real payments, tax documents, emails, messages, or state-changing calls to production APIs. Log in with the test accounts only. Never use a real account. In T3 Code, request a credential that only the user can give with `request_secret`.

Start the app with the project's own script. Identify the package manager by its lockfile and do not mix managers. Start your own server on a free port, bound to localhost only, in a way you can interrupt. Record its process id, and read the real port and URL from its output. Do not use or stop a server you did not start. If the project fixes the port and another service already uses it, ask the user. If dependencies are missing, install them with the project's package manager and report the lockfile changes. Removing packages, major version upgrades, global installs, and deleting `node_modules` or lockfiles need the user's permission.

### 4. Verify the map

Brief the verifier with `map.md`, the app URL, and the test accounts. The verifier checks the map against the running app, signed in as each role in turn. It follows menus and links from the entry page, visits pages with GET requests, and submits nothing except the login form. For each unit it records the navigation path and whether it was reachable. It lists pages and endpoints it reached that the map lacks, and mapped units it could not reach. It writes its corrections in its findings file and returns a summary.

The coordinator merges the corrections into `map.md` and builds `ledger.md`. If the verifier finds major gaps, the coordinator sends them back to the mapper for one more round. Units that the map marks with an external effect or a bulk destructive action get the status skipped, with the reason. The user's exclusions get the same status. The coordinator tells the user the number of units and batches, and does not wait for a reply.

### 5. Run the passes

Group the units into batches by area, such as one module or one set of related pages, with about five to eight units each, and keep units that share data together. Run batches one after another. Within a batch, run both testers in parallel when the pass uses both.

Each brief to a tester includes:

- The unit entries from `map.md` for that batch, and the checks of the pass
- The app URL, the accounts to use by name, and the test data that exists. Tell them never to write credentials in a report, a screenshot, or a log.
- The data rules. Each tester creates its own records, labeled with the run, its role, and the pass, and lists them in its findings file. It changes or deletes only records it created.
- The path of its findings file and the evidence directory
- The browser instructions below, for the exploratory tester
- The delegate permissions, the Git restriction, and the scope from this skill

Each tester writes its details to its findings file, and returns a short summary instead of the details: a verdict for each unit, the number of problems by severity, the findings file path, and what it could not test and why. A problem entry has the unit ID, steps to reproduce, expected and actual results, severity (blocker, major, or minor), and the evidence path. The coordinator updates the ledger from the summaries and reads the findings files in full only when it writes the report.

Browser instructions for the exploratory tester and the verifier. In T3 Code, use the preview tools:

- Call `preview_status`. Open your own tab with `preview_open`, with `reuseExistingTab` set to false and the app port as `target`, and use the returned `tabId` from then on. Status can report unavailable only because no tab exists yet.
- `preview_snapshot` reads the page and returns `aria-ref` locators for `preview_act`, which clicks, types, presses keys, selects, uploads, and scrolls. Locators expire after navigation or a new snapshot.
- `preview_wait_for` waits for text, an element, or a URL. `preview_resize` sets the viewport. `preview_set_appearance` switches light and dark. `preview_evaluate` runs JavaScript in the page for state that a snapshot can't show.
- Do not install or switch to another automation system. If no browser tool is available, report that and list what stayed unverified. Close the tab at the end with `t3_preview_close`.

#### Baseline

Start the automated tester once on the whole project. It runs the existing build, type checks, linters in check-only mode, and test suites, within the database rule above. It records each failure under the `baseline` row.

#### Pass 1, functional

Both testers, batch by batch. The automated tester works without a browser and sends HTTP requests to the app with the test accounts, for each unit in the batch:

- Valid requests, and each validation rule from the map at its boundary, with missing and extra fields
- The wrong method, no session, and the wrong role
- A few hand-written hostile strings, and whether output is escaped
- Status codes, error messages, and responses that expose secrets or stack traces
- Extra tests for the batch in a temporary directory outside the source tree, when that is practical

The exploratory tester uses each unit in the browser the way a user would:

- The main flow from start to finish, as each role that may use it
- Invalid input, empty fields, boundary values, and the message shown for each
- Loading, empty, and error states, and a repeated submit
- Reload and back navigation after the action, to confirm that saved data persists
- Console errors and failed requests during every step

Both testers apply the locale rule. When a unit involves dates, currency, numbers, documents, phone numbers, or addresses, compare what the page shows with what it sends, using the project's own formats. Try values where day and month could swap, cents, large values with thousands separators, zero, an empty field, an invalid value, month and year boundaries, and the resulting sort order and filters.

#### Pass 2, presentation

The exploratory tester only, batch by batch. For each unit:

- Narrow (about 360 px), tablet, and desktop widths, and dark appearance when the project supports it
- Long values, many rows, and empty lists, and text that overflows or is cut off
- Keyboard use, covering tab order, visible focus, Enter, and Escape, and the accessible names in the snapshot
- Console errors during every step

#### Pass 3, access and journeys

Both testers, in groups chosen by role and by journey instead of by area:

- The automated tester sends direct requests for every role and every unit, including no session at all, and checks that each is allowed or denied as the map says. With two accounts of the same role, it checks that one can't read or change the other's records by changing an identifier.
- The automated tester checks the app as a whole, covering logout and expired sessions, cookie flags, protection of state-changing requests, response headers, error pages that leak internals, and internal links that return errors, using GET requests only.
- The exploratory tester checks that each role sees only the controls it may use, and that denied pages show a clear message.
- The exploratory tester runs each journey from the map from start to finish with fresh test data, and checks that lists, totals, and reports agree with what the steps did.

#### Managing the run

- Update the ledger after every batch, and send the user one line with the batch, the units done, and the problems found so far.
- If a tester returns with units unfinished, put those units in one new batch. If they are unfinished again, mark them blocked with the reason.
- Do not retest a unit because it failed. The problem goes to the report.
- If a unit can't be tested because of an earlier failure, such as a login that does not work, mark it blocked with the reason.
- If the app stops responding, check its process and logs, and restart the app you started at most twice in a row. After that, stop testing, mark the remaining units not tested, and go to step 6.
- Do not end a pass early because few problems appeared, and do not add passes beyond these.

### 6. Stop the environment

The coordinator stops everything the run started, with no exception. This includes the app, watchers, and background processes, using the recorded process ids. Confirm that no tab or process from the run remains. Do not leave anything running, even when the user may want to inspect the app. Do not stop processes you did not start.

Delete the testers' temporary files and scratch copies. Keep the run directory, because it is the deliverable, and report its path. Delete `evidence/` only if the user said not to keep it.

### 7. Report

The coordinator writes `report.md` in the run directory with every problem, and sends the user a final reply with the results:

| Pass     | Units | Pass | Fail | Blocked | Skipped |
| -------- | ----- | ---- | ---- | ------- | ------- |
| Baseline | 1     | 0    | 1    | 0       | 0       |
| Pass 1   | 42    | 35   | 5    | 1       | 1       |

After the table, list:

- Every blocker and major problem, with the unit, steps to reproduce, expected and actual results, and evidence path. Give the number of minor problems, with the path of `report.md`.
- The units not tested, with the reason for each
- The delegate runs by role, with the model and effort
- The records the testers created, and the labels to find them, so the user can clean them up if the database is not disposable
- What the run started and stopped, such as the app and its port, the tabs, and any lockfile changes, and the command that starts the app again
- The run directory path, and follow-ups

Never report a unit as passed if no tester ran it. When a check fails, include the relevant output.

## Delegate permissions

All delegates may run non-destructive commands, such as builds, type checks, linters in check-only mode, test suites, and read-only diagnostics, and may send requests to the app the coordinator started. The exploratory tester and the verifier may open a browser tab for that app.

Delegates write only to the run directory and their own temporary directories, and the mapper writes only `map.md`. Testers never edit source files, and they may install packages inside their temporary directories. Delegates do not start or stop the app, a watcher, or any other long-running process. The coordinator owns them.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside temporary directories, the run directory, or build output
- Installing, upgrading, or removing project dependencies
- Any direct database access, including migrations, seeds, resets, and writes
- Load or stress tests, brute-force attempts, and automated fuzzing
- Bulk deletion, resets, imports, or any action that changes records the tester did not create
- Deploying, publishing, releasing, or pushing anywhere
- Sending real emails, payments, tax documents, notifications, or calls to production APIs that change state
- Exposing the app to the network, or binding it to an address other than localhost

If a delegate isn't sure whether a command is safe, it skips the command and explains what it would have run and why.

## Git restriction

The coordinator and all delegates may run only these read-only Git commands:

- `git --no-pager --no-optional-locks status`
- `git --no-pager diff --no-ext-diff --no-textconv`
- `git --no-pager show --no-ext-diff --no-textconv`
- `git --no-pager log`

Arguments that only filter or format the output are allowed, such as paths, commit ranges, `--stat`, `--oneline`, `-p`, and `-n`. Never use `--output` or any option that writes files.

Never run any other Git command or version-control operation. This includes commit, add, checkout, switch, restore, reset, stash, merge, rebase, pull, push, fetch, branch, tag, and clean. This restriction applies to the coordinator and all delegates, including their tools and subprocesses.

## Scope

Test the project's own pages, forms, and endpoints that the map lists, minus the user's exclusions. Pages hosted by third parties are out of scope. Nobody fixes anything. If a problem needs a fix, the user decides how to handle it, for example with squad or qa.

---
name: test-web-app
description: Test a requested change or flow in a web project's running app on isolated development data through an automated tester and an exploratory tester, report what passed and failed, and stop everything that was started. It reports problems and never fixes them. Usually follows a squad run. For Android apps use test-android-app. Use only when the user says 'test-web-app'.
---

# Test web app

Use this skill to test a change or flow in a web project's running app. It runs only when the user asks for it, usually after a squad run. Never start it automatically. The coordinator prepares an isolated environment, briefs two testers, collects their findings, and stops everything it started. The testers report problems and never fix them. For a loop that also fixes problems and retests, use qa. To test a whole project from a new chat, use test-web-full. For Android apps, use test-android-app.

## Roles

The coordinator is the agent in the conversation. It talks to the user, prepares and stops the environment, briefs each delegate, filters their findings, and reports back. The coordinator does not test on its own and does not edit source files. It is the only agent that starts or stops the app.

Use these exact models and reasoning efforts:

| Role               | Model                   |
| ------------------ | ----------------------- |
| Automated tester   | GPT 6.2 Sol xhigh       |
| Exploratory tester | Claude Sonnet 5.5 xhigh |

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh, so every brief must include all the context it needs.

## Workflow

### 1. Understand the task

Restate the task to the user in one or two sentences. The coordinator needs:

- The behavior or flow to check, and the expected result
- The URL, route, or entry point, and the account or role the flow needs
- The changed files and what changed in each, and the plan and its verification steps when a squad run came earlier in the conversation, including any changes made after it
- Whether the change touches dates, currency, documents, or other locale-dependent data

If `PROJECT.md` exists at the workspace root, read it for commands, ports, and variable names. Otherwise read the README, the package scripts, and the project's instruction files. If the coordinator can't tell what to test or how to start the app, it asks the user before starting anything.

### 2. Prepare the data

Run the app in a development or test configuration. Never use production or real customer data.

Check which database the configuration points to, without printing credentials. A disposable database needs no question, such as an in-memory database, a temporary file, or a container the test creates. If the configuration points to a real or shared database, stop and ask the user for permission before starting the app. The request explains:

- Which database and environment, and exactly what will run
- Which tables or records it affects, and how many
- Why it is needed, and whether an alternative avoids the database
- Whether it can be reversed, and the expected result

The permission covers only that action. Migrations, seeds, resets, and direct writes need the same permission. Delegates reach the database only through the app.

Use sandbox or test modes, or local stubs, for external services. Never trigger real payments, tax documents, emails, messages, or state-changing calls to production APIs. If part of the flow can't be tested without one, leave it out and report it.

Log in with test accounts that the project documents or seeds. If none exist, ask the user. Never use a real account. In T3 Code, request a credential that only the user can give with `request_secret`.

### 3. Start the app

Start the app with the project's own script, taken from `PROJECT.md`, the README, or the package scripts. Identify the package manager by its lockfile and do not mix managers.

Start your own server on a free port, bound to localhost only, in a way you can interrupt. Record its process id, and read the real port and URL from its output instead of assuming the default. Do not use or stop a server you did not start. If the project fixes the port and another service already uses it, ask the user.

If the app can't start because dependencies are missing, install them with the project's package manager and report the lockfile changes. Removing packages, major version upgrades, global installs, and deleting `node_modules` or lockfiles need the user's permission.

Create one evidence directory outside the project's source tree, in the scratchpad or the system temporary directory.

### 4. Brief the testers

Give both testers the same brief:

- The task from step 1, including the changed files, the plan, and its verification steps
- The app URL, the accounts to use by name, and the test data that exists. Tell them never to write credentials in a report, a screenshot, or a log.
- The data rules. Each tester creates its own records, labeled with its role, and does not change or delete records it did not create.
- The evidence directory path
- The delegate permissions, the Git restriction, and the scope from this skill

### 5. Testing

Start both testers in parallel.

The automated tester checks the app without a browser:

- Runs the existing build, type checks, linters in check-only mode, and test suites
- Writes extra tests for the changed behavior and its edge cases in a temporary directory outside the source tree, then runs them
- Sends HTTP requests to the running app with the test accounts, and checks status codes, validation errors, auth and role checks, and that responses expose no secrets
- Checks that code near the change still works, to catch regressions

The exploratory tester uses the app in a browser the way a user would. In T3 Code, it uses the preview tools:

- Call `preview_status`. Open its own tab with `preview_open`, with `reuseExistingTab` set to false and the app port as `target`, and use the returned `tabId` from then on. Status can report unavailable only because no tab exists yet, and opening a tab needs no user action.
- `preview_snapshot` reads the page and returns `aria-ref` locators for `preview_act`, which clicks, types, presses keys, selects, uploads, and scrolls. Locators expire after navigation or a new snapshot, so take a new snapshot first.
- `preview_wait_for` waits for text, an element, or a URL. `preview_resize` sets the viewport. `preview_set_appearance` switches light and dark. `preview_evaluate` runs JavaScript in the page for state that a snapshot can't show.
- Do not install or switch to another automation system. If no browser tool is available, report that and list what stayed unverified.

It checks, in this order, skipping items that do not apply:

- The main flow from start to finish, as the account or role the request names
- Invalid input, empty fields, boundary values, and the message shown for each
- Loading, empty, and error states, and a repeated submit
- Reload and back navigation after the action, to confirm that saved data persists
- Narrow (about 360 px), tablet, and desktop widths, and dark appearance when the project supports it
- Keyboard use, covering tab order, visible focus, Enter, and Escape, and the accessible names in the snapshot
- Access, when the change touches it, so that a user without the role can't reach the page or the action
- Console errors and failed requests during every step
- Security in the changed code, such as script text entered in a field and rendered back

Both testers apply the locale rule. When the change involves dates, currency, numbers, documents, phone numbers, or addresses, compare what the page shows with what it sends, using the project's own formats. Try values where day and month could swap, cents, large values with thousands separators, zero, an empty field, an invalid value, month and year boundaries, and the resulting sort order and filters.

Each tester saves screenshots and logs as evidence in the evidence directory, and returns:

- A verdict, pass or fail
- Each problem found, with steps to reproduce, expected and actual results, and severity (blocker, major, or minor)
- The commands it ran and their output, and the checks it ran with the result of each
- What it could not test and why

Testers close the tabs they opened and delete their other temporary files before returning.

### 6. Stop the environment

The coordinator stops everything the run started, with no exception, once both testers return. This includes the app, watchers, and background processes, using the recorded process ids. Confirm that no tab or process from the run remains. Do not leave anything running, even when the user may want to inspect the app. Do not stop processes you did not start.

Delete the evidence directory, unless the user asked to keep the evidence. In that case, report its path.

### 7. Report

The coordinator reviews the findings, merges duplicates, and drops reports that are out of scope. It records the reason for each dropped report. It does not fix anything.

Include the results in the final reply, using this format:

| Delegate                | Result         |
| ----------------------- | -------------- |
| GPT 6.2 Sol xhigh       | fail (1 major) |
| Claude Sonnet 5.5 xhigh | pass           |

After the table, list the checks that each tester ran with their results, the problems with reproduction steps and severity, the problems dropped and why, what could not be tested and why, with the steps the user can run to check it, and the follow-ups, including unrelated problems. Say what the run started and stopped, such as the app and its port, the tabs, the temporary files, and any lockfile changes. Give the command that starts the app again, for a user who wants to inspect it. Never report a check as passed if no tester ran it. When a check fails, include the relevant output.

## Delegate permissions

All delegates may run non-destructive commands, such as builds, type checks, linters in check-only mode, test suites, and read-only diagnostics, and may send requests to the app the coordinator started. The exploratory tester may open a browser tab for that app.

Testers never edit source files. They may write only to the evidence directory and their own temporary directories, and they may install packages inside their temporary directories. Delegates do not start or stop the app, a watcher, or any other long-running process. The coordinator owns them.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside temporary directories or build output
- Installing, upgrading, or removing project dependencies
- Any direct database access, including migrations, seeds, resets, and writes
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

Test only the request and the code it touches. Testers may report unrelated bugs they find, but nobody fixes them. List them as follow-ups in the final reply. If a problem needs a fix, the user decides how to handle it, for example with squad or qa.

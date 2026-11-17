---
name: test-android-full
description: Test an entire Android project from a cold start in a new chat. Map its screens, flows, deep links, permissions, and backend calls, check the map on the running app, run functional, presentation and configuration, and lifecycle and access passes over every mapped unit through testers, and report coverage and problems. It can run for hours, reports problems and never fixes them, and stops everything it started. For one change or flow use test-android-app. Use only when the user says 'test-android-full'.
---

# Test Android full

Use this skill to test a whole Android app, starting from a chat that knows nothing about it. It covers native Android projects, cross-platform projects such as React Native, Flutter, and Capacitor, and apps that wrap a web view. It runs only when the user asks for it. Never start it automatically. It is exhaustive by design. Delegates map the project, the coordinator checks the map on the running app, and testers go through every mapped screen and flow in three passes. A run can take hours and start many delegates. Invoking this skill is the explicit request for that depth. The testers report problems and never fix them. For one change or flow, use test-android-app. For a loop that also fixes problems and retests, use qa.

## Roles

The coordinator is the agent in the conversation. It talks to the user, prepares and stops the environment, keeps the ledger, briefs each delegate, filters their findings, and reports back. The coordinator does not map or test on its own and does not edit source files. It is the only agent that builds the app, installs it, starts or stops a backend, a bundler, or an emulator.

Use these exact models and reasoning efforts:

| Role               | Model                   |
| ------------------ | ----------------------- |
| Mapper             | GPT 6.2 Sol xhigh       |
| Verifier           | Claude Sonnet 5.5 high  |
| Automated tester   | GPT 6.2 Sol xhigh       |
| Exploratory tester | Claude Sonnet 5.5 xhigh |

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh, so every brief must include all the context it needs.

One device accepts one driver at a time. Only the verifier and the exploratory tester use the device, and the coordinator never has two of them running together.

## Run directory and ledger

A run is longer than one conversation can hold, so files on disk are the source of truth. Create one run directory outside the project's source tree, in the place the user names or else in the scratchpad or the system temporary directory. It holds:

- `map.md`, with one entry per unit
- `ledger.md`, with one row per unit and its status in each pass
- `findings/`, with one file per delegate run
- `evidence/`, with screenshots and logs

A unit is one screen, flow, deep link, background component, or journey. The ledger starts with a line for the status of the baseline run and a line for the first launch, then has one row per unit, like this:

| ID  | Unit          | Accounts | Pass 1 | Pass 2  | Pass 3  | Notes |
| --- | ------------- | -------- | ------ | ------- | ------- | ----- |
| S01 | LoginActivity | none     | pass   | pending | pending |       |

The statuses are pending, pass, fail, blocked, and skipped. A blocked or skipped status always has a reason in Notes. Re-read the ledger before every batch and after any pause or summary of the conversation. If the user gives the path of an earlier run directory, resume from its ledger instead of starting over, after checking that the environment is ready.

## Workflow

### 1. Understand the project and ask

Find the project to test. In a multi-project workspace, ask which one. Read `PROJECT.md` if it exists, as a hint only, and read the README, the Gradle files, the configuration, and the project's instruction files. Find the application id, the build variants, and the `minSdk` and `targetSdk`. Tell the user that the run can take hours.

Ask the user once, in a single message, only what no file can answer. After that, the run goes on without waiting, unless a step says to stop.

- The permission described in step 3, if the backend's database is real or shared
- A test account for each role, or where the project documents them
- Screens or areas to leave out
- The device to use. Prefer an emulator. A physical device only if the user names it.
- Where to put the run directory, if not the default

If the coordinator can't tell how to build the app, it asks that too. The mapper does not need the answers, so start it while waiting.

### 2. Map

Brief the mapper with the workspace path, the project, and any hints. The mapper reads files and runs read-only commands. It does not edit files. It writes `map.md` in the run directory and returns a summary with:

- The stack and UI toolkit, the modules, how to build and run the app, and the existing test commands
- The manifest components, with their intent filters, deep links, and exported flags, and the permissions the app requests
- The navigation graphs or routes and the start destination
- Every screen, with its class or composable file, its view model, its arguments, the data and backend calls it uses, the inputs with their validation rules from the code, the states it can show, the runtime permissions it needs, the accounts that may use it, and how to reach it
- The first-launch flow, such as onboarding, welcome screens, permission explanations, and the first login
- The backend base URLs per build type and the endpoints the app calls
- Background work, such as services, workers, receivers, notifications, and widgets, and which of it a person can trigger from the UI
- Units that send messages, take payments, call third-party services, or delete or overwrite data in bulk, each marked with that effect
- Journeys that cross several screens
- Dead code, such as screens without a navigation entry, and anything it could not determine from the code alone

### 3. Prepare the environment

Wait for the user's answers. Point the app at a local, development, or staging backend. Never use production or real customer data.

Before starting a backend, check which database its configuration points to, without printing credentials. A disposable database needs no question, such as an in-memory database, a temporary file, or a container the test creates. Prefer one when the project documents a way to create it. If the configuration points to a real or shared database, stop and ask the user for permission before starting it. The request explains:

- Which database and environment, and exactly what will run
- Which tables or records it affects, and roughly how many, since a full run creates and edits many records through the app
- Why it is needed, and whether an alternative avoids the database
- Whether it can be reversed, and the expected result

The permission covers only that action. Migrations, seeds, resets, and direct writes need the same permission. Test suites that reach such a database need it too. Delegates reach the database only through the app.

Use sandbox or test modes, or local stubs, for external services. Never trigger real payments, push notifications, SMS, emails, or state-changing calls to production APIs. Log in with the test accounts only. Never use a real account. In T3 Code, request a credential that only the user can give with `request_secret`.

Open the device. In T3 Code, call `device_list`, then `device_open` with the host and device ids. It boots the device if needed, so note whether it was already running. Elsewhere, use the harness's device tools or `adb`. List devices with `adb devices`, and pass `-s <serial>` on every command when more than one device is connected. To start an existing emulator without T3 Code, run it in the background and wait until `adb shell getprop sys.boot_completed` returns 1. If no emulator or system image exists, ask the user. Downloading system images with `sdkmanager` or creating a virtual device needs permission. If the device host is another machine, ask the user how the app should reach the backend.

Build the debug variant with the project's Gradle wrapper (`./gradlew`, or `gradlew.bat` on Windows), or with the project's own cross-platform command. Never build or install a release variant, and never use signing keys or keystore passwords. Run `./gradlew --status` before the build. If the build starts a Gradle daemon that was not running before, stop it in step 6. `./gradlew --stop` stops every daemon of that Gradle version, so use it only when none was running before the build. Install the debug build on the selected device only. If dependencies are missing, install them with the project's package manager and report the lockfile changes. Removing packages, major version upgrades, and global installs need the user's permission. Never uninstall the app or clear its data. If the app can't reach a local backend over HTTP, report the cause, because Android 9 and later block cleartext traffic by default and changing the manifest is a source edit.

Start the backend and the bundler, such as Metro, with the project's own commands. Keep them bound to localhost, on a free port, in a way you can interrupt, and record their process ids. An emulator reaches the host's loopback at `10.0.2.2`. Alternatively, run `adb reverse tcp:<port> tcp:<port>` so that the app uses `localhost`, which also works for a physical device over USB. Do not use or stop a process you did not start.

### 4. First launch and verify the map

Without clearing data, the first launch of the installed app can be seen only once. So before anything else, start the exploratory tester on the first-launch flow from the map, with the checks of pass 1. It leaves the app signed in, or at the first screen that needs an account.

Then brief the verifier with `map.md`, the application id, and the test accounts. The verifier checks the map on the device, signed in as each account in turn. It navigates from the launch screen through every screen it can reach, and submits nothing except the login form. For each unit it records the navigation path, a screenshot, and whether it was reachable. It lists screens it reached that the map lacks, and mapped screens it could not reach. It writes its corrections in its findings file and returns a summary.

The coordinator merges the corrections into `map.md` and builds `ledger.md`. If the verifier finds major gaps, the coordinator sends them back to the mapper for one more round. Units that the map marks with an external effect or a bulk destructive action get the status skipped, with the reason. The user's exclusions get the same status. The coordinator tells the user the number of units and batches, and does not wait for a reply.

### 5. Run the passes

Group the units into batches by area, such as one feature or one set of related screens, with about five to eight units each, and keep units that share data together. Run batches one after another. Within a batch, run both testers in parallel when the pass uses both.

Each brief to a tester includes:

- The unit entries from `map.md` for that batch, and the checks of the pass
- The application id, the backend address, the accounts to use by name, and the test data that exists. Tell them never to write credentials in a report, a screenshot, or a log.
- The data rules. Each tester creates its own records, labeled with the run, its role, and the pass, and lists them in its findings file. It changes or deletes only records it created.
- The path of its findings file and the evidence directory
- The device instructions below, with the host id and the device id, for the exploratory tester only
- The delegate permissions, the Git restriction, and the scope from this skill

Each tester writes its details to its findings file, and returns a short summary instead of the details: a verdict for each unit, the number of problems by severity, the findings file path, and what it could not test and why. A problem entry has the unit ID, steps to reproduce, expected and actual results, severity (blocker, major, or minor), the relevant log lines, and the evidence path. The coordinator updates the ledger from the summaries and reads the findings files in full only when it writes the report.

Device instructions for the verifier and the exploratory tester. In T3 Code:

- Call `device_open` with the host and device ids from the brief. Use the returned `agent-device` command, with all returned arguments, for every app launch, relaunch, deep link, tap, text entry, alert, setting, screenshot, and recording.
- Run `open <app>` before snapshots or input, and use `help <command>` to list options. Use `device_screenshot` to look at the screen.
- Use `adb` only for logs, port forwards, and what `agent-device` can't do.
- If a device tool or the command it returned fails, quote the error, follow its advice once, and then stop and report the blocker.
- Close the session at the end with `device_close`, without `shutdown`.

Without T3 Code, use the harness's device tools or `adb`. In every host:

- Do not install or switch to another automation system.
- Read the log of the app's process with `adb -s <serial> logcat -d --pid=$(adb -s <serial> shell pidof -s <package>)`, and the crash buffer with `logcat -b crash -d`. Look for `FATAL EXCEPTION`, ANR reports, and StrictMode violations.
- Never clear the device's log buffer.

#### Baseline

Start the automated tester once on the whole project. It runs the existing Gradle unit tests, type checks, and lint in check-only mode, within the database rule above. It does not run instrumented tests (`connected*AndroidTest`) unless the user asked, because they take over the device and can remove the app. It records each failure under the baseline line.

#### Pass 1, functional

Both testers, batch by batch. The automated tester works without the device, for each unit in the batch:

- Sends HTTP requests to the backend calls the units use, with the test accounts. It tries valid requests, each validation rule from the map at its boundary, missing and extra fields, no token, the wrong role, and a few hand-written hostile strings, and checks status codes and that responses expose no secrets.
- Reviews the units' code for state lost on rotation or process death, permission handling, input validation, and secrets
- Writes extra tests for the batch in a temporary directory outside the source tree, when that is practical

The exploratory tester uses each unit on the device the way a user would:

- The main flow from start to finish, as each account that may use it
- Invalid input, empty fields, boundary values, and the message shown for each
- Loading, empty, and error states
- Runtime permission prompts the unit triggers, covering allow, deny, and deny with don't ask again
- The back button and the navigation stack
- Crashes and errors in the log during every step

Both testers apply the locale rule. When a unit involves dates, currency, numbers, documents, phone numbers, or addresses, compare what the screen shows with what the app sends, using the project's own formats. Try values where day and month could swap, cents, large values with thousands separators, zero, an empty field, an invalid value, month and year boundaries, and the resulting sort order and filters.

#### Pass 2, presentation and configuration

The exploratory tester only, batch by batch. For each setting in turn, it applies the setting, visits every unit in the batch, and restores the setting before the next one. It changes only one setting at a time:

- Rotation, on units with forms, lists, or media
- Font scale 1.3 and then 2.0
- Dark theme
- A second locale, and right-to-left layout when the app supports it
- A tablet or foldable screen, when the app supports one and a device is available
- Accessibility, covering labels on controls, touch targets of at least 48 dp, contrast, and focus order. The UI hierarchy from `adb shell uiautomator dump` shows labels and sizes when the device tool doesn't.
- Long values, many rows, and empty lists, and text that overflows or is cut off

The tester lists every setting it changed and confirms that it restored it.

#### Pass 3, lifecycle, access, and journeys

Both testers, in groups chosen by account and by journey instead of by area:

- The exploratory tester sends each unit to the background and brings it back, and ends the process of a backgrounded app with `adb shell am kill <package>` when `agent-device` has no equivalent, to check that the unit restores its state.
- The exploratory tester checks units that load data with a slow network, with no network, and with a server error, and checks what the user sees and can retry.
- The exploratory tester opens every deep link and intent filter from the map through the device tool, with valid values and with missing or wrong extras.
- The exploratory tester checks that each account sees only the screens and controls it may use, and runs each journey from the map from start to finish with fresh test data.
- The automated tester sends direct requests to the backend for every account, including no token, and checks that each is allowed or denied as the map says. With two accounts of the same role, it checks that one can't read or change the other's records by changing an identifier.
- The automated tester reviews the app as a whole, covering exported components without a permission, backup and debuggable settings, network security configuration, WebView settings, and secrets in the code.

#### Managing the run

- Update the ledger after every batch, and send the user one line with the batch, the units done, and the problems found so far.
- If a tester returns with units unfinished, put those units in one new batch. If they are unfinished again, mark them blocked with the reason.
- Do not retest a unit because it failed. The problem goes to the report.
- If a unit can't be tested because of an earlier failure, such as a login that does not work, mark it blocked with the reason.
- A crash of the app is a problem to report. The tester relaunches the app and continues.
- If the device stops responding, close and open it again with the device tools, at most twice in a row. The debug build stays installed. After that, stop testing, mark the remaining units not tested, and go to step 6. The same applies to a backend you started that stops responding.
- Do not end a pass early because few problems appeared, and do not add passes beyond these.

### 6. Stop the environment

The coordinator stops everything the run started, with no exception, once no delegate is running. This includes the backend, the bundler, watchers, and background processes, using the recorded process ids, and the Gradle daemon rule from step 3. Remove the `adb reverse` mappings it added with `adb reverse --remove tcp:<port>`. In T3 Code, call `device_close` for the device it opened, and pass `shutdown` when it booted that emulator. Elsewhere, shut down an emulator it started with `adb -s <serial> emu kill`. Leave a device that was already running as it was. Do not leave anything running, even when the user may want to inspect the app. Do not stop processes you did not start.

The debug build stays installed on the device. Report it. Delete the testers' temporary files and scratch copies. Keep the run directory, because it is the deliverable, and report its path. Delete `evidence/` only if the user said not to keep it.

### 7. Report

The coordinator writes `report.md` in the run directory with every problem, and sends the user a final reply. Start it with the device and the build, such as the device name, the Android version, the build variant, and the version of the app. Then include the results:

| Pass         | Units | Pass | Fail | Blocked | Skipped |
| ------------ | ----- | ---- | ---- | ------- | ------- |
| Baseline     | 1     | 0    | 1    | 0       | 0       |
| First launch | 1     | 1    | 0    | 0       | 0       |
| Pass 1       | 38    | 30   | 6    | 1       | 1       |

After the table, list:

- Every blocker and major problem, with the unit, steps to reproduce, expected and actual results, the relevant log lines, and the evidence path. Give the number of minor problems, with the path of `report.md`.
- The units not tested, with the reason for each
- The delegate runs by role, with the model and effort
- The records the testers created, and the labels to find them, so the user can clean them up if the database is not disposable
- What the run started and stopped, such as the backend and its port, the bundler, the emulator, the `adb reverse` mappings, the settings restored, and the debug build left installed, and the commands that start the app again
- The run directory path, and follow-ups

Never report a unit as passed if no tester ran it. When a check fails, include the relevant output.

## Delegate permissions

All delegates may run non-destructive commands, such as type checks, linters in check-only mode, unit test suites, and read-only diagnostics, and may send requests to the backend the coordinator started. The verifier and the exploratory tester may use the device the coordinator names.

Delegates write only to the run directory and their own temporary directories, and the mapper writes only `map.md`. Testers never edit source files, and they may install packages inside their temporary directories. Delegates do not build the app, install it, or start or stop the backend, a bundler, or an emulator. The coordinator owns them. Gradle may start a daemon during a unit test run. Leave it for the coordinator to stop.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside temporary directories, the run directory, or build output
- Editing manifests or build configuration
- Installing, upgrading, or removing project dependencies, or downloading system images and SDK packages
- Uninstalling apps, clearing app data, factory resets, or any use of root
- Any direct database access, including migrations, seeds, resets, and writes
- Random input tools such as `adb shell monkey`, load or stress tests, brute-force attempts, and automated fuzzing
- Bulk deletion, resets, imports, or any action that changes records the tester did not create
- Release builds, signing, deploying, publishing, releasing, or pushing anywhere
- Sending real emails, payments, push notifications, SMS, or calls to production APIs that change state
- Exposing a backend or a bundler to the network

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

Test the app's own screens, flows, and backend calls that the map lists, minus the user's exclusions. Third-party screens, such as system dialogs and external sign-in pages, are out of scope except for the entry and return. Nobody fixes anything. If a problem needs a fix, the user decides how to handle it, for example with squad or qa.

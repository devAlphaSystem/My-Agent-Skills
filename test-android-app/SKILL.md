---
name: test-android-app
description: Build an Android project's debug app, install it on an emulator or a device the user names, test a requested change or flow through an automated tester and an exploratory tester, report what passed and failed, and stop everything that was started. It reports problems and never fixes them. Usually follows a squad run. For web apps use test-web-app. Use only when the user says 'test-android-app'.
---

# Test Android app

Use this skill to test a change or flow in an Android app. It covers native Android projects, cross-platform projects such as React Native, Flutter, and Capacitor, and apps that wrap a web view. It runs only when the user asks for it, usually after a squad run. Never start it automatically. The coordinator builds and installs the debug app, briefs two testers, collects their findings, and stops everything it started. The testers report problems and never fix them. For a loop that also fixes problems and retests, use qa. To test a whole app from a new chat, use test-android-full. For web apps, use test-web-app.

## Roles

The coordinator is the agent in the conversation. It talks to the user, prepares and stops the environment, briefs each delegate, filters their findings, and reports back. The coordinator does not test on its own and does not edit source files. It is the only agent that builds the app, starts or stops a backend, a bundler, or an emulator, and installs on the device.

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
- The screen, deep link, or entry point, and the account the flow needs
- The changed files and what changed in each, and the plan and its verification steps when a squad run came earlier in the conversation, including any changes made after it
- The application id, the build variants, and the `minSdk` and `targetSdk`, taken from the Gradle files
- Whether the change touches dates, currency, documents, or other locale-dependent data

If `PROJECT.md` exists at the workspace root, read it for commands and conventions. Otherwise read the README, the Gradle files, and the project's instruction files. If the coordinator can't tell what to test or how to build the app, it asks the user before starting anything.

### 2. Prepare the data

Point the app at a local, development, or staging backend. Never use production or real customer data.

Before starting a backend, check which database its configuration points to, without printing credentials. A disposable database needs no question, such as an in-memory database, a temporary file, or a container the test creates. If the configuration points to a real or shared database, stop and ask the user for permission before starting it. The request explains:

- Which database and environment, and exactly what will run
- Which tables or records it affects, and how many
- Why it is needed, and whether an alternative avoids the database
- Whether it can be reversed, and the expected result

The permission covers only that action. Migrations, seeds, resets, and direct writes need the same permission. Delegates reach the database only through the app.

Use sandbox or test modes, or local stubs, for external services. Never trigger real payments, push notifications, SMS, emails, or state-changing calls to production APIs. If part of the flow can't be tested without one, leave it out and report it.

Log in with test accounts that the project documents or seeds. If none exist, ask the user. Never use a real account. In T3 Code, request a credential that only the user can give with `request_secret`.

### 3. Prepare the device and the app

Choose the device. Prefer an emulator. Use a physical device only when the user names it, because it holds real data and other apps. In T3 Code, call `device_list`, then `device_open` with the host and device ids. It boots the device if needed, so note whether it was already running. Elsewhere, use the harness's device tools or `adb`. List devices with `adb devices`, and pass `-s <serial>` on every command when more than one device is connected. To start an existing emulator without T3 Code, run it in the background and wait until `adb shell getprop sys.boot_completed` returns 1.

If no emulator or system image exists, ask the user. Downloading system images with `sdkmanager` or creating a virtual device changes files outside the project folder and needs permission. If the device host is another machine, ask the user how the app should reach the backend.

Build the debug variant with the project's Gradle wrapper (`./gradlew`, or `gradlew.bat` on Windows), or with the project's own cross-platform command. Never build or install a release variant, and never use signing keys or keystore passwords. Run `./gradlew --status` before the build. If the build starts a Gradle daemon that was not running before, stop it in step 6. `./gradlew --stop` stops every daemon of that Gradle version, so use it only when none was running before the build.

Install the debug build on the selected device only. Do not uninstall the app or clear its data without the user's permission, and say what would be lost. If the app can't reach a local backend over HTTP, report the cause. Android 9 and later block cleartext traffic by default, and changing the manifest or the network configuration is a source edit.

Start the backend and the bundler, such as Metro, with the project's own commands. Keep them bound to localhost, on a free port, in a way you can interrupt, and record their process ids. An emulator reaches the host's loopback at `10.0.2.2`. Alternatively, run `adb reverse tcp:<port> tcp:<port>` so that the app uses `localhost`, which also works for a physical device over USB. Do not use or stop a process you did not start.

Create one evidence directory outside the project's source tree, in the scratchpad or the system temporary directory.

### 4. Brief the testers

Give both testers the same brief:

- The task from step 1, including the changed files, the plan, and its verification steps
- The application id, the entry point, the accounts to use by name, the backend address, and the test data that exists. Tell them never to write credentials in a report, a screenshot, or a log.
- The data rules. Each tester creates its own records, labeled with its role, and does not change or delete records it did not create.
- The evidence directory path
- The delegate permissions, the Git restriction, and the scope from this skill

Give the exploratory tester the host id and the device id as well. Only the exploratory tester uses the device.

### 5. Testing

Start both testers in parallel.

The automated tester checks the app without the device:

- Runs the existing Gradle unit tests, type checks, and lint in check-only mode. It does not run instrumented tests (`connected*AndroidTest`) unless the user asked, because they take over the device and can remove the app.
- Writes extra tests for the changed behavior in a temporary directory outside the source tree when that is practical, then runs them
- Reviews the changed code for Android risks, such as state lost on rotation or process death, permissions in the manifest, exported components, cleartext traffic, and secrets in the code
- Sends HTTP requests to the backend with the test accounts, and checks status codes, validation errors, auth, and that responses expose no secrets
- Checks that code near the change still works, to catch regressions

The exploratory tester uses the installed debug app the way a user would. In T3 Code, it calls `device_open` with the host and device ids from the brief, then uses the returned `agent-device` command, with all returned arguments, for every app launch, relaunch, deep link, tap, text entry, alert, setting, screenshot, and recording. It runs `open <app>` before snapshots or input, uses `help <command>` to list options, and uses `device_screenshot` to look at the screen. It uses `adb` only for logs, port forwards, and what `agent-device` can't do. If a device tool or the command it returned fails, it quotes the error, follows its advice once, and then stops and reports the blocker. Without T3 Code, it uses the harness's device tools or `adb`. It does not install or switch to another automation system.

It checks, in this order, skipping items that do not apply:

- The main flow from start to finish, as the account the request names
- Invalid input, empty fields, boundary values, and the message shown for each
- Runtime permission prompts, covering allow, deny, and deny with don't ask again
- Rotation during a form or a long operation, and whether the state survives
- Going to the background and returning, and ending the process of a backgrounded app to check that it restores its state. It uses `adb shell am kill <package>` when `agent-device` has no equivalent.
- The back button and the navigation stack, and deep links or share intents when the change touches them
- A slow or missing network and a server error, and what the user sees and can retry
- Font scale (1.3 and larger), dark theme, and a second locale, or right-to-left layout when the app supports it
- A phone and a tablet or foldable screen when the app supports them
- Accessibility, covering labels on controls, touch targets of at least 48 dp, contrast, and focus order
- Crashes and errors in the log during every step

It reads the log of the app's process with `adb -s <serial> logcat -d --pid=$(adb -s <serial> shell pidof -s <package>)`, and the crash buffer with `logcat -b crash -d`. It looks for `FATAL EXCEPTION`, ANR reports, and StrictMode violations, and does not clear the device's log buffer. The UI hierarchy from `adb shell uiautomator dump` shows labels and sizes when the device tool doesn't. It changes a device setting only to test it, one at a time, such as rotation, font scale, theme, locale, or network mode, and restores each one afterward.

Both testers apply the locale rule. When the change involves dates, currency, numbers, documents, phone numbers, or addresses, compare what the screen shows with what the app sends, using the project's own formats. Try values where day and month could swap, cents, large values with thousands separators, zero, an empty field, an invalid value, month and year boundaries, and the resulting sort order and filters.

Each tester saves screenshots and log excerpts as evidence in the evidence directory, and returns:

- A verdict, pass or fail
- Each problem found, with steps to reproduce, expected and actual results, severity (blocker, major, or minor), and the relevant log lines
- The commands it ran and their output, and the checks it ran with the result of each
- The settings it changed and restored
- What it could not test and why

The exploratory tester closes its device session with `device_close`, without `shutdown`. Testers delete their other temporary files before returning.

### 6. Stop the environment

The coordinator stops everything the run started, with no exception, once both testers return. This includes the backend, the bundler, watchers, and background processes, using the recorded process ids, and the Gradle daemon rule from step 3. Remove the `adb reverse` mappings it added with `adb reverse --remove tcp:<port>`. In T3 Code, call `device_close` for the device it opened, and pass `shutdown` when it booted that emulator. Elsewhere, shut down an emulator it started with `adb -s <serial> emu kill`. Leave a device that was already running as it was. Do not leave anything running, even when the user may want to inspect the app. Do not stop processes you did not start.

The debug build stays installed on the device. Report it.

Delete the evidence directory, unless the user asked to keep the evidence. In that case, report its path.

### 7. Report

The coordinator reviews the findings, merges duplicates, and drops reports that are out of scope. It records the reason for each dropped report. It does not fix anything.

Start the final reply with the device and the build, such as the device name, the Android version, the build variant, and the version of the app. Then include the results, using this format:

| Delegate                | Result         |
| ----------------------- | -------------- |
| GPT 6.2 Sol xhigh       | fail (1 major) |
| Claude Sonnet 5.5 xhigh | pass           |

After the table, list the checks that each tester ran with their results, the problems with reproduction steps, severity, and relevant log lines, the problems dropped and why, what could not be tested and why, with the steps the user can run to check it, and the follow-ups, including unrelated problems. Say what the run started and stopped, such as the backend and its port, the bundler, the emulator, the `adb reverse` mappings, the settings restored, and the debug build left installed. Give the commands that start the app again, for a user who wants to inspect it. Never report a check as passed if no tester ran it. When a check fails, include the relevant output.

## Delegate permissions

All delegates may run non-destructive commands, such as type checks, linters in check-only mode, unit test suites, and read-only diagnostics, and may send requests to the backend the coordinator started. The exploratory tester may use the device the coordinator names.

Testers never edit source files. They may write only to the evidence directory and their own temporary directories, and they may install packages inside their temporary directories. Delegates do not build the app, install it, or start or stop the backend, a bundler, or an emulator. The coordinator owns them. Gradle may start a daemon during a unit test run. Leave it for the coordinator to stop.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside temporary directories or build output
- Editing manifests or build configuration
- Installing, upgrading, or removing project dependencies, or downloading system images and SDK packages
- Uninstalling apps, clearing app data, factory resets, or any use of root
- Any direct database access, including migrations, seeds, resets, and writes
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

Test only the request and the code it touches. Testers may report unrelated bugs they find, but nobody fixes them. List them as follow-ups in the final reply. If a problem needs a fix, the user decides how to handle it, for example with squad or qa.

---
name: qa
description: Trigger this if the user says 'qa'.
---

# QA

Use this skill to test changes in the workspace. It runs only when the user asks for it, usually after a squad run. Never start it automatically. The coordinator briefs the testers, collects their findings, sends real bugs to the fixer, and repeats until the tests pass.

## Roles

The coordinator is the agent in the conversation. It briefs each delegate, filters and passes findings between them, and reports to the user. The coordinator does not test or fix code on its own.

Use these exact models and reasoning efforts:

| Role               | Model                                                                          |
| ------------------ | ------------------------------------------------------------------------------ |
| Automated tester   | GPT 6.1 Sol xhigh                                                              |
| Exploratory tester | Claude Sonnet 5.5 xhigh                                                        |
| Fixer              | Claude Sonnet 5.5 xhigh, or Claude Opus 5.5 xhigh for high complexity problems |

A problem is high complexity when at least one of these applies:

- Its root cause is not clear from the findings and the reproduction steps.
- The fix spans several modules or layers.
- A mistake would be costly or hard to undo, such as a change to auth, security, payments, or stored data.
- An earlier fix for the same problem failed a retest.

The coordinator starts the fixer on Claude Opus 5.5 xhigh for high complexity problems, and on Claude Sonnet 5.5 xhigh otherwise. This is the only model choice the coordinator makes. The final report states the model used for each fixer and why.

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh, so every brief must include all the context it needs.

## Brief

Give both testers the same brief:

- The original request
- The list of changed files and what changed in each
- The plan and its verification steps, when coming from squad
- The project type, how to build and run it, and the URLs, commands, or entry points to check

If a squad run happened earlier in the conversation, include its results and any changes made after it, such as fixes the user asked for. Otherwise, the coordinator builds the brief from the user's request and the files the user points to. If the coordinator can't tell what to test, it asks the user.

## Testing

Start both testers in parallel.

The automated tester checks the code with tools:

- Runs the existing build, type checks, linters in check-only mode, and test suites
- Writes extra tests for the changed behavior and its edge cases in a temporary directory outside the source tree, then runs them
- Checks that code near the change still works, to catch regressions

The exploratory tester uses the result the way a user would:

- Opens the app in a browser, emulator, or simulator, runs the binary, or calls the CLI or API with sample inputs, depending on the project type
- Tries the main flow, invalid inputs, empty states, errors, and limits
- Checks UI and UX where the change is user-facing, including accessibility, responsive layout, and console errors
- Looks for security problems in the changed code, such as missing input validation, exposed secrets, and broken auth checks

Each tester returns:

- A verdict: pass or fail
- Each problem found, with steps to reproduce, expected and actual results, and severity (blocker, major, or minor)
- The commands it ran and their output
- What it could not test and why

Testers delete their temporary files before returning.

## Fixing

The coordinator reviews the findings, merges duplicates, and drops reports that are out of scope or not reproducible. It records the reason for each dropped report.

The coordinator sends blocker and major problems to the fixer, along with the brief and the reproduction steps. The fixer fixes the cause, runs the relevant checks, and returns the changed files and check output. If a fix needs a design change, the fixer stops and reports it, and the coordinator asks the user how to proceed.

Minor problems go into the final reply as follow-ups unless the fix is a one-line change in a file already touched.

After each fix, start a new round with both testers. Each full test pass by both testers counts as one round. Stop when both testers return pass, or after five rounds. If the problems remain after five rounds, report them to the user and stop.

## Delegate permissions

All delegates may open a browser, start local dev servers, emulators, and simulators, and run non-destructive commands such as builds, type checks, linters in check-only mode, test suites, and read-only diagnostics.

Testers never edit source files. They may write only to their temporary directories. Only the fixer may edit source files, and only to fix reported problems.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside temporary directories, build output, or the files the fixer must change
- Installing, upgrading, or removing project dependencies. Testers may install packages inside their temporary directories.
- Database migrations, resets, or writes to real data
- Deploying, publishing, releasing, or pushing anywhere
- Sending real emails, payments, notifications, or calls to production APIs that change state

If a delegate isn't sure whether a command is safe, it skips the command and explains what it would have run and why.

## Git restriction

The coordinator and all delegates may run only these read-only Git commands:

- `git --no-pager --no-optional-locks status`
- `git --no-pager diff --no-ext-diff --no-textconv`
- `git --no-pager show --no-ext-diff --no-textconv`
- `git --no-pager log`

Arguments that only filter or format the output are allowed, such as paths, commit ranges, `--stat`, `--oneline`, `-p`, and `-n`. Never use `--output` or any option that writes files.

Never run any other Git command or version-control operation. This includes commit, add, checkout, switch, restore, reset, stash, merge, rebase, pull, push, fetch, branch, tag, and clean. This restriction applies to the coordinator and all delegates, including their tools and subprocesses. Make all changes by editing workspace files directly.

## Scope

Test only the request and the code it touches. Testers may report unrelated bugs they find, but the fixer does not fix them. List them as follow-ups in the final reply.

## Report

Include the QA results in the final reply, using this format:

| Delegate                | Round 1          | Round 2 |
| ----------------------- | ---------------- | ------- |
| GPT 6.1 Sol xhigh       | fail (2 major)   | pass    |
| Claude Sonnet 5.5 xhigh | fail (1 blocker) | pass    |

Adjust the rounds and columns to match the actual runs. After the table, list the fixer model used for each fix and why, the problems fixed, the problems dropped and why, what could not be tested, and the follow-ups.

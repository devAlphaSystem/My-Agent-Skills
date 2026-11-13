---
name: debug
description: Trigger this if the user says 'debug'.
---

# Debug

Use this skill when the user reports a bug and the cause is unknown. The coordinator runs the investigation through delegates, confirms the root cause, and writes a brief the user can pass to the squad skill. This skill never fixes the bug and never starts squad automatically.

## Roles

The coordinator is the agent in the conversation. It talks to the user, briefs each delegate, passes results between them, and reports back. The coordinator does not investigate on its own, except for small clarifications needed to write a brief.

Use these exact models with xhigh reasoning effort:

| Role         | Model                 |
| ------------ | --------------------- |
| Investigator | GPT 6.1 Sol xhigh     |
| Analyst      | Claude Opus 5.5 xhigh |

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh, so every brief must include all the context it needs.

## Workflow

### 1. Understand the bug

Restate the bug to the user in one or two sentences. The coordinator needs:

- What happens and what should happen instead
- Where it happens, such as a page, command, endpoint, or platform
- How often it happens, and whether it started after a known change
- Error messages, logs, or screenshots the user already has

If the coordinator can't tell what the bug is or where to look, it asks the user before starting any delegate.

### 2. Investigation

Brief the investigator with the bug description, the project type, how to build and run the project, and the relevant paths. The investigator gathers evidence. It does not propose a fix or edit source files.

The investigator must:

- Try to reproduce the bug and record the exact steps, inputs, and environment that trigger it
- Collect error messages, stack traces, logs, console output, and network requests
- Find the code paths involved, with file paths and line references
- Check dependency versions and configuration that could matter
- Search the internet for known issues, such as bug trackers, changelogs, and forum threads, each with a link

The investigator returns its findings, says whether it reproduced the bug, and lists what it could not check and why.

If the investigator can't reproduce the bug, the coordinator asks the user for more details before moving on. The analysis can continue without a reproduction only if the evidence is strong enough, and the final report must say so.

### 3. Analysis

Brief the analyst with the bug description and the full investigation report. The analyst lists the possible causes, ranks them by likelihood, and tests them one at a time.

To test a hypothesis, the analyst may add logging, write small reproduction scripts, or change code in a temporary directory outside the source tree, such as a scratch copy of the affected files. The analyst never edits source files.

When the evidence points to several unrelated causes, the coordinator may run up to three analysts in parallel, each testing a different hypothesis.

A cause counts as confirmed only when the analyst can show it. For example, a change in the scratch copy makes the bug disappear, or a script triggers the bug on demand through that code path. If the analyst only has a strong suspicion, it reports it as unconfirmed.

The analyst returns:

- Each hypothesis it tested, what it did, and the result
- The confirmed root cause, with file paths and line references
- The evidence that confirms it
- Other places in the code with the same problem, if any
- Possible fix directions, without choosing one

Analysts delete their temporary files before returning.

If no hypothesis is confirmed, the coordinator sends the new evidence back to the investigator for one more round. Stop after three rounds and report what is known.

### 4. Report

The coordinator sends the user a final reply with:

- The root cause, or the strongest unconfirmed hypothesis, stated plainly
- Whether the bug was reproduced, and the steps to reproduce it
- The evidence, including key logs and the hypotheses ruled out
- Other places with the same problem
- A squad brief, ready to paste, that includes the bug, the root cause, the affected files, the reproduction steps, and the fix directions

The coordinator stops after the report. The user decides whether to ask questions, investigate further, or start squad.

## Delegate permissions

All delegates may open a browser, start local dev servers, emulators, and simulators, and run non-destructive commands such as builds, type checks, linters in check-only mode, test suites, and read-only diagnostics.

No delegate edits source files. Delegates may write only to their temporary directories.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside temporary directories or build output
- Installing, upgrading, or removing project dependencies. Delegates may install packages inside their temporary directories.
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

Investigate only the reported bug. If delegates find unrelated problems, list them as follow-ups in the final reply.

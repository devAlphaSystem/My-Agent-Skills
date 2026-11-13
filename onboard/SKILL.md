---
name: onboard
description: Trigger this if the user says 'onboard'.
---

# Onboard

Use this skill to map a project and save the result in `PROJECT.md` at the workspace root. Other skills read this file to write shorter and more accurate briefs. Run it once per project, and again when the project changes a lot. Never start it automatically.

## Roles

The coordinator is the agent in the conversation. It briefs each delegate, merges their results, writes `PROJECT.md`, and reports to the user. The coordinator does not map the project on its own, except for small clarifications needed to write a brief.

Use these exact models and reasoning efforts:

| Role     | Model                  |
| -------- | ---------------------- |
| Mapper   | GPT 6.1 Sol xhigh      |
| Verifier | Claude Sonnet 5.5 high |

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh, so every brief must include all the context it needs.

## Workflow

### 1. Check the current state

If `PROJECT.md` already exists, read it and tell the user that this run will update it. Keep any section marked as written by the user. If the workspace has several projects, list them and ask the user which ones to map, unless the user already said.

### 2. Mapping

Brief the mapper with the workspace path and the projects to map. The mapper reads files and runs read-only commands. It does not edit files.

The mapper must return:

- The project type, languages, frameworks, and their versions
- The package manager and runtime versions, such as Node, Python, or Java
- Commands to install, build, run, test, lint, type check, and format, taken from scripts, Makefiles, and config files
- The folder structure, with one line on what each main folder holds
- Entry points, such as the main file, routes, CLI commands, or API endpoints
- Code conventions, such as naming, file layout, state management, error handling, and test location
- External services the project talks to, such as databases, queues, and third-party APIs, with the config files that set them up
- Required environment variables, by name only
- Recent activity from `git log`, such as the most changed areas
- Anything unclear or contradictory, such as two test runners or stale scripts

### 3. Verification

Brief the verifier with the mapper's report. The verifier checks each claim against the code and runs the build, test, lint, and type check commands to confirm that they work. It does not edit files.

The verifier returns:

- Each command it ran, whether it worked, how long it took, and the error if it failed
- Claims from the report that are wrong, with the correct information
- Important things the mapper missed

If the verifier finds major gaps, the coordinator sends them back to the mapper for one more round.

### 4. Write PROJECT.md

The coordinator writes `PROJECT.md` with these sections:

- Overview: what the project is, in two or three sentences
- Stack: languages, frameworks, runtimes, and versions
- Commands: each command and whether it was verified
- Structure: main folders and entry points
- Conventions: the patterns new code should follow
- External services and environment variables, by name only
- Known issues: failing commands, contradictions, and open questions
- Last updated: the date of this run

In multi-project workspaces, write one section per project, or one `PROJECT.md` per project folder if the user prefers.

Run the text through the unslop rules before saving.

### 5. Report

The coordinator tells the user:

- Where it saved `PROJECT.md`, and whether it created or updated it
- Which commands failed during verification
- Open questions that only the user can answer

## Secrets

Never write secret values in `PROJECT.md` or in any report. This includes values found in `.env` files, config files, and Git history. List environment variables by name and cite only the file where they appear.

## Delegate permissions

All delegates may run non-destructive commands, such as builds, type checks, linters in check-only mode, test suites, local dev servers, and read-only diagnostics.

No delegate edits files. Only the coordinator writes, and only `PROJECT.md`.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside build output
- Installing, upgrading, or removing dependencies. If the build needs an install first, the verifier reports it and skips the command.
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

Map the project only. Do not fix failing commands or suggest refactors. List problems under known issues.

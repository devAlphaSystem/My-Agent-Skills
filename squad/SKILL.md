---
name: squad
description: Trigger this if the user says 'squad'.
---

# Squad

Use this skill when the user asks for a fix, addition, change, or removal in the workspace. The coordinator runs the work through delegates with fixed roles: research, planning, and implementation.

## Roles

The coordinator is the agent in the conversation. It talks to the user, briefs each delegate, passes results between them, and reports back. The coordinator does not research, plan, or implement on its own, except for small clarifications needed to write a brief.

Use these exact models with xhigh reasoning effort:

| Role        | Model                   |
| ----------- | ----------------------- |
| Researcher  | GPT 6.1 Sol xhigh       |
| Planner     | Claude Sonnet 5.5 xhigh |
| Implementer | Claude Opus 5.5 xhigh   |

Do not add, replace, or substitute models, even if the user suggests others. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their xhigh reasoning effort, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh, so every brief must include all the context it needs.

## Workflow

The workflow runs from start to finish without waiting for user approval. Stop and ask the user only when the request is unclear or a step needs a permission this skill does not grant.

### 1. Understand the request

Restate the request to the user in one or two sentences. If the goal, scope, or affected project is unclear, ask before starting any delegate.

### 2. Research

Brief the researcher with the request, the project type, and the relevant paths. The researcher only gathers information. It does not propose a solution or edit files.

The researcher must return:

- Files, functions, and modules the request touches, with paths and line references
- How the current code works in those areas, and any existing patterns or conventions to follow
- Dependencies, versions, and configuration that matter
- External references from the internet, such as official docs, changelogs, known issues, and examples, each with a link
- Open questions or unknowns it could not resolve

The coordinator reads the report, checks that it answers the request, and asks the researcher for follow-up if something important is missing.

### 3. Planning

Brief the planner with the request and the full research report.

The planner should explore more than one way to solve the problem. It may build and test prototypes, run benchmarks, and compare approaches. All prototypes go in a temporary directory outside the source tree, such as a scratch copy of the affected files. The planner never edits source files.

When the problem has several reasonable solutions, the coordinator may run up to three planners in parallel. Give each one the same brief plus a different direction to explore, such as "minimize changes", "optimize for performance", or "use the library's built-in feature". The coordinator then compares the plans and either picks the best one or merges the strongest parts into one plan. Record the reason for the choice.

Each planner must return a plan that covers:

- The approaches it tried, what it measured or observed for each, and why the chosen one wins
- Exact files to create, change, or delete, and what changes in each
- Performance impact
- Security impact, including input validation, auth, secrets, and data exposure
- UI and UX impact when the change is user-facing, including accessibility and responsive behavior
- Simplicity and maintainability tradeoffs
- Edge cases and failure modes
- How to verify the change, with the exact checks and commands to run

Planners delete their temporary prototypes before returning.

### 4. Implementation

Once the plan is final, brief the implementer with the request, the research report, and the plan. The implementer edits the workspace and follows the plan. If it finds that the plan is wrong or incomplete, it stops and reports the problem instead of improvising a different design.

The implementer must run the verification steps from the plan and return:

- A list of changed files with a short description of each change
- The output of each check it ran, and whether it passed
- Any deviation from the plan and the reason for it

If a check fails, the implementer fixes it and runs the check again. If the fix needs a design change, the coordinator sends the problem back to a planner, then passes the revised plan to the implementer.

### 5. Report

The coordinator sends the user a final reply with:

- What changed and where
- Which approaches the planners compared and why the chosen one won
- Verification results
- Deviations from the plan
- Open issues or follow-ups, if any

## Delegate permissions

All delegates may open a browser to read docs and inspect local or deployed pages. All delegates may run non-destructive commands, such as builds, type checks, linters in check-only mode, test suites, local dev servers, emulators, and read-only diagnostics.

Only the implementer may edit source files, and only to carry out the plan. The researcher stays read-only. Planners may write only to their temporary prototype directories.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside the plan, temporary prototype directories, or build output
- Installing, upgrading, or removing dependencies in the project. Planners may install packages inside a temporary prototype directory to test an approach. The implementer may change project dependencies only if the plan says so.
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

Keep the work limited to the user's request. If the researcher or a planner suggests unrelated improvements, list them as follow-ups in the final reply and leave them out of the plan.

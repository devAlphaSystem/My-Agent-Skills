---
name: council
description: Trigger this if the user says 'council'.
---

# Council

Before an audit, fix known issues and run relevant checks without using Git. Give council the updated implementation; do not defer known fixes until reviews return. If the user wants to discard the implementation and explore new ideas, skip fixes to code being replaced.

Bring in exactly two delegates to help with the user's request, usually for review or feedback. Always use these exact models with xhigh reasoning effort:

- Claude Sonnet 5.5 xhigh
- GPT 6.1 Sol xhigh

Do not add, replace, or substitute models, even if the user suggests other council members. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a council member and does not count toward the two delegates. It returns a link for each source, and the agent that started it checks the source before relying on the result. If either required model or reasoning level is unavailable, report the limitation rather than substituting.

In T3 Code, use `orchestrator_capabilities` to resolve the two exact models and their xhigh reasoning effort, then `delegate_task` to start both delegates in parallel. Elsewhere, use the harness's subagent tools.

## Supported projects

The council works with any kind of project, including:

- Websites and web apps
- Native desktop apps
- Mobile apps (iOS, Android, cross-platform)
- CLIs and scripts
- Libraries, APIs, backends, and services
- Monorepos and multi-project workspaces

Adapt the review to the project type. Use the checks, tooling, and verification methods that fit the platform, such as browser inspection for websites, emulators or simulators for mobile, running the binary for desktop apps, and sample invocations for CLIs and scripts. In multi-project workspaces, identify which projects the request affects and focus there.

## Delegate permissions

Delegates may do the following when it helps the audit:

- **Open a browser** to view websites, web apps, local dev servers, docs, or deployed pages. They may navigate, inspect rendered output, check responsiveness, read console output, and take screenshots.
- **Run non-destructive commands** related to the audit, such as:
  - Builds, type checks, linters, and formatters in check-only mode
  - Test suites
  - Starting local dev servers, emulators, simulators, or the app itself
  - Running CLIs or scripts with safe sample inputs
  - Reading logs, listing files, and inspecting dependencies
  - Read-only diagnostics, profiling, and audit tools such as Lighthouse, accessibility checkers, and `npm audit`

Delegates must not run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside temporary or build output
- Auto-fixing or rewriting source files, such as `--fix` or `--write` modes
- Installing, upgrading, or removing dependencies unless the coordinator explicitly approves it
- Database migrations, resets, or writes to real data
- Deploying, publishing, releasing, or pushing anywhere
- Sending real emails, payments, notifications, or calls to production APIs that change state
- Any Git or version-control operation

If a delegate isn't sure whether a command is safe, it should skip it and explain what it would have run and why.

## Git restriction

The coordinator and all delegates may run only these read-only Git commands:

- `git --no-pager --no-optional-locks status`
- `git --no-pager diff --no-ext-diff --no-textconv`
- `git --no-pager show --no-ext-diff --no-textconv`
- `git --no-pager log`

Arguments that only filter or format the output are allowed, such as paths, commit ranges, `--stat`, `--oneline`, `-p`, and `-n`. Never use `--output` or any option that writes files.

Never run any other Git command or version-control operation. This includes commit, add, checkout, switch, restore, reset, stash, merge, rebase, pull, push, fetch, branch, tag, and clean. This restriction applies to the coordinator and all delegates, including their tools and subprocesses. Make all changes by editing workspace files directly.

## Running the council

Give each delegate the request and relevant context, including the project type, how to build or run it, and any URLs or entry points to check. For review or feedback, keep their work read-only with respect to source files. They may still browse and run non-destructive commands as described above. Collect their responses and summarize the useful findings, disagreements, and recommended next steps.

For a code review, ask each delegate for a score out of 10. Count each full review pass as one round. A fresh review after fixes starts a new round.

Include the council review results in the final reply, using this format:

```markdown
## Council review

Rounds: 2

| Delegate                | Round 1 | Round 2 |
| ----------------------- | ------- | ------- |
| Claude Sonnet 5.5 xhigh | 7/10    | 9/10    |
| GPT 6.1 Sol xhigh       | 8/10    | 10/10   |
```

Adjust the rounds, scores, and table columns to match the actual reviews. If a delegate gives no score, write `no score`. Do not invent one.

If you agree with their recommendations, make the changes and request a fresh review after. Continue this process until both delegates are satisfied with the outcome.

Do not let repeated reviews result in scope creep. It is your job to fight to keep scope limited.

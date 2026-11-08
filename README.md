# Skills

Reusable agent skills for project mapping, implementation, debugging, code review, testing, delegate model selection, shell command logging, and editing prose. Each directory contains a `SKILL.md` file with the skill's trigger, workflow, and rules.

I use these skills in T3 Code with Codex on the Plus plan and Claude on the Pro plan. Some skills depend on T3 Code's `delegate_task` to coordinate Codex and Claude delegates in the same workflow. T3 Code is required to run those skills as configured in my setup.

## Available skills

### Original skills - T3 Code

| Skill                             | Purpose                                                                                | Result                                                                                      |
| --------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [onboard](onboard/SKILL.md)       | Map a project's structure, stack, commands, and conventions, then verify the findings. | A `PROJECT.md` file at the workspace root.                                                  |
| [squad](squad/SKILL.md)           | Research, plan, and implement a requested change through delegates with fixed roles.   | Workspace changes and a verification report.                                                |
| [debug](debug/SKILL.md)           | Investigate a bug with an unknown cause and test possible explanations.                | Evidence, a confirmed cause or an explicitly unconfirmed hypothesis, and a brief for squad. |
| [qa](qa/SKILL.md)                 | Run automated and exploratory testing, send confirmed problems to a fixer, and retest. | A report of test results, fixes, limitations, and follow-ups.                               |
| [council](council/SKILL.md)       | Get independent feedback from two delegates and review again after accepted fixes.     | Review findings and, for code reviews, scores for each round.                               |
| [pickmodel](pickmodel/SKILL.md)   | Choose the model for every delegate when used with a delegation workflow.              | User-selected models with `xhigh` effort, reported by role.                                 |
| [pragmatic](pragmatic/SKILL.md)   | Keep effort proportional to practical benefit and defect risk.                         | Useful improvements and disclosed minor issues without disproportionate investigation.      |
| [commandlog](commandlog/SKILL.md) | Record every shell command each agent runs, with its expected outcome and reason.      | A `commands.json` file in `CommandLogs` on the Desktop, grouped by workspace.               |

### Third-party skills

Skills by other authors, included here with links to their original sources.

| Skill                     | Purpose                                                              | Result                                       | Source                                                                                     |
| ------------------------- | -------------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------ |
| [unslop](unslop/SKILL.md) | Remove common AI writing patterns while preserving meaning and tone. | Edited prose using plain, concrete language. | [Cursor pstack](https://github.com/cursor/plugins/blob/main/pstack/skills/unslop/SKILL.md) |

## Usage

Make the desired skill directories available to your agent's skill loader, keeping each `SKILL.md` inside its directory. The loading mechanism depends on the host. This repository contains instruction files and has no installation script or application to build.

For the workflow skills, include the skill name in your request and describe the task:

```text
onboard this project and document its structure and verified commands.

squad add input validation to the settings form using the existing patterns.

debug why saving the settings form fails with an empty optional field.

qa the settings form changes, including invalid inputs and error states.

council review the settings form changes for correctness and maintainability.

pickmodel squad add input validation to the settings form using the existing patterns.

pragmatic squad improve the settings form using the existing patterns.

pragmatic council review the changes and keep effort proportional to practical impact.

pragmatic qa test the changes and defer rare, low-impact defects that need disproportionate effort.

commandlog squad add input validation to the settings form using the existing patterns.

commandlog debug why saving the settings form fails with an empty optional field.
```

Invoke `unslop` explicitly through your host's skill mechanism and provide the text to edit. Its metadata sets `disable-model-invocation: true`.

Workflow skills run as separate requests. `debug` produces a brief for squad but does not implement a fix or start squad. `onboard` and `qa` also require an explicit request. Use `pickmodel` together with `onboard`, `squad`, `debug`, `qa`, or `council` in the same request to choose delegate models. Use `commandlog` together with any of them, or with an ordinary task that runs shell commands, to log those commands.

## Models and orchestration

By default, the five delegation workflows require these exact models, all with `xhigh` reasoning effort:

| Skill   | Roles and models                                                                              |
| ------- | --------------------------------------------------------------------------------------------- |
| onboard | Mapper: GPT 6.1 Sol. Verifier: Claude Sonnet 5.5.                                             |
| squad   | Researcher: GPT 6.1 Sol. Planner: Claude Sonnet 5.5. Implementer: Claude Opus 5.5.            |
| debug   | Investigator: GPT 6.1 Sol. Analyst: Claude Sonnet 5.5.                                        |
| qa      | Automated tester: GPT 6.1 Sol. Exploratory tester: Claude Sonnet 5.5. Fixer: Claude Opus 5.5. |
| council | Exactly two delegates: Claude Sonnet 5.5 and GPT 6.1 Sol.                                     |

These model assignments are required unless `pickmodel` is invoked. Without `pickmodel`, if a required model or reasoning level is unavailable, report the limitation instead of substituting another model.

With `pickmodel`, the user chooses each delegate's model through a selectable-options tool before it starts. The only allowed models are Claude Opus 5.5, Claude Sonnet 5.5, and GPT 6.1 Sol, always with `xhigh` effort. The workflow's assigned model becomes the recommended option when it is allowed and available. Unavailable models are excluded; if none are available, the workflow stops.

Every new delegate requires a fresh answer, including later review rounds and replacements. All choices for a parallel group must be collected before any delegate in that group starts. Async questions require an actual user response. Pickmodel overrides model assignments only; roles, prompts, delegate counts, workflow, scope, and permissions remain unchanged. The final report includes each delegate's role, model, and effort.

In T3 Code, the skills use `orchestrator_capabilities` to resolve the models and `delegate_task` to start delegates. Using them in another host requires equivalent delegation support for the required models. `unslop` does not define delegate roles or fixed model requirements.

## Choosing a workflow

Use `onboard` when a project needs a shared map in `PROJECT.md`. Use `squad` when the requested change is ready for research, planning, and implementation. If a bug's cause is unknown, start with `debug`, then pass its findings to squad in a separate request.

After implementation, request `qa` to exercise the changed behavior or `council` for review and feedback. Use `unslop` when editing documentation or other prose.

Add `pickmodel` to a delegation workflow when you want to select the model for each delegate.

Add `pragmatic` to keep investigation, fixes, and optimization proportional to practical impact. Clear, low-cost improvements remain required. It overrides optional performance work and permits deferring rare, minor defects when a proper fix would take disproportionate effort. It also avoids speculative scale and unsupported compatibility work, and permits deferring rare, minor defects when a fix would put common flows at greater risk. Documented capacity and support commitments still apply. Known issues and failed checks remain visible. Rarity does not excuse security problems, destructive consequences, or unmet explicit requirements. Hours of work or hundreds of thousands of tokens for an expected gain of 1% to 5% require an explicit request for that depth of optimization. It starts no delegates and can also guide work without another skill.

Add `commandlog` to keep an audit trail of shell commands. It changes no roles, models, workflow, or scope of the other skill, and it does not review, block, or approve commands. It starts no delegates.

## Command log

With `commandlog`, every agent, including the coordinator and all delegates, saves an entry before it runs each shell command. Non-shell tool calls and calls to the logging helper are not recorded. If an entry cannot be saved, the command does not run.

- **Location:** `<Desktop>\CommandLogs\<workspace folder name>\commands.json`, where `<Desktop>` comes from PowerShell's `GetFolderPath('Desktop')`. Setting `COMMANDLOG_ROOT` replaces the log root.
- **Format:** one JSON array of entries with exactly four fields: `timestamp` (set by the helper), `command`, `expectedOutcome`, and `reason`. Agents write the last two as one short English sentence each.
- **Helper:** the coordinator creates `log_command.ps1` in the log root. It uses a named mutex and an atomic file replace, so parallel delegates can append to the same file safely. Entries are only appended, never edited, reordered, or deleted.
- **Secrets:** tokens, passwords, and API keys inside a command are replaced with `<redacted>`.
- **Final report:** states the log file path, the number of entries recorded in the run, and any command that could not be recorded.

## Scope and permissions

Each skill defines its own role boundaries. Researchers, reviewers, and testers generally keep source files unchanged. Squad assigns source edits to its implementer; QA assigns fixes to its fixer. Onboard writes only `PROJECT.md`, and debug keeps experiments outside the source tree.

The delegation skills restrict Git to read-only status, diff, show, and log operations. Their files specify the permitted options. They also restrict destructive operations, production writes, publishing, and dependency changes. Project and session instructions may impose stricter limits on commands, servers, or file changes. When `commandlog` is active, agents may also write inside the log root, which is the only exception to these file rules.

Read the relevant `SKILL.md` before invoking a skill for its complete permissions, stopping conditions, and report format.

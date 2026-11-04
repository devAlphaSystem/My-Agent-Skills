# Skills

Reusable agent skills for project mapping, implementation, debugging, code review, testing, and editing prose. Each directory contains a `SKILL.md` file with the skill's trigger, workflow, and rules.

I use these skills in T3 Code with Codex on the Plus plan and Claude on the Pro plan. Some skills depend on T3 Code's `delegate_task` to coordinate Codex and Claude delegates in the same workflow. T3 Code is required to run those skills as configured in my setup.

## Available skills

### Original skills - T3 Code

| Skill                       | Purpose                                                                                | Result                                                                                      |
| --------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [onboard](onboard/SKILL.md) | Map a project's structure, stack, commands, and conventions, then verify the findings. | A `PROJECT.md` file at the workspace root.                                                  |
| [squad](squad/SKILL.md)     | Research, plan, and implement a requested change through delegates with fixed roles.   | Workspace changes and a verification report.                                                |
| [debug](debug/SKILL.md)     | Investigate a bug with an unknown cause and test possible explanations.                | Evidence, a confirmed cause or an explicitly unconfirmed hypothesis, and a brief for squad. |
| [qa](qa/SKILL.md)           | Run automated and exploratory testing, send confirmed problems to a fixer, and retest. | A report of test results, fixes, limitations, and follow-ups.                               |
| [council](council/SKILL.md) | Get independent feedback from two delegates and review again after accepted fixes.     | Review findings and, for code reviews, scores for each round.                               |

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
```

Invoke `unslop` explicitly through your host's skill mechanism and provide the text to edit. Its metadata sets `disable-model-invocation: true`.

Skills run as separate requests. `debug` produces a brief for squad but does not implement a fix or start squad. `onboard` and `qa` also require an explicit request.

## Models and orchestration

The five delegation workflows require these exact models, all with `xhigh` reasoning effort:

| Skill   | Roles and models                                                                              |
| ------- | --------------------------------------------------------------------------------------------- |
| onboard | Mapper: GPT 6.1 Sol. Verifier: Claude Sonnet 5.5.                                             |
| squad   | Researcher: GPT 6.1 Sol. Planner: Claude Sonnet 5.5. Implementer: Claude Opus 5.5.            |
| debug   | Investigator: GPT 6.1 Sol. Analyst: Claude Sonnet 5.5.                                        |
| qa      | Automated tester: GPT 6.1 Sol. Exploratory tester: Claude Sonnet 5.5. Fixer: Claude Opus 5.5. |
| council | Exactly two delegates: Claude Sonnet 5.5 and GPT 6.1 Sol.                                     |

These are required model selections, not suggested alternatives. If a required model or reasoning level is unavailable, report the limitation instead of substituting another model.

In T3 Code, the skills use `orchestrator_capabilities` to resolve the models and `delegate_task` to start delegates. Using them in another host requires equivalent delegation support for the required models. `unslop` does not define delegate roles or fixed model requirements.

## Choosing a workflow

Use `onboard` when a project needs a shared map in `PROJECT.md`. Use `squad` when the requested change is ready for research, planning, and implementation. If a bug's cause is unknown, start with `debug`, then pass its findings to squad in a separate request.

After implementation, request `qa` to exercise the changed behavior or `council` for review and feedback. Use `unslop` when editing documentation or other prose.

## Scope and permissions

Each skill defines its own role boundaries. Researchers, reviewers, and testers generally keep source files unchanged. Squad assigns source edits to its implementer; QA assigns fixes to its fixer. Onboard writes only `PROJECT.md`, and debug keeps experiments outside the source tree.

The delegation skills restrict Git to read-only status, diff, show, and log operations. Their files specify the permitted options. They also restrict destructive operations, production writes, publishing, and dependency changes. Project and session instructions may impose stricter limits on commands, servers, or file changes.

Read the relevant `SKILL.md` before invoking a skill for its complete permissions, stopping conditions, and report format.

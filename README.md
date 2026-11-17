# Skills

Reusable agent skills for project mapping, implementation, debugging, code review, testing, running web and Android apps to verify changes, keeping interface work consistent with existing patterns, delegate model selection, and editing prose. Each directory contains a `SKILL.md` file with the skill's trigger, workflow, and rules.

I use these skills in T3 Code with Codex on the Plus plan and Claude on the Max plan. Some skills depend on T3 Code's `delegate_task` to coordinate Codex and Claude delegates in the same workflow. T3 Code is required to run those skills as configured in my setup.

## Available skills

### Original skills - T3 Code

| Skill                                           | Purpose                                                                                                                                                                                                                  | Result                                                                                                                    |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| [onboard](onboard/SKILL.md)                     | Map a project's structure, stack, commands, and conventions, then verify the findings.                                                                                                                                   | A `PROJECT.md` file at the workspace root.                                                                                |
| [squad](squad/SKILL.md)                         | Research, plan, and implement a requested change through delegates with fixed roles.                                                                                                                                     | Workspace changes and a verification report.                                                                              |
| [debug](debug/SKILL.md)                         | Investigate a bug with an unknown cause and test possible explanations.                                                                                                                                                  | Evidence, a confirmed cause or an explicitly unconfirmed hypothesis, and a brief for squad.                               |
| [qa](qa/SKILL.md)                               | Run automated and exploratory testing, send confirmed problems to a fixer, and retest.                                                                                                                                   | A report of test results, fixes, limitations, and follow-ups.                                                             |
| [test-web-app](test-web-app/SKILL.md)           | Run a web project's app on isolated data and test a change or flow through two testers, without fixing anything.                                                                                                         | A report of test results, problems, limitations, and follow-ups, with everything the run started stopped.                 |
| [test-android-app](test-android-app/SKILL.md)   | Build and install an Android project's debug app and test a change or flow through two testers, without fixing anything.                                                                                                 | A report of test results, problems, limitations, and follow-ups, with everything the run started stopped.                 |
| [test-web-full](test-web-full/SKILL.md)         | From a new chat with no knowledge of the project, map a web project, verify the map on the running app, and test every unit in three passes, without fixing anything.                                                    | A run directory with the map, ledger, findings, and report, and everything the run started stopped.                       |
| [test-android-full](test-android-full/SKILL.md) | From a new chat with no knowledge of the project, map an Android app, verify the map on a device, and test every unit in three passes, without fixing anything.                                                          | A run directory with the map, ledger, findings, and report, and everything the run started stopped.                       |
| [council](council/SKILL.md)                     | Get independent feedback from two delegates and review again after accepted fixes.                                                                                                                                       | Review findings and, for code reviews, scores for each round.                                                             |
| [pickmodel](pickmodel/SKILL.md)                 | Choose the model for every delegate when used with a delegation workflow.                                                                                                                                                | User-selected models and efforts, reported by role.                                                                       |
| [pragmatic](pragmatic/SKILL.md)                 | Keep effort proportional to practical benefit and defect risk.                                                                                                                                                           | Useful improvements and disclosed minor issues without disproportionate investigation.                                    |
| [ui-consistency](ui-consistency/SKILL.md)       | Make new or changed interface reuse the project's own components, variants, fixed properties, and flows, and keep the rules in UI.md. Saying the skill's name adds reviewers, screenshots, measurements, and recordings. | A `UI.md` file at the workspace root that grows with each interface task, and a conformance summary of the checks it ran. |

The four test skills for apps adapt the approach of the `test-t3-app` and `test-t3-mobile` skills from [T3 Code](https://github.com/pingdotgg/t3code/tree/main/.agents/skills) (MIT license) to any project, with the permission rules of this repository.

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

test-web-app check that the settings form saves an empty optional field and rejects invalid input.

test-android-app check that the login screen keeps its state after a rotation.

test-web-full test the whole project and report every problem.

test-android-full test the whole app on the emulator and report every problem.

ui-consistency map this project's interface patterns and save them in UI.md.

ui-consistency squad add a customers page with an edit drawer and a delete confirmation.

council review the settings form changes for correctness and maintainability.

pickmodel squad add input validation to the settings form using the existing patterns.

pragmatic squad improve the settings form using the existing patterns.

pragmatic council review the changes and keep effort proportional to practical impact.

pragmatic qa test the changes and defer rare, low-impact defects that need disproportionate effort.
```

Invoke `unslop` through your host's skill mechanism and provide the text to edit. The agent can also select it on its own when asked to make writing sound less AI-generated, and `onboard` applies its rules before saving `PROJECT.md`.

The agent can also apply `ui-consistency` at a light level whenever a task builds or changes an interface's appearance or behavior. Its examples are not an exhaustive checklist: it discovers the project's actual patterns for the affected components, composition, states, and interactions, including client-only behavior and project-specific elements the skill does not name. Whether the host loads the skill on its own is up to the host, so this is not guaranteed. At the light level the agent does the discovery, the decisions, the checks, and the recording itself, starts no extra delegates, and opens no browser. It checks the behavior in code only and says so. It records what it finds in `UI.md`, and creates the file when it does not exist.

Saying `ui-consistency` with a task adds two delegates, a code reviewer and a visual checker, using platform-appropriate screenshots, measurements, recordings, or observed interactions. Web checks use the browser; native Android checks use a test device and supported configurations, without requiring desktop widths or DOM measurements. This costs more time and tokens, and it runs the project's app locally. A real or shared database needs your detailed permission first, and without the app, the required browser or device tools, or test accounts the run falls back to code checks and reports what stayed unconfirmed at runtime. Each review round reruns only the reviewers that the fix affects, with at most three rounds. A small change can skip the reviewers, and the report says so.

Reusing a component does not prove its states are usable. Runtime checks verify that feedback is perceivable, required controls are reachable, and input navigation works through the interaction's boundaries, including forward and reverse keyboard traversal on web. Code-only and headless DOM checks leave those properties unconfirmed. A material failure in the requested flow remains a failed check when inherited from a reference; fix its authorized root cause and check affected uses, or report it unresolved. Defects outside the task stay follow-ups, and these checks do not authorize a broad redesign.

What the skill does with `UI.md` when you say `ui-consistency` with a task:

- If `UI.md` does not exist, it creates focused entries for the task and offers project-wide mapping without blocking the task. Mapping starts only when you accept the offer or explicitly request it.
- If `UI.md` exists but holds only entries from earlier tasks, it asks once per project whether to map the whole project first. If you decline or do not answer, it continues with the focused task and does not start the mapping.
- If `UI.md` already holds mapped entries, it reuses them and verifies the ones the task touches against the code, without mapping again.

Saying `ui-consistency` with no task, or explicitly requesting project-wide mapping, maps the project and confirms the properties and interactions it can check in the running app. Updating a specific pattern stays focused on that entry.

When the project's instruction file does not mention `UI.md`, the skill asks once per project before appending one line that tells agents to read the relevant `UI.md` entries before interface work. It edits nothing without your approval, and it never creates an instruction file. If you decline, it records that and does not ask again. Whether a host loads that file is up to the host, so the line does not guarantee that other agents read `UI.md`.

Workflow skills run as separate requests. `debug` produces a brief for squad but does not implement a fix or start squad. `onboard`, `qa`, `test-web-app`, `test-android-app`, `test-web-full`, and `test-android-full` also require an explicit request. Use `pickmodel` together with `onboard`, `squad`, `debug`, `qa`, `test-web-app`, `test-android-app`, `council`, or `ui-consistency` when it starts delegates, in the same request to choose delegate models.

## Models and orchestration

Only these models and efforts are allowed:

| Model             | Effort            | Used for                                                                                                                                                                                                                                                                                                                                                                                                                                       | Edits code     |
| ----------------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| Claude Opus 5.5   | `xhigh`           | The most capable model, and the strongest at open-ended work that needs sustained judgment. The right choice for heavier problems, such as debugging, and planning and implementing high complexity tasks.                                                                                                                                                                                                                                     | Yes            |
| Claude Sonnet 5.5 | `xhigh`           | Creates and edits code intelligently. Better at code than GPT 6.2 Sol, so it edits code by default, mostly for new code of medium to high complexity.                                                                                                                                                                                                                                                                                          | Yes            |
| Claude Sonnet 5.5 | `high`            | The best middle ground between cost and speed. Default for talking directly with the user, explanations, quick code fixes, and checking other models' work.                                                                                                                                                                                                                                                                                    | Yes            |
| Claude Haiku 5.5  | `high`            | Very fast and very cheap, good at reading many files quickly. Much weaker than GPT 6.2 Sol and Sonnet 5.5, and hallucinates more than other recent Anthropic models, so it is never the only check on another model's work. Narrow read-only tasks only: objective evaluations, such as running a command and recording the result, summaries, and light information lookups, including internet searches. Any agent can start it at any time. | Never          |
| GPT 6.2 Sol       | `high` or `xhigh` | Main information aggregator. Strong at creating and editing code, but Sonnet 5.5 is better, so it does not edit by default. Studies the code more, so its opinion matters in council and squad. Runs tests too. Workflows use `xhigh` by default; `high` applies when the user chooses it.                                                                                                                                                     | Not by default |

By default, the ten delegation workflows require these exact models:

| Skill             | Roles and models                                                                                                                                                                                                                                     |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onboard           | Mapper: GPT 6.2 Sol xhigh. Verifier: Claude Sonnet 5.5 high.                                                                                                                                                                                         |
| squad             | Researcher: GPT 6.2 Sol xhigh. Planner: Claude Sonnet 5.5 xhigh, or Claude Opus 5.5 xhigh for high complexity tasks. Implementer: Claude Sonnet 5.5 xhigh, or Claude Opus 5.5 xhigh when the coordinator or the plan rates the task high complexity. |
| debug             | Investigator: GPT 6.2 Sol xhigh. Analyst: Claude Opus 5.5 xhigh.                                                                                                                                                                                     |
| qa                | Automated tester: GPT 6.2 Sol xhigh. Exploratory tester: Claude Sonnet 5.5 xhigh. Fixer: Claude Sonnet 5.5 xhigh, or Claude Opus 5.5 xhigh for high complexity problems.                                                                             |
| test-web-app      | Automated tester: GPT 6.2 Sol xhigh. Exploratory tester: Claude Sonnet 5.5 xhigh.                                                                                                                                                                    |
| test-android-app  | Automated tester: GPT 6.2 Sol xhigh. Exploratory tester: Claude Sonnet 5.5 xhigh.                                                                                                                                                                    |
| test-web-full     | Mapper: GPT 6.2 Sol xhigh. Verifier: Claude Sonnet 5.5 high. Automated tester: GPT 6.2 Sol xhigh. Exploratory tester: Claude Sonnet 5.5 xhigh.                                                                                                       |
| test-android-full | Mapper: GPT 6.2 Sol xhigh. Verifier: Claude Sonnet 5.5 high. Automated tester: GPT 6.2 Sol xhigh. Exploratory tester: Claude Sonnet 5.5 xhigh.                                                                                                       |
| council           | Exactly two delegates: Claude Sonnet 5.5 xhigh and GPT 6.2 Sol xhigh.                                                                                                                                                                                |
| ui-consistency    | Mapping: Mapper: GPT 6.2 Sol xhigh. Verifier: Claude Sonnet 5.5 high. Visual checker: Claude Sonnet 5.5 xhigh. Applying, full level: Code reviewer: GPT 6.2 Sol xhigh. Visual checker: Claude Sonnet 5.5 xhigh. The light level adds no delegates.   |

The squad and qa files define the objective criteria for high complexity, such as open-ended requests, changes across several modules, and costly or hard to undo mistakes.

Any agent, including the coordinator and the delegates, can start Claude Haiku 5.5 high at any time for light information lookups, including internet searches. This needs no permission, is not a delegate role, does not count toward council's two delegates, and needs no `pickmodel` question. Haiku stays read-only. It returns a link for each source, and the agent that started it checks the source before relying on the result.

These model assignments are required unless `pickmodel` is invoked. Without `pickmodel`, if a required model or reasoning level is unavailable, report the limitation instead of substituting another model.

With `pickmodel`, the user chooses each delegate's model through a selectable-options tool before it starts, then chooses the effort when the model allows more than one (Claude Sonnet 5.5 and GPT 6.2 Sol, high or xhigh). Claude Haiku 5.5 is not offered for roles that edit source files. GPT 6.2 Sol is offered for them but is never the recommended option. The workflow's assigned model and effort become the recommended options when they are allowed and available. Unavailable models are excluded; if none are available, the workflow stops.

Every new delegate requires a fresh answer, including later review rounds and replacements. All choices for a parallel group must be collected before any delegate in that group starts. Async questions require an actual user response. Pickmodel overrides model assignments only; roles, prompts, delegate counts, workflow, scope, and permissions remain unchanged. The final report includes each delegate's role, model, and effort.

In T3 Code, the skills use `orchestrator_capabilities` to resolve the models and `delegate_task` to start delegates. Using them in another host requires equivalent delegation support for the required models. `unslop` does not define delegate roles or fixed model requirements. The testers and verifiers of the four test skills for apps, and the visual checker of `ui-consistency`, use T3 Code's browser (`preview_*`) and device (`device_*`) tools when available, and the skills say what to do in other hosts.

## Choosing a workflow

Use `onboard` when a project needs a shared map in `PROJECT.md`. Use `squad` when the requested change is ready for research, planning, and implementation. If a bug's cause is unknown, start with `debug`, then pass its findings to squad in a separate request.

After implementation, request `qa` to exercise the changed behavior or `council` for review and feedback. Use `test-web-app` or `test-android-app` to run the app on isolated data and have two testers check a specific change or flow. They report problems and fix nothing. Use `qa` instead when confirmed problems should be fixed and retested in the same run. Use `unslop` when editing documentation or other prose.

Use `test-web-full` or `test-android-full` in a new chat to test a whole project that the agent knows nothing about. They map the project, verify the map on the running app, and test every unit in three passes. A run can take hours and start many delegates, and invoking the skill is the explicit request for that depth. They report problems and fix nothing, and they ask their questions once, at the start. Do not combine them with `pickmodel`, which asks before every delegate.

Add `ui-consistency` to a request that builds or changes interface, with or without `squad`, so that the work reuses the project's shared elements, variants, and flows instead of recreating them. Saying it also has a code reviewer and a visual checker examine the result. The visual checker opens the running app, compares the new screen with the reference page, measures the fixed properties, and records the flows. It can also check a failed request by making one approved local request fail on the client. That simulates a network failure, not a backend error, and the report says so. Every run adds what it learns to `UI.md`, so the file grows with the project. Entries from a task cover only the comparable uses that task examined, and the file states the count and its scope. Run it once with a request to map the project; it inventories the patterns and confirms the properties and flows it can reach in the running app, listing anything unverified. The skill can offer to add one line to the project's instruction file that points agents to `UI.md`, but only with your approval. Pending and declined offers are recorded so the skill does not ask repeatedly.

Add `pickmodel` to a delegation workflow when you want to select the model for each delegate.

Add `pragmatic` to keep investigation, fixes, and optimization proportional to practical impact. Clear, low-cost improvements remain required. It overrides optional performance work and permits deferring rare, minor defects when a proper fix would take disproportionate effort. It also avoids speculative scale and unsupported compatibility work, and permits deferring rare, minor defects when a fix would put common flows at greater risk. Documented capacity and support commitments still apply. Known issues and failed checks remain visible. Rarity does not excuse security problems, destructive consequences, or unmet explicit requirements. Hours of work or hundreds of thousands of tokens for an expected gain of 1% to 5% require an explicit request for that depth of optimization. It starts no delegates and can also guide work without another skill.

## Scope and permissions

Each skill defines its own role boundaries. Researchers, reviewers, and testers generally keep source files unchanged. Squad assigns source edits to its implementer; QA assigns fixes to its fixer. Onboard writes only `PROJECT.md`, and debug keeps experiments outside the source tree. Besides the changes a task asks for, `ui-consistency` writes only `UI.md`. It edits the project's existing instruction file only to append the one line pointing to `UI.md` that you approved for that project, and it never creates an instruction file. Its reviewers edit nothing. Its coordinator prepares the runtime and starts and stops the test services or emulators. The visual checker may launch and relaunch the prepared native app through the device tools. The run stops only the app and processes it started. The screenshots and recordings go to an evidence directory outside the source tree, which it keeps because they are the deliverable. In the four test skills for apps, only the coordinator builds, starts, and stops the app, backend, bundler, and emulator. It stops everything the run started before it reports, with no exception, and asks for permission before the run uses a real or shared database. The testers never edit source files. The full variants also keep a run directory outside the source tree, with the map, ledger, findings, and report, and leave it in place because it is the deliverable.

The delegation skills restrict Git to read-only status, diff, show, and log operations. Their files specify the permitted options. They also restrict destructive operations, production writes, publishing, and dependency changes. Project and session instructions may impose stricter limits on commands, servers, or file changes.

Read the relevant `SKILL.md` before invoking a skill for its complete permissions, stopping conditions, and report format.

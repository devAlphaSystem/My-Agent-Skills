---
name: pickmodel
description: Let the user choose the model and reasoning effort of every delegate before it starts, overriding the fixed models of the other skill. Combine it with any skill that starts delegates. Use only when the user says 'pickmodel'.
---

# Pickmodel

Use this skill together with any other skill that starts delegates. It overrides how that skill assigns models, whatever the skill is and however it words its model rules, and the other skill does not need to mention pickmodel. Pickmodel takes precedence over every model rule in the other skill.

## Allowed models

Only these models are allowed, each with the reasoning effort shown:

| Model             | Effort allowed | Edits project files                        |
| ----------------- | -------------- | ------------------------------------------ |
| Claude Opus 5.5   | xhigh          | Yes                                        |
| Claude Sonnet 5.5 | high or xhigh  | Yes                                        |
| Claude Haiku 5.5  | high           | Never                                      |
| GPT 6.2 Sol       | high or xhigh  | Only when the user chooses it for the role |

Do not offer, accept, or start any other model or reasoning effort, even if the other skill names one or the user suggests one.

Claude Haiku 5.5 never edits project files. Do not offer it for a role that edits source files, such as the implementer in squad or the fixer in qa, or any role that the other skill lets change project files. Offer it only for read-only roles. GPT 6.2 Sol is strong at writing and editing code, but Claude Sonnet 5.5 is better, so no workflow assigns code edits to it. It is offered for roles that edit source files only because the user makes the choice, and it is never the recommended option for them. GPT 6.2 Sol may also create and run tests that do not change project source files.

## Override

The other skill assigns a fixed model to each role and says not to substitute it. While pickmodel is active, ignore those assignments. The user picks one of the allowed models, and the effort when the model allows more than one, for every delegate. If the other skill names an allowed model for a role, that model is the recommended option, and so is the effort the other skill names for it. If it names a model outside the allowed list, no option is recommended. If the other skill names more than one model for a role, such as an escalation rule, recommend the one its default rule selects. If it names no model for a role, no option is recommended.

Everything else in the other skill still applies, including roles, workflow, permissions, the Git restriction, and scope.

## Rule

Before every delegate starts, ask the user which model to use and, for Claude Sonnet 5.5 and GPT 6.2 Sol, which effort. This applies to every new delegate, with no exceptions:

- The first delegate and every later one.
- Each delegate in a parallel group.
- Repeat rounds, such as a fresh council review after fixes.
- Delegates that replace a failed or stopped delegate.

Never reuse an earlier answer. Never assume the user wants the same model as last time. Never start a delegate before the user answers.

One answer can cover several delegates in only one case: delegates with the same role start together in one parallel group, and the user chooses one choice for all of them (step 2 of the Steps). The answer covers only those delegates, which the question names. Every delegate that starts later asks again, whether it starts in another group or round, alone, or as a replacement.

The one exception is Claude Haiku 5.5 high used for light information lookups, including internet searches. Any agent can start it at any time without asking the user. It is not a workflow delegate, so it needs no question, is not counted in the final table of delegates, and stays read-only. It returns a link for each source, and the agent that started it checks the source before relying on the result.

## How to ask

Ask with the selectable-options tool of the harness. Never ask in plain text, and never write a multiple choice question as a chat message.

| Harness           | Tool                 |
| ----------------- | -------------------- |
| Codex in T3 Code  | `request_user_input` |
| Claude in T3 Code | `AskUserQuestion`    |
| Other harnesses   | The equivalent tool  |

Prefer the native tool for the current harness. In T3 Code, if neither `request_user_input` nor `AskUserQuestion` is usable in the current session (unavailable, disabled, or returning a technical error), use `request_user_input_async` if it is exposed. Do not assume another harness's tool is available. A user dismissal is not a technical failure; handle it under "If the user does not choose".

With `request_user_input_async`, put each question in `questions[].title` and its choices in `questions[].options`. Its `{ accepted: true }` result only confirms submission. Keep each question associated with its delegate and wait for the actual answer, which arrives as a later user message. Ask the effort question only after the model answer arrives. If needed, end the turn without starting delegates and resume after the answer arrives. Never treat submission, a preselected option, silence, or elapsed time as a choice.

If no working selectable-options tool remains, stop and explain which tools are unavailable or failed. Ask the user to enable a supported tool or switch to a mode that allows one. Do not fall back to a text question and do not start the delegate.

## Steps

1. Call `orchestrator_capabilities` to resolve the allowed models, their provider instances, and the efforts each one lists. If a model or effort is unavailable, tell the user which one is missing and offer only the others. If no allowed model is available for a role, stop and report it. A model that T3 Code warns has no usage left counts as unavailable. The other skill's GPT 6.2 Sol usage-limit fallback does not apply while pickmodel is active. A delegate that stops because its model ran out of usage is replaced by a new delegate, which gets a fresh question.
2. For each role that has two or more delegates in the parallel group about to start, ask first whether the user wants one choice for all of them or one for each. Put these questions in one call, one per role, with two options: "One choice for all N <role> delegates" and "A choice for each". Name the role and the number of delegates, and say what each delegate will do in one short sentence. Skip this step when every role in the group has a single delegate. A dismissal is handled like any other (see "If the user does not choose").
3. Ask the model question first, one per delegate you are about to start, or one per role when its delegates share one choice. Put all model questions for a parallel group in one call. If the tool limits the number of questions, split them into several calls.
4. Write each question so the user knows which delegate it is for. Name the role, such as "Researcher", "Analyst", or "Council member 1", and say what the delegate will do in one short sentence. For a role with one choice for all, name the role and the number of delegates, and say that the answer applies to all of them.
5. For a role that does not edit source files, give one option per available model, each with a short description:
   - Claude Opus 5.5 (xhigh)
   - Claude Sonnet 5.5 (high or xhigh)
   - Claude Haiku 5.5 (high)
   - GPT 6.2 Sol (high or xhigh)

   For a role that edits source files, offer only Claude Opus 5.5, Claude Sonnet 5.5, and GPT 6.2 Sol, and never mark GPT 6.2 Sol as recommended. Add "(Recommended)" to the label of the option that the other skill names for that role, if any, and list it first. Do not add any other option. If the tool adds an "Other" option on its own, treat it as unsupported (see below). If the tool accepts only option strings, keep the model labels as those strings and put their short descriptions in the question text.

6. Ask the effort question only for delegates, or roles with one choice for all, whose model answer was Claude Sonnet 5.5 or GPT 6.2 Sol, in a second call after the model answers. Offer high and xhigh, with "(Recommended)" on the effort the other skill names for that role, if any. Claude Opus 5.5 (xhigh) and Claude Haiku 5.5 (high) have a single allowed effort, so do not ask about it. Collect a valid model answer, and an effort answer where one is needed, for every delegate in the group before starting any of them, including with async questions.
7. After receiving and validating the answers, start each delegate with `delegate_task`, and give every delegate of a role with one choice for all the choice for its role. Map each choice to the exact provider instance and model ID resolved in step 1, and explicitly set that provider's effort option to the chosen effort in `target.options`. Do not change, upgrade, or downgrade the selection.
8. Report the choices in the final reply, as a table of role, model, and effort, with the number of delegates when one choice covered several. Include delegates started in later rounds.

## If the user does not choose

An async question without a response is still pending. Do not ask it again merely because the tool returned `{ accepted: true }`.

If the user dismisses the question or does not select an option, do not start the delegate. Ask again once. If the user dismisses it twice, stop and report that the run is paused until they choose.

If the user answers with any model or effort outside the allowed combinations, or with Claude Haiku 5.5 for a role that edits source files, say which combinations are allowed for that role and ask again with the same options.

## Scope

Pickmodel only controls which model each delegate uses. It does not change roles, prompts, permissions, or the number of delegates. Do not use it to add delegates the other skill does not call for.

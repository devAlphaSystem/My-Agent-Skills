---
name: change
description: Add, edit, or remove a feature in one request. Researches first, then takes a light path for a small, low-risk change or the full path otherwise. The full path runs squad, the full ui-consistency review, and qa. The light path runs one implementer and the testers the change needs. Both apply impeccable's design rules to interface work, pragmatic to testing, and unslop to all text, and end with a short plain-language summary, the test results, and notes. Use only when the user says 'change'.
---

# Change

Use this skill when the user says 'change' followed by a feature to add, edit, or remove. One request runs the whole job, from research to tested code, and the user reads one short report at the end. It works from a new chat and needs no earlier context.

The run always starts with research. The research shows what the change involves, and the coordinator then picks one of two paths. The light path is for a small, low-risk change and starts few delegates. The full path is for everything else.

This skill composes other skills:

| Skill          | What it does in this skill                                                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| squad          | Its researcher runs on both paths. On the full path, its planner and implementer build the change.                                         |
| ui-consistency | Keeps interface work on the project's patterns and records them in `UI.md`, at the light level on the light path and full on the full path |
| impeccable     | Sets the design quality floor for interface work, and gives the exploratory tester its interface audit                                     |
| qa             | Tests the result and fixes confirmed problems                                                                                              |
| pragmatic      | Keeps the testing, fixing, and retesting proportional to practical impact                                                                  |
| unslop         | Sets the rules for every piece of text the run writes, including interface copy, docs, and the final reply                                 |

Saying 'change' is the user's explicit request for all of them for this task, including the full level of ui-consistency on the full path and the qa run. Read the `SKILL.md` of each one before starting, and follow it except where this file says otherwise.

squad, qa, pragmatic, ui-consistency, and unslop are in the same skill collection as this file. impeccable is a separate skill. If a skill the run needs is not installed, tell the user which one is missing and stop before starting any delegate. impeccable and ui-consistency are needed only when the change touches the interface.

## Coordinator

The coordinator is the agent in the conversation. It talks to the user, briefs each delegate, picks the path, passes results between the stages, and reports back. It never creates or edits code, however small the change. Every code change goes to an implementer or qa's fixer. The coordinator writes only `UI.md`, as ui-consistency describes.

The delegates and their models are the ones each skill defines, plus the light implementer under "Light path". When the user also says 'pickmodel', it applies to every delegate of the run.

## Rules between the skills

- An explicit request from the user wins over every skill.
- For interface work, the project's existing patterns, as ui-consistency finds them, win over impeccable's taste. impeccable governs craft, such as spacing, hierarchy, states, accessibility, and copy, inside those patterns, and the design of a new element that has no pattern. A change is a refinement in impeccable's terms unless the user asks for a redesign.
- Do not run impeccable's `init`, `document`, `live`, or `hooks` commands during the run. If impeccable reports missing or stale context, such as no `PRODUCT.md`, continue on the existing implementation and list it in the notes.
- pragmatic applies to the testing stage only. The research and the build follow squad as written.
- Permissions, the Git restriction, and stopping conditions are the union of the skills involved. When two skills differ, the stricter rule applies.
- When the change deletes files that existed before the task and the host's instructions require permission for that, ask the user once before the implementer starts, and list the files.

## Workflow

The workflow runs from start to finish without waiting for approval. Stop and ask only when the request is unclear or a step needs a permission that the skills do not grant.

### 1. Understand

Restate the change in one or two sentences. Read `PROJECT.md` and `UI.md` at the workspace root when they exist, and pass the relevant parts to the delegates.

Decide whether the change touches the interface, which is anything a user sees or interacts with, including labels, messages, and emails. If it does not, ui-consistency and impeccable do not apply, and the report says so.

### 2. Research

Run squad's researcher, with squad's brief and these additions:

- For interface changes, the discovery of ui-consistency step 1, with the patterns, the counts, and their scope.
- The facts the path decision needs: the files the change would touch, whether it reuses an existing implementation, which shared code it would change and how many places use that code, and whether it touches any area in the list under step 3.

### 3. Choose the path

Take the light path only when the research shows all of these:

- The change is small, such as a few files in one module or layer.
- It follows an existing implementation or pattern, or it is self-contained.
- It leaves no design decision open.
- It adds or upgrades no dependency, and changes no database schema, migration, environment variable, or public API.
- It does not change how data is written, stored, or migrated, and does not touch auth, permissions, security, or payments. Showing data that the code already loads is fine.
- It changes no shared code that many places use, such as a shared component, helper, layout, or style, beyond adding to it without changing what the existing uses get.

Otherwise, take the full path. When the research leaves a check uncertain, take the full path.

The user can choose the path by saying 'change quick' or 'change full'. A user's choice wins. When the user chose the light path and the research shows a risk from the list above, say so in one line and continue on the light path.

Tell the user the path and the reason in one line, then continue without waiting.

### 4. Light path

#### Build

Start one implementer on Claude Sonnet 5.5 high, with the request, the research report, and the rules of squad's implementer, except that it works without a plan. Start it on Claude Sonnet 5.5 xhigh instead when the change adds or changes logic beyond wiring existing behavior, such as a new calculation, condition, or validation rule. It picks the simplest change that follows the existing code, runs the project's relevant checks, and returns the changed files and the output of each check. For interface changes, the coordinator first writes the decision table of ui-consistency step 2 from the research, and the implementer follows ui-consistency step 3, runs impeccable's setup, and reads its craft floor before editing interface files, as impeccable describes. For every piece of text it writes, it follows the unslop rules.

The implementer stops and reports when the change turns out larger than the research showed, or when a check from step 3 no longer holds. The coordinator then moves the run to the full path, as described under "Moving to the full path".

When an implementer on Claude Sonnet 5.5 high returns with a failing check it could not fix, or its result fails a qa round, the next attempt goes to a new implementer on Claude Sonnet 5.5 xhigh. Brief it with the research, the changes already made, the failing check or the qa findings, and the earlier implementer's report. It replaces qa's fixer on the light path from then on. When the change still fails with Claude Sonnet 5.5 xhigh, move to the full path. Each escalation and its reason goes in the report.

#### Interface rules

For interface changes, the coordinator runs ui-consistency steps 4 and 5 at the light level: its checks in code and the `UI.md` update. No reviewers run.

#### Test

Run qa with pragmatic's rules in every brief, with only the testers the change needs:

- The exploratory tester alone, when the change only affects what users see or how they interact, such as text, layout, style, or a control wired to existing behavior.
- The automated tester alone, when the change only affects logic and has no interface effect.
- Both testers, when the change affects both.

The coordinator states which testers ran and why. For interface changes, the exploratory tester uses impeccable's audit as a read-only checklist for the changed screens. Confirmed blocker and major problems go to the next fix attempt: to the xhigh implementer, as described under Build, when the high implementer made the change, and otherwise to qa's fixer, with the same additions as the light implementer. Run at most two rounds. When a problem remains after two rounds, or a fix needs a design change, move to the full path.

### 5. Full path

#### Build

Run squad's planning and implementation with the research from step 2. Add to the briefs:

- Planner: for interface changes, the decision table of ui-consistency step 2 and the `UI.md` entries to add or correct. The planner reads impeccable's instructions for the kind of work, as its routing describes, and states the design direction for new elements.
- Implementer: for interface changes, follow ui-consistency step 3, and run impeccable's setup and read its craft floor before editing interface files, as impeccable describes. For every piece of text it writes, such as interface copy, messages, docs, and comments, follow the unslop rules.

#### Interface review

For interface changes, run ui-consistency steps 4 to 6 at the full level: the coordinator's checks, the `UI.md` update, and the code reviewer and visual checker. Accepted findings go to squad's implementer, with at most three review rounds, as ui-consistency describes. Its rule for skipping the reviewers on a small change still applies, and the report says when they were skipped and why.

#### Test

Run qa with both testers and pragmatic's rules in every brief. Build the qa brief from the squad results and the interface review. Add to the briefs:

- Exploratory tester: for interface changes, use impeccable's audit as a read-only checklist for the changed screens. It reports problems and fixes nothing.
- Fixer: the same additions as squad's implementer above.

After a qa fix that changes the interface, the coordinator updates the affected `UI.md` entries. qa's exploratory tester covers the fixed screens, so the interface reviewers do not run again.

### Moving to the full path

When the light path moves to the full path, keep the research and the changes already made. Brief the planner with both, and with the reason for the move, then continue with step 5. A qa round of the light path does not count toward qa's five rounds on the full path. The report says that the run moved and why.

### Skipping the tests

Testing always runs once the build changed files. Skip it only when the build changed nothing, such as when the request turned out to need no change or the implementer stopped before editing. Say why in the report.

### 6. Report

The final reply replaces the separate reports of squad, qa, and ui-consistency. Follow the unslop rules. Use this format:

```markdown
## What was done

A plain explanation in two to four sentences, for someone who does not code. Say what the user can now do, or no longer sees, and where. No file names or jargon here.

## Test results

The qa table, then one line for each check the run executed, such as build, tests, lint, and the interface review, with pass or fail. Mark checks that did not run as not run, with the reason.

## Notes

Only what the user needs to act on or know: deferred problems with their impact, what could not be tested and why, decisions made without the user, conflicts for the user to settle, and follow-ups. Leave this section out when there is nothing to say.

## Details

- Path: light or full, the reason, and whether the run moved from light to full and why
- Changed files, one line each
- Models used, by role, and the reason for each choice between models
- `UI.md` entries added or corrected, when the interface changed
```

Never report a check as passed when it did not run, and never call a deferred problem fixed.

## Git restriction

The coordinator and all delegates may run only these read-only Git commands:

- `git --no-pager --no-optional-locks status`
- `git --no-pager diff --no-ext-diff --no-textconv`
- `git --no-pager show --no-ext-diff --no-textconv`
- `git --no-pager log`

Arguments that only filter or format the output are allowed, such as paths, commit ranges, `--stat`, `--oneline`, `-p`, and `-n`. Never use `--output` or any option that writes files.

Never run any other Git command or version-control operation. This includes commit, add, checkout, switch, restore, reset, stash, merge, rebase, pull, push, fetch, branch, tag, and clean. This restriction applies to the coordinator and all delegates, including their tools and subprocesses. The implementers and qa's fixer make all code changes by editing workspace files directly.

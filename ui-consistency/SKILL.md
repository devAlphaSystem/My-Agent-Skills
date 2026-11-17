---
name: ui-consistency
description: Preserve a project's existing interface conventions across any UI change, including shared components, visual foundations, page composition, states, and interactions. Discover and reuse the patterns relevant to the task, even when this skill does not name them, and record them in UI.md. Light level for interface work without extra delegates or a browser. Full level when the user says 'ui-consistency' with a task, alone or with squad, adding code and visual reviews. Mapping when the user says 'ui-consistency' with no task or asks to map the project's interface patterns.
---

# UI consistency

Use this skill so that a new or changed interface looks and behaves like the rest of the product, without the user repeating its conventions in each request. Discover the conventions from the project and preserve them within their intended purpose, variants, and supported states. Reusing the right component is not enough when the work overrides its constraints, omits a required state, or bypasses its interaction flow.

The skill works at three levels.

- Applying, light covers any task that builds or changes interface when the user did not say 'ui-consistency'. The agent does the discovery, the decisions, the checks, and the recording in `UI.md` itself, in steps 1 to 5 and 7 under "Applying the patterns". It adds no delegates and opens no browser or device session. With squad, squad's own researcher, planner, and implementer do their usual parts. It creates `UI.md` when the file does not exist. The agent may select this level on its own.
- Applying, full runs when the user says 'ui-consistency' with a task, alone or with squad. It adds step 6, where two reviewers check the result. One reads the code. The other compares the running surface with the reference through platform-appropriate evidence. Saying 'ui-consistency' authorizes the local app and the matching browser or device checks, within the rules under "Running the app and inspecting the interface".
- Mapping studies the project and saves the patterns in `UI.md` at the workspace root. It runs when the user says 'ui-consistency' with no task, explicitly requests project-wide mapping, or accepts the offer under "Starting a run". A task that updates a specific pattern stays focused on that pattern. Never start a project-wide mapping merely because the file is missing.

Web projects are the main target, whether the interface comes from server templates, a component framework, or global scripts and styles. On Android, apply the same method to themes, styles, shared layouts or composables, dialogs, and snackbars, with the device tools in place of the browser.

## Coverage

All component names, families, and sample tables in this skill are illustrative, not an exhaustive checklist. A pattern is in scope because the task uses or changes it, not because this skill names it. Discover project-specific patterns from the actual code, comparable screens, and shared implementations. Absence from this skill or from `UI.md` is not evidence that no existing pattern exists. Automatic loading still depends on the host; naming the skill explicitly requests its workflow.

For the affected interface, cover its visual foundations and composition, every relevant element and state, and the interactions that connect them. This includes client-only behavior as well as server actions, and the project's responsive, keyboard, focus, and accessibility conventions. Apply the same evidence and reuse rules at each level, from an individual control to a multi-step flow. Examine only what the task affects; this does not authorize a project-wide redesign or audit.

For every discovered need, decide which existing implementation and variant to use, or record the search that established that none fits. If the need is genuinely new, use the project's existing foundations and composition style, and justify the addition without inventing an unnecessary abstraction. Do not restrict discovery, decisions, or verification to the examples below.

## Starting a run

When the user says 'ui-consistency' with a task, check `UI.md` first:

- It does not exist. Discover only the patterns needed by the task and create focused entries in step 5. Offer project-wide mapping once, using the same approval and offer-status rules below; do not make the focused task wait for the answer.
- It exists and no entry says it comes from a mapping. Check the Overview for an earlier mapping offer. If none exists, ask the user once per project whether to map the whole project first, and say what the mapping adds. Record the offer and its pending or answered status with the date. If the user agrees, map first. If the user declines or has not answered, do not start the mapping, and continue with the focused application. Do not repeat a pending or declined offer. A pending question is not approval for the added work; a later explicit mapping request can still start it.
- It exists and has mapped entries. Reuse them, and verify the entries that the task touches against the code, as in step 1. Do not map again.

A mapped entry covers its own pattern and the projects the Overview lists. It does not mean that the whole project is mapped. For a need with no entry, find the implementation in the code, as in step 1. A request for project-wide mapping, or 'ui-consistency' with no task, runs the mapping. Updating a named entry does not expand the task into a whole-project inventory. The mapping offer explains that it adds a mapper, a verifier, a visual checker, and platform-appropriate runtime checks.

## Evidence and decisions

A pattern is how the project already solves a UI need. Rank the evidence in this order:

1. A rule the project documents, such as an entry in `UI.md`, an instruction file, or a design document
2. A shared implementation, such as a component, partial, helper, global function, CSS class, or token
3. The way most comparable uses do it, with the count and its scope stated

The most common way is evidence, not law. Older screens can hold inconsistencies. When a shared implementation exists but most pages bypass it, the project has two patterns, and that is a conflict.

- Comparable uses. Compare flows that serve the same purpose, such as edit drawers with edit drawers, not every page of the project. State the count with its denominator and scope, such as 6 of 7 edit drawers in the customers and orders modules. A count without its scope misleads a later agent.
- Variants. A pattern can have variants that the project documents or names by purpose, such as a standard edit drawer and a wide drawer for reports. Identify each variant, its purpose, and the defaults and constraints that the variants share. Use the variant whose purpose matches the task. A wider variant does not allow an arbitrary width for an edit drawer.
- Fixed and variable properties. Within one variant and one breakpoint, a property that is identical in every comparable use, such as the width of the standard drawer, is fixed. Do not set it locally, in an inline style, or in a page-specific class. A property that differs by use, such as the title, the content, and the actions, is variable. Pass it through the pattern's own parameters.
- Behavior. Trace each affected interaction through its relevant states, including interactions that never call the server. Follow the established handling of input, navigation, focus, dismissal, feedback, and state changes where applicable. For an asynchronous operation, find the project's waiting, success, failure, and repeated-action behavior for that kind of operation. Reuse the matching flow; do not impose a blocking processing dialog on unrelated interactions that use a different established pattern.
- Defects. A majority never justifies keeping a known security, accessibility, or correctness defect, such as output that skips escaping. A reference is evidence of convention, not proof of usability. A defect reproduced in the task's flow remains relevant when inherited from a shared implementation. Keep the sound canonical pattern and fix the authorized root cause rather than adding a page-specific workaround. If the correction needs broader changes or permissions, report the affected requirement as unresolved and request that scope; do not silently omit it or call the broken flow compliant. Unrelated reference defects remain follow-ups.
- User-requested deviations. An explicit request from the user overrides an inferred convention for the scope of the request. Record the reason in `UI.md` and in the report. Do not change the shared default or other screens to fit the deviation.
- Conflicts. If the evidence splits, no documented rule settles it, and the choice changes what the user sees, ask the user one concrete question. Give the two options, the counts with their scope, and a reference page for each. If the choice is minor or easy to reverse, follow the more common option and say so.
- Existing deviations. Do not copy them, and do not fix them outside the task. List them as follow-ups.
- No shared implementation. Follow the way most comparable uses do it. Do not create a component, partial, or abstraction for it unless the project already does that for this pattern or the task asks.

## UI.md

`UI.md` is the project's record of its interface patterns, written for any agent to read before interface work. It lives at the workspace root. In a multi-project workspace, use the one in the folder of the project the task touches, or one section per project if the user prefers. Both Applying and Mapping write it. Only the coordinator writes it, and delegates never do.

It has these sections:

- Overview, covering the interface stack, where shared elements live, how scripts and styles load, the projects and the date of the last mapping, and mapping or instruction-pointer offers with their status and date
- Patterns, with one entry for each
- Formats and language, covering dates, money, numbers, documents, labels, and messages
- Deviations, listing existing inconsistencies that new work must not copy, and the deviations that the user requested, with the reason
- Open questions, listing conflicts that only the user can settle
- How to use this file, in three lines that tell an agent to read the relevant entries before interface work and to confirm them against the code
- Last updated, with the date of the run

Size each pattern entry to its pattern, and keep it short:

- A task entry can be one compact bullet or paragraph. It gives the implementation and a reference page, the purpose and the variant, the constraints and the flow that matter, the evidence with its count, scope, and date, and whether it is confirmed in code only or at runtime. For example: "Result selection: reuse the shared selection control, reference the current search page. Preserve its keyboard navigation and focus handling. Task entry from 3 of 3 comparable selection controls in search and settings, 2026-11-17. Confirmed in code only." Replace sample names and counts with the project's actual evidence.
- A mapped entry can add more detail: what the pattern is and where it lives, when to use each variant, how to call it with a reference page, its fixed and variable properties per variant and breakpoint, its behavior, the basis, and the status with the date. It still links to files instead of copying code, and it does not hold a report.
- Describe open, close, success, failure, and repeated submit only for a pattern that has them, such as a drawer, a modal, or a server action. A static badge needs none of them.

Writing rules:

- Create the file when it does not exist, with these sections. If it exists with another structure, keep its content and add the sections it lacks.
- Record only what you inspected in code or directly observed on the platform. Distinguish reuse of a pattern from confirmation that its states are usable. Code and headless DOM checks do not establish rendered visibility, layout, or native focus traversal. Before saving, reopen each file an entry cites, and confirm that it exists and says what the entry says.
- Add or correct only the entries for the patterns the work examined. Do not rewrite or reorder the others. If the code contradicts an entry, correct it and say so in the report.
- An entry the user wrote or marked as decided stays as it is. If the code contradicts it, record the contradiction under Open questions. Record the user's answer to a conflict as a decided rule, with the date.
- A pattern learned from a task covers only the comparable uses that task examined. State the count, the denominator, and the scope in the entry, so a later agent does not read a partial count as a full one.
- Link to files instead of copying code. Do not link to evidence files, because they live outside the project.
- Run the text through the unslop rules, and use no emojis. Never write secret values.

## Applying the patterns

Keep the work proportional to the task. Look only at the elements the task needs.

### 1. Discover

Read `UI.md` if it exists, and the project's instruction files for interface conventions. Inspect the affected screen or flow and list its actual UI needs, including composition, visual foundations, controls, states, and interactions. Derive this list from the project and the request, rather than matching only familiar component names. Include formats and product language when the task displays or accepts data.

For each need, find the implementation the project uses. Open at least two comparable uses when the project has them, and identify the variant that fits the task. Read the shared files they use, such as components, partials, global scripts and styles, and tokens. Trace each relevant interaction through its states and callers, whether it runs locally or calls the server. If `UI.md` has an entry for a need, confirm it against the code before relying on it, because it can be stale. If it has none, rank the evidence from the code as above. Note what to record in `UI.md` for each need, including the count of comparable uses you examined and its scope.

### 2. Decide

Before editing, write the decisions in a short table. These rows illustrate the format; replace them with every relevant need discovered for the task, including needs not named in this skill:

| Need              | Implementation to use     | Reference page           | Constraints kept             |
| ----------------- | ------------------------- | ------------------------ | ---------------------------- |
| Selection control | Project selection control | Existing results screen  | Keyboard and focus behavior  |
| Screen layout     | Project layout            | Existing settings screen | Spacing and responsive rules |

Reuse is the default. A new component, a copy of an existing one, or a local override of a fixed property needs a reason that comes from what the task requires. A preference does not count. If a shared element lacks something the task needs, make the smallest extension that preserves correct existing contracts, defaults, and variants, and check the affected uses. A directly related defect correction follows the same rule; preserving a defect is not a compatibility requirement. Changes to unrelated appearance or interaction contracts need an explicit decision, rather than being hidden in a shared fix.

With squad, the researcher reports the patterns for the task's needs, and the planner puts this table in the plan. The researcher also returns the evidence and the counts with their scope, and the planner lists the entries to add or correct in `UI.md`.

### 3. Implement

Follow the table for each affected element, composition, state, and interaction. Use the project's own way of loading scripts and styles and of passing data to the page. Reuse the established flow for the same kind of operation, including its applicable feedback and failure behavior. Do not add inline styles or one-off classes that replace shared tokens or fixed properties. Leave `UI.md` for step 5.

### 4. Check the work

Check the result against the table before reporting, at every level. The agent runs these checks itself, on the changed and new files. When the project is in a Git repository, it may also use the read-only diff command under "Git restriction" for the tracked files. It does not need Git for the checks.

- Search for duplicates, such as new markup, scripts, or styles that repeat a shared element
- Search for local overrides of fixed properties, including inline styles and page-specific classes
- Check every need in the decision table, including local interactions, state transitions, composition, and relevant responsive and accessibility behavior
- Trace each affected flow from its trigger to its resulting states, and confirm that applicable feedback, recovery, and repeated-action handling follow the matching project flow
- Trace where each result is presented, what can cover or clip it, and how focus enters, traverses, and leaves the interaction. A message in the DOM or a successful callback is insufficient evidence that the user can perceive or act on the result
- Check that the new code does not copy a defect from the reference page, such as output that skips escaping

At the light level, say in the report that the behavior was checked in code only, and that saying 'ui-consistency' adds the reviewers and platform checks. Identify visibility, layout, and native input behavior as unconfirmed even when simulated DOM events pass. Never report a runtime check that did not run or hide a confirmed failure behind successful reuse.

### 5. Record in UI.md

At every level, the coordinator updates `UI.md` after the checks in step 4 and before any reviewer starts, following the section "UI.md". Without squad, the agent does it. Use the decision table, the research findings when squad ran the research, and the results of the checks. Add the entries for the patterns the task examined, and correct the ones the code contradicts. Mark each entry as confirmed in code only, with the date. Record new deviations and conflicts, and the deviations the user requested with the reason. Delegates do not write the file.

### 6. Validate with reviewers (full level)

Start the reviewers after step 5, so that they review the implementation and the `UI.md` entries together. Skip them when the change is small, such as a text change, a single style value that follows a pattern, or code cleanup with no rendered or interaction effect. Say in the report that the reviewers were skipped and why, and that the behavior was checked in code only. Do not skip them for a new screen or interactive surface or flow, or a material change to existing composition, states, or interaction patterns, even if the element is not named in this skill. Fixing an existing interaction defect does not qualify for exemption merely because no new component was added.

The coordinator prepares the app as described under "Running the app and inspecting the interface", then starts both reviewers in parallel when their checks are possible. If runtime prerequisites are unavailable, start the code reviewer and skip the visual checks that are blocked; if none can run, do not start the visual checker. Report them as not run and runtime as unconfirmed, without repeating rounds solely to obtain a missing verdict. The brief for each includes the request, the changed files and what changed in each, the decision table, the reference pages, the `UI.md` entries from step 5, and the rules under "Evidence and decisions". Give the visual checker also the web URL or native application id and device identifiers, the available test accounts by name, the evidence directory path, and the instructions for that platform.

The code reviewer checks the work without running it:

- Reads the changed and new files, the decision table, the `UI.md` entries, the reference pages, and the shared files they use
- Searches the whole project for an existing element that the work duplicates or missed, including elements that `UI.md` does not list
- Checks that no fixed property is overridden, through inline styles, page-specific classes, or local rules, that variable values go through the pattern's own parameters, and that scripts and styles load as they do on the reference page
- Checks that the decision table covers the task's actual UI needs, including project-specific patterns not named in this skill
- Traces each affected interaction and state transition in code and compares it with the matching reference flow, including client-only behavior
- Follows state output locations, layering, clipping, focus routing, and dismissal handlers instead of treating an existing component call or a DOM assertion as sufficient confirmation
- Checks the formats and language of labels and messages against the project's own
- Checks that the `UI.md` entries match the code, with their counts and scope

The visual checker compares the running screen with the reference:

- Opens the reference and the new or changed surface in matching supported configurations and states, and saves a screenshot of each. On web, cover a desktop width and a narrow width of about 360 px. On Android, use the supported device configurations relevant to the task under "Android runtime"; do not impose desktop widths on a phone-only app.
- Checks the relevant visual and composition constraints for the matching variant and state. On web, use `preview_evaluate` for bounds and computed styles. On Android, use device snapshots and the bounds or state evidence the device tools expose; DOM measurements do not apply to native UI. Return comparable values when available and mark unavailable measurements as unconfirmed. Different content does not require identical measurements everywhere.
- Exercises each affected interaction, including client-only flows, and compares its relevant states and keyboard or focus behavior with the reference. Records flows that need temporal evidence, using the platform's available tools. For asynchronous actions, checks the matching processing, success, failure, and repeated-action behavior. On web, a read-only observer can establish fast transitions; on native UI, use available device evidence and report transitions the tools cannot establish.
- Checks effective usability of the resulting states in each relevant configuration: required feedback must be perceivable and controls needed in that state must be reachable. Inspect covering layers, clipping, and intended local scrolling; DOM presence or nonzero bounds alone do not establish visibility. On web, screenshots and read-only hit testing can help distinguish a covered element from a visible one. Do not move elements, raise their stacking order, or alter focus programmatically to manufacture passing evidence.
- Traverses the relevant input sequence through entry, intermediate controls, boundaries, and return or dismissal, using native browser/device actions. On web, include Tab and Shift+Tab past the first and last controls, then test applicable dismissal keys from the positions the flow permits. Distinguish normal browser-chrome transitions from covered in-page focus. Preserve intended modality: do not impose a focus trap on every component or convert a nonmodal surface to modal as a shortcut. A user must retain a visible, usable route through or out of the interaction.
- Checks the failure path of each server action when it can be done safely, with the failure-path check under "Running the app and inspecting the interface", and checks the console for errors. Invalid input alone does not show how the flow handles a failed request.
- Creates its own test records, labeled with the run, and changes only records it created. It skips an action with a real external effect and lists it as unchecked.

Each reviewer returns a verdict, pass or fail, and each finding with the convention or usability, accessibility, or correctness requirement it violates, its reproduction context, and file/line or runtime evidence. Identify a sound existing implementation when one is available; otherwise describe the scoped root-cause correction where known, or state what remains unknown. Do not invent a replacement when the canonical implementation contains the defect. Also return what could not be checked and why. The visual checker adds the paths of screenshots and recordings.

The coordinator reviews the findings, merges duplicates, and drops unsupported preferences or findings outside the task. A material usability, accessibility, or correctness failure in the requested flow remains a failed check even when the reference behaves identically. Label it inherited rather than new, but do not discard it solely for matching the reference. Defects confined to unrelated reference flows stay follow-ups. Record each dropped report's reason. Accepted findings go to the implementer for a scoped root-cause fix and affected-use checks, subject to existing permissions; a broader redesign needs authorization. Without squad the agent implements; with squad start an implementer run. An unfixed failure remains explicit when permission, runtime access, or the round limit prevents completion.

Review rounds:

- After each fix, update the affected `UI.md` entries before the next review, so that the reviewers see the code and the entries together.
- Rerun only the reviewers that the fix affects. A change to CSS, markup, an interaction, or shared interface behavior needs both reviewers. A documentary change, or a clarification in code with no rendered or behavioral effect, needs only the code reviewer.
- Keep the other reviewer's earlier verdict only when you can explain why the fix did not touch what that reviewer checked. State the explanation in the report.
- Run at most three review rounds, counting the first review as round 1. Stop when every reviewer's last verdict is pass, or after the third round, and report the deviations that remain.
- Correct entries contradicted by a review before the affected re-review, within the same round limit. After the reviews, add only the verification status and date supported by their results. State which properties and flows were confirmed at runtime and which remain unchecked or failed; measuring a width does not confirm the failure path. An interaction exercised and directly observed in the running app counts as runtime evidence without a numeric measurement. Parts not exercised or measured at runtime retain their code-only or unchecked status. Do not silently change a pattern rule after its final review.

### 7. Report

Include the conformance summary in the final reply, in this format. Name in the last column only the checks that ran:

| Need              | Implementation used       | Constraints kept          | Checked by         |
| ----------------- | ------------------------- | ------------------------- | ------------------ |
| Selection control | Project selection control | Keyboard, focus handling  | Flow trace in code |
| Screen layout     | Project layout            | Spacing, responsive rules | Code search        |

At the full level, the last column also names the reviewers, screenshots, and recordings that covered each row. Add the review results in this format, with the rounds and the verdicts adjusted to the actual reviews. Write `retained` with the reason for a verdict kept without a rerun, and `not run` with the reason for a check that did not run:

| Delegate                                 | Round 1 | Round 2                                  |
| ---------------------------------------- | ------- | ---------------------------------------- |
| GPT 6.2 Sol xhigh (code reviewer)        | fail    | pass                                     |
| Claude Sonnet 5.5 xhigh (visual checker) | pass    | retained, the fix had no rendered effect |

Then add the measurements that matter, as element, property, reference value, and new value. Show the key screenshots, one of the reference page and one of the new page for each need and width, when the host can display images. Link the recordings and give the evidence directory path.

After that, list pattern deviations, in-scope defects (new or inherited), unrelated follow-ups, and their evidence and status separately. Reuse conformance does not override a failed behavior check. State which `UI.md` entries changed, what was checked in code only, and what stayed unconfirmed; a verified property does not confirm the whole flow. If the project's instruction file does not mention `UI.md`, follow "Pointing agents to UI.md".

## Mapping the project

The coordinator is the agent in the conversation. See "Delegates" for the roles and models. It briefs each delegate, merges their results, writes `UI.md`, and reports to the user. The coordinator does not map on its own, except for small clarifications needed to write a brief.

### 1. Check the current state

If `UI.md` exists, read it and tell the user that this run will update it. Keep any entry the user wrote or marked as decided. Entries that came from a task cover only the comparable uses that task examined, so give them to the mapper as hints to recheck over the whole project. If the workspace has several projects, list them and ask which ones to map, unless the user already said. If `PROJECT.md` exists, give it to the mapper as a hint.

### 2. Mapping

Brief the mapper with the workspace path and the projects to map. The mapper reads files and runs read-only commands. It does not edit files.

The mapper must return:

- The interface stack, covering the templating or component framework, the CSS approach and framework with versions, the layout files, and how scripts and styles load
- The shared elements and where they live, such as components, partials, helpers, global scripts, styles, and tokens
- Derive the families of patterns from the project's actual interface, including its domain-specific features. For each family, return every implementation and variant found, its purpose, the number of comparable uses and the scope of that count, and a usage example from the best reference page. Visual foundations, composition, controls, navigation, data presentation, feedback, and multi-step flows are examples, not a closed inventory. Include any other family the project uses.
- For each variant of a pattern, the fixed properties and the variable ones, based on every comparable use, and its behavior at narrow widths. A variant that serves another purpose does not make a property variable for the others.
- The relevant states and transitions of each interaction pattern, including client-only flows, traced for a reference use; for asynchronous operations, the matching processing, success, failure, and repeated-action handling, and uses that bypass it
- The formats and language the product uses for dates, money, numbers, documents, labels, and messages
- Conventions that the project documents
- Deviations, such as forks of shared elements, local overrides of fixed properties, hand-built copies, and inline styles that replace tokens, each with a file reference
- Conflicts with no clear winner, with the counts and their scope for each option
- Known security, accessibility, or correctness defects in the shared elements or in how pages use them, so that the entries do not preserve them
- Anything it could not determine from the code alone

### 3. Verification

Brief the verifier with the mapper's report. The verifier checks each claim against the code. It repeats the searches behind the counts, confirms that the files and examples exist, and checks that the fixed properties are fixed in every use of the same variant. It does not edit files and does not start the app. A claim about runtime behavior that reading can't confirm is marked as not confirmed at runtime.

The verifier returns the claims that are wrong, with the correct information, and the important patterns the mapper missed. If it finds major gaps, the coordinator sends them back to the mapper for one more round.

### 4. Runtime check

The coordinator prepares the app as described under "Running the app and inspecting the interface", and briefs the visual checker with the corrected map, the web URL or native application id and device identifiers, the test accounts by name, the evidence directory path, and the platform instructions. Use the same platform-specific comparison rules as in Applying step 6. For each pattern in the map, the visual checker opens a reference use and:

- Saves screenshots in the matching supported web viewports or native device configurations relevant to the pattern
- Checks the fixed properties of each variant on reachable uses, up to five, with the applicable browser or device tools, and returns the evidence and sample scope. If values differ within the same variant and configuration, check them against the shared default and documented rule. Report a local override as a deviation; do not make a fixed property variable merely because an existing page violates it. If no rule settles the difference, record it as a conflict.
- Exercises a reference use of each interaction pattern through its relevant states, including local interactions and keyboard or focus behavior, and records flows that need temporal evidence. Returns available observer logs or device evidence for short-lived transitions, listing anything unconfirmed
- Checks the failure path of each such flow when it can be done safely, with the failure-path check under "Running the app and inspecting the interface"
- Creates its own labeled test records, changes only records it created, and skips an action with a real external effect, listing it as unchecked

The coordinator records which properties and interactions the checker confirmed, with the date, and corrects the others. If the app can't run, the required browser or device tools are unavailable, the test accounts are unavailable, or the user declines the database permission, skip the checks that this blocks, and keep the map checked in code. Unchecked properties and interactions stay marked as not confirmed at runtime.

### 5. Write UI.md

The coordinator writes `UI.md` as described under "UI.md", with an entry for every pattern the map documents. State in each Basis line that the entry comes from a mapping, with the date, and record in the Overview the projects the mapping covered. A mapped entry replaces an earlier entry from a task for the same pattern. Keep the entries the user wrote or marked as decided. Never write secret values in `UI.md` or in any report.

### 6. Report

The coordinator tells the user where it saved `UI.md` and whether it created or updated it, how many patterns it documented, which were confirmed at runtime, and the conflicts that only the user can answer. It gives the evidence directory path, and shows the key screenshots when the host can display images. If the project's instruction file does not mention `UI.md`, it follows "Pointing agents to UI.md". If the run started because a task came with the request, continue with the task using the steps under "Applying the patterns".

## Pointing agents to UI.md

To guide future agents to `UI.md`, check whether the project's existing instruction file, such as `AGENTS.md` or `CLAUDE.md`, already points to it. If it does not, check the Overview for a previous pointer offer. If none exists, ask the user once per project to approve adding one line. Ask at the end of the run, in the final reply or with the host's question tool, and do not hold up the work for the answer. Record the proposed target, line, date, and pending status in `UI.md`, and update the status when the user answers. Do not repeat a pending or declined offer, and never edit the instruction file before the user approves.

- Show the target file and the exact line in the question. Use a line such as: Before interface work, read the relevant entries in `UI.md` and confirm them against the code. If `UI.md` is not in the same folder as the instruction file, write its path in the line.
- When the user approves, append only that line to the end of the file, and keep the existing text as it is. Report the change.
- When the user declines, leave the file unchanged, record the decision in the Overview of `UI.md` with the date so later runs do not ask again, and say in the report that future agents are not necessarily guided to `UI.md`.
- When the target is ambiguous, such as several instruction files with no clear main one, or when the project has none, do not choose one and do not create a file. Report the suggested line and leave the decision to the user.
- Do not claim that the line makes every host load `UI.md`. It only tells an agent that reads the instruction file where to look.

## Delegates

The coordinator is the agent in the conversation, or squad's coordinator when squad runs the task. It briefs the delegates, filters their findings, writes `UI.md`, and reports back. In Mapping and at the full level, it does not map, or replace a reviewer, on its own. At the light level there are no delegates, and the agent does the discovery, the decisions, the checks, and the recording itself. Without squad, the agent also does the implementation, as in council. With squad, the squad roles do the research, the plan, and the implementation.

Use these exact models and reasoning efforts:

| Role           | Model                   | Used in                          |
| -------------- | ----------------------- | -------------------------------- |
| Mapper         | GPT 6.2 Sol xhigh       | Mapping                          |
| Verifier       | Claude Sonnet 5.5 high  | Mapping                          |
| Code reviewer  | GPT 6.2 Sol xhigh       | Applying, full level             |
| Visual checker | Claude Sonnet 5.5 xhigh | Mapping and applying, full level |

Do not add, replace, or substitute models, even if the user suggests others. The only exception is Claude Haiku 5.5 high, which any agent can start at any time for light, read-only information lookups, including internet searches. It is not a delegate role. It returns a link for each source, and the agent that started it checks the source before relying on the result. If a required model or reasoning level is unavailable, report the limitation to the user and stop.

In T3 Code, use `orchestrator_capabilities` to resolve the exact models and their reasoning efforts, then `delegate_task` to start each delegate. Elsewhere, use the harness's subagent tools. Each delegate starts fresh, so every brief must include all the context it needs.

## Running the app and inspecting the interface

The visual checker inspects the project's own app. The coordinator prepares the build and runtime environment and is the only agent that starts or stops servers, bundlers, or emulators. The visual checker may launch and relaunch the prepared native test app through the device tools while exercising its UI. Browser instructions apply to web content, not native Android surfaces.

- Run the app in a development or test configuration. Never use production or real customer data.
- Check which database the configuration points to, without printing credentials. A disposable database needs no question, such as an in-memory database, a temporary file, or a container the test creates. If it is real or shared, do not start the app or run any test action against it without the user's detailed, bounded permission. The request explains which database and environment, exactly what will run, including each test action that reads or writes data, which records it affects and roughly how many, why it is needed, whether it can be reversed, and the expected result. The permission covers only what the request describes, and another action needs a new request.
- Log in with test accounts that the project documents or seeds. Never use a real account. If a check needs a credential that the project does not provide, ask the user for it, and in T3 Code use `request_secret`, never plain chat. Without the credential, skip the checks that need it.
- Use sandbox or test modes, or local stubs, for external services. Never trigger real payments, emails, messages, tax documents, or state-changing calls to production APIs.
- For a web app or a required local backend, use the project's own script and package or build tool. Choose a free local port through supported configuration. If the project fixes the port, use it only when free; otherwise report the blocker and never stop another process. Bind local services to localhost, retain their terminal sessions or process ids, and use the printed URL or derive it from configuration and confirm readiness. Do not use or stop a server you did not start. A native app does not require a web server or URL unless its architecture actually uses one. If dependencies are missing, use the project's own package or build tool and report lockfile changes. Removing packages, major version upgrades, and global installs need permission.
- When the checks end, stop everything you started, with no exception, and close the tabs. Do not stop processes you did not start.
- If the app can't run, the required browser or device tools are unavailable, the accounts are unavailable, or the user declines a permission, continue with the checks in code that still make sense, and report what stayed unconfirmed at runtime.

Create one evidence directory outside the project's source tree, in the place the user names or else in the scratchpad or the system temporary directory. It holds the screenshots and recordings. It is a deliverable, so keep it and report its path.

Browser instructions for the visual checker. In T3 Code:

- Call `preview_status`. Open its own tab with `preview_open`, with `reuseExistingTab` set to false and the app port as `target`, and use the returned `tabId` from then on.
- `preview_snapshot` with `save` set to true writes a PNG and returns its path. It also returns `aria-ref` locators for `preview_act`, which clicks, types, presses keys, selects, and scrolls. Locators expire after navigation or a new snapshot. `preview_resize` sets the viewport, `preview_set_appearance` switches light and dark, and `preview_wait_for` waits for text, an element, or a URL.
- `preview_recording_start` and `preview_recording_stop` record the tab. The stop call returns the path of the file, up to 50 MiB. Record one flow at a time, from the trigger to the end, and stop the recording after each one.
- `preview_evaluate` runs JavaScript in the page. Use it to read measurements and to install read-only observers that log events. Do not use it to change the page, except for the temporary request rejection under "Failure-path check".
- Close the tab with `t3_preview_close`.

Without T3 Code, use the harness's browser tool. Do not install or switch to another automation system. If recording is not available, take a sequence of screenshots and say so.

### Android runtime

- The coordinator identifies the debug build command, application id, supported device configurations, and any required test backend from the project. Build and install only its isolated debug app using the existing Gradle wrapper or cross-platform tooling. Never replace a real app, clear its data, or uninstall it without permission.
- In T3 Code, call `device_list` and select an available test emulator, or a physical device the user explicitly named. The coordinator calls `device_open` with its `hostId` and `deviceId` and records whether it was already running. Give both identifiers to the visual checker. If the platform or tool is unavailable, report that blocker rather than installing a new SDK or switching automation systems.
- The visual checker calls `device_open` for the selected device and uses the returned `agent-device` command, with all its arguments, for launches, relaunches, deep links, input, settings, snapshots, and recordings. Run `open <app>` before snapshots or input, consult `help <command>` for supported operations, and use `device_screenshot` to inspect the display. Use `adb` only for builds, installs, logs, or connectivity, never as a substitute UI driver. If a device tool fails, quote the error and follow its advice once; if it still fails, stop device checks and report the blocker. In other hosts, use their available device surface; if none exists, report native runtime checks as unavailable.
- Compare the reference and changed screens on the same device and state, using configurations the app supports. Exercise only task-relevant configuration changes and restore them afterward through the same device tools. Native screenshots, snapshots, and observed interactions are valid evidence; do not claim DOM/CSS measurements or accessibility behavior that the tools did not expose or exercise. If recording is unavailable, use a screenshot sequence and state its limits.
- The fetch-rejection technique below applies only to browser content. For native failure paths, use an existing, approved test mechanism or report the path unverified. Never introduce production calls or source edits to manufacture a passing result.
- Before reporting, stop only the test app and environment processes this run started and close its own device session if the tool supports that. Preserve a device that was already running. Keep test data intact unless its removal was separately authorized.

### Failure-path check

Invalid input alone does not show how a server action handles failure, because validation can stop the request before it is sent. To check the failure feedback of an action, the visual checker may make one request fail on the client, only when all of these hold:

- The brief names the action's local URL and HTTP method, and the flow uses `fetch`. The check applies to requests that match that URL and method, and passes every other request through unchanged.
- The patched `fetch` rejects the matching request before it leaves the page. Other requests keep their original behavior and remain subject to the test's permissions.
- One script installs the patch, triggers the action, waits for the flow to end with a bounded timeout, and restores the original `fetch` and disconnects its observers in a `finally` block. A timeout is an incomplete check, not a pass. Confirm that the instrumentation is removed before another flow; if cleanup cannot be confirmed, close that test tab.
- Apart from triggering the selected action, the script makes no DOM changes, intercepts no other requests, calls no production service, and edits no source files. Read-only observers are allowed, such as the one that logs when the processing element becomes visible and hidden.

If one of these cannot hold, such as an action that uses another request API, a URL that is not local, or a browser tool that cannot run the script this way, skip the check and report it as unchecked.

This check simulates a failure of the client network. It does not show how the backend reports its own errors, and the report must say so. Record the result as confirmed against a simulated client network failure, never as a confirmed backend error. The patch changes only the outcome of the matching request. It must not hide console errors or change how the page reacts to the rejection, and the visual checker reports what the page does.

## Delegate permissions

All delegates may run non-destructive commands, such as searches, line counts, linters in check-only mode, and read-only diagnostics. The visual checker may open a browser tab for the app the coordinator started.

The mapper, the verifier, and the code reviewer edit no files. The visual checker writes only to the evidence directory, and uses browser or device tools only within the platform rules above. Only the coordinator writes `UI.md`, and it edits an instruction file only to append the one line the user approved under "Pointing agents to UI.md". Only the implementer edits source files, to fix accepted deviations or carry out the task. Delegates do not edit instruction files or start or stop servers, bundlers, emulators, or unrelated processes. The visual checker may launch and relaunch the prepared native test app through its device tools.

No delegate may run destructive or side-effecting commands, including:

- Deleting, moving, or overwriting files outside the evidence directory
- Installing, upgrading, or removing dependencies
- Any direct database access, and any action that changes records the visual checker did not create
- Deploying, publishing, releasing, or pushing anywhere
- Sending real emails, payments, tax documents, notifications, or calls to production APIs that change state
- Exposing the app to the network, or binding it to an address other than localhost

If a delegate isn't sure whether a command is safe, it skips the command and explains what it would have run and why.

## Git restriction

While delegates run, the coordinator and all delegates may run only these read-only Git commands:

- `git --no-pager --no-optional-locks status`
- `git --no-pager diff --no-ext-diff --no-textconv`
- `git --no-pager show --no-ext-diff --no-textconv`
- `git --no-pager log`

Arguments that only filter or format the output are allowed, such as paths, commit ranges, `--stat`, `--oneline`, `-p`, and `-n`. Never use `--output` or any option that writes files. The checks in step 4, including at the light level, use only these read-only commands, and only when the project is a Git repository.

Never run any other Git command or version-control operation. This includes commit, add, checkout, switch, restore, reset, stash, merge, rebase, pull, push, fetch, branch, tag, and clean. This restriction applies to the coordinator and all delegates, including their tools and subprocesses. Make all changes by editing workspace files directly.

## Scope

Work only on the interface the task touches, and on `UI.md`. The one exception is the approved pointer line under "Pointing agents to UI.md", which is the only change the skill makes to an instruction file. This skill never starts a visual redesign, never changes existing screens to match a pattern unless the task asks, and never fixes inconsistencies across the project on its own. List them as follow-ups.

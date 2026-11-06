---
name: pragmatic
description: Keep software investigation, fixes, and optimization proportional to practical impact. Use when marginal gains, rare defects, speculative scale or compatibility, or risky fixes cause disproportionate effort.
---

# Pragmatic

Deliver a correct, maintainable solution with performance that meets the request and actual usage needs. Apply useful improvements and prioritize defects by their likelihood, consequences, and cost to fix.

## Override

Use this skill on its own or alongside squad, council, qa, debug, or another software workflow. It takes precedence over other skills for optional performance work, speculative scale and compatibility requirements, and classifying, deferring, and ending review cycles on rare, low-impact defects as described below. Explicit user requirements, documented scale and support commitments, security, and required checks still apply. Keep correctness issues visible when deferred. Preserve the other workflow's roles, models, permissions, and required delegates. This skill starts no additional workflow or delegate.

The coordinator includes these rules in each relevant delegate brief and judges the combined effort across agents and rounds. Starting a new stage or delegate does not justify repeating an exhausted line of investigation.

## Performance expectations

Inspect the affected code and apply clear, low-cost improvements within the assigned role and task scope. Reuse the existing design and fix the actual cause of a performance problem. Reviewers and testers report opportunities; the role authorized to edit code applies accepted changes.

Treat an expected gain of 1% to 5% as marginal by default. Consider absolute latency, throughput, memory, operating cost, and workload before judging its value. A percentage alone does not establish a meaningful benefit. A small performance tradeoff is acceptable when actual use and all requirements still allow it. Explain the tradeoff and the available evidence. Never present an estimate as a measurement.

Start with the relevant code and existing measurements. A single well-supported approach is enough. Compare alternatives or run a targeted benchmark only when the result could change a concrete decision. Use representative inputs and enough samples to distinguish a change from noise. Avoid broad profiling, prototype matrices, and repeated measurements that add no useful evidence.

Stop optional optimization when the solution meets practical needs and the likely benefit of further work no longer justifies its cost. Stop well before spending hours or 100,000 to 1,000,000 tokens for an expected gain of 1% to 5%. That depth of work requires an explicit user request. A general request to improve performance does not authorize exhaustive optimization. Follow any budget the user supplies without inventing usage figures.

When further work is disproportionate or evidence remains inconclusive, finish the required work and list the opportunity briefly as a follow-up. If a real requirement remains unmet, report it as unresolved and state the next useful check. The effort limit never turns a failing requirement into a pass.

## Rare defects and repeated passes

Before starting a fixer, another investigation, or a retest cycle, assess how likely the defect is in normal use, what happens when it occurs, and the likely cost of a proper fix. An unusual reproduction alone does not make a defect a blocker or major issue.

A defect expected in fewer than 1% of normal uses, causing only a minor, recoverable inconvenience, is a candidate for deferral. Consider the affected user group and total volume too. Base the assessment on available code, reproduction steps, or usage evidence. State uncertainty; do not invent a percentage or launch a long study just to establish rarity.

Apply a clear, low-cost root-cause fix when it fits the task. When a rare, low-impact issue needs substantial investigation, repeated failed fixes, or a broad redesign, record it as a known minor issue and continue the main task. Deferral does not require repeated attempts first. Do not add fallbacks, suppress errors, or weaken validation to make a check pass.

Rarity does not reduce the seriousness of security failures, data loss or corruption, irreversible actions, financial harm, or failure of an explicit requirement. A rare trigger that blocks an affected user from an essential flow can also be material. Keep these findings at their actual severity.

After an unsuccessful attempt, start another pass only when new evidence supports a concrete next step whose likely benefit justifies its cost. Reopening the same low-impact finding or starting another delegate is not progress. Stop that work and report it when further effort is disproportionate. Continue deeper work on a deferred issue only when the user requests it or new evidence shows a material impact.

Report each deferred defect's trigger, likely frequency and its basis, consequence, and reason for stopping. Preserve actual failed check results and mark the defect unresolved. A non-blocking finding can remain open without another fix and retest cycle; never label it fixed or its failing check passed.

## Scale, support, and correction risk

- Judge capacity against actual usage, documented limits, and requested growth. Do not optimize for extreme volumes, concurrency, or file sizes with no demonstrated requirement. A stress-test failure beyond those needs does not alone justify redesign or block acceptance. Preserve inexpensive safeguards and address failures within the required capacity.
- Follow the project's support policy. Do not add compatibility for old versions or unusual configurations explicitly outside that policy just to satisfy an isolated reproduction. Check relevant documentation and configuration before classifying an environment as unsupported. Missing tests or an unclear policy do not establish that it is unsupported.
- Compare the consequence of a rare, minor defect with the regression risk of its proposed correction. Prefer a small root-cause fix. If resolving it would change widely used shared behavior, add substantial complexity, or put common flows at greater risk than the defect itself, defer it with the reason. A difficult fix does not lower the severity of a material defect or excuse an unmet explicit requirement.

Record the evidence for the scale or support boundary and the practical impact of deferred work. Findings confined to these out-of-scope conditions do not trigger repeated fix and review cycles. If the issue also affects supported usage or required capacity, assess that impact normally.

## With other workflows

- In squad, selecting a sound approach from code analysis is sufficient. Prototypes and benchmarks are optional when they would only rank marginal alternatives. Include low-cost improvements in the plan. Skip extra planners and replanning that only pursue marginal performance gains. Defer eligible rare defects without reopening the plan solely for them.
- In council, end the review when only optional performance suggestions or eligible deferred defects remain. Record them and any disagreement. Delegate satisfaction and a perfect score do not justify another round.
- In qa, optional optimizations and eligible rare, minor defects do not by themselves block acceptance or trigger a fixer and retest cycle. Report open findings and actual check results separately. Material defects and violated requirements keep their normal severity and verification.
- In debug, focus on the reported issue and stop once the cause has sufficient evidence for the handoff, or defer an eligible rare defect under the rules above. Leave fixes to the requested implementation work.

In the final report, briefly state relevant improvements, measured or estimated tradeoffs, and deferred work with its practical impact. Keep required functional and security verification in the original workflow.

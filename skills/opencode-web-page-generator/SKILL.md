---
name: opencode-web-page-generator
description: Manager for web-page work. Continuously orchestrate planner, builder, and evaluator roles by reading `agents/*.md` and launching generic subagents until the shared artifacts show the work is complete.
compatibility: opencode
metadata:
  workflow: iterative
  runtime: opencode
---

# OpenCode Web Page Generator

Use this skill as a manager that continuously orchestrates the workflow.

- Do not do planning, implementation, or review in the manager session.
- Do not restate the subagent instructions.
- Keep manager prompts compact, but continue routing until the completion criteria are met.
- Treat `agents/planner.md`, `agents/builder.md`, and `agents/evaluator.md` as role instruction files.
- Start generic subagents from those files instead of assuming `planner`, `builder`, or `evaluator` are registered OpenCode agent types.

Role files:

- `planner` at `agents/planner.md`
- `builder` at `agents/builder.md`
- `evaluator` at `agents/evaluator.md`

## Shared Artifacts

- `product_spec.md`
- `sprint_contract.md`
- `handoff.md`
- `qa_report.md`
- `design-system/MASTER.md`
- `design-system/pages/*.md`

Templates:

- `references/product-spec-template.md`
- `references/sprint-contract-template.md`
- `references/handoff-template.md`
- `references/qa-report-template.md`

Reuse these files. Do not invent parallel documents.

## Handoff Contract

`handoff.md` is the continuity artifact for agent reset, not only end-of-sprint wrap-up.

Use it in two cases:

- a role finishes its round and needs to hand off to the next role
- the current subagent is approaching context saturation and should stop early so a fresh subagent can continue from `handoff.md`

Do not wait for hard failure. Reset proactively when the working context is getting crowded.

## Manager Rules

- Read only what is needed at each step to choose the next hop.
- Use progressive disclosure for all skills and references: load only the skill or reference file needed for the current decision, then only the specific sections needed for the current subtask.
- Do not preload every skill file, every reference file, or full generated docs just because they might become relevant later.
- Prefer file paths and brief state summaries over pasting artifact contents into manager prompts.
- When dispatching, read the relevant file in `agents/`, then launch a generic subagent with that file's instructions plus a compact task prompt.
- Prefer a fresh subagent session when the role changes or a sprint ends.
- Prefer a fresh subagent session when the current session reaches the context reset threshold, even if the role does not change.
- After one subagent finishes, re-check the artifacts and route the next subagent instead of stopping early.
- Do not use `--agent planner`, `--agent builder`, or `--agent evaluator` unless those names are actually registered in the runtime.
- When dispatching work that may start local servers or run integration checks, remind the subagent to use only bounded commands or detached background processes with redirected stdio.
- Treat a missing `handoff.md` after apparent implementation success as a likely builder-session hang during verification, not as evidence that review should be skipped.
- Treat a refreshed `handoff.md` that explicitly requests continuation as a normal reset signal. Dispatch a fresh subagent of the same role when that is the shortest path forward.

## Context Reset Threshold

Do not rely on exact token counts. Use the following operational threshold instead.

Reset the current subagent when any of these becomes true:

- the session has already gone through 12 or more substantive tool calls
- the session has already gone through 3 or more build, test, or verification rounds and meaningful work still remains
- continuing safely would require carrying more than about 10 project files or large command outputs in active context
- the subagent is starting to restate prior state from memory instead of re-reading artifacts
- the subagent has completed a coherent slice of work, but more than a small follow-up remains and a fresh session would be cleaner

When the threshold is reached:

- refresh `handoff.md` immediately
- record exact current state, finished work, remaining scope, blockers, and the next concrete actions
- stop the current subagent after the handoff write succeeds
- launch a fresh subagent that starts from artifacts, especially `handoff.md`, instead of continuing the bloated session

## Routing Rules

1. Missing or stale `product_spec.md` or `sprint_contract.md` -> `planner`
2. Approved sprint ready to implement -> `builder`
3. Build round stopped and `handoff.md` refreshed -> `evaluator`
4. Blocking `qa_report.md` -> `planner` by default, `builder` only for small clear fixes
5. Current role requested reset in `handoff.md` -> fresh subagent of the same role unless another routing rule has higher priority

## Orchestration Rule

In one manager invocation, keep dispatching the next required subagent until the stop condition is satisfied.

- Typical loop is `planner -> builder -> evaluator`, then repeat as needed.
- After each subagent finishes, re-read the relevant artifacts and decide the next hop.
- Stop only when the completion criteria are satisfied.

## Dispatch Prompt Standard

When starting the next subagent:

- read the corresponding role file from `agents/`
- pass that role file's instructions into a generic subagent run
- include the task brief in one short paragraph
- name only the relevant artifact files
- state the exact output files or outcomes expected from that subagent
- if `handoff.md` exists, tell the subagent to treat it as the primary continuity source for the current round
- remind the subagent to read external skills and references progressively instead of loading everything up front
- avoid long summaries when the files already exist on disk
- do not inline replacement instructions that bypass the subagent file

## Stop

Stop only when:

- `qa_report.md` has no unresolved blocking findings
- the current sprint `Done Means` is satisfied
- `handoff.md` is sufficient for continuation or closeout

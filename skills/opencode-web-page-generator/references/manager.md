# Manager Reference

## Responsibility

The manager coordinates the workflow. It continuously orchestrates the planner, builder, and evaluator until the work is complete.

Manager responsibilities:

- decide which subagent runs next
- ensure the shared artifact set exists
- route blocking review findings into the next round
- continue dispatching subagents until the exit criteria are satisfied
- reset long-running work into a fresh subagent when `handoff.md` indicates context saturation

The manager must not:

- do the builder's implementation work
- skip evaluator review and accept the result directly
- send the builder before `sprint_contract.md` exists
- repeat downstream instructions in detail
- produce long summaries when file paths are enough
- substitute a generic Task worker for `planner`, `builder`, or `evaluator`

## Required Reads

Read the minimum needed to choose the next hop.

Preferred order:

1. Check whether `product_spec.md` and `sprint_contract.md` exist.
2. If implementation state matters, read `handoff.md`.
3. If review state matters, read `qa_report.md`.
4. Read the full artifact bodies only when the route is still ambiguous.

If a required artifact is missing, route to the subagent that owns it and continue the workflow from the refreshed artifacts.

`handoff.md` is also the reset artifact for same-role continuation. Do not assume a refreshed handoff always means the role is finished; check whether it requests a fresh session of the same role.

## Stage Gates

Before build:

- `product_spec.md` exists
- `sprint_contract.md` exists
- current sprint scope is explicit
- no unresolved blocking finding prevents the round

Before review:

- the builder subagent has stopped changing code for the round
- `handoff.md` has been refreshed
- a run or build path is executable
- no builder verification command is still holding the session open
- `handoff.md` does not indicate that the builder stopped only for a context reset with unfinished core implementation

## Dispatch Standard

Dispatch with a compact prompt:

- brief task summary
- relevant artifact file paths
- exact expected outputs

Let the subagent read templates and artifacts itself. Do not paste large artifact bodies into the dispatch prompt unless the file cannot be read normally.
Dispatch only to the named subagents defined by this skill, not a generic substitute.
After each subagent returns, re-check the artifacts and dispatch the next required subagent.

When dispatching the builder for any task that may start a local server or run integration checks, explicitly remind it to use only bounded commands or detached background processes with redirected stdio.
When dispatching any fresh replacement subagent after a reset, tell it to start from `handoff.md` and artifacts instead of replaying the prior chat.

## Routing Rules

- If `qa_report.md` contains blocking findings, the default next hop is the planner subagent.
- If the issue is only a small implementation gap and `sprint_contract.md` is already clear, the next hop can be the builder subagent.
- If `product_spec.md` itself has drifted from the task, fix the spec first and then plan the next sprint.
- If the builder appears stalled and `handoff.md` is still missing, assume the round is blocked inside builder-side verification or a long-running command before assuming implementation is incomplete.
- If `handoff.md` says `context-reset`, default to a fresh subagent of the same role unless another artifact clearly requires a role change first.
- Continue routing until the exit criteria are met.

## Context Reset

Prefer a fresh session instead of continuing long chat history when:

- roles change
- a sprint ends
- the current session starts repeating or losing constraints
- the page direction or brief changes materially
- `handoff.md` records that the context reset threshold was reached

In a fresh session, read artifacts first instead of replaying old chat history.

## Exit Criteria

The manager may end the workflow only when:

- `qa_report.md` has no unresolved blocking findings
- the current sprint's `Done Means` is satisfied
- `handoff.md` is sufficient for a new subagent to continue

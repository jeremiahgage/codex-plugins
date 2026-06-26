---
name: ultradev
description: Use only when the user explicitly invokes `$ultradev` or directly asks for a generic development orchestrator that ingests an implementation plan, chooses serial subagent phases, spawns fresh-context worker subagents one at a time, runs a required `$ultrasimplify` simplification phase, and verifies the completed work end to end.
---

# Ultradev

Use this skill to implement an existing plan through staged subagent work while retaining orchestrator ownership of integration and verification.

The `$ultrasimplify` phase is a required phase gate on every successful run. Do not let normal diff review, worker self-review, local cleanup, or final verification substitute for it.

When tradeoffs conflict, prefer correct, maintainable implementation and adequate verification over minimizing runtime, token use, or delegation overhead. Treat simplification as reducing unnecessary code complexity, not as a reason to skip required context, review, or verification.

## Workflow

1. Ingest the plan.
   - Confirm the plan is concrete enough to implement. If it lacks required files, behavior, interfaces, or acceptance criteria, inspect the repo first and ask only for missing product intent that cannot be discovered.
   - Restate the implementation goal, non-goals, constraints, global acceptance criteria, and required verification checks before delegating.

2. Decide the phase structure.
   - Use one worker subagent per phase.
   - Default to 2-5 phases for meaningful plans, 1 phase for trivial plans, and more only when the plan clearly has more independent serial milestones.
   - Keep tightly coupled edits in the same phase. Split phases by ownership boundary, dependency order, or verification checkpoint.
   - Define each phase with a clear objective, allowed write scope, required context, phase acceptance criteria, and expected summary.

3. Spawn subagents serially.
   - Spawn exactly one worker subagent at a time.
   - Use fresh context for every worker; do not fork full orchestrator context unless the user explicitly requests it.
   - Give each worker only the original plan summary, current repo facts needed for the phase, prior phase handoffs, its write scope, constraints, and acceptance criteria.
   - Tell each worker it is not alone in the codebase, must not revert others' work, must not spawn additional agents, and must adapt to existing or prior-phase changes.
   - Require each worker's final response to emit the Phase Handoff Schema.

4. Integrate after each phase.
   - Wait for the current worker before starting the next phase.
   - Review the worker's reported changes and inspect the relevant diff or files.
   - If the worker misses acceptance criteria, reports a blocker, or leaves the phase incomplete, do not start the next phase; inspect the failure, re-scope and respawn that phase, fix locally if small, or halt with an explicit blocker.
   - Forward the worker's Phase Handoff record unchanged to the next phase; if you make local integration edits, amend the record to reflect them.
   - Use phase handoffs as checkpoints. Do not stage, commit, create branches, or push unless the user explicitly asks.
   - Resolve integration problems locally when small and clearly within the phase intent; otherwise create a narrow follow-up phase.

5. Run the required `$ultrasimplify` phase.
   - Run this phase after implementation and repair phases, before final verification, for every successful run.
   - Before simplifying, define the exact simplification scope and the behavior, contracts, telemetry, user-facing flows, and acceptance criteria that must stay fixed.
   - Review the candidate diff for unnecessary abstraction, duplicated logic, dead code, awkward control flow, excessive configuration, and names that obscure intent.
   - Invoke `$ultrasimplify` explicitly from the parent ultradev orchestrator. Do not spawn a generic simplification subagent and ask that child to invoke `$ultrasimplify`.
   - Load and follow `$ultrasimplify`'s own skill instructions for this phase.
   - Give `$ultrasimplify` a self-contained brief with the original plan summary, completed phase handoffs, exact files or modules in scope, invariants, relevant repo docs or examples, required verification, and expected report format.
   - Let `$ultrasimplify` run its normal cleanup review, approved simplification implementation, and behavior-preservation verification workflow.
   - Reject simplification results that trade readability for cleverness, introduce a new architecture, or expand cleanup into unrelated refactoring.
   - Inspect the `$ultrasimplify` result before finalizing. If no meaningful simplification exists, leave the diff unchanged and report that explicitly.
   - If `$ultrasimplify` is unavailable, stop with an explicit blocker instead of silently substituting local cleanup.

6. Verify the full work as orchestrator.
   - After simplification completes, inspect the entire diff and reconcile it with the original plan and global acceptance criteria.
   - Run the project's relevant tests, builds, linters, type checks, or manual verification discovered from the repo.
   - Fix failures locally when small and obvious. For larger failures, spawn a serial repair phase with a narrow scope, then rerun the required `$ultrasimplify` phase before final verification.
   - Do not claim completion until the full work is verified or the remaining blocker is explicit and actionable.

## Phase Handoff Schema

Every worker emits this record as its final response, and the orchestrator forwards it unchanged to later phases. The three dependency-carrying fields (`public_interfaces_added_or_changed`, `invariants_established`, `deferred_followups`) are what make serial handoff reliable; workers must populate them even when the change feels self-contained.

```text
Phase handoff:
- changed_files: paths touched this phase
- public_interfaces_added_or_changed: signatures, contracts, or exports later phases may depend on, or "none"
- invariants_established: guarantees later phases must preserve, or "none"
- behavior_changes: user- or system-visible changes, or "none"
- checks_run: checks or tests run this phase and their results
- deferred_followups: known work intentionally left for a later phase, or "none"
- unresolved_issues: blockers or residual risks, or "none"
```

## Subagent Prompt Shape

Use implementation phase prompts with this structure:

```text
You are implementing phase {n} of {total} for a planned development task.

Original plan summary:
{brief plan}

Prior phase handoffs:
{phase handoff records or "None"}

Your phase objective:
{objective}

Write scope:
{files/modules/areas}

Constraints:
- You are not alone in the codebase; do not revert edits made by others.
- Do not spawn subagents.
- Keep changes within this phase unless a small adjacent edit is required for correctness.
- Follow existing repo conventions.
- When tradeoffs conflict, prefer correctness, maintainability, and adequate verification over minimizing runtime, token use, or delegation overhead.

Acceptance criteria:
{criteria}

When done, emit the Phase Handoff record:
- changed_files: paths touched this phase
- public_interfaces_added_or_changed: signatures, contracts, or exports later phases may depend on, or "none"
- invariants_established: guarantees later phases must preserve, or "none"
- behavior_changes: user- or system-visible changes, or "none"
- checks_run: checks or tests run this phase and their results
- deferred_followups: known work intentionally left for a later phase, or "none"
- unresolved_issues: blockers or residual risks, or "none"
```

Use the required `$ultrasimplify` phase brief with this structure:

```text
Use $ultrasimplify to simplify the completed implementation for a planned development task.

Original plan summary:
{brief plan}

Completed phase handoffs:
{phase handoff records}

Simplification target:
{exact files/modules/areas}

Behavior and contracts that must stay fixed:
{acceptance criteria, external contracts, telemetry, user-facing flows, and other invariants}

Relevant repo context:
{repo docs, conventions, and nearby examples to follow}

Instructions:
- Run the normal Ultrasimplify workflow for this target.
- Simplify without changing functionality, behavior, contracts, telemetry, user-facing flows, or acceptance criteria.
- Prefer small local edits that remove duplication, dead code, unnecessary indirection, awkward control flow, or unclear names.
- Treat simplification as reducing unnecessary code complexity, not as a reason to skip required context, review, or verification.
- Do not introduce new architecture, broaden scope, hide simple logic in clever abstractions, or refactor unrelated code.
- You are not alone in the codebase; do not revert edits made by others.
- If no meaningful simplification exists, leave the files unchanged and say so.

Verification:
{checks to run, or checks already run that must remain valid}

When done, report:
- changed files, or "none"
- simplifications made and why they are simpler
- behavior-preservation checks
- verification run and results
- unresolved issues or residual risks
```

## Final Response

Report the completed phase sequence, major changes, simplification performed or explicitly "none", verification performed, and any residual risks. Keep the final user-facing summary concise and make clear whether the original plan is fully implemented.

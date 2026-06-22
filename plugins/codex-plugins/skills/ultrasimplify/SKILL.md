---
name: ultrasimplify
description: Use only when the user explicitly invokes `$ultrasimplify` or directly asks for a generic simplification orchestrator that scopes an existing implementation, runs parallel cleanup review agents, applies approved behavior-preserving simplifications, and verifies the simplified code.
---

# Ultrasimplify

Use this skill to simplify existing code while preserving behavior. The orchestrator owns scope, invariants, delegation, review, and verification.

The parallel cleanup review is required for every successful run. Do not substitute local cleanup, ordinary review, or final verification for the fresh-context cleanup lanes.

## Workflow

1. Determine the simplification target.
   - Prefer the user's explicit focus when present.
   - If no focus is provided, use current working-tree changes: staged, unstaged, and relevant untracked files.
   - If the working tree is clean, use the diff against `main`.
   - If no focus or diff is discoverable, ask for the simplification target.
   - Reduce unrelated changes to the smallest coherent simplification pass, and call out any excluded areas before delegating.

2. Build the behavior-preservation brief.
   - Read the target files, nearby callers, relevant tests, and 1-2 nearby examples that match the code in scope.
   - Read applicable repo rules or docs, such as `AGENTS.md`, `README.md`, architecture, style, testing, and error-handling docs when present.
   - Identify the behavior, public APIs, data contracts, auth rules, persistence paths, telemetry, logging, accessibility, user-facing flows, and acceptance criteria that must not change.
   - Surface contradictions, ambiguous scope, or risks that simplification could hide behavior changes before delegating.

3. Spawn four cleanup review agents in parallel.
   - Use fresh context for every review agent; do not fork full orchestrator context unless the user explicitly requests it.
   - Prefer read-only explorer-style agents for cleanup review lanes.
   - Give every agent the same target summary, changed surface, behavior-preservation brief, relevant repo context, and verification expectations.
   - Assign one lane per agent:
     - Code reuse: existing helpers, utilities, shared modules, duplicate logic, and hand-rolled patterns that should use repo-local APIs.
     - Simplification: redundant state, dead code, unnecessary indirection, copy-paste, parameter sprawl, awkward nesting, and unclear names.
     - Efficiency: redundant computation, repeated I/O, duplicate API calls, missed safe concurrency, hot-path bloat, leaks, and overly broad reads.
     - Abstraction fit: right ownership layer, boundary violations, misplaced responsibility, repo conventions, and applicable `AGENTS.md` or README guidance.
   - Tell agents to review only; not to edit files, spawn subagents, broaden scope, or hunt for correctness bugs unless a bug directly affects cleanup safety.
   - Require each agent to report concrete cleanup opportunities with evidence, why behavior is preserved, suggested fix direction, checks or searches performed, and residual risk.

4. Reconcile and approve cleanup findings.
   - Wait for all cleanup review agents to finish before applying changes.
   - Surface each agent's concise lane summary to preserve coverage.
   - Deduplicate overlapping findings and merge supporting evidence.
   - Reject findings that change behavior, contracts, telemetry, user experience, error handling, ownership boundaries, or repo architecture without explicit approval.
   - Reject cleanup that trades readability for cleverness, hides simple logic in abstractions, expands into unrelated refactoring, or relies on weak evidence.
   - If no meaningful safe simplification exists, leave files unchanged and report that directly.

5. Apply approved simplifications with one implementation worker.
   - Use a single fresh-context simplification implementation worker after findings are approved.
   - Give the worker only the approved findings, exact write scope, invariants, relevant repo context, and required verification.
   - Tell the worker not to spawn subagents, not to broaden scope, not to revert unrelated edits, and not to change functionality.
   - Require the worker to leave files unchanged and say so if an approved finding proves unsafe or not worth applying.
   - Inspect the changed files or diff directly after the worker completes.
   - Fix a small obvious issue locally only when it is clearly caused by the simplification; otherwise re-scope and respawn the worker once, or halt with an explicit blocker.

6. Verify the simplified code.
   - Run the relevant tests, builds, linters, type checks, or manual checks discovered from the repo and changed surface.
   - Confirm verification matches the files changed and the behavior-preservation brief.
   - Fix small obvious failures locally when they are clearly caused by the simplification; otherwise report the blocker with the failing command and reason.
   - Do not claim completion until behavior preservation is verified or the remaining blocker is explicit and actionable.

## Simplification Rules

Prefer:

- removing duplication
- deleting dead or redundant code
- flattening unnecessary nesting
- collapsing needless indirection
- reducing state, configuration, and moving parts
- replacing special-case logic with clearer shared logic
- extracting small focused helpers when they improve readability
- using names that reveal intent
- explicit readable control flow over compact clever code

Avoid:

- changing user-visible behavior
- changing public APIs or data/backend contracts unless explicitly requested
- removing telemetry, logging, auth checks, accessibility, cleanup, permission handling, or error handling as "cleanup"
- introducing a new architecture or broad pattern without approval
- moving logic across established ownership boundaries without a strong local reason
- hiding simple logic behind abstractions
- dense one-liners, nested ternaries, or clever control flow
- broad rewrites when local incremental edits would work

## Subagent Prompt Shape

Use cleanup review lane prompts with this structure:

```text
You are one lane of a parallel cleanup review for an existing implementation.

Simplification target:
{exact files/modules/areas}

Changed surface:
{full changed files/modules/diff summary}

Behavior and contracts that must stay fixed:
{public APIs, data contracts, telemetry, auth, persistence, logging, accessibility, user-facing flows, acceptance criteria, and other invariants}

Relevant repo context:
{repo docs, rules, tests, nearby examples, and conventions to follow}

Your cleanup lane:
{one of: code reuse, simplification, efficiency, abstraction fit}

Lane focus:
{narrow lane-specific concerns and examples}

Constraints:
- Review only; do not edit files.
- Do not hunt for correctness bugs unless a bug directly affects cleanup safety.
- Do not recommend changes that alter functionality, user-visible behavior, public APIs, data contracts, telemetry, error handling, or ownership boundaries.
- Do not introduce new architecture, broaden scope, or suggest unrelated refactoring.
- Do not hide simple logic in clever abstractions.
- Do not spawn subagents.

Verification:
{checks the final simplification should preserve or rerun}

When done, report:
- cleanup opportunities with file and line evidence when possible
- why each suggestion preserves behavior
- minimal fix direction
- checks or searches performed
- lane coverage summary
- residual risks or "none"
```

Use the simplification implementation worker prompt with this structure:

```text
You are applying approved cleanup findings to an existing implementation without changing functionality.

Simplification target:
{exact files/modules/areas}

Approved cleanup findings:
{deduplicated findings approved by the orchestrator}

Behavior and contracts that must stay fixed:
{public APIs, data contracts, telemetry, auth, persistence, logging, accessibility, user-facing flows, acceptance criteria, and other invariants}

Relevant repo context:
{repo docs, rules, tests, nearby examples, and conventions to follow}

Constraints:
- Apply only approved cleanup findings that are still safe after inspecting the code.
- Do not change functionality, user-visible behavior, public APIs, data contracts, telemetry, error handling, or ownership boundaries unless explicitly stated above.
- Do not introduce new architecture, broaden scope, or refactor unrelated code.
- Do not hide simple logic in clever abstractions.
- Do not spawn subagents.
- You are not alone in the codebase; do not revert edits made by others.
- If an approved finding is unsafe or not worth applying, leave that area unchanged and say why.
- If no meaningful simplification remains, leave files unchanged and say so.

Verification:
{checks to run, or checks already run that must remain valid}

When done, report:
- changed files, or "none"
- approved findings applied and skipped
- simplifications made and why they are simpler
- behavior-preservation checks
- verification run and results
- unresolved issues or residual risks
```

## Final Response

Report the cleanup review lane coverage, what was simplified, why the result is simpler, how behavior preservation was checked, verification performed, and any residual risks. If no meaningful simplification was possible, say that directly. Keep the final user-facing summary concise.

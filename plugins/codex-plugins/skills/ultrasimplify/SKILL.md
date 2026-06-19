---
name: ultrasimplify
description: Use only when the user explicitly invokes `$ultrasimplify` or directly asks for a generic simplification orchestrator that scopes an existing implementation, spawns one fresh-context simplification worker, preserves behavior and contracts, reviews the result, and verifies the simplified code.
---

# Ultrasimplify

Use this skill to simplify existing code while preserving behavior. The orchestrator owns scope, invariants, delegation, review, and verification.

The simplification worker is required for every successful run. Do not substitute local cleanup, ordinary review, or final verification for the fresh-context simplification pass.

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

3. Spawn exactly one simplification worker.
   - Use fresh context for the worker; do not fork full orchestrator context unless the user explicitly requests it.
   - Give the worker a self-contained brief with exact files, relevant repo context, invariants, simplification priorities, verification requirements, and expected report format.
   - Tell the worker not to spawn subagents, not to broaden scope, not to revert unrelated edits, and not to change functionality.
   - Require the worker to leave files unchanged and say so if no meaningful simplification exists.

4. Review the worker result.
   - Inspect the changed files or diff directly.
   - Reject changes that alter behavior, contracts, telemetry, user experience, error handling, or repo architecture without explicit approval.
   - Reject cleanup that trades readability for cleverness, hides simple logic in abstractions, or expands into unrelated refactoring.
   - If the worker misses scope or verification requirements, re-scope and respawn the worker once, fix a small obvious issue locally, or halt with an explicit blocker.

5. Verify the simplified code.
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

Use simplification worker prompts with this structure:

```text
You are simplifying an existing implementation without changing functionality.

Simplification target:
{exact files/modules/areas}

Current behavior and contracts that must stay fixed:
{public APIs, data contracts, telemetry, auth, persistence, logging, accessibility, user-facing flows, acceptance criteria, and other invariants}

Relevant repo context:
{repo docs, rules, tests, nearby examples, and conventions to follow}

Simplification priorities:
- Remove duplication, dead code, redundant state, unnecessary indirection, awkward nesting, or unclear names.
- Prefer small local edits that make the code easier to read and maintain.
- Extract helpers only when they improve clarity.

Constraints:
- Do not change functionality, user-visible behavior, public APIs, data contracts, telemetry, error handling, or ownership boundaries unless explicitly stated above.
- Do not introduce new architecture, broaden scope, or refactor unrelated code.
- Do not hide simple logic in clever abstractions.
- Do not spawn subagents.
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

Report what was simplified, why the result is simpler, how behavior preservation was checked, verification performed, and any residual risks. If no meaningful simplification was possible, say that directly. Keep the final user-facing summary concise.

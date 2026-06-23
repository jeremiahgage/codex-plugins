---
name: ultrareview-plan
description: Review and refine an existing implementation plan with focused subagent reviewers, then synthesize one revised, agent-ready replacement plan. Use when the user supplies a plan and asks for subagent, multi-agent, parallel-agent, or independent-agent review, critique, validation, refinement, or plan improvement before implementation.
---

# Ultrareview Plan

Use this skill to turn a supplied implementation plan into one revised, decision-complete handoff plan for a later implementation agent. This is a planning-only workflow.

## Required Input

Require an existing plan. Optional context may include:

- the original task, ticket, issue, or requirements
- a branch, diff, file path, package path, failing test output, or repo focus
- product, API, compatibility, or rollout constraints
- reviewer lenses requested by the user

If no plan is supplied, stop and ask the user for the plan to review. Do not scout the repo or generate a fresh plan from only a task brief.

## Non-Negotiable Rules

- Do not edit files, apply patches, stage, commit, run codegen, run formatters, or spawn implementation workers.
- Review the supplied plan. Do not ask subagents to independently generate a new plan from scratch.
- Keep the parent agent responsible for final accuracy, conflict resolution, scope control, and the final revised plan.
- Let reviewer subagents inspect the repo as needed to validate the supplied plan's claims, touchpoints, constraints, and omissions.
- Read repo instructions and relevant local skills before finalizing the revised plan.
- Ask the user only for material product, API, scope, or tradeoff decisions that block a correct revised plan.
- Choose conservative defaults for non-blocking ambiguity and list them under assumptions.
- Do not include unrelated working-tree state, untracked files, subagent bookkeeping, raw reviewer notes, or speculative refactors in the final plan.

## Plan Quality Bar

A final plan is ready only when a capable implementation agent can proceed without product, API, or scope decisions. It should state:

- the goal and expected user/system outcome
- current repo facts that matter, with enough specificity to ground the plan
- hard constraints from docs, architecture, tests, APIs, or compatibility requirements
- non-goals and scope edges that prevent overreach
- behavior contract, state/data expectations, and important edge cases
- likely touchpoints, ownership boundaries, and public interfaces or types when relevant
- acceptance checks that define done behavior
- verification commands or scenarios, including focused tests when known
- assumptions/defaults and only true blocking questions

## Workflow

1. Confirm and frame the supplied plan:
   - Extract the plan goal, proposed changes, assumptions, explicit decisions, likely touchpoints, tests, and open questions.
   - Read nearest repo instructions and project docs that govern the plan's scope.
   - Inspect likely target files, adjacent patterns, current tests, and current git status only as needed to validate or revise the supplied plan.
   - Identify applicable local skills, architecture constraints, interfaces, test expectations, and ambiguity.
   - If the plan itself is missing or a blocker is not discoverable from the repo, ask a concise clarification before spawning reviewers.
2. Choose reviewer subagents adaptively:
   - Use 2 reviewers for narrow, low-risk plan refinements.
   - Use 3 reviewers by default.
   - Use 4 reviewers for cross-layer, migration, data-contract, navigation, backend, or high-risk plans.
   - Prefer independent reviewer agents with `fork_context: false` when the agent tool supports it.
3. Assign focused review lenses:
   - architecture and local-pattern fit
   - behavior, data, API, state, or backend contract correctness
   - tests, edge cases, and failure modes
   - implementation simplicity, maintainability, and risk
   - rollout, compatibility, or migration risk when relevant
4. Require each reviewer to return:
   - `plan_summary`
   - `repo_validation`
   - `must_fix_issues`
   - `should_consider`
   - `plan_strengths_to_preserve`
   - `missing_decisions`
   - `recommended_plan_changes`
   - `confidence`
5. Synthesize with one separate non-editing agent:
   - Spawn one synthesis agent with `fork_context: false` when supported.
   - Pass only the supplied plan, optional task context, parent repo-validation facts, repo constraints, and raw reviewer outputs.
   - Ask it to produce a full replacement `<proposed_plan>`, not a critique report, patch list, or reviewer summary.
   - Require conservative defaults for non-blocking ambiguity and explicit blocking questions only when implementation cannot proceed correctly without the user's decision.
6. Parent review:
   - Reconcile reviewer disagreements against repo evidence.
   - Verify the synthesis is complete, scoped, and decision-ready.
   - Remove critique language, subagent references, raw reviewer notes, speculative work, unrelated refactors, and unnecessary implementation prescription.
   - Preserve important strengths of the original plan when they remain valid.
   - Ensure the final answer is one complete replacement plan that can be handed directly to an implementation agent.

## Delegation Prompt Templates

Reviewer prompt:

```text
You are one independent reviewer for a supplied implementation plan. Do not edit files.

Task context:
<optional original task, ticket, branch, diff, repo focus, constraints>

Supplied plan:
<plan>

Known repo constraints:
<parent repo-validation facts and applicable rules>

Review lens:
<one specific lens>

Inspect the codebase only as needed to validate the supplied plan. Review the plan; do not create an unrelated plan from scratch.

Return:
- plan_summary
- repo_validation
- must_fix_issues
- should_consider
- plan_strengths_to_preserve
- missing_decisions
- recommended_plan_changes
- confidence

Ground must-fix issues and missing decisions in repo evidence or explicit uncertainty. Keep recommendations focused on changes to the supplied plan.
```

Synthesis prompt:

```text
You are synthesizing independent reviews of a supplied implementation plan. Do not edit files.

Task context:
<optional original task, ticket, branch, diff, repo focus, constraints>

Supplied plan:
<plan>

Parent repo-validation facts and constraints:
<facts>

Reviewer outputs:
<raw outputs>

Produce one concise replacement implementation plan wrapped in a single <proposed_plan> block. The plan must be complete enough for a capable implementation agent to execute without reading the reviewer notes. It must not be a critique report, diff against the old plan, or patch list.

Include repo facts, hard constraints, non-goals, behavior/data/API contracts, likely touchpoints, public interfaces or types when relevant, test cases or scenarios, verification, and assumptions/defaults. Include blocking questions only when implementation cannot proceed correctly without the user's decision. Remove unrelated repo state, subagent bookkeeping, speculative refactors, and routine implementation mechanics.
```

## Final Output

Return one concise replacement plan wrapped in a single `<proposed_plan>` block. Include:

- `Title`
- `Summary`
- `Repo Facts`
- `Key Changes`
- `Public Interfaces/Types` when relevant
- `Behavior/Data/API Contracts` when relevant
- `Scope Edges`
- `Test Cases/Scenarios`
- `Verification`
- `Assumptions/Defaults`
- `Blocking Questions` only if any true blockers remain

The final output must read as the plan to execute, not as a review of the prior plan. Keep sections compact and omit sections that truly do not apply. Do not include exhaustive file-by-file edits, symbol-by-symbol instructions, or detailed implementation sequences unless repo evidence shows those details are required to prevent a concrete mistake.

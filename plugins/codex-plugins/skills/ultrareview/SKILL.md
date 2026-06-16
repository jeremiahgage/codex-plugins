---
name: ultrareview
description: Use only when the user explicitly invokes `$ultrareview` or directly asks for a parallel subagent code review workflow that ingests a focus or current changes, spawns fresh-context review agents at the same time across review lanes, surfaces each agent's summary, and compiles a prioritized issue list.
---

# Ultrareview

Use this skill to run a parallel, multi-lane code review while retaining orchestrator ownership of the final prioritized findings.

## Workflow

1. Determine the review target.
   - Use the user's provided focus when present.
   - If no focus is provided, review current changes: staged diff, unstaged diff, and relevant untracked files.
   - If no focus and no current changes are discoverable, ask for the review target.
   - Summarize the target, changed areas, known constraints, and any explicit review priorities before delegating.

2. Decide the review lanes.
   - Use parallel lanes that match the change's risk and surface area.
   - Default lanes: correctness and data integrity, architecture and style consistency, performance and scalability, tests and coverage, security and error handling, user experience and accessibility.
   - Use 2-3 lanes for small changes, 4-6 lanes for larger or riskier changes.
   - Merge irrelevant or overlapping lanes. Add a domain-specific lane only when the repo or focus clearly calls for it.
   - Partition lanes by concern, not file ownership. Every lane should see the whole changed surface unless the target is explicitly narrower.
   - Define each lane with a narrow review concern and expected evidence.

3. Spawn review subagents in parallel.
   - Spawn all review agents at the same time.
   - Prefer read-only explorer-style agents for review lanes.
   - Use fresh context for every agent; do not fork full orchestrator context unless the user explicitly requests it.
   - Give each agent the shared review target summary, the full changed surface, and its lane-specific focus.
   - Tell agents not to edit files, not to spawn more agents, and not to duplicate other lanes except where a finding is directly relevant.
   - Require each agent to report actionable findings with severity, file and line evidence when possible, rationale, and a concise lane summary.

4. Collect and reconcile results.
   - Wait for all review agents to finish before producing the final review.
   - Surface each agent's concise summary to preserve review coverage.
   - Deduplicate overlapping findings and merge supporting evidence.
   - Verify high-severity claims against the code or diff before presenting them when feasible.

5. Compile the final issue list.
   - Lead with findings, ordered by severity and confidence.
   - For each issue include file and line reference when available, impact, and the minimal fix direction.
   - Include open questions only when they affect review confidence or implementation safety.
   - If no issues are found, say so clearly and note residual risk or unverified areas.

## Severity Scale

- `critical`: Likely data loss, security exposure, outage, or broken core workflow.
- `high`: Real user-facing bug or serious regression risk.
- `medium`: Correctness, maintainability, or test gap that should be fixed.
- `low`: Minor issue, polish, or future-risk note.

## Subagent Prompt Shape

Use prompts with this structure:

```text
You are reviewing one lane of a parallel code review.

Shared review target:
{focus or current-change summary}

Changed areas:
{full changed surface: files/modules/diff summary}

Your review lane:
{lane name and narrow focus}

Instructions:
- Review only; do not edit files.
- Do not spawn subagents.
- Review the full changed surface through your assigned concern; do not limit review to a preselected file subset.
- Prioritize actionable bugs, regressions, missing tests, and maintainability risks.
- Use severity labels: critical, high, medium, low.
- Avoid broad style preferences unless they create real consistency or maintenance risk.
- Provide file and line evidence when possible.

When done, report:
- findings with severity, evidence, impact, and fix direction
- checks or searches performed
- lane coverage summary
- residual uncertainty
```

## Final Response

Use a review-first format:

1. Findings ordered by severity.
2. Open questions or assumptions.
3. Agent coverage summary.
4. Residual risk or test gaps.

Keep summaries concise. Do not bury findings below process notes.

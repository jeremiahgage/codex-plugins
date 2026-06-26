---
name: ultraadversarialreview
description: Use only when the user explicitly invokes `$ultraadversarialreview` or directly asks for a Claude-backed adversarial code review workflow that runs `claude -p`, validates Claude's candidate findings against local code, and returns prioritized confirmed issues.
---

# Ultra Adversarial Review

Use this skill to run a Claude-backed adversarial review while retaining Codex ownership of final findings. Claude output is raw candidate input, not the review result.

This is a review-only workflow. Do not edit files, apply patches, stage, commit, create branches, push, run formatters that rewrite files, or spawn implementation workers.

## Non-Negotiable Rules

- Do not report Claude's findings verbatim as confirmed.
- Every confirmed finding must be independently validated against the local code, diff, tests, docs, or reachable behavior.
- Reject or downgrade false positives, speculative issues, duplicates, non-reachable behavior, weak style preferences, and findings without actionable impact.
- If `claude -p` is unavailable, fails, times out, or returns unusable output, stop with an explicit blocker instead of substituting another review workflow.
- If the review target likely contains secrets, credentials, private keys, tokens, or other sensitive material, stop before invoking Claude and report the blocker.

## Workflow

1. Determine the review target.
   - Prefer the user's explicit focus when present.
   - If no focus is provided, review current changes: staged diff, unstaged diff, and relevant untracked files.
   - If no focus and no current changes are discoverable, ask for the review target.
   - Summarize the target, changed areas, known constraints, and any explicit review priorities before invoking Claude.

2. Inspect for sensitive material before external review.
   - Review the target diff or files for likely secrets, credentials, private keys, tokens, certificates, customer data, or other material that should not be sent to an external model.
   - If sensitive material is likely present, do not run Claude. Stop with a concise blocker and identify the affected path or diff area without exposing the secret.
   - If only non-sensitive code and ordinary test fixtures are present, continue.

3. Build the Claude adversarial review prompt.
   - Include the review target summary, changed surface, explicit user priorities, relevant constraints, and any checks already run.
   - Ask Claude to focus on real bugs, regressions, missed edge cases, security issues, data loss, compatibility breaks, concurrency issues, and meaningful test gaps.
   - Ask Claude to avoid style-only preferences, broad rewrites, speculative architecture opinions, and findings without concrete file or behavior evidence.
   - Require structured candidate findings with severity, file/line evidence when possible, impact, reasoning, and suggested fix direction.

4. Run Claude in bounded read-only mode.
   - Use `claude -p --permission-mode plan --no-session-persistence "<adversarial review prompt>"`.
   - Do not use bypass-permissions flags.
   - Do not allow Claude to edit files or perform implementation work.
   - If the command fails, is unavailable, or returns unusable output, stop with an explicit blocker that includes the failing command shape and reason.

5. Validate Claude's candidate findings locally.
   - Read the cited code, relevant diff, nearby callers, tests, and docs needed to confirm or reject each candidate.
   - For each candidate decide:
     - confirmed: true issue with local evidence and reachable impact
     - downgraded: real but lower severity, narrower impact, duplicate, or better framed as residual risk/test gap
     - rejected: false positive, unsupported, speculative, not reachable, or not actionable
   - Merge duplicates and preserve the strongest evidence.
   - Verify high-severity claims against the code or diff before presenting them.

6. Compile the final review.
   - Lead with numbered confirmed findings ordered by severity and confidence.
   - For each confirmed finding include severity, file and line reference when available, impact, evidence, and minimal fix direction.
   - Include rejected or downgraded Claude findings only as a concise secondary section.
   - Include residual risks or test gaps last.
   - If no confirmed findings remain after validation, say so clearly and summarize what was checked.

## Severity Scale

- `critical`: Likely data loss, security exposure, outage, or broken core workflow.
- `high`: Real user-facing bug or serious regression risk.
- `medium`: Correctness, maintainability, compatibility, or test gap that should be fixed.
- `low`: Minor issue, polish, or future-risk note.

## Claude Prompt Shape

Use a prompt with this structure:

```text
You are performing an adversarial code review. Review only; do not edit files.

Review target:
{focus or current-change summary}

Changed surface:
{files/modules/diff summary or explicit target}

Known constraints and priorities:
{user priorities, repo constraints, acceptance criteria, or "None"}

Instructions:
- Find real bugs, regressions, missed edge cases, security issues, data loss risks, compatibility breaks, concurrency issues, and meaningful test gaps.
- Prioritize actionable issues with concrete evidence.
- Avoid style-only preferences, broad rewrites, speculative architecture opinions, and findings without reachable impact.
- Do not modify files or propose unrelated refactors.

When done, return candidate findings only:
- severity: critical, high, medium, or low
- file/line evidence when possible
- issue
- impact
- reasoning
- minimal fix direction
- confidence
```

## Validation Record

Before finalizing, maintain an internal validation record for Claude's candidates:

```text
Candidate validation:
- claude_claim:
- status: confirmed | downgraded | rejected
- local_evidence:
- severity_after_validation:
- reason:
- final_disposition:
```

## Final Response

Use a review-first format:

1. Numbered confirmed findings ordered by severity and confidence. Use numbered entries for issues found, not bullets.
2. Rejected or downgraded Claude findings.
3. Residual risks or test gaps.

Do not bury confirmed findings below process notes. Keep the final user-facing summary concise, and make clear that Claude's output was independently validated before inclusion.

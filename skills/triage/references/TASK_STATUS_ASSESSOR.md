# Task Status Assessor

Use this workflow to determine the real state of one task. This is the core
decision protocol for the rest of the pack.

Apply the core invariants from `../SKILL.md`. Use `TASK_NOTE_TEMPLATE.md` for
persisted task notes.

## Inputs

- One task key, PR, branch, Slack thread, or local worktree.
- The canonical task note from `TASK_NOTE_TEMPLATE.md`, if it exists.
- Linear issue and comments.
- Related GitHub issues, branches, PRs, checks, reviews, mergeability, and
  unresolved review threads.
- Related Slack QA/dev threads.
- Related Notion specs.
- Local worktree status and unpushed changes.

## Source Rules

- Linear is the canonical task source if a Linear issue exists.
- GitHub PRs are implementation state.
- Local git is the source for unpushed or resumed work.
- Slack is QA/dev signal and human decision context.
- Notion is spec/context only.
- Do not assume a task is done just because code changed.
- Do not assume a task is blocked just because a note says so; verify current
  PR/check/comment state.

## Decision Tree

Answer these in order:

1. Does a real task exist, or is this an orphan/duplicate/stale signal?
2. Is the task missing reproduction, expected behavior, target repo, owner, or
   scope?
3. Has implementation started locally or on a branch?
4. Are all required repos represented by PRs or explicit non-code actions?
5. Is there an open PR for each implementation branch?
6. Are any PRs draft, conflicted, stale, or missing required commits?
7. Are there unresolved human review comments or requested changes?
8. Are there Copilot/bot comments after the latest push?
9. Are Copilot comments new, repeated, contradictory, already addressed, or
   low-signal?
10. Are required checks failing, pending, missing, or green?
11. Are failures caused by this change, unknown, or proven unrelated?
12. Are approvals, QA signoff, product answers, or release gates missing?
13. Is the task merged/shipped/closed?

## State Selection

Choose exactly one:

- `new`: issue exists and no implementation is confirmed.
- `needs_context`: next implementation step would require guessing.
- `in_progress`: code/worktree/branch exists but is not review-ready.
- `ready_for_review`: PR exists, no known blocking comments, ready for human
  review.
- `review_iteration`: review feedback needs action or explicit rejection.
- `ready_to_merge`: all required gates pass.
- `blocked_human`: waiting on a person, external team, QA, approval, product
  answer, infra owner, or release decision.
- `blocked_ci`: CI is failing, pending too long, missing, or unknown and not
  proven unrelated.
- `blocked_merge_conflict`: PR has merge conflicts and needs rebase from a
  freshly updated staging/main branch.
- `done`: merged/shipped/closed complete.
- `ignore`: no useful action; duplicate, orphan, obsolete, or low-signal only.

Tie-breakers:

- Human unresolved review beats Copilot comments: `review_iteration`.
- Unknown CI failure beats ready states: `blocked_ci`.
- Proven unrelated infra/deploy failure that needs another owner:
  `blocked_human`.
- Merge conflict beats review-ready and merge-ready states:
  `blocked_merge_conflict`.
- Missing expected behavior beats implementation: `needs_context`.
- Merged PR but Linear stale: usually `done` with a Linear-update action, unless
  QA/release is explicitly still open.

## Questions

Ask questions only when the answer changes the next action. Ask at most five,
ordered by unblock value. Prefer concrete choices:

- "Should expired coupons return 400 or be ignored?"
- "Is mobile checkout in scope for this task or a follow-up?"
- "Which repo owns this contract?"

If one reasonable assumption is safe and reversible, state it and continue.

## Forbidden

- Do not change code.
- Do not post external comments unless explicitly asked.
- Do not mark done without checking PR merge state or explicit closure.
- Do not merge, push, close, approve, dismiss, or request reviews.
- Do not treat Copilot as authoritative.

## Task Note Update

Update or propose an update to the canonical task note:

- `state`
- `confidence`
- `last_checked_at`
- `Summary`
- `Review rounds`
- `Open comments`
- `Local validation`
- `Blockers`
- `Next action`
- External write drafts, if needed

## Output Contract

Return strict JSON plus no extra prose when another agent will consume it:

```json
{
  "state": "review_iteration",
  "confidence": "high",
  "evidence": [
    "GitHub PR owner/repo#482 has 2 unresolved human comments",
    "Copilot left 3 comments after the latest push",
    "Linear ENG-1234 is still In Progress",
    "CI passed except an unrelated e2e flake"
  ],
  "blockers": [
    "Unresolved human review comments"
  ],
  "next_action": "Address human comments first; reject low-signal Copilot comments with rationale",
  "recommended_execution_prompt": "Work in the current worktree. Address the two unresolved human review comments on owner/repo#482. Do not apply Copilot suggestions unless they are valid after reading the code. Run the affected tests and typecheck.",
  "requires_human": false,
  "note_update": {
    "path": ".codex/tasks/ENG-1234.md",
    "summary": "State review_iteration; human comments are the next blocker"
  }
}
```

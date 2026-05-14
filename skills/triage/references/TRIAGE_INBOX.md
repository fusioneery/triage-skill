# Triage Inbox

Use this workflow for a daily or repeated sweep of active work across Linear,
GitHub, Slack, Notion, and local task notes.

Apply the core invariants from `../SKILL.md`. Use `TASK_NOTE_TEMPLATE.md` for
persisted task notes.

## Goal

Determine the real status of every active work item and return the next few
actions that matter most.

## Sources To Read

- Local notes: `.codex/tasks/*.md`.
- Linear assigned/open issues, especially issues in active workflow states.
- GitHub open PRs authored by the user, PRs requesting the user's review, and
  PRs linked from active Linear issues.
- GitHub unresolved review threads, latest reviews, checks, approvals, merge
  conflicts, draft state, and branch freshness.
- Slack QA/dev mentions and linked task threads. Search by task key, PR number,
  branch, and error text.
- Notion specs linked from Linear, Slack, or GitHub.

## Task Universe

Build the task set from the union of:

- Active Linear issues.
- Open GitHub PRs authored by the user.
- Local task notes.
- Slack threads that mention a task key, branch, PR, or urgent QA/dev issue.

De-duplicate by Linear key first. If no Linear key exists, group by PR or branch
and mark as `ignore`, `new`, or `needs_context` depending on evidence.

## Classification

For every task, apply `TASK_STATUS_ASSESSOR.md` and assign exactly one state:

```text
new
needs_context
in_progress
ready_for_review
review_iteration
ready_to_merge
blocked_human
blocked_ci
blocked_merge_conflict
done
ignore
```

Do not leave a task as "unknown" in the final output. If evidence is missing,
use `needs_context`.

## Immediate Action Ranking

Rank "Needs immediate action" by:

1. User-blocking production/QA regressions.
2. PRs with failing checks caused by the change.
3. Human review comments that block merge.
4. Merge conflicts on otherwise valuable work.
5. Copilot review rounds where the comment is valid or probably valid.
6. Fresh tasks with clear repro and small scope.

Rank "Waiting on humans" by:

1. Required approval missing.
2. Product/QA decision needed.
3. Infra/release owner needed.
4. Reviewer has not responded.

Classify low-signal Copilot churn, stale duplicates, and unrelated watch items
as "Safe to ignore/watch".

## Questions

Ask questions only for tasks where the next action depends on user input. Keep
them short and batched:

- "For ENG-1234, should expired coupons return 400 or be ignored?"
- "For ENG-1277, is mobile checkout in scope?"

If more than five questions appear, return the top five and mark the rest as
`needs_context`.

## Forbidden

- Do not execute code changes.
- Do not merge, push, approve, dismiss, or close anything.
- Do not post to Slack, Linear, GitHub, or Notion unless explicitly asked.
- Do not infer completion from a local branch alone.
- Do not make Copilot comments equal to human review.

## Task Note Handling

For every task:

1. Read the note if it exists.
2. Recompute state from live sources.
3. Update the task note after assessment.
4. Append evidence instead of erasing prior history.

## Output Format

```markdown
# Triage summary

## Needs immediate action
- ENG-1234 - review_iteration - PR blocked by two human review comments.
- ENG-1277 - blocked_ci - API check fails on changed test.

## Waiting on humans
- ENG-1190 - blocked_human - missing backend approval on owner/api#882.

## Ready to merge
- ENG-1204 - ready_to_merge - frontend#482 green, approved, no conflicts.

## Safe to ignore/watch
- ENG-1212 - ignore - Copilot repeated a rejected suggestion from round 3.

## Recommended next 3 actions
1. Fix ENG-1234 human review comments.
2. Investigate ENG-1277 failing API check.
3. Ping backend reviewer for ENG-1190 approval.

## Questions
- ENG-1301: Should expired coupons return 400 or be ignored?
```

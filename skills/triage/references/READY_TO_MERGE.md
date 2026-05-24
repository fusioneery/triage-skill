# Ready To Merge

Use this workflow when a task or PR looks done but is not merged, or when the user
asks why it is not merged yet.

Apply the core invariants from `../SKILL.md`. Use `TASK_NOTE_TEMPLATE.md` for
persisted task notes.

## Goal

Decide whether the task is truly ready to merge and identify the exact remaining
gate if not.

## Required Sources

- Canonical task note.
- Linear status, assignee, linked PRs, QA/test-plan comments, and approval or
  release requirements.
- All required GitHub PRs for the task.
- PR mergeability, branch freshness, draft state, required checks, approvals,
  unresolved review threads, requested changes, labels, and required reviewers.
- Copilot/bot comments and any explicit rejection replies.
- Local branch status only if unpushed changes may exist.
- Slack/QA threads for signoff if the task note or Linear says QA is required.

## Ready Criteria

A task is ready to merge only if:

- All required PRs exist.
- All required PRs are not draft.
- All required CI checks pass, or failures are confirmed non-required and
  unrelated.
- No merge conflicts.
- No unresolved human review comments.
- No active requested-changes review.
- Required approvals exist.
- Copilot comments are addressed, already addressed, or explicitly rejected with
  rationale.
- Required QA plan or handoff exists in Linear when QA is needed.
- No unpushed local changes are required for the PR.

If any criterion fails, canonical state is not `ready_to_merge`.

## Decision Substates

Use the canonical states in the task note, but the assessment may include a
more precise readiness substate:

- `ready_to_merge`
- `ready_to_merge_blocked_on_approval`
- `ready_to_merge_blocked_on_ci`
- `ready_to_merge_blocked_on_conflict`
- `ready_to_merge_blocked_on_review`
- `ready_to_merge_blocked_on_qa`
- `ready_to_merge_blocked_on_copilot_round`
- `not_ready_implementation_remaining`

Map these back to canonical states:

- approval, QA, product, release, or external owner needed -> `blocked_human`
- failing/unknown required checks -> `blocked_ci`
- merge conflicts -> `blocked_merge_conflict`
- unresolved review or Copilot round -> `review_iteration`
- implementation still missing -> `in_progress`
- all gates pass -> `ready_to_merge`

## Forbidden

- Do not merge. This assessment is read-only.
- **NEVER enable auto-merge.** Do not invoke `gh pr merge ... --auto`, do not
  call the `enablePullRequestAutoMerge` GraphQL mutation, and do not hit the
  `PUT /repos/{owner}/{repo}/pulls/{number}/automerge` REST endpoint. A
  `ready_to_merge` verdict is a recommendation to the human, not a license
  for the agent to merge or arm an auto-merge. Auto-merge fires silently on
  green CI and skips the explicit user confirmation this workflow exists to
  produce.
- **NEVER flip a draft PR to ready-for-review** as part of a ready-to-merge
  assessment. Use `markPullRequestReadyForReview` only on explicit user
  request that names the PR.
- Do not approve your own PR.
- Do not dismiss reviews or resolve threads unless explicitly asked.
- Do not post approval nudges unless explicitly asked.
- Do not call "ready" while required checks are pending.
- Do not treat Copilot comments as blockers if they were explicitly rejected
  with evidence.

## Writes

Update or propose the task note:

- canonical state
- readiness substate
- evidence for each required PR
- exact remaining gate
- local validation summary
- QA test plan draft if needed
- Slack/Linear/GitHub message drafts if a human is needed

## Output Format

```markdown
# Ready-to-merge assessment

State: ready_to_merge_blocked_on_approval
Canonical state: blocked_human

## Evidence
- owner/frontend#482 CI passed.
- owner/api#913 CI passed.
- No unresolved review threads.
- Missing required approval on owner/api#913.

## Action for user
Ping backend reviewer in Slack.

## Slack message
Hey, owner/api#913 is green and has no unresolved threads. Could you take the final approval pass when you have a moment?

## QA test plan to post to Linear
1. Apply valid discount code.
2. Apply expired discount code.
3. Apply stacked coupon scenario.
4. Verify checkout total and backend order payload.
```

For machine consumption:

```json
{
  "canonical_state": "blocked_human",
  "readiness_state": "ready_to_merge_blocked_on_approval",
  "confidence": "high",
  "remaining_gate": "Missing required backend approval on owner/api#913",
  "next_action": "Ping backend reviewer",
  "can_merge_now": false
}
```

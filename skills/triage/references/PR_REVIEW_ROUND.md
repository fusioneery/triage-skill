# PR Review Round

Use this workflow when a PR has human or Copilot review feedback and the question
is whether to change code, reject feedback, wait, or merge.

Apply the core invariants from `../SKILL.md`. Use `TASK_NOTE_TEMPLATE.md` for
persisted task notes.

## Goal

Assess the current review round as signal, not as an instruction queue.

Copilot comments are advisory, not authoritative. Human reviewer comments have
priority unless they are clearly stale or contradicted by newer decisions.

## Sources To Read

- Canonical task note and previous `Review rounds`.
- GitHub PR timeline, latest commits, latest push SHA, reviews, unresolved
  threads, check runs, mergeability, and approvals.
- Copilot/bot comments and any previous replies rejecting or addressing them.
- Previous task-note evidence about accepted/rejected Copilot suggestions.
- Local branch diff if code changed since the last review round.
- Linear/Slack/Notion only when review feedback depends on product or QA
  context.

## Metrics To Compute

Return these explicitly:

- `round_number`
- `copilot_comments_new`
- `copilot_comments_repeated`
- `copilot_comments_contradict_previous_round`
- `human_comments_new`
- `human_comments_unresolved`
- `code_changed_since_last_round`
- `ci_status`
- `local_validation_status`

Round counting:

- Increment the round when new review feedback appears after a push or after the
  previous round was recorded.
- Track Copilot and human rounds separately when possible.
- If history is missing, infer conservatively from PR timeline and state
  confidence as `medium` or `low`.

## Comment Classification

For every human and Copilot comment, classify as:

```text
valid
probably_valid
low_signal
contradictory
already_addressed
harmful
```

Definitions:

- `valid`: real issue after reading code/spec.
- `probably_valid`: likely real, needs small implementation or test change.
- `low_signal`: nitpick, style-only, vague, or not worth another round.
- `contradictory`: conflicts with previous accepted change or current spec.
- `already_addressed`: current code already satisfies it.
- `harmful`: would break behavior, security, tests, or agreed design.

## Copilot Rules

- Read code around each Copilot comment before accepting it.
- Compare with previous review rounds and task note history.
- If `round_number >= 3` and a Copilot comment is `low_signal`,
  `contradictory`, or `already_addressed`, recommend rejecting it with a concise
  explanation.
- Never apply a Copilot suggestion blindly if it reverts intentional code from a
  previous round.
- If Copilot repeats a previously rejected suggestion without new evidence,
  draft a rejection instead of editing code.
- If Copilot finds a real bug, fix it like any other review comment.

## Human Review Rules

- Address unresolved human comments before Copilot comments.
- If a human comment is stale because code moved or changed, reply with the
  evidence and ask whether more is needed.
- If a human comment requires product/QA decision, move task to
  `blocked_human`.

## CI And Validation

- Record remote CI status.
- If local validation was run, include exact commands and results.
- If CI fails and local validation is green, decide whether the failure is
  related, unknown, or external.
- Do not call a PR ready to merge while required checks are failing or pending
  unless the failure is explicitly non-required and unrelated.

## Forbidden

- Do not apply all Copilot comments mechanically.
- Do not dismiss or resolve GitHub threads unless explicitly asked and allowed.
- Do not push, merge, or request re-review unless the user asked.
- **NEVER enable auto-merge** at the end of a review round, even when all
  comments are addressed and CI is green. Do not invoke `gh pr merge ... --auto`,
  do not call `enablePullRequestAutoMerge`, do not hit the
  `PUT /pulls/{number}/automerge` REST endpoint. The end of a review round is
  a transition to `ready_to_merge` (read `READY_TO_MERGE.md`), not a merge.
- **NEVER flip a draft PR to ready-for-review** from inside a review-round
  assessment without an explicit user instruction that names the PR.
- Do not ignore human requested changes.
- Do not rewrite prior task-note history to make the current round look clean.

## Writes

Update or propose the task note:

- `Review rounds`
- `Open comments`
- `Blockers`
- `Next action`
- GitHub reply drafts for rejected/stale comments

External write drafts should be neutral and specific. Do not mention tool
provenance.

## Output Format

```markdown
# Review round assessment

PR: owner/frontend#482
Round: 4

## Human comments
- 1 unresolved, valid, should address.

## Copilot comments
- 2 new comments.
- 1 repeats a previous suggestion already rejected.
- 1 contradicts the round 2 fix.
- Recommendation: do not change code for Copilot comments; reply with rationale.

## CI and validation
- CI: passing.
- Local validation: not run in this assessment.

## Next action
Address the human comment, push one focused commit, then wait for final human approval.
```

For machine consumption, return:

```json
{
  "state": "review_iteration",
  "round_number": 4,
  "human_comments_unresolved": 1,
  "copilot_comments_new": 2,
  "copilot_comments_repeated": 1,
  "copilot_comments_contradict_previous_round": 1,
  "code_changed_since_last_round": true,
  "ci_status": "pass",
  "local_validation_status": "not_run",
  "next_action": "Address human comment; reject low-signal Copilot comments with rationale"
}
```

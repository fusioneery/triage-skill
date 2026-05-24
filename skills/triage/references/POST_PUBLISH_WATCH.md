# Post Publish Watch

Use immediately after this skill opens or updates a PR.

This workflow is operational: launch it as a watcher subagent and keep the
current turn open while the subagent polls the PR. Do not replace it with a
passive recommendation to check later.

## Subagent Contract

Launch exactly one watcher subagent for the PR unless the user explicitly
disabled watching.

Give the subagent:

- PR URL
- repository
- branch
- head SHA
- task key
- task note path
- worktree path
- validation commands already run
- whether one Copilot-fix restart has already happened

The watcher subagent owns polling, Copilot classification, and any valid
Copilot follow-up fix during the watch window. The parent agent must not edit,
commit, push, or rebase the same branch while the watcher subagent is active.

If the environment cannot launch subagents, run this workflow in the parent
agent and say that subagent launch was unavailable.

## Defaults

- `total_watch_window`: 25 minutes
- `poll_interval`: 3 minutes
- `target_pr`: the PR just opened or updated

## Process

1. Record the PR URL, head SHA, and watch start time in the task note.
2. Poll the PR immediately once.
3. Poll every 3 minutes until 25 minutes elapse or a stop condition occurs.
4. On each poll, check:
   - new Copilot review comments
   - new Copilot PR comments
   - new human review comments
   - CI status
   - mergeability
   - latest GitHub Deployment for the head SHA (state and `environment_url`)
5. The first time a Deployment for the current head SHA reaches
   `state=success`, load `FEATURE_STAND_OPEN.md` and run it once. If a later
   push produces a new head SHA inside the same watch window, the trigger
   re-arms for the new SHA. Do not reopen the stand on subsequent polls of
   the same SHA. If the repo never produces a deployment, skip silently.
6. If new Copilot comments appear, load `PR_REVIEW_ROUND.md` and classify them.
7. If Copilot comments are `valid` or `probably_valid`, the watcher subagent
   fixes, validates, commits, pushes, and restarts the watch window once from
   the new head SHA.
8. If Copilot comments are `low_signal`, `already_addressed`, `contradictory`,
   or `harmful`, do not edit code; draft replies but do not post unless asked.
9. If human comments appear, stop and return `review_iteration`.
10. If CI fails, stop and return `blocked_ci` unless the failure is proven
    unrelated.
11. If no comments appear before timeout, return the latest PR state.

## Stop Conditions

- Human review comments appear.
- CI fails and the failure is not proven unrelated.
- Mergeability becomes blocked by conflicts.
- The 25 minute watch window elapses.
- One Copilot-fix restart has already happened and new Copilot comments appear
  again; the watcher subagent returns `review_iteration` with the review-round
  assessment instead of looping indefinitely.

## Task Note Updates

Update the task note after each poll with:

- PR URL
- head SHA
- watch start time
- latest poll time
- poll count
- Copilot comments found, addressed, and rejected
- human comments found
- CI state
- mergeability
- latest deployment state and `environment_url` for the head SHA
- whether the feature stand has already been opened for the head SHA
- current state
- next action

## Output

Report:

- PR URL
- elapsed watch time
- poll count
- Copilot comments found, addressed, and rejected
- human comments found
- CI state
- mergeability
- feature stand URL opened (raw and final, with `customBackendUrl` if applied) and open mechanism
- final task state
- next action

## Forbidden

- The watcher is **read-only**. It does not push, rebase, merge, or post
  comments. Those decisions stay with the parent agent (and the user) after
  the watcher returns.
- **NEVER enable auto-merge** when the watcher returns `clean`, `timeout`, or
  any other terminal state. Do not invoke `gh pr merge ... --auto`, do not
  call `enablePullRequestAutoMerge`, do not hit the
  `PUT /pulls/{number}/automerge` REST endpoint. The watcher signals
  readiness for the *next* human decision, not a license to merge silently.
- **NEVER respond to a `clean` watcher exit** with `gh pr merge`,
  `enablePullRequestAutoMerge`, `markPullRequestReadyForReview`,
  `convertPullRequestToDraft`, or any equivalent. Route the signal to the
  user; let them decide the merge.
- Do not flip a draft PR to ready-for-review from inside the watcher (or
  from the parent on receiving a watcher result) without an explicit user
  instruction that names the PR.
- One watcher per PR. Never run two watchers in parallel on the same branch.

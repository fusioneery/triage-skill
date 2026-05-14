# Continue Task

Use this workflow to resume an existing task from the current worktree, branch, PR,
or task note.

Apply the core invariants from `../SKILL.md`. Use `TASK_NOTE_TEMPLATE.md` for
persisted task notes.

## Goal

Reconstruct what was already done, what remains, and the next executable step.
Then either produce an execution prompt or start implementation if the user
explicitly asked you to continue coding, or if `$triage <LINEAR-ID>` was
invoked without limiting words such as "status", "only assess", "report only",
"no publish", or "do not push".

## Sources To Read

1. Canonical task note from `TASK_NOTE_TEMPLATE.md`.
2. Linear issue and latest comments.
3. Linked Notion spec, if any.
4. Linked Slack threads, if any.
5. GitHub PRs, branches, checks, reviews, and unresolved threads.
6. Local worktree: `git status`, current branch, staged files, recent commits,
   diff against base, `.triage/` artifacts, and test outputs if present.

## Protocol

1. Identify the task key and canonical note path.
2. Read the current task note. If missing, create one using
   `TASK_NOTE_TEMPLATE.md`.
3. Verify live Linear and GitHub state; do not rely on stale note status.
4. Inspect local git status before edits. Preserve unrelated dirty changes.
5. Determine "already done" from merged commits, PR diff, local diff, and tests.
6. Determine "not done" from open blockers, review comments, failing checks,
   missing tests, or unimplemented spec items.
7. Decide whether clarification is needed. If yes, ask only blocking questions.
8. Decide whether implementation is allowed by the user request.
9. If the Linear issue has no existing branch, create or use
   `feature/<LINEAR_TASK_ID>-<short-title-of-task>` based on `origin/staging`
   unless the issue or repo clearly specifies another base.
10. Return an execution prompt or implement the next step directly.
11. If code changes were made, follow the publish tail.

## Scope Control

- Continue the existing task; do not expand scope because a related issue is
  visible.
- If new work is medium/large, recommend a child Linear issue instead of folding
  it into the current PR.
- If multiple repos are involved, name the repo for each next action.
- If the current worktree is not on the right branch, stop and report the safest
  command rather than editing blind.
- For Linear issues with no existing branch, default to
  `feature/<LINEAR_TASK_ID>-<short-title-of-task>`. Preserve the task ID exactly
  and shorten the title into lowercase kebab case. Base the branch on
  `origin/staging` unless the issue or repo clearly says otherwise.
- PR titles should be `<TASK-KEY>: <Task title>` — task key first, colon,
  then the verbatim Linear title. No Conventional Commit prefix.

## Forbidden

- Do not discard or revert unrelated local changes.
- Do not force-push unless explicitly asked or a guarded force-with-lease retry
  is required by the owning workflow after a rejected normal push.
- Do not create a new implementation direction when a task note or review thread
  already defines the next action.
- Do not mark a Copilot-only round as mandatory without classifying it.
- Do not change code if the task is `needs_context`, `blocked_human`, or
  `ready_to_merge` unless the requested action is specifically about that state.

## Writes

Update or propose the task note:

- state
- confidence
- last_checked_at
- summary
- already done
- not done
- blockers
- next action
- execution prompt
- PR URL and QA test plan, if code was published
- code review findings (blocking + non-blocking) from `requesting-code-review`,
  if it was run

## Publish Tail

If code changes were made:

1. Run required validation (typecheck, build, lint, circular, relevant tests).
   Treat this as a gate, not as a deliverable — its outcome decides whether to
   proceed, but the commands themselves do not appear in the user-facing
   report or PR description.
2. If validation passes, run the `requesting-code-review` skill on the staged
   diff. Prefer dispatching it as a separate subagent; run inline only if the
   environment cannot launch subagents. The parent agent must not edit, commit,
   push, or rebase the branch while the review subagent is active. Skip this
   step only if the user said "no review", "skip review", or equivalent.
3. Address blocking findings surfaced by the review before push. Record
   non-blocking findings as follow-ups in the task note; do not silently drop
   them.
4. Generate a QA test plan from the implemented diff with three sections:
   - **Steps to reproduce** — concrete user actions to exercise the change.
   - **Cases to check** — edge cases, error paths, role/permission variants,
     empty/large inputs, and any contract boundaries the diff crosses.
   - **Other places potentially involved** — adjacent code paths, feature
     flags, shared utilities, or callers that might be affected. Include this
     only when the diff actually touches shared modules; omit otherwise rather
     than padding with speculation.
5. Commit with a neutral Conventional Commit message. Prefer
   `<type>(<TASK-KEY>): <summary>` when a task key exists.
6. Push the feature branch.
7. Check whether a PR already exists for the branch.
8. If yes, update it. If no, open a ready PR against `staging` unless the repo
   or issue clearly specifies another base. Do not open the PR as draft.
9. Ensure the PR title follows `<TASK-KEY>: <Task title>` and the PR
   description is neutral. Include the QA test plan from step 4 in the PR
   description.
10. Add the `triage` label/tag to the PR.
11. Do not mention Codex, Claude Code, AI, automated assistance, or other
    execution environments in commit messages, PR titles, or PR descriptions.
12. Update the task note with the PR URL and the QA test plan.
13. Launch a watcher subagent using `POST_PUBLISH_WATCH.md` unless the user
    says "no watch", "return immediately", "status only", or "report only".

If validation fails, do not commit or push unless the failure is known unrelated
and the task state should become `blocked_human` or `blocked_ci`.

## Output Format

```markdown
# Continue plan for ENG-1234

## Already done
- Frontend UI validates discount code.
- API endpoint accepts coupon metadata.

## Not done
- Expired coupon behavior is not covered.
- Missing integration test for stacked coupons.

## Need clarification
- Should expired coupon return 400 or be ignored?

## Execution prompt
Work in the current worktree. Add API integration test for stacked coupons.
Do not touch frontend unless tests reveal a contract mismatch.
```

If implementation was published, finish with:

```markdown
## QA test plan

### Steps to reproduce
1. Sign in as a Pro-tier account.
2. Add an expired coupon (`EXPIRED10`) at checkout.
3. Submit the order.

### Cases to check
- Expired coupon → 400 with the new error code; checkout total unchanged.
- Coupon expiring mid-session (server clock crosses expiry while cart open).
- Stacked coupons where one is expired and one is valid.
- Free / Pro / Enterprise role variants.
- Empty cart and max-items cart.

### Other places potentially involved
- `discount/applyCoupon` is also called from the renewal flow — verify it
  still surfaces the legacy error message there.
- `feature.expiredCouponHardFail` flag gates the new behavior; confirm OFF
  state preserves old soft-ignore path.

## Code review findings
- Blocking: none.
- Non-blocking: extract `isCouponExpired` to shared utility (follow-up).
```

Machine output:

```json
{
  "state": "in_progress",
  "implemented": true,
  "changed_files": ["<path>"],
  "qa_test_plan": {
    "steps_to_reproduce": ["<step>"],
    "cases_to_check": ["<case>"],
    "adjacent_areas": ["<area or null>"]
  },
  "code_review": {
    "ran": true,
    "blocking": ["<finding or empty>"],
    "non_blocking": ["<finding or empty>"]
  },
  "next_action": "<remaining step>"
}
```

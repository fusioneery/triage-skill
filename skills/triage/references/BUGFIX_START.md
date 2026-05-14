# Bugfix Start

Use this workflow for a new bug or regression where the next step is to establish
repro, expected behavior, target repo, and the first failing test before coding.

Apply the core invariants from `../SKILL.md`. Use `TASK_NOTE_TEMPLATE.md` for
persisted task notes.

## Goal

Turn a bug report into a scoped, evidence-backed implementation start without
guessing or adding orchestration overhead.

## Sources To Read

- Linear issue title, description, priority, labels, links, attachments, and
  latest comments.
- Slack QA/dev report, screenshots, videos, logs, timestamps, and reproduction
  details.
- Notion specs, product docs, or previous decisions for expected behavior.
- Related GitHub issues, PRs, releases, deploys, and recent commits.
- Local repos that likely own the affected surface.
- Existing task note, if the task key already exists.

## Reproduction Checklist

Collect:

- Environment: prod, staging, local, browser/device, app version, feature flag.
- Actor: role, team, account, permissions, tenant, data shape.
- Steps to reproduce.
- Expected behavior.
- Actual behavior.
- Frequency and first-seen time.
- Error IDs, logs, screenshots, network responses, or trace links.
- Regression window or suspected PR/deploy, if known.

## Clarification Questions

Ask up to five high-signal questions only when reproduction or expected behavior
is incomplete. Prioritize questions that unblock a failing test:

1. Exact repro steps.
2. Expected behavior.
3. Environment/account.
4. Scope boundary.
5. Severity or rollout impact.

If the bug is urgent and enough evidence exists, state assumptions and proceed
with a minimal repro.

## Worktree Protocol

If the user asked for implementation:

1. Check local git status first.
2. Identify the likely repo(s).
3. If creating a worktree manually, refresh the repo default integration branch
   first:
   - `git checkout <base>`
   - `git pull --ff-only origin <base>`
   Usually `<base>` is `staging`, `main`, or `master`, based on repo convention.
4. Create a fresh branch/worktree from the repo default integration branch
   (`staging`, `main`, or `master`, based on repo convention).
5. If a triage tool already prepared a detached worktree from `origin/<base>`,
   do not switch branches inside it; continue in the returned worktree.
6. Do not overwrite unrelated local changes.

If this is assessment-only, do not create a worktree; return the exact command
you would run.

## Test-First Protocol

Before changing product code:

1. Identify the smallest failing test that would prove the bug.
2. Prefer an existing test harness near the affected code.
3. If no harness exists, describe the manual repro and add the smallest
   automated test that fits project patterns.
4. Run the failing test when feasible.
5. Only then change code.

## Forbidden

- Do not code from a vague Slack report if expected behavior is unknown.
- Do not create broad refactors while starting a bugfix.
- Do not touch multiple repos unless evidence shows a cross-repo contract issue.
- Do not skip the failing-test proposal.
- Do not push, post, or create external issues unless explicitly asked.

## Task Note

Create or update `.codex/tasks/<TASK-KEY>.md`:

- state: `new`, `needs_context`, or `in_progress`
- confidence
- last_checked_at
- active_worktree and active_branch, if known
- repro facts
- expected behavior source
- likely repos
- first failing test
- open questions
- next action

## Output Format

```markdown
# Bugfix start for ENG-1234

## Repro
- Environment: staging.
- Steps: apply expired coupon during checkout.
- Actual: checkout total keeps discount.
- Expected: unclear.

## Likely owner
- API owns coupon validation.
- Frontend only displays API response.

## Questions
1. Should expired coupons return 400 or be silently ignored?

## Minimal failing test
Add API integration test for expired coupon validation in the checkout coupon
test suite.

## Next action
Ask product/QA the expired-coupon behavior before implementation.
```

Machine output:

```json
{
  "state": "needs_context",
  "likely_repos": ["owner/api"],
  "questions": ["Should expired coupons return 400 or be silently ignored?"],
  "minimal_failing_test": "API integration test for expired coupon validation",
  "next_action": "Ask product/QA for expected expired-coupon behavior"
}
```

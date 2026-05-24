---
name: triage
description: Use this skill when the user wants to triage engineering work across Linear, GitHub, Slack, Notion, and local worktrees; assess the real status of tasks and PRs; handle Copilot or human review rounds; continue an active task; decide merge readiness; start a bugfix; prepare human handoff for CI/CD, QA, approval, or external blockers; or steer ongoing work in a task/worktree.
---

# Triage

This skill helps triage engineering tasks using connected tools for Linear,
GitHub, Slack, Notion, and local repositories.

## Core rule

Before coding, always determine the real task state.

Linear is the canonical task source when a Linear issue exists. GitHub PRs
represent implementation and review state. Slack represents QA/dev signal.
Notion represents product/spec context. Local worktree and task notes represent
execution state.

## Task states

Every task must be classified into exactly one state:

- `new`
- `needs_context`
- `in_progress`
- `ready_for_review`
- `review_iteration`
- `ready_to_merge`
- `blocked_human`
- `blocked_ci`
- `blocked_merge_conflict`
- `done`
- `ignore`

## Routing

Load only the workflow reference needed for the user's request:

- For a full overview of active work: read `references/TRIAGE_INBOX.md`.
- For one task status assessment: read `references/TASK_STATUS_ASSESSOR.md`.
- For GitHub PR review comments, Copilot loops, or human review rounds: read
  `references/PR_REVIEW_ROUND.md`.
- For continuing implementation in an existing task/worktree: read
  `references/CONTINUE_TASK.md`.
- For deciding whether a task is ready to merge: read
  `references/READY_TO_MERGE.md`.
- For injecting new user instructions into active work: read
  `references/STEER_ACTIVE_TASK.md`.
- For starting a bugfix from fresh context: read `references/BUGFIX_START.md`.
- For CI/CD, approval, QA, or external blockers: read
  `references/HUMAN_BLOCKED.md`.
- After automatically opening or updating a PR from `$triage <LINEAR-ID>`:
  launch a watcher subagent using `references/POST_PUBLISH_WATCH.md` unless the
  user says "no watch", "return immediately", "status only", or "report only".
- For opening the deployed feature stand once the PR's deploy succeeds (with
  optional `customBackendUrl` from a paired BE PR): read
  `references/FEATURE_STAND_OPEN.md`. Loaded automatically from the watcher
  on first deploy success; can also be loaded standalone when the user asks
  to open the feature stand for a task that already has a deployed PR.
- For task note format and update rules: read
  `references/TASK_NOTE_TEMPLATE.md`.

If multiple references match, load the smallest set that covers the request.
For example, a PR with Copilot comments usually needs `PR_REVIEW_ROUND.md`; a
task that is green but missing approval usually needs `READY_TO_MERGE.md`, and
may need `HUMAN_BLOCKED.md` only if a human-facing handoff is required.

## Persistent task note

For every non-trivial task, maintain a task note.

Preferred path:

```text
.codex/tasks/<LINEAR-ID>.md
```

If no Linear issue exists:

```text
.codex/tasks/<source>-<id>.md
```

Use `references/TASK_NOTE_TEMPLATE.md`.

## Invariants

- **NEVER enable auto-merge on any PR.** Do not invoke `gh pr merge ... --auto`, do not call the `enablePullRequestAutoMerge` GraphQL mutation, do not hit the `PUT /repos/{owner}/{repo}/pulls/{number}/automerge` REST endpoint, and do not ask a downstream agent or workflow to do it. Auto-merge fires silently the moment required checks go green, which bypasses the explicit human-in-the-loop confirmation this skill is built around. Triage opens PRs; humans merge them.
- **NEVER merge a PR opened during a triage run**, even if the same agent opened it. Triage's job ends at "PR opened, body written, label applied, watcher dispatched". The merge is a separate, human-authored step.
- **NEVER flip a draft PR to ready-for-review without an explicit user confirmation that names the PR.** `gh pr ready` / `markPullRequestReadyForReview` triggers required-reviewer auto-assignment and CI side-effects; it is a write, not a status read.
- Do not assume a task is complete because a PR exists.
- Do not assume a PR is ready because CI is green.
- Do not blindly apply Copilot comments.
- Human review comments outrank Copilot comments.
- QA feedback can reopen a task even if the PR is green.
- If CI fails locally and remotely because of our code, fix the code.
- If CI fails remotely but local validation passes and the failure is unrelated,
  stop and prepare human handoff.
- Before manually creating a task branch or worktree from an integration branch,
  refresh that integration branch with `git checkout <base>` and
  `git pull --ff-only origin <base>`; usually `<base>` is `staging`, `main`, or
  `master`.
- If merge conflicts exist, recommend rebase from freshly updated staging/main.
- If approvals are missing, prepare a QA/test plan and Slack ping.
- Never start coding before determining task state.
- Always update the task note after assessment.
- When the user invokes `$triage <LINEAR-ID>` without limiting words like
  "status", "only assess", "report only", "no publish", or "do not push", treat
  it as permission to execute the next safe task step end-to-end after task
  state is determined.
- If implementation changes are made under that permission and all required
  validation passes, run the `requesting-code-review` skill once on the staged
  diff before pushing, then commit, push, and open or update a ready PR
  automatically. Address any blocking findings the review surfaces before push;
  non-blocking findings can be folded in or recorded in the task note as
  follow-ups. Prefer running the review as a separate subagent when the
  environment supports it; otherwise run inline. Skip if the user said
  "no review", "skip review", or equivalent.
  Add the `triage` label/tag to the PR. Do not open it as draft. Do not merge.
- If the task description names multiple environments, domains, or repos (e.g.
  separate FE/BE/infra repos, or paired changes across services), opening one
  PR is not sufficient. Publish all required related PRs before starting the
  post-publish watcher. Treat the publish step as complete only when every
  required PR is open and linked in the task note.
- Local validation (typecheck/build/lint/circular/tests) is a precondition for
  push, not a deliverable. Do not surface the list of validation commands in
  the user-facing report or PR description. Instead, generate a QA test plan
  with three sections: **Steps to reproduce**, **Cases to check**, and
  **Other places potentially involved** (the last is optional — include only
  when the diff touches shared modules, contracts, or feature flags). Put the
  QA test plan in both the PR description and the user-visible final report.
- After automatically opening or updating a PR from `$triage <LINEAR-ID>`,
  launch a watcher subagent using `references/POST_PUBLISH_WATCH.md` unless the
  user says "no watch", "return immediately", "status only", or "report only".
  The parent agent must not continue code edits in the same branch while the
  watcher subagent is active.
- Commit messages must follow Conventional Commits. Prefer
  `<type>(<LINEAR-ID>): <summary>` when a Linear issue exists.
- New task branches must use
  `feature/<LINEAR_TASK_ID>-<short-title-of-task>`, with the task ID preserved
  and the title shortened into lowercase kebab case.
- PR titles must use `<TASK-KEY>: <Task title>`, e.g.
  `ENG-1234: Reject expired coupons in checkout`. Take the title verbatim from
  the Linear issue (or task source) — do not kebab-case it. Do not prepend a
  Conventional Commit type prefix such as `feat:` or `fix:`; the task key alone
  is the prefix.
- Do not post to Linear, GitHub, Slack, or Notion unless the user explicitly
  asks for posting. Draft exact text otherwise. Opening or updating a ready PR
  under the publish rule above is allowed; posting comments/status updates is
  not.
- Keep external write drafts neutral. Do not mention tool provenance, Codex,
  Claude Code, AI, automated assistance, or other execution environments in
  commit messages, PR titles, PR descriptions, Slack text, Linear text, or
  GitHub comments.
- When introducing a new ENV variable in code (or renaming an existing one),
  register it in Secrets Manager for **every** environment before merging:
  ephemeral, staging, Prod EU, and Prod US. Missing it in any environment causes
  silent failures on deploy. Surface this as a follow-up checklist item in the
  task note and PR description; do not rely on memory.
- Do not run the manual base-refresh checkout inside a worktree that was already
  prepared by triage tooling; those worktrees are intentionally detached from
  `origin/<base>`.
- When a task has a linked infra/schema/database PR, verify the merged infra PR
  diff or the schema source of truth before finalizing app-side column names,
  generated DB types, codegen assumptions, or migration compatibility. Do not
  rely only on the app branch submodule pointer, generated types, local checkout
  state, or review comments; those may be stale or rebased incorrectly. Treat
  the merged infra PR diff and schema files as authoritative, then update app
  code, DB types/codegen outputs, tests, and submodule pointer to match.

## Source priority

Use this priority when sources disagree:

1. Current GitHub PR/check/review/mergeability state for implementation facts.
2. Current local git worktree for unpushed or in-progress implementation facts.
3. Linear for canonical task identity, owner, priority, and workflow state.
4. Slack for QA signal and human decisions.
5. Notion for expected behavior and background.
6. Existing task note for history and previous decisions, unless live sources
   contradict it.

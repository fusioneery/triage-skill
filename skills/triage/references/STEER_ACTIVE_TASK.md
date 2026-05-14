# Steer Active Task

Use this workflow when the user adds new direction to an active task, such as
"also check mobile" or "do not touch frontend".

Apply the core invariants from `../SKILL.md`. Use `TASK_NOTE_TEMPLATE.md` for
persisted task notes.

## Goal

Preserve the current task context while recording the new steering instruction,
deciding whether it changes scope, and updating the next action.

## Sources To Read

- Canonical task note and `Steering log`.
- Current Linear issue and linked PRs.
- Current local worktree status.
- Current review comments and blockers.
- Slack/Notion context only if the steering instruction refers to product, QA,
  design, or external decisions.

## Protocol

1. Do not discard the current plan.
2. Read the current task note.
3. Record the user's exact steering instruction under `Steering log`.
4. Determine whether the instruction is:
   - `constraint`: changes what must not be done.
   - `small_investigation`: can be checked before the next push.
   - `scope_addition`: adds implementation work to the same PR.
   - `separate_task`: should become a child Linear issue.
   - `conflict`: contradicts current implementation, review, or spec.
5. Decide whether current canonical state changes.
6. Update next actions and execution prompt.
7. If the instruction creates a blocker, move state to `needs_context` or
   `blocked_human`.

## Scope Rules

- Small investigation: integrate into the next action.
- Small implementation in same touched surface: can stay in the task if it does
  not invalidate review scope.
- Medium/large implementation, new repo, or separate product behavior:
  recommend child Linear issue.
- Contradiction with human review/spec: stop and ask a focused question.
- Contradiction with Copilot only: classify the Copilot comment; do not let it
  override user steering by default.

## Forbidden

- Do not restart the task from scratch.
- Do not erase prior blockers or review-round history.
- Do not silently expand PR scope.
- Do not create a child issue or post comments unless explicitly asked.
- Do not keep coding if the steering instruction makes expected behavior
  ambiguous.

## Task Note Update

Append:

```markdown
## Steering log

### <YYYY-MM-DD>
User added: <exact user instruction>
Assessment: <constraint|small_investigation|scope_addition|separate_task|conflict> - <why>
Updated next action: <new first action>
```

Then update:

- `state`, if changed.
- `Summary`.
- `Blockers`.
- `Next action`.
- `External write drafts`, if a child issue or clarification is needed.

## Output Format

```markdown
# Steering update for ENG-1234

## Added instruction
Check whether mobile checkout uses the same coupon contract.

## Assessment
small_investigation - this should be checked before the final API push, but it
does not change scope unless mobile uses a different contract.

## Updated next action
Before pushing the API fix, inspect mobile checkout coupon usage and mention the
result in the PR summary.

## Execution prompt
Continue the current task. Before final validation, inspect mobile checkout
coupon usage. Do not change mobile code unless it already depends on the API
contract being changed.
```

Machine output:

```json
{
  "state": "in_progress",
  "steering_type": "small_investigation",
  "scope_changed": false,
  "requires_child_task": false,
  "next_action": "Inspect mobile checkout coupon usage before final API push"
}
```

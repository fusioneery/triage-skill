# Task Note Template

Use one small persisted artifact for every non-trivial task so any agent can
resume without replaying the whole conversation.

Preferred path when a Linear issue exists:

```text
.codex/tasks/<LINEAR-ID>.md
```

Fallback path when no Linear issue exists:

```text
.codex/tasks/<source>-<id>.md
```

## Template

```markdown
# <TASK-ID> — <title>

## State
state:
confidence:
last_checked_at:
active_worktree:
active_branch:

## Sources
linear:
github_prs:
slack_threads:
notion_pages:

## Related PRs
<!-- One line per repo. Role is one of: frontend, backend, mobile, infra, other.
     FEATURE_STAND_OPEN.md uses the `backend` role to wire customBackendUrl. -->
- frontend: <owner>/<repo>#<pr>
- backend:  <owner>/<repo>#<pr>

## Deployments
<!-- Latest known deployment per repo for this task. Updated by the watcher. -->
- frontend stand: <url>
- backend stand:  <url>

## Summary
...

## Review rounds
| Round | Date | Trigger | Copilot | Human | Result |
|---|---|---|---:|---:|---|
| 1 | ... | initial PR | 3 | 1 | addressed |
| 2 | ... | after push | 2 | 0 | 1 rejected |
| 3 | ... | after push | 4 | 0 | low-signal, rejected |

## Open comments
...

## Local validation
typecheck:
tests:
build:
last_command_output_summary:

## Blockers
...

## Steering log
...

## Next action
...
```

## Update Rules

- Replace `state`, `confidence`, `last_checked_at`, `active_worktree`, and
  `active_branch` with the current assessment values.
- Keep `Sources` current with Linear, PR, Slack, and Notion links.
- Add a `Related PRs` line as soon as a sibling PR (BE pair, mobile, infra)
  is opened or discovered. The `backend` role is what `FEATURE_STAND_OPEN.md`
  reads to apply `customBackendUrl`; without it, the stand opens without the
  param and falls back to a search heuristic.
- Update `Deployments` from the watcher whenever a new deployment URL is
  resolved for either repo.
- Keep `Summary` short and evidence-backed.
- Update `Review rounds` after every review assessment.
- Update `Open comments` with unresolved human and Copilot comments.
- Update `Local validation` only from commands actually run or verified.
- Remove a blocker only when current evidence proves it is gone.
- Append steering entries instead of overwriting history.
- Keep `Next action` concrete and executable. It should be the next thing a
  human or agent should do.

## Output Contract

When creating or updating a note, report:

```json
{
  "task_key": "<TASK-ID>",
  "note_path": ".codex/tasks/<TASK-ID>.md",
  "state": "<canonical state>",
  "confidence": "low|medium|high",
  "note_updated": true,
  "next_action": "<single next action>"
}
```


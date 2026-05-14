# Human Blocked

Use this workflow when the task cannot make safe progress because a person,
external team, approval, QA step, product answer, or unrelated infrastructure
problem is required.

Apply the core invariants from `../SKILL.md`. Use `TASK_NOTE_TEMPLATE.md` for
persisted task notes.

## Goal

Stop coding, summarize evidence, draft the exact human-facing messages, and
mark the task as blocked.

## Sources To Read

- Canonical task note and latest blockers.
- GitHub PR checks, review threads, approvals, mergeability, and latest commit.
- Local validation evidence, if already run.
- Linear status, comments, owner, labels, and linked specs.
- Slack thread where the blocker was reported or should be escalated.
- Notion/spec only if expected behavior is the blocker.

## Blocker Types

Use `blocked_human` when waiting for:

- Product behavior decision.
- QA repro, confirmation, or signoff.
- Required reviewer approval.
- Infra/release owner for unrelated deploy/check failure.
- Access, credential, feature flag, data fixture, or environment owner.
- Human judgment on contradictory review feedback.

Use `blocked_ci` when:

- Required checks fail and the cause is unknown.
- Required checks fail due to the PR's changed files.
- Checks are pending/stuck and no external owner has been identified yet.

If CI failure is proven unrelated and requires infra/release action, canonical
state becomes `blocked_human`.

## Evidence Requirements

Before declaring a human blocker, collect enough evidence to avoid dumping an
unclear problem on someone else:

- What is blocked.
- What was tried.
- Exact failing check, review thread, missing approval, or unknown decision.
- Why the agent should stop coding.
- Who or which channel is the likely owner.
- Exact next answer/action needed.

## Forbidden

- Do not keep coding around a missing product/QA decision.
- Do not retry random fixes for unrelated CI/CD failures.
- Do not post vague "blocked" messages.
- Do not blame another owner; state observable facts.
- Do not mark done or ready to merge while the blocker exists.
- Do not post to Slack, Linear, or GitHub unless explicitly asked.

## Writes

Update or propose the task note:

- `state: blocked_human` or `state: blocked_ci`
- `confidence`
- `last_checked_at`
- `Blockers`
- `Summary`
- `Next action`
- exact Slack/Linear/GitHub drafts

## Slack Message Template

```text
Hey, <task/PR> is blocked on <specific blocker>. Evidence:
- <fact 1>
- <fact 2>

Can you <specific requested action>?
```

Keep it short. Include PR/check links when available.

## Linear Comment Template

```text
Blocked on <specific blocker>.

Evidence:
- <fact 1>
- <fact 2>

Next action needed: <owner/action>.
```

## Output Format

```markdown
# Human blocker

Reason: CI/CD failure unrelated to this PR.
Canonical state: blocked_human

## Evidence
- Local typecheck passes.
- Local affected tests pass.
- Failed GitHub check is deploy preview infra timeout.
- No changed files touch deploy config.

## Slack message
Hey, owner/frontend#482 is blocked by a deploy preview infra timeout. Local validation passes and the diff does not touch deploy config. Can someone from infra/release check the preview job?

## Linear comment
Blocked on deploy preview infra timeout. Local typecheck and affected tests pass, and this diff does not touch deploy config. Next action needed: infra/release check the preview job.

## Next action
Wait for infra/release response; do not make code changes for this blocker.
```

Machine output:

```json
{
  "state": "blocked_human",
  "reason": "Deploy preview infra timeout unrelated to changed files",
  "evidence": ["Local typecheck passes", "Affected tests pass"],
  "next_action": "Ask infra/release to inspect preview job",
  "requires_human": true
}
```

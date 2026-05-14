# triage-skill

A Claude Code / agent skill for triaging engineering work across Linear, GitHub,
Slack, Notion, and local worktrees. It assesses real task status, decides merge
readiness, handles Copilot or human review rounds, starts bugfixes, prepares
human handoffs (CI/CD, QA, approvals, external blockers), and steers ongoing
work in active tasks/worktrees.

Maintained by Acme Inc.

## What's inside

```text
skills/triage/
├── SKILL.md
└── references/
    ├── BUGFIX_START.md
    ├── CONTINUE_TASK.md
    ├── FEATURE_STAND_OPEN.md
    ├── HUMAN_BLOCKED.md
    ├── POST_PUBLISH_WATCH.md
    ├── PR_REVIEW_ROUND.md
    ├── READY_TO_MERGE.md
    ├── STEER_ACTIVE_TASK.md
    ├── TASK_NOTE_TEMPLATE.md
    ├── TASK_STATUS_ASSESSOR.md
    └── TRIAGE_INBOX.md
```

`SKILL.md` is the entry point. The agent loads only the reference files it
needs for the user's current request.

## Install

Requires Node.js 18+.

This repo follows the [`skill`](https://www.npmjs.com/package/skill) CLI layout
(`skills/<name>/SKILL.md`). Point `SKILL_BASE_URL` at this repo, then install:

```bash
SKILL_BASE_URL=https://github.com/fusioneery/triage-skill/tree/main \
  npx skill skills/triage
```

That writes the skill into `.codebuddy/skills/triage/` in the current project.

### Install into Claude Code

Claude Code reads skills from `~/.claude/skills/`. To install there directly:

```bash
git clone https://github.com/fusioneery/triage-skill.git
mkdir -p ~/.claude/skills
cp -r triage-skill/skills/triage ~/.claude/skills/triage
```

Or for a single project:

```bash
mkdir -p .claude/skills
cp -r triage-skill/skills/triage .claude/skills/triage
```

After install, the skill is discoverable by name `triage` and activates when
you ask the agent to triage tasks, assess PR status, continue an active task,
handle review rounds, decide merge readiness, etc.

## Usage

Trigger phrases that activate the skill include:

- "triage my inbox"
- "what's the status of PLA-1234"
- "continue the active task"
- "is this PR ready to merge"
- "start a bugfix for <issue>"
- "handle the Copilot review round"
- "open the feature stand for <task>"

The skill expects the following tools/integrations to be available to the
agent: GitHub (`gh` CLI or MCP), Linear (MCP), Slack (MCP), Notion (MCP), and a
local git worktree.

## Conventions enforced by the skill

- Task notes live at `.codex/tasks/<TASK-KEY>.md`.
- Branches: `feature/<TASK-KEY>-<short-kebab-title>`.
- PR titles: `<TASK-KEY>: <verbatim task title>` — no Conventional Commit
  prefix on PR titles.
- Commit messages follow Conventional Commits, preferring
  `<type>(<TASK-KEY>): <summary>` when a task ID exists.
- External writes (Linear/GitHub/Slack/Notion) are drafted, not posted,
  unless the user explicitly asks for posting. Opening or updating a ready PR
  under the `$triage <TASK-KEY>` permission is allowed.
- New env variables must be registered across all environments (ephemeral,
  staging, prod-eu, prod-us, …) before merge.

See `skills/triage/SKILL.md` for the full set of invariants and source
priority order.

## License

MIT — see [LICENSE](./LICENSE).

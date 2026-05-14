# Feature Stand Open

Use after a push triggers a deploy and the corresponding deployment for the
PR's head SHA reaches `state=success`. Loaded from `POST_PUBLISH_WATCH.md` the
first time deploy success is observed in a watch iteration. Can also be loaded
standalone when the user asks to open the feature stand for a task that
already has a deployed PR.

Run this once per watch window. Do not reopen the stand on subsequent polls.

## Inputs

- `pr_url`, `repo` (`<owner>/<name>`), `head_sha` for the FE PR just pushed.
- `task_id` (Linear ID or `<source>-<id>`) and task note path.

## 1. Resolve the FE feature-stand URL

The deployment is the source of truth (the rocket-icon "deployed to <env>"
line on the PR). Workflow run output is a fallback.

```bash
# Find the latest deployment for this head SHA.
gh api "repos/<owner>/<name>/deployments?sha=<head_sha>&per_page=10" \
  --jq '.[0]' > /tmp/dep.json
dep_id=$(jq -r '.id' /tmp/dep.json)

# Latest status for that deployment.
gh api "repos/<owner>/<name>/deployments/${dep_id}/statuses?per_page=10" \
  --jq '.[0] | {state, environment_url, log_url, target_url}'
```

- If `state` is `pending`/`in_progress`/`queued`, this iteration is too early —
  return to the watch loop and try again next poll. Do not block.
- If `state` is `failure`/`error`, do not open a stand. Return to the watch
  loop; CI handling in `POST_PUBLISH_WATCH.md` step 9 owns the failure.
- If `state` is `success`, take `environment_url` as the FE stand URL. If it
  is empty, fall back to `target_url`. If both are empty, the deploy workflow
  did not publish a URL; record this in the task note and stop — there is
  nothing to open.

If no deployment exists for the head SHA at all, the repo does not deploy on
push. Stop silently; do not invent a URL.

## 2. Resolve the related BE pair (optional)

A BE pair only adds the `customBackendUrl` query param. If no BE pair is
found, open the FE stand URL as-is.

Order of resolution:

1. Read the task note at the path provided. Look for an explicit
   `Related PRs` section that lists repo + PR pairs with roles, e.g.:
   ```
   ## Related PRs
   - frontend: <owner>/app#123
   - backend:  <owner>/api#456
   ```
   If a `backend` entry exists, use that PR.
2. If the task note has no usable `Related PRs` section, search across the
   org for sibling PRs by task id:
   ```bash
   gh search prs "<TASK-ID>" in:title --json url,repository,title,number,state
   ```
   Filter to open PRs in repos other than the FE repo. If exactly one
   candidate looks like a backend (repo name matches `/api|backend|be|server/i`
   or the task note hints at it), use it. If multiple candidates remain,
   list them in the report and skip the param — do not guess.

For the chosen BE PR, repeat step 1 (latest deployment for its head SHA) to
get the BE `environment_url`. If the BE deployment is not yet successful,
proceed without the param and note this in the task note; do not block the FE
open on BE deploy.

## 3. Build the open URL

If a BE stand URL was resolved:

```
<fe_stand_url>?customBackendUrl=<urlencoded BE stand url>
```

If the FE URL already has a query string, append with `&` instead of `?`.
Always URL-encode the BE URL value (e.g. via `jq -rn --arg u "$be" '$u|@uri'`).

## 4. Detect environment and open

Probe whether cmux is reachable:

```bash
cmux ping >/dev/null 2>&1 && echo cmux || echo other
```

- `cmux` reachable: open the URL in a browser pane in the relevant workspace.
  Look up the workspace ref by triage session name (see
  `src/run/cmux-launcher.ts:120` for the parsing rule); if not found, omit
  `--workspace` and let cmux open it in the current workspace.
  ```bash
  cmux new-surface --type browser --url "<open_url>" \
    [--workspace workspace:<n>] --focus true
  ```
- Not reachable (Codex native app, plain terminal, CI shell, etc.): emit a
  text instruction back to the user, exact format:
  ```
  Open the feature stand:
  <open_url>
  ```
  Do not attempt `open <url>`, `xdg-open`, or any other OS handler. Codex
  native receives only text instructions per project policy.

## 5. Update the task note

Append under the post-publish section:

- FE stand URL (raw, without query params)
- BE stand URL if resolved, or the reason it was skipped (no pair found,
  multiple candidates, BE deploy not yet successful)
- Final opened URL (with `customBackendUrl` if applied)
- Open mechanism used: `cmux` or `text-prompt`
- Timestamp

## Output

Add to the watcher's report:

- FE stand URL
- BE stand URL (if any) and whether `customBackendUrl` was applied
- Open mechanism used

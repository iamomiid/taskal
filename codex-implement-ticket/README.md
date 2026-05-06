# Codex Implement Ticket

Pick the next eligible issue, implement it with Codex, and open or update a pull request.

This composite GitHub Action is designed for repositories running on a self-hosted runner with Codex already installed and authenticated.

It includes a usage guard that checks remaining Codex usage before starting work and defers the run when usage is too low.

## What It Does

When triggered, the action:

1. Ensures the required automation labels exist
2. Selects the next eligible open issue labeled `codex:implement`
3. Prevents parallel work if another issue is already active
4. Marks the issue as running
5. Checks remaining Codex usage against a configured threshold
6. Reads optional model and reasoning settings from issue labels
7. Resets the workspace, creates `codex/issue-<number>`, and runs Codex
8. Verifies that a PR exists for the branch
9. Updates labels and comments on the issue with the result

## Requirements

- Self-hosted runner
- `codex` CLI installed and authenticated
- `gh`, `jq`, `curl`, and `git` installed
- Repository permissions for:
  - `contents: write`
  - `issues: write`
  - `pull-requests: write`

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `github-token` | Yes | - | GitHub token used by `gh` and checkout |
| `repository` | Yes | - | Repository in `owner/name` format |
| `event-name` | Yes | - | Triggering GitHub event name |
| `event-issue-number` | No | `""` | Issue number from the triggering issue event, if any |
| `base-branch` | No | `master` | Base branch for implementation PRs |
| `usage-threshold` | No | `"20"` | Minimum remaining usage percentage required for Codex to run |

## Example

```yaml
name: Implement Tickets

on:
  issues:
    types: [opened, labeled]
  schedule:
    - cron: "*/30 * * * *"
  workflow_dispatch:

jobs:
  implement-next:
    runs-on: self-hosted
    permissions:
      contents: write
      issues: write
      pull-requests: write
    steps:
      - uses: iamomiid/taskal/codex-implement-ticket@main
        with:
          github-token: ${{ github.token }}
          repository: ${{ github.repository }}
          event-name: ${{ github.event_name }}
          event-issue-number: ${{ github.event.issue.number }}
          base-branch: master
          usage-threshold: "20"
```

## Labels

This action automatically creates and manages:

- `codex:implement`
- `codex:queued`
- `codex:running`
- `codex:pr-open`
- `codex:needs-human`
- `codex:error`

It also reads optional execution settings from labels on the issue:

- `codex:model:<model-name>`
- `codex:reasoning:<effort>`

Defaults:

- model: `gpt-5.4`
- reasoning: `medium`

## Behavior Notes

- If no eligible issue exists, the run exits cleanly.
- If another issue is already labeled `codex:running` or `codex:pr-open`, the run exits to avoid overlapping work.
- If remaining usage is below the configured threshold, the issue is re-queued and retried on the next scheduled run.
- The action hard-resets and cleans the repository before Codex starts work in order to guarantee a clean branch state.
- If Codex does not push the branch, the issue is marked `codex:needs-human` and no PR is created.

## License

MIT

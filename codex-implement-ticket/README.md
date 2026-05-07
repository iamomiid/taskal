# Codex Implement Ticket

Pick the next eligible issue, implement it with Codex, and open or update a pull request.

This composite GitHub Action is designed for repositories running on a self-hosted runner with Codex already installed and authenticated.

It includes a usage guard that checks remaining Codex usage before starting work and defers the run when usage is too low.

## What It Does

When triggered, the action:

1. Ensures the required automation labels exist
2. Selects the next eligible open issue labeled `codex:implement`
3. Limits active work based on the configured cap for running issues and open implementation PRs
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

## GitHub Settings Required for PR Creation

To allow this action to open pull requests, configure the repository in GitHub under:

`Settings -> Actions -> General -> Workflow permissions`

Required settings:

- `Read and write permissions`
- `Allow GitHub Actions to create and approve pull requests`

If these are not enabled, the branch may be pushed successfully while PR creation still fails.

## Inputs

| Name                    | Required | Default  | Description                                                                                             |
| ----------------------- | -------- | -------- | ------------------------------------------------------------------------------------------------------- |
| `github-token`          | No       | `""`     | Optional GitHub token used by `gh` and checkout. If omitted, the action falls back to `github.token`    |
| `elevated-github-token` | No       | `""`     | Optional elevated GitHub token used only as a fallback for branch push and PR create or edit operations |
| `repository`            | Yes      | -        | Repository in `owner/name` format                                                                       |
| `event-name`            | Yes      | -        | Triggering GitHub event name                                                                            |
| `event-issue-number`    | No       | `""`     | Issue number from the triggering issue event, if any                                                    |
| `base-branch`           | No       | `master` | Base branch for implementation PRs                                                                      |
| `max-active-issues`     | No       | `"1"`    | Maximum number of issues that may be actively running or waiting in an open PR at the same time         |
| `usage-threshold`       | No       | `"20"`   | Minimum remaining usage percentage required for Codex to run                                            |

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
          repository: ${{ github.repository }}
          event-name: ${{ github.event_name }}
          event-issue-number: ${{ github.event.issue.number }}
          elevated-github-token: ${{ secrets.WORKFLOW_TOKEN }}
          base-branch: master
          max-active-issues: "3"
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
- reasoning effort: `medium`

## Behavior Notes

- If no eligible issue exists, the run exits cleanly.
- The action counts open `codex:running` and `codex:pr-open` issues together and only starts new work when that total is below `max-active-issues`.
- Issues that already have `codex:running` or `codex:pr-open` are not eligible for re-selection, so open implementation PRs stay in review/feedback mode instead of being re-implemented.
- If remaining usage is below the configured threshold, the issue is re-queued and retried on the next scheduled run.
- The action hard-resets and cleans the repository before Codex starts work in order to guarantee a clean branch state.
- If Codex does not push the branch, the issue is marked `codex:needs-human` and no PR is created.
- The action uses `github-token` or `github.token` for labels, comments, and issue state. If workflow-file writes or PR creation need stronger permissions, set `elevated-github-token` so only push and PR mutation operations retry with the elevated token.

## License

MIT

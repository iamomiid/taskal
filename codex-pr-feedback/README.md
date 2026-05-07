# Codex PR Feedback

Apply pull request feedback with Codex and push the updated branch.

This composite GitHub Action is designed for repositories running on a self-hosted runner with Codex already installed and authenticated.

It includes a usage guard that checks remaining Codex usage before running and skips the task when usage is too low.

## What It Does

When triggered from a PR comment, review comment, or submitted review:

1. Adds an `eyes` reaction to acknowledge the request
2. Resolves the pull request number
3. Checks remaining Codex usage against a configured threshold
4. Reads optional model and reasoning settings from the linked issue labels
5. Checks out the PR branch
6. Runs Codex against the supplied feedback
7. Commits and pushes any resulting changes
8. Replies back on the PR thread

If a prior Codex session id is stored in the PR body, the action resumes that session to preserve context.

## Requirements

- Self-hosted runner
- `codex` CLI installed
- `gh`, `jq`, `curl`, and `git` installed
- Repository permissions for:
  - `contents: write`
  - `pull-requests: write`
  - `issues: write`

## Inputs

| Name                    | Required | Default | Description                                                                                          |
| ----------------------- | -------- | ------- | ---------------------------------------------------------------------------------------------------- |
| `github-token`          | No       | `""`    | Optional GitHub token used by `gh` and checkout. If omitted, the action falls back to `github.token` |
| `elevated-github-token` | No       | `""`    | Optional elevated GitHub token used only as a fallback for branch push and PR edit operations        |
| `repository`            | Yes      | -       | Repository in `owner/name` format                                                                    |
| `event-name`            | Yes      | -       | Triggering GitHub event name                                                                         |
| `issue-number`          | No       | `""`    | Issue number for `issue_comment` events                                                              |
| `pull-request-number`   | No       | `""`    | Pull request number for review events                                                                |
| `issue-comment-author`  | No       | `""`    | Author login for `issue_comment` events                                                              |
| `review-author`         | No       | `""`    | Author login for `pull_request_review` events                                                        |
| `review-comment-id`     | No       | `""`    | Review comment id for `pull_request_review_comment` events                                           |
| `issue-comment-id`      | No       | `""`    | Issue comment id for `issue_comment` events                                                          |
| `review-id`             | No       | `""`    | Review id for `pull_request_review` events                                                           |
| `feedback`              | Yes      | -       | The feedback text to apply                                                                           |
| `usage-threshold`       | No       | `"20"`  | Minimum remaining usage percentage required for Codex to run                                         |

## Example

```yaml
name: PR Feedback

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  pull_request_review:
    types: [submitted]

jobs:
  apply-feedback:
    runs-on: self-hosted
    if: |
      (github.event_name == 'issue_comment' && github.event.issue.pull_request != null && contains(github.event.comment.body, '@taskal')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@taskal')) ||
      (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@taskal'))
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: iamomiid/taskal/codex-pr-feedback@main
        with:
          repository: ${{ github.repository }}
          event-name: ${{ github.event_name }}
          elevated-github-token: ${{ secrets.WORKFLOW_TOKEN }}
          issue-number: ${{ github.event.issue.number }}
          pull-request-number: ${{ github.event.pull_request.number }}
          issue-comment-author: ${{ github.event.comment.user.login }}
          review-author: ${{ github.event.review.user.login }}
          review-comment-id: ${{ github.event.comment.id }}
          issue-comment-id: ${{ github.event.comment.id }}
          review-id: ${{ github.event.review.id }}
          feedback: ${{ github.event.comment.body || github.event.review.body }}
```

## Optional Issue Labels

If the pull request closes a linked issue, the action reads these labels from that issue:

- `codex:model:<model-name>`
- `codex:reasoning:<effort>`

Defaults:

- model: `gpt-5.4`
- reasoning effort: `medium`

## Behavior Notes

- If remaining usage is below the configured threshold, the action comments on the PR and exits without running Codex.
- The action stores a hidden `codex-session-id` marker in the PR body so later runs can resume context.
- If there are no file changes after Codex runs, no commit is created.
- The action uses `github-token` or `github.token` for comments and normal PR reads. If workflow-file writes need stronger permissions, set `elevated-github-token` so only push and PR edit operations retry with the elevated token.

## License

MIT

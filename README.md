# Taskal

Open-source GitHub Actions for running Codex-driven repository work on your own self-hosted runner.

This repository currently includes two composite actions:

- [`codex-pr-feedback`](./codex-pr-feedback): apply requested pull request feedback when someone mentions `@taskal`
- [`codex-implement-ticket`](./codex-implement-ticket): pick the next labeled issue, implement it with Codex, and open or update a pull request

These actions are built for teams that want to keep automation inside their own GitHub repository and runner environment.

Both actions include a built-in usage guard that checks remaining Codex usage before starting expensive work.

## What You Need

Both actions expect a self-hosted runner with:

- `codex` CLI installed and authenticated
- `gh` installed and authenticated through the provided GitHub token
- `jq` installed
- `curl` installed
- `git` available

The actions also read Codex auth data from:

- `$CODEX_HOME/auth.json`, or
- `~/.codex/auth.json`

## Actions

### `codex-pr-feedback`

Listens for PR comments or reviews that mention `@taskal`, checks out the PR branch, asks Codex to apply the requested changes, pushes the result, and replies back on the PR.

Docs: [`codex-pr-feedback/README.md`](./codex-pr-feedback/README.md)

### `codex-implement-ticket`

Looks for the next open issue labeled `codex:implement`, asks Codex to implement it, pushes a branch, and ensures a PR exists.

Docs: [`codex-implement-ticket/README.md`](./codex-implement-ticket/README.md)

## Quick Start

Example workflow for PR feedback:

```yaml
name: PR Feedback

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  pull_request_review:
    types: [submitted]

concurrency:
  group: pr-feedback
  cancel-in-progress: false

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
      - name: Apply PR feedback
        uses: iamomiid/taskal/codex-pr-feedback@main
        with:
          github-token: ${{ github.token }}
          repository: ${{ github.repository }}
          event-name: ${{ github.event_name }}
          issue-number: ${{ github.event.issue.number }}
          pull-request-number: ${{ github.event.pull_request.number }}
          issue-comment-author: ${{ github.event.comment.user.login }}
          review-author: ${{ github.event.review.user.login }}
          review-comment-id: ${{ github.event.comment.id }}
          issue-comment-id: ${{ github.event.comment.id }}
          review-id: ${{ github.event.review.id }}
          feedback: ${{ github.event.comment.body || github.event.review.body }}
          usage-threshold: "20"
```

Example workflow for issue implementation:

```yaml
name: Implement Tickets

on:
  issues:
    types: [opened, labeled]
  schedule:
    - cron: "*/30 * * * *"
  workflow_dispatch:

concurrency:
  group: implement-tickets
  cancel-in-progress: false

jobs:
  implement-next:
    runs-on: self-hosted
    permissions:
      contents: write
      issues: write
      pull-requests: write
    steps:
      - name: Implement next ticket
        uses: iamomiid/taskal/codex-implement-ticket@main
        with:
          github-token: ${{ github.token }}
          repository: ${{ github.repository }}
          event-name: ${{ github.event_name }}
          event-issue-number: ${{ github.event.issue.number }}
          base-branch: master
          usage-threshold: "20"
```

## Labels Used by `codex-implement-ticket`

The implementation action manages these labels automatically:

- `codex:implement`
- `codex:queued`
- `codex:running`
- `codex:pr-open`
- `codex:needs-human`
- `codex:error`

It can also read optional configuration labels from the issue:

- `codex:model:<model-name>`
- `codex:reasoning:<effort>`

Examples:

- `codex:model:gpt-5.4`
- `codex:reasoning:high`

If no labels are provided, both actions default to:

- model: `gpt-5.4`
- reasoning effort: `medium`

## Important Notes

- These actions are intended for self-hosted runners, not GitHub-hosted runners.
- Both actions check remaining Codex usage and pause or defer work when the configured threshold is reached.
- The usage guard depends on the current Codex auth file format and the ChatGPT usage endpoint remaining compatible.
- `codex-implement-ticket` resets and cleans the working tree before starting work inside the checked-out repository.
- `codex-pr-feedback` resumes prior Codex context when it finds a stored session id in the PR body.

## Versioning

Until tagged releases are published, examples may reference `@main`. For production use, prefer pinning to a tag or commit SHA once releases are available.

## Contributing

Contributions are welcome. Start with [`CONTRIBUTING.md`](./CONTRIBUTING.md).

## Formatting

This repo uses Prettier for Markdown, YAML, and JSON formatting.

```bash
npm install
npm run format
```

## Security

Please report vulnerabilities according to [`SECURITY.md`](./SECURITY.md).

## License

[MIT](./LICENSE)

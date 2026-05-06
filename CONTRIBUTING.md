# Contributing

Thanks for considering a contribution.

## Before You Start

- Open an issue for bugs, feature ideas, or behavior changes that affect public action inputs or workflow behavior
- Keep pull requests focused and easy to review
- Update documentation when changing action inputs, outputs, labels, or runner requirements

## Development Notes

This repository contains composite GitHub Actions. Most changes will happen in:

- [`codex-pr-feedback/action.yml`](./codex-pr-feedback/action.yml)
- [`codex-implement-ticket/action.yml`](./codex-implement-ticket/action.yml)

Please keep these points in mind:

- The actions are designed for self-hosted runners
- Shell steps should remain portable across common Linux runner setups
- New external dependencies should be avoided unless they materially simplify the action
- Backward compatibility matters for action inputs and label conventions

## Testing Changes

Before opening a pull request:

1. Validate YAML syntax for changed workflow or action files
2. Test on a self-hosted runner with `codex`, `gh`, `jq`, `curl`, and `git` installed
3. Run `npm run format` and confirm `npm run format:check` passes
4. Confirm the documented examples still match the supported inputs

## Pull Requests

Please include:

- a short summary of the change
- the motivation behind it
- any required migration notes for existing users

## Code of Conduct

By participating in this project, you agree to follow [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md).

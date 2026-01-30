# Puller Action

GitHub Action for syncing content from social platforms using [Puller](https://github.com/socialsbase/puller).

## Usage

```yaml
- uses: socialsbase/puller-action@v1
  env:
    VIBE_FOREM_API_KEY: ${{ secrets.VIBE_FOREM_API_KEY }}
  with:
    platform: devto
    output-dir: content
```

## Inputs

| Input            | Required | Default    | Description                                |
| ---------------- | -------- | ---------- | ------------------------------------------ |
| `platform`       | Yes      | `devto`    | Platform to pull from                      |
| `output-dir`     | Yes      | `content`  | Directory for pulled content               |
| `branch`         | No       | `live`     | Target branch for commits                  |
| `since`          | No       | -          | Only pull articles since date (YYYY-MM-DD) |
| `exclude-drafts` | No       | `false`    | Skip draft articles                        |
| `force`          | No       | `false`    | Re-pull existing articles                  |
| `dry-run`        | No       | `false`    | Preview only, no commits                   |
| `structure`      | No       | `platform` | Folder structure: `platform` or `flat`     |
| `commit-message` | No       | auto       | Custom commit message                      |
| `version`        | No       | `latest`   | Puller version to use                      |

## Outputs

| Output          | Description                                     |
| --------------- | ----------------------------------------------- |
| `pulled-count`  | Number of articles pulled                       |
| `skipped-count` | Number of articles skipped                      |
| `committed`     | Whether changes were committed (`true`/`false`) |
| `commit-sha`    | Commit SHA if changes were committed            |

## Permissions

The action requires `contents: write` permission to push commits to your repository. By default, the `GITHUB_TOKEN` only has read permissions. You must explicitly grant write access:

```yaml
jobs:
  sync:
    runs-on: ubuntu-latest
    permissions:
      contents: write
```

Without this permission, the action will fail with a "Missing required permission: contents: write" error.

## Complete Workflow Example

```yaml
name: Sync Dev.to Content

on:
  schedule:
    - cron: "0 6 * * *" # Daily at 6 AM UTC
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Pull articles
        id: puller
        uses: socialsbase/puller-action@v1
        env:
          VIBE_FOREM_API_KEY: ${{ secrets.VIBE_FOREM_API_KEY }}
        with:
          platform: devto
          output-dir: content
          branch: live
          exclude-drafts: true

      - name: Summary
        run: |
          echo "Pulled: ${{ steps.puller.outputs.pulled-count }}"
          echo "Skipped: ${{ steps.puller.outputs.skipped-count }}"
          echo "Committed: ${{ steps.puller.outputs.committed }}"
```

## How It Works

This action downloads the [Puller](https://github.com/socialsbase/puller) binary from the latest release and runs it to pull content from the specified platform. The pulled content is automatically committed to the target branch.

## License

MIT

# Automation

Central repository for GitHub Actions

## Available Workflows

### PNPM Bump Dependencies

This workflow is used to bump the dependencies to the latest version in a PNPM project.

How to use:

```yaml
name: Bump dependencies
on:
  workflow_dispatch:
  schedule:
    - cron: "0 0 * * 1"

jobs:
  deps-bump:
    uses: ./.github/workflows/pnpm-bump.yml
    secrets:
      GH_TOKEN: ${{ secrets.GH_TOKEN }}
```

where `GH_TOKEN` is a GitHub token for the user that you want to create the PRs as.

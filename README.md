# Automation

Central repository for GitHub Actions.

> [!IMPORTANT]
> All of these workflows require PNPM _and_ that PNPM manages your Node versions.
> See [the documentation](https://pnpm.io/cli/env) for more information.

## Available Workflows

### Bump Dependencies

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
    uses: markgaze/automation/.github/workflows/bump.yml@main
    secrets:
      GH_TOKEN: ${{ secrets.GH_TOKEN }}
```

where `GH_TOKEN` is a GitHub token for the user that you want to create the PRs as.

#### Inputs

| Input            | Description                             | Default                                      |
| ---------------- | --------------------------------------- | -------------------------------------------- |
| `bump-branch`    | The branch to commit the changes to.    | `bump`                                       |
| `commit-message` | The commit message to use for the bump. | `Bump dependencies`                          |
| `title`          | The title of the PR.                    | `Bump dependencies`                          |
| `body`           | The body of the PR.                     | `Bumping dependencies via GitHub Actions...` |
| `turbo-cache`    | Whether to use the Turborepo cache.     | `false`                                      |

### Format Code

This workflow is used to format the code in a PNPM project.

How to use:

```yaml
name: Format code
on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  format:
    uses: markgaze/automation/.github/workflows/format.yml@main
    secrets:
      GH_TOKEN: ${{ secrets.GH_TOKEN }}
```

where `GH_TOKEN` is a GitHub token for the user that you want to create the format commit as.

#### Inputs

| Input            | Description                             | Default           |
| ---------------- | --------------------------------------- | ----------------- |
| `commit-message` | The commit message to use for the bump. | `[ci] format`     |
| `branch`         | The branch to format the code on.       | `github.head_ref` |
| `command`        | The script to run to format the code.   | `format`          |
| `turbo-cache`    | Whether to use the Turborepo cache.     | `false`           |

## Extra Options

### Turbo Cache

If you use Turborepo and want to configure the cache, you can add the `turbo-cache` input to the workflow. For example:

```yaml
name: Format code
on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  format:
    uses: markgaze/automation/.github/workflows/format.yml@main
    secrets:
      GH_TOKEN: ${{ secrets.GH_TOKEN }}
with:
  turbo-cache: true
```

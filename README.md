# Automation

Central repository for GitHub Actions.

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

| Input                         | Description                                                     | Default                                      |
| ----------------------------- | --------------------------------------------------------------- | -------------------------------------------- |
| `bump-branch`                 | The branch to commit the changes to.                            | `bump`                                       |
| `commit-message`              | The commit message to use for the bump.                         | `Bump dependencies`                          |
| `title`                       | The title of the PR.                                            | `Bump dependencies`                          |
| `body`                        | The body of the PR.                                             | `Bumping dependencies via GitHub Actions...` |
| `turbo-cache`                 | Whether to use the Turborepo cache.                             | `false`                                      |
| `install-node`                | Whether to install Node.js using actions/setup-node.            | `false`                                      |
| `node-version`                | Node.js version to install (only used if install-node is true). | `lts/*`                                      |
| `require-minimum-release-age` | Require `pnpm-workspace.yaml` to contain `minimumReleaseAge`.   | `true`                                       |

If your repository does not use `minimumReleaseAge` in `pnpm-workspace.yaml`, you can disable the check by setting `require-minimum-release-age: false`:

```yaml
jobs:
  deps-bump:
    uses: markgaze/automation/.github/workflows/bump.yml@main
    with:
      require-minimum-release-age: false
```

It is strongly recommended to use `minimumReleaseAge` in `pnpm-workspace.yaml` to ensure that the updated dependencies are less likely to contain compromised versions.

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
```

#### Inputs

| Input            | Description                                                     | Default           |
| ---------------- | --------------------------------------------------------------- | ----------------- |
| `commit-message` | The commit message to use for the bump.                         | `[ci] format`     |
| `branch`         | The branch to format the code on.                               | `github.head_ref` |
| `command`        | The script to run to format the code.                           | `format`          |
| `turbo-cache`    | Whether to use the Turborepo cache.                             | `false`           |
| `install-node`   | Whether to install Node.js using actions/setup-node.            | `false`           |
| `node-version`   | Node.js version to install (only used if install-node is true). | `lts/*`           |

## Extra Options

### Node.js Installation

By default, these workflows use PNPM to manage Node.js versions. However, you can optionally install Node.js using GitHub Actions' setup-node action instead. This is useful if you don't want to rely on PNPM's environment management.

To enable Node.js installation:

```yaml
jobs:
  format:
    uses: markgaze/automation/.github/workflows/format.yml@main
    with:
      install-node: true
      node-version: "20" # or "lts/*", "18", etc.
```

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
    with:
      turbo-cache: true
```

### NPM Token

If you have a private package registry, you can add the `NPM_TOKEN` secret to the workflow. For example:

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
    with:
      turbo-cache: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

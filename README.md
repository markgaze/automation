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

| Input                         | Description                                                   | Default                                      |
| ----------------------------- | ------------------------------------------------------------- | -------------------------------------------- |
| `bump-branch`                 | The branch to commit the changes to.                          | `bump`                                       |
| `commit-message`              | The commit message to use for the bump.                       | `Bump dependencies`                          |
| `title`                       | The title of the PR.                                          | `Bump dependencies`                          |
| `body`                        | The body of the PR.                                           | `Bumping dependencies via GitHub Actions...` |
| `node-version-file`           | Node.js version file to determine the version to install.     | `""`                                         |
| `require-minimum-release-age` | Require `pnpm-workspace.yaml` to contain `minimumReleaseAge`. | `true`                                       |

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

| Input               | Description                                               | Default           |
| ------------------- | --------------------------------------------------------- | ----------------- |
| `commit-message`    | The commit message to use for the bump.                   | `[ci] format`     |
| `branch`            | The branch to format the code on.                         | `github.head_ref` |
| `command`           | The script to run to format the code.                     | `format`          |
| `turbo-cache`       | Whether to use the Turborepo cache.                       | `false`           |
| `node-version-file` | Node.js version file to determine the version to install. | `""`              |

## Extra Options

### Node.js Version

Node.js is installed by default. The version is determined by the `node-version-file` input, which should point to a file like `.nvmrc` or `.node-version` in your repository.

Example:

```yaml
jobs:
  format:
    uses: markgaze/automation/.github/workflows/format.yml@main
    with:
      node-version-file: ".nvmrc"
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

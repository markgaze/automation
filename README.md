# Automation

Central repository for GitHub Actions.

This repository provides both **reusable workflows** and **composite actions** for PNPM-based Node.js projects.

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

## Composite Actions

In addition to reusable workflows, this repository provides composite actions that can be used as steps within your own workflows. This gives you more flexibility to combine multiple actions or add custom steps before/after the automation tasks.

### Bump Action

Use the bump composite action to update dependencies in your workflow:

```yaml
name: Custom bump workflow
on:
  workflow_dispatch:

jobs:
  bump:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Bump dependencies
        uses: markgaze/automation/bump@main
        with:
          bump-branch: bump
          commit-message: "Bump dependencies"
          title: "Bump dependencies"
          body: "Bumping dependencies via GitHub Actions..."
          node-version-file: ".nvmrc"
          require-minimum-release-age: true
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

#### Bump Action Inputs

| Input                         | Description                                                   | Default                                      | Required |
| ----------------------------- | ------------------------------------------------------------- | -------------------------------------------- | -------- |
| `bump-branch`                 | The branch to commit the changes to.                          | `bump`                                       | No       |
| `commit-message`              | The commit message to use for the bump.                       | `Bump dependencies`                          | No       |
| `title`                       | The title of the PR.                                          | `Bump dependencies`                          | No       |
| `body`                        | The body of the PR.                                           | `Bumping dependencies via GitHub Actions...` | No       |
| `node-version-file`           | Node.js version file to determine the version to install.     | `""`                                         | No       |
| `require-minimum-release-age` | Require `pnpm-workspace.yaml` to contain `minimumReleaseAge`. | `true`                                       | No       |
| `GH_TOKEN`                    | GitHub token to create the PR.                                | `${{ github.token }}`                        | No       |
| `NPM_TOKEN`                   | NPM token to authenticate to a private package registry.      | `""`                                         | No       |

### Format Action

Use the format composite action to format code in your workflow:

```yaml
name: Custom format workflow
on:
  push:
    branches:
      - main

jobs:
  format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Format code
        uses: markgaze/automation/format@main
        with:
          commit-message: "[ci] format"
          branch: ${{ github.head_ref }}
          command: "format"
          turbo-cache: false
          node-version-file: ".nvmrc"
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

#### Format Action Inputs

| Input               | Description                                               | Default                  | Required |
| ------------------- | --------------------------------------------------------- | ------------------------ | -------- |
| `commit-message`    | The commit message to use for the formatting changes.     | `Format all files correctly` | No       |
| `branch`            | The branch to format the code on.                         | `${{ github.head_ref }}` | No       |
| `command`           | The script to run to format the code.                     | `format`                 | No       |
| `turbo-cache`       | Whether to use the Turborepo cache.                       | `false`                  | No       |
| `node-version-file` | Node.js version file to determine the version to install. | `""`                     | No       |
| `NPM_TOKEN`         | NPM token to authenticate to a private package registry.  | `""`                     | No       |

### When to Use Composite Actions vs Workflows

- **Use reusable workflows** when you want a complete, pre-configured job that handles everything from checkout to PR creation
- **Use composite actions** when you need more control, want to add custom steps, or need to combine multiple actions in a single job

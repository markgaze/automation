# Setup: Automated Dependency Update Issues

## Goal

Set up this repository so that:

1. A GitHub issue can be created automatically on a schedule or manually
2. That issue instructs an AI agent to:
   - upgrade dependencies
   - validate the project
   - fix straightforward breakages
   - open a pull request
3. The issue is assigned to `copilot`
4. Only one open dependency-update issue exists at a time
5. The issue content is tailored to this repository

## What to do

Inspect the repository and create the files and configuration needed for this setup.

Do not ask questions unless absolutely necessary. Infer the correct setup from the repo.

## 1. Inspect the repository

Determine all of the following from the repo contents.

### Package manager

Detect from lockfiles:

- `pnpm-lock.yaml` -> `pnpm`
- `bun.lockb` or `bun.lock` -> `bun`
- `yarn.lock` -> `yarn`
- `package-lock.json` -> `npm`

If multiple exist, prefer:

1. `pnpm`
2. `bun`
3. `yarn`
4. `npm`

### Repository structure

Determine whether this is:

- a single package repo
- a workspace repo
- a monorepo

Look for common tooling such as:

- `turbo.json`
- `nx.json`
- `workspace.json`
- `package.json` with `workspaces`

### Available validation scripts

Inspect the root `package.json` and use the repo’s existing top-level scripts where possible.

Detect which of these scripts actually exist:

- `format`
- `format:check`
- `lint`
- `typecheck`
- `test`
- `test:unit`
- `test:integration`
- `check`
- `build`

If the repo has an existing root command that already orchestrates validation across packages, prefer that instead of inventing per-package commands.

### Commit style

Inspect recent commit history and match the existing commit style.

For example, if the repo uses conventional commits, keep using that style.

## 2. Create the issue template

Create this file:

`.github/ISSUE_TEMPLATE/dependency-update.md`

Make the content concise and tailored to this repository.

Requirements:

- mention the detected package manager
- use the correct dependency upgrade command for that package manager
- tell the agent to run only scripts that actually exist in this repo
- prefer root/orchestrated commands if this is a monorepo
- tell the agent to use a commit message style consistent with the repo history
- tell the agent to create a PR
- keep the instructions short and practical

Use this structure, filling in repo-specific details:

```md
---
name: 🔄 Dependency Update (AI Task)
about: Upgrade dependencies and validate the project
title: "chore: upgrade dependencies to latest"
labels: ["dependencies", "automated"]
---

## Task

Upgrade all dependencies to latest and ensure the project still passes validation. Then open a PR.

## Package Manager

This repo uses: `<DETECTED_PACKAGE_MANAGER>`

## Upgrade

Use:

`<UPGRADE_COMMAND>`

## Install

Run the appropriate install command for this repo.

## Validate

Run the relevant existing root validation commands for this repo.

Use only scripts that actually exist.

Preferred order:

- format or format:check
- lint
- typecheck
- test
- build

Use repo-level/orchestrated commands where available.

## Fix

If dependency upgrades cause failures:

- apply minimal fixes only
- do not refactor unrelated code
- do not downgrade dependencies unless necessary

## Commit

Use a commit message consistent with the existing history.

Example:

`chore(deps): upgrade dependencies to latest`

## PR

Create a PR using the same style as the commit history.

Include:

- summary of dependency updates
- any compatibility fixes made
- confirmation of checks run and passed

## Done

- dependencies updated
- install succeeds
- relevant checks pass
- PR created
```

Replace placeholders with real values based on the repo.

Do not leave generic placeholders in the final file.

## 3. Create the workflow

Create this file:

`.github/workflows/dependency-update-issue.yml`

The workflow must:

- run on `workflow_dispatch`
- run weekly at around 1am on Monday morning UTC
- read the issue template file
- extract the issue title from frontmatter
- extract the issue body from the markdown after frontmatter
- check whether an open issue with the same title already exists
- if no matching open issue exists:
  - create the issue
  - assign it to `copilot`
- if a matching open issue already exists:
  - do nothing

Use this workflow content:

```yaml
name: Create dependency update issue

on:
  workflow_dispatch:
  schedule:
    - cron: "0 1 * * 1"

permissions:
  contents: read
  issues: write

jobs:
  create-issue:
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Read issue template
        id: template
        shell: bash
        run: |
          file=".github/ISSUE_TEMPLATE/dependency-update.md"

          title=$(awk '
            BEGIN { in_frontmatter=0 }
            /^---$/ && in_frontmatter==0 { in_frontmatter=1; next }
            /^---$/ && in_frontmatter==1 { exit }
            in_frontmatter==1 && /^title:/ {
              sub(/^title:[[:space:]]*/, "", $0)
              gsub(/^"/, "", $0)
              gsub(/"$/, "", $0)
              print
            }
          ' "$file")

          body=$(awk '
            BEGIN { dash_count=0 }
            /^---$/ { dash_count++; next }
            dash_count >= 2 { print }
          ' "$file")

          {
            echo "title<<EOF"
            echo "${title:-chore: upgrade dependencies to latest}"
            echo "EOF"
            echo "body<<EOF"
            echo "$body"
            echo "EOF"
          } >> "$GITHUB_OUTPUT"

      - name: Check for existing open issue
        id: existing
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.COPILOT_ASSIGN_TOKEN }}
          script: |
            const title = ${{ toJson(steps.template.outputs.title) }};

            const { data: issues } = await github.rest.issues.listForRepo({
              owner: context.repo.owner,
              repo: context.repo.repo,
              state: "open",
              labels: "automated",
              per_page: 100
            });

            const match = issues.find(issue => issue.title === title);

            core.setOutput("exists", match ? "true" : "false");
            core.setOutput("issue_number", match ? String(match.number) : "");

      - name: Create issue
        id: create
        if: steps.existing.outputs.exists != 'true'
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.COPILOT_ASSIGN_TOKEN }}
          script: |
            const title = ${{ toJson(steps.template.outputs.title) }};
            const body = ${{ toJson(steps.template.outputs.body) }};

            const issue = await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title,
              body,
              labels: ["dependencies", "automated"]
            });

            core.setOutput("issue_number", String(issue.data.number));

      - name: Assign to Copilot
        if: steps.existing.outputs.exists != 'true'
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.COPILOT_ASSIGN_TOKEN }}
          script: |
            const issue_number = Number(${{ toJson(steps.create.outputs.issue_number) }});

            await github.rest.issues.addAssignees({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number,
              assignees: ["copilot"]
            });
```

Only change this workflow if needed to make it work correctly with the repository.

Do not remove the duplicate-issue protection.

## 4. Secret requirements

Assume the repository owner will add this secret:

`COPILOT_ASSIGN_TOKEN`

This should be a PAT from a user that can assign issues in the repository.

Do not rename this secret unless there is a strong reason.

## 5. Final requirements

After making the changes:

- ensure the issue template is tailored to this repo
- ensure the workflow is valid
- keep the instructions concise
- do not add unrelated files
- do not perform unrelated refactors

## Expected result

Make the changes directly in the repository so that:

- `.github/ISSUE_TEMPLATE/dependency-update.md` exists
- `.github/workflows/dependency-update-issue.yml` exists
- the template is customized to the repo
- the workflow creates at most one open dependency-update issue at a time
- new issues are assigned to `copilot`

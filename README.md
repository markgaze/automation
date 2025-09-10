# Automation

Central repository for GitHub Actions

## Available Workflows

### PNPM Bump Dependencies

This workflow is used to bump the dependencies to the latest version in a PNPM project.

How to use:

```yaml
uses: ./.github/workflows/pnpm-bump.yml
with:
  GH_TOKEN: ${{ secrets.GH_TOKEN }}
```

where `GH_TOKEN` is a GitHub token with the `contents` and `pull-requests` permissions.

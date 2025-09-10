# GitHub Actions Automation Repository
This is a GitHub Actions workflows repository that provides reusable workflows for PNPM-based Node.js projects. It contains no buildable application code - only workflow definitions.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively
- Validate workflows:
  - Install actionlint: `wget https://github.com/rhysd/actionlint/releases/download/v1.7.7/actionlint_1.7.7_linux_amd64.tar.gz && tar -xzf actionlint_1.7.7_linux_amd64.tar.gz && chmod +x actionlint`
  - Run workflow validation: `./actionlint -color`
  - Expected time: Under 1 minute. Set timeout to 2 minutes minimum.
- Clean up validation files: `rm -f actionlint actionlint_1.7.7_linux_amd64.tar.gz LICENSE.txt && rm -rf docs man` after validation

## Critical Understanding
- **DO NOT attempt to build this repository** - it contains no source code, package.json, or build configuration
- **DO NOT look for npm/pnpm commands** - this repository provides workflows FOR projects that use PNPM, but is not itself a Node.js project
- **DO NOT search for test scripts** - the only validation is actionlint for workflow syntax
- **This is NOT an application** - it's a collection of reusable GitHub Actions workflows

## Repository Structure
- `.github/workflows/bump.yml` - Reusable workflow for dependency bumping in PNPM projects
- `.github/workflows/format.yml` - Reusable workflow for code formatting in PNPM projects  
- `.github/workflows/lint.yml` - Workflow for linting GitHub Actions files with actionlint
- `.github/dependabot.yml` - Dependabot configuration for GitHub Actions updates
- `README.md` - Comprehensive documentation with usage examples

## Validation
- **ALWAYS run actionlint validation** after making workflow changes: `./actionlint -color`
- **NEVER CANCEL actionlint** - it completes in under 1 minute
- Test workflow syntax changes by running actionlint before committing
- **No manual testing scenarios exist** - this repository contains no runnable applications
- **No build validation needed** - there is nothing to build

## Workflow Details
### bump.yml
- Bumps dependencies using `pnpm update -rL` in consuming projects
- Requires PNPM setup via `pnpm/action-setup@v4`
- Creates PR with dependency updates
- **Consumer projects must have PNPM** - workflow will fail otherwise

### format.yml  
- Runs formatting command (default: `pnpm run format`) in consuming projects
- Commits formatting changes automatically
- Requires PNPM setup and project with format script

### lint.yml
- Runs actionlint on GitHub Actions workflows
- Uses Docker image `rhysd/actionlint:1.7.7`
- Runs on every push and pull request

## Common Tasks
### Validating Workflow Changes
1. Install actionlint: `wget https://github.com/rhysd/actionlint/releases/download/v1.7.7/actionlint_1.7.7_linux_amd64.tar.gz && tar -xzf actionlint_1.7.7_linux_amd64.tar.gz && chmod +x actionlint`
2. Run validation: `./actionlint -color` (under 1 minute, set 2+ minute timeout)
3. Clean up: `rm -f actionlint actionlint_1.7.7_linux_amd64.tar.gz LICENSE.txt && rm -rf docs man`

### Making Workflow Changes
1. Edit workflow files in `.github/workflows/`
2. Validate syntax with actionlint
3. Update README.md if changing inputs/outputs
4. Commit changes - no build step required

## Time Expectations
- **actionlint validation: Under 1 minute** - Set timeout to 2+ minutes minimum
- **No builds exist** - Do not wait for or attempt builds
- **No test suites exist** - Do not look for or run tests beyond actionlint

## What NOT to Do
- Do not try to run `npm install`, `pnpm install`, or similar package manager commands
- Do not look for package.json, build scripts, or application code
- Do not attempt to "run" or "start" the application - there is none
- Do not search for test files or test runners
- Do not try to build, compile, or bundle anything
- Do not create development environments - this is a workflow-only repository

## Quick Reference
Repository contains exactly:
- 3 workflow files (.github/workflows/*.yml)
- 1 dependabot config (.github/dependabot.yml)  
- 1 documentation file (README.md)
- Git repository metadata (.git/)

That's it. No other files should exist in a clean repository state.
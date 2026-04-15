# AI Instructions

This directory contains instruction files for LLM agents.

## `bump.md`

This instruction file is for setting up automated dependency-update issues and workflow for a Copilot Agent, it will not work with any other agent without modification.

Copy this prompt to Copilot CLI (or similar):

```text
Configure this repo with https://github.com/markgaze/automation/blob/main/ai/bump.md
```

If you already have a token that can assign issues in the repository, use this prompt instead:

```text
Configure this repo with https://github.com/markgaze/automation/blob/main/ai/bump.md, using `TOKEN_NAME_HERE` in place of `COPILOT_ASSIGN_TOKEN`
```

where `TOKEN_NAME_HERE` is the name of the secret that contains the token.

## Update an existing dependency-update setup

If a repository already has the generated files, use this prompt to update the
existing setup from the latest instructions:

```text
Update this repository's dependency-update setup using https://github.com/markgaze/automation/blob/main/ai/bump.md as the source of truth.

Steps:
1. Inspect the current `.github/ISSUE_TEMPLATE/dependency-update.md` and `.github/workflows/dependency-update-issue.yml`.
2. Compare the current setup with the latest requirements in the source instructions.
3. Update both files to match the latest behaviour and requirements.
4. Preserve this repository's intentional local customizations (for example, secret names or repo-specific commands).
5. Keep the instructions concise and practical.
```

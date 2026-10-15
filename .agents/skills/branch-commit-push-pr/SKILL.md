---
name: branch-commit-push-pr
description: Use when preparing a branch, staging changes, writing Conventional Commits, pushing, or creating a pull request. Enforces branch safety, zero credential leaks, and publication risk disclosure.
---

# Branch, Commit, Push, and Pull Request

## Authorization and Risk

- **Never push directly to `main`**.
- Do not commit, push, create a release, or open/merge a pull request unless the user explicitly requests that specific action. Authorization to edit code does not authorize a git push or PR creation.
- A push to branches or `main` can trigger GitHub Actions workflows that publish images to GHCR and mirror them to Docker Hub. Before pushing, review workflow triggers and inform the user of external publication risks.
- Stage only intended files. Exclude `.codegraph` database state, vulnerability scan reports, local test images, or temporary artifacts.
- Never add AI or tool attribution to commit messages or PR descriptions.

## Preparing the Change

1. Inspect `git status --short --branch`, git remotes, and the pending diff.
2. If on `main`, switch to a descriptive feature branch (`feat/*`, `fix/*`, `chore/*`, `docs/*`). Do not hijack or rebase existing branches unexpectedly:
   ```bash
   git checkout -b <type>/<description>
   ```
3. Run `git diff --check` to check for whitespace errors or conflict markers.
4. Run deterministic validation relevant to the changed files (e.g. `hadolint`, workflow YAML validation).
5. Explicitly stage only the targeted files:
   ```bash
   git add <path1> <path2>
   ```
6. Review staged changes using `git diff --cached --stat` and `git diff --cached` to ensure zero unintended modifications or secrets exist.

## Conventional Commit Format

Format commits using Conventional Commits:

```text
<type>(<scope>): <imperative summary>

[optional body explaining motivation and context]
```

- Allowed types: `feat`, `fix`, `docs`, `chore`, `ci`, `test`.
- Common scopes: `baseos`, `runtime`, `workflow`, `deps`, `agents`.
- Examples:
  - `feat(runtime): add Python 3.12 Wolfi runtime image`
  - `fix(ci): scope BaseOS discovery to baseOS directory`
  - `ci(workflow): add runtime build and publish pipeline`
  - `docs(agents): define AGENTS.md and agent skills`

## Push and Pull Request

1. Confirm explicit user authorization before pushing:
   ```bash
   git push -u origin <branch-name>
   ```
2. If opening a pull request (via `gh pr create` or web):
   - Outline the problem, changes made, and validation executed.
   - List any live CI checks or external publications that will be triggered.
   - Do not claim external registry artifacts exist until CI successfully publishes them.

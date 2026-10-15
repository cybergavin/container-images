# Agent Instructions

This repository builds, tests, scans, signs, publishes, and mirrors hardened BaseOS and language runtime container images.
Keep this file concise; load only the matching procedure from `.agents/skills/`.

## Operating Principles

- **Preserve user context**: Start with `git status --short --branch`. Never reformat, revert, stage, or commit unrelated changes.
- **Define observable success first**: Know the exact success criteria and failure modes before modifying code.
- **Smallest coherent change**: Make minimal, surgical edits. Avoid speculative abstractions, unnecessary refactoring, or collateral churn.
- **Deterministic verification over assumptions**: Never assume code works. Run linters, syntax checks, local smoke builds, and `git diff --check` before reporting completion.
- **Fail fast and surface blockers**: Stop immediately when preconditions fail. Report exact error outputs rather than silently masking them.
- **Safe external actions**: Never push directly to `main`. Do not push container images, push git branches, dispatch workflows, or open PRs unless explicitly instructed.
- **Zero credential leaks**: Never commit or log tokens, credentials, vulnerability report outputs, local test images, or `.codegraph` database files.

## Skill Routing

- Use `$container-image-change` when modifying packages, users, labels, build arguments, or commands in an existing BaseOS or runtime Dockerfile.
- Use `$add-runtime-image` when adding a new runtime family, version, or variant under `runtimes/`.
- Use `$container-release-workflow` when changing or reviewing GitHub Actions workflows for discovery, tags, builds, scans, signing, GHCR publication, or Docker Hub mirroring.
- Use `$branch-commit-push-pr` when preparing a branch, staging changes, writing Conventional Commits, pushing, or preparing a pull request.

Load only the single skill matching the task. Use multiple skills only if the request spans multiple distinct operations.

## Sources of Truth

When sources disagree, follow this hierarchy:
1. Current working tree and Git status
2. Active `.github/workflows/*.yml` workflows and security configurations
3. Existing Dockerfiles under `baseOS/` and `runtimes/`
4. `README.md` and repository documentation

Key layout:
- `baseOS/<base>/Dockerfile`: Hardened minimal BaseOS images (Wolfi, Alpine, AlmaLinux, Amazon Linux).
- `runtimes/<runtime>/<version>/<variant>/Dockerfile`: Language runtime images built upon hardened BaseOS.
- `.github/workflows/validate.yml`: PR validation workflow running non-publishing lint, syntax, test builds, and smoke tests.
- `.github/workflows/baseos-build-publish.yml`: BaseOS discovery, local testing, Trivy/Grype scans, GHCR publication, Docker Hub mirror, Cosign signing.
- `.github/workflows/runtime-build-publish.yml`: Runtime discovery, BaseOS inheritance, runtime smoke tests, scans, GHCR publication, Docker Hub mirror, Cosign signing.
- `templates/GrypeReport.tmpl`: Template for generating supplemental HTML vulnerability reports.

## Repository Contracts

- **Security & User baseline**: Every container image must standardise on non-root `appuser:appgroup` (UID/GID 1001), working directory `/app`, and drop privileges before CMD/ENTRYPOINT.
- **Package manager guardrails**: BaseOS package managers wrap upgrade commands (e.g. `/sbin/apk` blocking `upgrade`) to ensure reproducibility. Runtimes must preserve and pass this guardrail check.
- **Base image inheritance**: Runtimes in CI build on top of published Cybergavin BaseOS images (`ghcr.io/cybergavin/<variant>:latest`). Keep Dockerfile `ARG BASE_IMAGE` defaults usable for standalone local builds.
- **Registries & Naming**:
  - BaseOS: `ghcr.io/cybergavin/<base>` mirrored to `docker.io/cybergavin/<base>`.
  - Runtimes: `ghcr.io/cybergavin/<runtime>-<variant>` mirrored to `docker.io/cybergavin/<runtime>-<variant>`.
- **Tag convention**: `${VERSION_ID}-b${num}-${YYYY.MM}`, `${VERSION_ID}`, and `latest`.
- **Quality & Security Gates**: Every build must pass Hadolint, local container smoke tests, Trivy scanning (blocking on CRITICAL and HIGH CVEs), Grype HTML reporting, Cosign keyless OIDC signing + verification, and SBOM + provenance attachment. Never bypass or lower gate thresholds to make a build pass.
- **Actions Pinning**: All third-party GitHub Actions must be pinned to full 40-character commit SHAs with semantic version comments. Non-trivial shell steps must use `set -euo pipefail`.

## Workflow & Verification

1. Review `git status --short` and identify the target scope.
2. Load the matching skill from `.agents/skills/`.
3. Plan the minimal required edits and state verification criteria.
4. Execute the edit and run deterministic verification:
   - For Dockerfiles: `hadolint` (with workflow ignore rules) and local container run/test when Docker is available.
   - For Workflows: YAML parser validation (`python3 -c "import yaml; yaml.safe_load(...)"`), `actionlint` / ShellCheck when available, SHA pin validation.
   - For Git: `git diff --check` and `git status --short`.
5. Report changes concisely: behavior modified, files changed, verification performed, and any external actions awaiting user approval.

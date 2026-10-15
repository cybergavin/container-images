---
name: container-release-workflow
description: Use when creating, updating, or reviewing GitHub Actions CI/CD workflows for discovery, builds, functional testing, Trivy/Grype scans, Cosign signing, GHCR publication, or Docker Hub mirroring.
---

# Container Release Workflows

## Scope and Architecture

This repository maintains three primary GitHub Actions workflows:
- `.github/workflows/validate.yml`: PR validation running non-publishing lint, syntax, test builds, and smoke tests.
- `.github/workflows/baseos-build-publish.yml`: Builds and publishes minimal BaseOS container images (`ghcr.io/cybergavin/<base>`).
- `.github/workflows/runtime-build-publish.yml`: Builds and publishes language runtime container images (`ghcr.io/cybergavin/<runtime>-<variant>`).

Both workflows share standard architectural guarantees:
1. Matrix discovery via `find` and `jq`.
2. Dynamic build tag calculation via `skopeo list-tags`: `${VERSION_ID}-b${num}-${YYYY.MM}`.
3. Local test build before any remote push.
4. Functional verification (startup, binary check, user check, package upgrade guardrails).
5. Vulnerability scanning: Trivy (blocking on HIGH / CRITICAL) + Grype HTML reporting (`templates/GrypeReport.tmpl`).
6. Dual-registry publishing: Primary GHCR push with embedded SBOM + provenance; non-blocking mirror to Docker Hub via `skopeo copy`.
7. Keyless Cosign signing using GitHub Actions OIDC (`id-token: write`) and immediate signature verification.
8. Attestation artifact extraction (SPDX JSON and in-toto provenance JSON).

## Workflow Invariants

- **Pin Actions by SHA**: Always pin third-party actions to full 40-character commit SHAs, adding a comment indicating the release tag (e.g. `uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`).
- **Least Privilege**: Only request necessary workflow permissions (`contents: write`, `packages: write`, `id-token: write`, `attestations: write`).
- **Discovery isolation**:
  - `baseos-build-publish.yml` MUST discover only `baseOS` Dockerfiles (`find baseOS -mindepth 2 -name Dockerfile`).
  - `runtime-build-publish.yml` MUST discover only `runtimes` Dockerfiles (`find runtimes -mindepth 4 -name Dockerfile`).
  - Never use broad root searches like `find . -name Dockerfile` that cross domain boundaries.
- **Robust Shell scripting**: All shell steps must use `set -euo pipefail`. Double-quote shell expansions.
- **Fail-safe gates**: Do not weaken vulnerability or functional gates to allow a failing build to publish.

## Validation Procedure

1. **YAML Syntax Validation**:
   Ensure valid YAML structure using Python:
   ```bash
   python3 -c "import yaml; yaml.safe_load(open('.github/workflows/baseos-build-publish.yml')); yaml.safe_load(open('.github/workflows/runtime-build-publish.yml')); print('Valid YAML')"
   ```
2. **Action Linting**:
   If `actionlint` is installed, run it against workflow files:
   ```bash
   actionlint .github/workflows/*.yml
   ```
3. **Trace Step Logic**:
   Manually trace variable interpolation (`$IMAGE`, `$BUILD_TAG`, `$BASE_IMAGE`, `$LOCAL_IMAGE`) through discovery, local build, test, scan, push, mirror, sign, and attestation.
4. **Git Inspection**:
   Run `git diff --check` and verify no unintended edits or secret leaks exist in the workflow files.

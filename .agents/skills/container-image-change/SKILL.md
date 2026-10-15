---
name: container-image-change
description: Use when changing packages, users, labels, build arguments, commands, or hardening in an existing BaseOS or runtime Dockerfile. Preserves non-root appuser, guardrails, and CI contracts.
---

# Modify an Existing Container Image

## Understand the Scope & Contracts

1. Read the target Dockerfile and its corresponding workflow (`.github/workflows/baseos-build-publish.yml` or `.github/workflows/runtime-build-publish.yml`).
2. Identify the exact package, tool, or label being modified. Verify upstream package names and dependencies before editing.
3. Keep Dockerfile `ARG BASE_IMAGE` default pointing to an unauthenticated public image (e.g. `cgr.dev/chainguard/wolfi-base:latest`) so developers can build locally without registry credentials. CI workflows override `BASE_IMAGE` when needed.

## Invariants to Preserve

- **Standardized non-root user**:
  - User `appuser` (UID 1001), group `appgroup` (GID 1001).
  - Working directory `/app` owned by `appuser:appgroup`.
  - Must finish with `USER appuser`. Root is only used during build steps requiring elevated setup.
- **Package guardrails**:
  - Never circumvent or remove package manager wrapper scripts (e.g. `/sbin/apk` blocking `upgrade`). Images must pass the package upgrade rejection test.
- **OCI & Maintainer Labels**:
  - Keep OpenContainers and Cybergavin labels accurate (`org.opencontainers.image.*`, `cg.image.*`, `build_label=${BUILD_LABEL}`).
- **Minimal attack surface**:
  - Avoid installing unnecessary packages or compilers into the final layer.
  - Clean package manager caches within the same `RUN` command.
  - Avoid leaving credentials, build-time tokens, or temporary files in the container layers.

## Validation Procedure

1. **Linting**:
   Run Hadolint against the modified Dockerfile:
   ```bash
   hadolint --ignore DL3018,DL3059,DL3041,DL3006,DL3066 <path/to/Dockerfile>
   ```
2. **Local Build** (when Docker is available):
   Build the image locally with a test tag:
   ```bash
   docker build -t test-image:local -f <path/to/Dockerfile> .
   ```
3. **Functional Smoke Tests**:
   Execute the same tests the CI workflow performs:
   - Startup check:
     ```bash
     docker run --rm test-image:local echo "startup ok"
     ```
   - User check:
     ```bash
     docker run --rm test-image:local id appuser
     ```
   - Upgrade guardrail check (must exit non-zero):
     ```bash
     ! docker run --rm test-image:local apk upgrade
     ```
   - Runtime version check (for runtimes, e.g. Python):
     ```bash
     docker run --rm test-image:local python --version
     docker run --rm test-image:local python -m pip --version
     ```
4. **Git Inspection**:
   Run `git diff --check` and verify no unintended changes or temporary files were staged.

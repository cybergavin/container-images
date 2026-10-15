---
name: add-runtime-image
description: Use when adding a new runtime family, version, or variant under runtimes/. Defines directory hierarchy, Dockerfile construction, workflow registration, and test verification.
---

# Add a New Runtime Container Image

## Directory Structure Contract

Runtime images follow a strict 4-level directory hierarchy under `runtimes/`:
```text
runtimes/<runtime>/<version>/<variant>/Dockerfile
```
Examples:
- `runtimes/python/3.12/wolfi/Dockerfile`
- `runtimes/python/3.11/alpine/Dockerfile`
- `runtimes/node/20/wolfi/Dockerfile`

Each path component defines the artifact identity:
- `<runtime>`: Runtime family (e.g., `python`, `node`, `golang`).
- `<version>`: Primary version ID (e.g., `3.12`, `20`, `1.22`).
- `<variant>`: BaseOS variant (e.g., `wolfi`, `alpine`, `almalinux`, `amazonlinux`).
- Resulting image name: `<runtime>-<variant>` (e.g., `python-wolfi`).

## Implementation Checklist

1. **Create the Dockerfile**:
   - `ARG BASE_IMAGE` default pointing to an unauthenticated public base (e.g. `cgr.dev/chainguard/wolfi-base:latest`).
   - `ARG BUILD_LABEL`.
   - Comprehensive OCI and Cybergavin metadata labels:
     - `org.opencontainers.image.title="cg-<runtime>-runtime"`
     - `org.opencontainers.image.version="<version>"`
     - `cg.image.runtime="<runtime>"`
     - `cg.image.runtime.version="<version>"`
     - `build_label=${BUILD_LABEL}`
   - Standard non-root `appuser:appgroup` (UID/GID 1001), `/app` directory, and `USER appuser`.
   - Set sensible default command/entrypoint (e.g. `CMD ["python", "--version"]`).

2. **Register in Workflow** (`.github/workflows/runtime-build-publish.yml`):
   - In `Set BASE_IMAGE and runtime test behavior`:
     - If adding a new `<variant>`, ensure the `case "$VARIANT"` block maps it to the Cybergavin BaseOS image (`ghcr.io/cybergavin/<variant>:latest`) and sets `package_manager`.
     - If adding a new `<runtime>` family, extend `case "$RUNTIME"` with:
       - `primary_check_cmd` (e.g. `node --version`)
       - `primary_check_pattern` (e.g. `v${VERSION_ID}`)
       - `secondary_check_cmd` (e.g. `npm --version`)

3. **Update Documentation**:
   - If adding a new runtime family or major version, update `README.md` with usage instructions and pull examples.

## Verification

1. **Lint**:
   ```bash
   hadolint --ignore DL3018,DL3059,DL3041,DL3006,DL3066 runtimes/<runtime>/<version>/<variant>/Dockerfile
   ```
2. **Local Build**:
   ```bash
   docker build -t test-runtime:local runtimes/<runtime>/<version>/<variant>
   ```
3. **Smoke Tests**:
   - Verify non-root user: `docker run --rm test-runtime:local id appuser`
   - Verify primary command: `docker run --rm test-runtime:local <primary_cmd>`
   - Verify package guardrail: `! docker run --rm test-runtime:local apk upgrade`
4. **Workflow Discovery**:
   Verify the discovery command finds the new Dockerfile:
   ```bash
   find runtimes -mindepth 4 -name Dockerfile | sort
   ```

# Hardened BaseOS & Runtime Container Images

A secure, hardened collection of minimal base container images and language runtimes for platform engineering, AI/LLM workloads, and application deployment. Standardized across multiple Linux distributions with built-in security controls, supply chain attestations, and dual-registry distribution.

[Watch this video](https://www.youtube.com/watch?v=zwlAeCTpTRc) for an overview of the base software in this repository.

---

## Available Images

### 1. BaseOS Images

| Base Image | Description | Size | libc | Package Manager | Security Posture | Optimal Use Cases |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **AlmaLinux Minimal** | RHEL-compatible, stable, enterprise-grade distro | 🟨 Medium | `glibc` | `microdnf` | CVE-patched, slower cadence | Legacy enterprise apps, RHEL workloads |
| **Alpine** | Minimalist, musl-based, widely used in container environments | 🟩 Tiny | `musl` | `apk` | Frequent updates, fast CVE patching | Scratch-like microservices, static Go/Rust apps |
| **Amazon Linux Minimal** | AWS-optimized distro with tight EC2/ECS/Lambda integration | 🟨 Medium | `glibc` | `microdnf` | AWS-patched, fast sync with AWS CVEs | AWS-hosted apps, Lambda container images, EKS |
| **Wolfi** | Hardened, minimal, `glibc`-based with apk-like build | 🟦 Tiny | `glibc` | `melange` + `apk` | SBOM-native, zero-CVE posture | LLM apps, secure microservices, distroless-style |

### 2. Language Runtime Images

| Runtime Image | Version | Base OS | Primary Registry | Mirrored Registry | Features |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Python** | `3.12` | Wolfi | `ghcr.io/cybergavin/python-wolfi:3.12` | `docker.io/cybergavin/python-wolfi:3.12` | Python 3.12, pip, setuptools, wheel, non-root user |
| **Python** | `3.13` | Wolfi | `ghcr.io/cybergavin/python-wolfi:3.13` | `docker.io/cybergavin/python-wolfi:3.13` | Python 3.13, pip, setuptools, wheel, non-root user |
| **Node.js** | `24` | Wolfi | `ghcr.io/cybergavin/node-wolfi:24` | `docker.io/cybergavin/node-wolfi:24` | Node.js 24, npm, production environment, non-root user |

---

## Common Features & Hardening

All BaseOS and runtime images include:
- **Non-root user**: Default execution as `appuser` (UID 1001) with `appgroup` (GID 1001).
- **Dedicated workdir**: `/app` with proper non-root ownership.
- **Package Guardrails**: Package manager wrappers prevent runtime `upgrade` commands to preserve immutability and reproducibility.
- **Minimal Attack Surface**: Based on minimal/distroless variants with package caches cleaned.
- **Standardized Metadata**: OpenContainers and Cybergavin metadata annotations (`org.opencontainers.image.*`, `cg.image.*`, `build_label`).

---

## Security & Supply Chain Integrity

1. **Vulnerability Management**:
   - **Trivy scanning**: High and Critical severity vulnerabilities strictly block image publication in release workflows.
   - **Grype reporting**: Supplemental comprehensive HTML vulnerability reports generated via [`templates/GrypeReport.tmpl`](templates/GrypeReport.tmpl).
2. **Supply Chain Attestation**:
   - **SBOM generation**: Embedded and standalone SPDX Software Bill of Materials (`.spdx.json`).
   - **Provenance**: In-toto build provenance tracked with maximum detail (`mode=max`).
   - **Keyless Signing**: Images signed using Sigstore Cosign via GitHub Actions OIDC and immediately verified against the repository identity.

---

## CI/CD Workflows

The repository uses three specialized GitHub Actions workflows:

1. **[PR Validation](docs/WORKFLOWS.md#1-pr-validation-workflow-validateyml)** (`validate.yml`):
   Runs non-publishing quality gates on PRs targeting `main` (Hadolint, workflow action SHA pinning, agent skills frontmatter, local test builds, and smoke tests).
2. **[BaseOS Release](docs/WORKFLOWS.md#2-baseos-build-publish-workflow-baseos-build-publishyml)** (`baseos-build-publish.yml`):
   Discovers `baseOS/` Dockerfiles, runs pre-publish tests/scans, publishes to GHCR, mirrors to Docker Hub, signs with Cosign, and publishes attestation artifacts.
3. **[Runtime Release](docs/WORKFLOWS.md#3-runtime-build-publish-workflow-runtime-build-publishyml)** (`runtime-build-publish.yml`):
   Discovers `runtimes/` Dockerfiles, resolves base images, runs runtime smoke tests, publishes to GHCR (`:BUILD_TAG`, `:<version>`, `:latest`), mirrors to Docker Hub, and signs with Cosign.

> For detailed architecture, step breakdowns, and Mermaid sequence diagrams, see [**docs/WORKFLOWS.md**](docs/WORKFLOWS.md).

---

## Image Tagging Convention

Images are published to both **GHCR** (`ghcr.io/cybergavin/<name>`) and **Docker Hub** (`cybergavin/<name>`):

- **BaseOS format**: `{base-version}-b{build-number}-{YYYY.MM}` and `latest`
  - Example: `ghcr.io/cybergavin/alpine:3.22.1-b01-2025.07`
- **Runtime format**: `{runtime-version}-b{build-number}-{YYYY.MM}`, `{runtime-version}`, and `latest`
  - Example: `ghcr.io/cybergavin/python-wolfi:3.12-b01-2026.10`, `3.12`, `latest`

---

## 🚀 Usage

### Pull Images

```bash
# Pull BaseOS image from GHCR
docker pull ghcr.io/cybergavin/wolfi:latest

# Pull Python runtime from GHCR or Docker Hub
docker pull ghcr.io/cybergavin/python-wolfi:3.13
docker pull cybergavin/python-wolfi:3.13

# Pull Node.js runtime from GHCR or Docker Hub
docker pull ghcr.io/cybergavin/node-wolfi:24
docker pull cybergavin/node-wolfi:24
```

### Run Containers

```bash
# Run BaseOS container (runs as non-root appuser)
docker run --rm ghcr.io/cybergavin/alpine:latest echo "Hello from Alpine!"

# Run Python runtime container
docker run --rm ghcr.io/cybergavin/python-wolfi:3.13 python -c "print('Hardened Python runtime ready!')"

# Run Node.js runtime container
docker run --rm ghcr.io/cybergavin/node-wolfi:24 node -e "console.log('Hardened Node.js runtime ready!')"
```

### Application Dockerfile Example

```dockerfile
FROM ghcr.io/cybergavin/python-wolfi:3.12

# Copy application files (appuser ownership already configured)
COPY --chown=appuser:appgroup requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appgroup . .

# Run application as non-root appuser
CMD ["python", "app.py"]
```

---

## 🔍 Verification

### Verify Image Signatures

```bash
cosign verify ghcr.io/cybergavin/python-wolfi:3.12 \
  --certificate-identity-regexp="^https://github.com/cybergavin/container-images/" \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com
```

### Download SBOM & Attestations

```bash
# Download SBOM
cosign download sbom ghcr.io/cybergavin/python-wolfi:3.12

# Download Provenance Attestation
cosign download attestation ghcr.io/cybergavin/python-wolfi:3.12
```

---

## 🤖 Agentic AI & Customizations

This repository is configured for autonomous AI pair programming (Codex, Antigravity, and other agents):
- [`AGENTS.md`](AGENTS.md): Repository operating principles, sources of truth, security contracts, and skill routing.
- `.agents/skills/`: On-demand modular workflows:
  - [`branch-commit-push-pr`](.agents/skills/branch-commit-push-pr/SKILL.md): Safe git branching, conventional commits, and staging hygiene.
  - [`container-image-change`](.agents/skills/container-image-change/SKILL.md): Modifying existing BaseOS or runtime Dockerfiles.
  - [`add-runtime-image`](.agents/skills/add-runtime-image/SKILL.md): Adding new language runtime versions or variants.
  - [`container-release-workflow`](.agents/skills/container-release-workflow/SKILL.md): Managing CI/CD pipelines, action SHA pinning, and release gates.
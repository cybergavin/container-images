# BaseOS Container Images

A secure, hardened collection of minimal base container images for platform engineering and application deployment. This repository provides standardized base images across multiple Linux distributions with built-in security controls and supply chain attestations.

## Available Images

| Base Image       | Description                                                           | Size      | libc    | Package Manager   | Security Posture                     | Optimal Use Cases                                                                                       |
| ---------------- | --------------------------------------------------------------------- | --------- | ------- | ----------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| **AlmaLinux Minimal**    | RHEL-compatible, stable, enterprise-grade distro                      | 🟨 Medium | `glibc` | `microdnf`             | CVE-patched, slower cadence          | - Legacy enterprise apps<br>- Python/C apps expecting RHEL<br>- Compliance-required workloads           |
| **Alpine**       | Minimalist, musl-based, widely used in container environments         | 🟩 Tiny   | `musl`  | `apk`             | Frequent updates, fast CVE patching  | - Scratch-like microservices<br>- Fast CI/CD<br>- Static Go/Rust apps             |
| **Amazon Linux Minimal** | AWS-optimized distro with tight EC2/ECS/Lambda integration            | 🟨 Medium | `glibc` | `microdnf`       | AWS-patched, fast sync with AWS CVEs | - AWS-hosted apps<br>- Lambda container images<br>- EKS/ECS-optimized workloads                         |
| **Wolfi**        | Hardened, minimal, `glibc`-based with apk-like `melange`/`apko` build | 🟦 Tiny   | `glibc` | `melange` + `apk` | SBOM-native, hardened by default     | - Zero CVE base images<br>- LLM apps<br>- SLSA-compliant builds<br>- Multi-arch distroless-style images |


## Image Specifications

### Common Features
All baseOS images include:
- **Pre-installed Tools**: curl, openssl, bash, jq, unzip, tzdata
- **Standardized User**: `appuser` (UID 1001) with `appgroup` (GID 1001)
- **Working Directory**: `/app` with proper ownership
- **Package Guardrails**: Wrapper scripts prevent `upgrade` commands
- **Metadata Labels**: OpenContainers annotations for source and maintainer info

### 🔒 Security Features

#### Hardening
- **Non-root user:** All images include an appuser for running applications securely
- **Package upgrade prevention:** Images are configured to prevent package upgrades to maintain consistency
- **Minimal attack surface:** Based on minimal/distroless variants where available

#### Vulnerability Management
- **Trivy scanning:** Critical and high-severity vulnerabilities block image publication
- **Grype scanning:** Additional vulnerability insights for comprehensive security assessment
- **Automated reporting:** Vulnerability reports generated and uploaded as artifacts

#### Supply Chain Security
- **SBOM generation:** Software Bill of Materials automatically generated for all images
- **Provenance attestation:** Build provenance tracked with maximum detail
- **Cosign signing:** Images signed with keyless signing using GitHub OIDC
- **Signature verification:** Automatic verification of image signatures post-build

### 📋 Build Process

#### Automated Triggers
- **Scheduled builds:** Runs every fortnight at 10 AM UTC (0 10 */14 * *)
- **Manual dispatch:** Can be triggered manually via GitHub Actions

#### Build Pipeline
- **Discovery:** Automatically discovers all Dockerfiles in the repository
- **Linting:** Hadolint validation of Dockerfiles
- **Local build:** Images built locally for testing before publication
- **Functional testing:** Container startup and security configuration validation
- **Vulnerability scanning:** Trivy and Grype security assessments
- **Publication:** Images pushed to GitHub container registry (`ghcr.io`) only after all tests pass
- **Attestation:** SBOM and provenance attached to published images
- **Signing:** Cosign keyless signing with GitHub OIDC

### 📦 Image Tags
- Images are published with versioning based on the base OS version:

```
{base-os-version}-b{build-number}-{YYYY.MM}
```
**Examples:**

`2023-b01-2025.01` - Amazon Linux 2023, first build of January 2025
`9.5-b02-2025.01` - AlmaLinux 9.5, second build of January 2025
`3.21-b01-2025.01` - Alpine 3.21, first build of January 2025

- All images are also tagged as `latest`.

### 🚀 Usage

#### Pull Images

```bash
# Pull latest Alpine-based image
docker pull ghcr.io/cybergavin/alpine:latest

# Pull specific versioned build
docker pull ghcr.io/cybergavin/alpine:3.22.1-b01-2025.07
```

#### Run Containers

```bash
# Run as non-root appuser
docker run --rm ghcr.io/cybergavin/alpine:latest echo "Hello from Alpine!"

# Interactive shell
docker run -it ghcr.io/cybergavin/almalinux:latest
```

#### Dockerfile Usage

```dockerfile
FROM ghcr.io/cybergavin/alpine:3.22.1-b01-2025.07

# Switch to root to install packages
USER root
RUN apk add --no-cache python3 py3-pip

# Create a venv owned by appuser
RUN python3 -m venv /opt/venv && \
    chown -R appuser:appgroup /opt/venv

# Drop privileges
USER appuser
WORKDIR /app

# Use venv by default
ENV PATH="/opt/venv/bin:$PATH"

# Copy app files
COPY --chown=appuser:appgroup . .

# Install Python deps into the venv
RUN pip install --no-cache-dir -r requirements.txt

# Run your app (replace with your actual app)
CMD ["python", "app.py"]

```

### 🔍 Verification

#### Verify Image Signatures
```bash
# Verify with cosign
cosign verify ghcr.io/cybergavin/alpine:latest \
  --certificate-identity-regexp="^https://github.com/cybergavin/container-images/" \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com
```

#### View Attestations
```bash
# View SBOM
cosign download sbom ghcr.io/cybergavin/alpine:latest

# View provenance
cosign download attestation ghcr.io/cybergavin/alpine:latest
```

### 📊 Build Artifacts
Each successful build generates the following artifacts:

#### Vulnerability Reports
- Trivy Report (TrivyScanReport.md) - Vulnerability scan for build pass/fail
- Grype Report (GrypeScanReport.html) - Comprehensive vulnerability analysis

#### Attestations
- SBOM (sbom-{base}-{tag}.spdx.json) - Software Bill of Materials in SPDX format
- Provenance (provenance-{base}-{tag}.json) - Build provenance information
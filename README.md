# BaseOS Container Images

A secure, hardened collection of minimal base container images for platform engineering and application deployment. This repository provides standardized base images across multiple Linux distributions with built-in security controls and supply chain attestations.

## Image Specifications

### Common Features
All baseOS images include:
- **Pre-installed Tools**: curl, openssl, bash, jq, unzip, tzdata
- **Standardized User**: `appuser` (UID 1001) with `appgroup` (GID 1001)
- **Working Directory**: `/app` with proper ownership
- **Package Guardrails**: Wrapper scripts prevent `upgrade` commands
- **Metadata Labels**: OpenContainers annotations for source and maintainer info

### Package Management Restrictions
```bash
# Allowed operations
apk add --no-cache python3        # ✅ Install new packages
microdnf install -y nodejs        # ✅ Install new packages

# Blocked operations
apk upgrade                       # ❌ Blocked by guardrails
microdnf upgrade -y              # ❌ Blocked by guardrails
```

The wrapper scripts display clear error messages directing users to contact platform teams for upgrade requests.

## Available Images

- **AlmaLinux** - Enterprise-grade RHEL-compatible base
- **Alpine** - Minimal, security-focused musl-based distribution  
- **Amazon Linux** - AWS-optimized distribution
- **Wolfi** - Distroless-style base with glibc and package manager

## Features

### Security Controls
- **Vulnerability Scanning**: All images undergo Trivy security scanning for CRITICAL and HIGH vulnerabilities
- **Supply Chain Security**: Images are signed with Cosign and include SLSA provenance attestations
- **SBOM Generation**: Software Bill of Materials (SBOM) in SPDX format for compliance and transparency
- **Package Management Guardrails**: Wrapper scripts prevent runtime package upgrades (`apk upgrade`, `microdnf upgrade`) to maintain image stability
- **Non-root User**: Includes pre-configured `appuser` (UID 1001) with dedicated `/app` workspace
- **Standardized User Model**: Consistent user/group configuration across all distributions

### Build Process
- **Automated Builds**: Fortnightly scheduled builds ensure images stay current with security patches
- **Dockerfile Linting**: Hadolint validation enforces best practices
- **Functional Testing**: Functional tests validate container startup and security controls
- **Standardized Tooling**: All images include essential tools (curl, openssl, bash, jq, unzip, tzdata)
- **Metadata Compliance**: OpenContainers annotations for source attribution and maintainer information

### Distribution Strategy
- **Versioning**: Images tagged with `{VERSION_ID}-b{BUILD_NUMBER}-{YYYY.MM}` format
- **Latest Tags**: Always-current `latest` tag for each distribution
- **Registry**: Published to GitHub Container Registry (ghcr.io)

## Usage

### Basic Usage

```bash
# Pull latest Alpine-based image
docker pull ghcr.io/cybergavin/alpine:latest

# Pull specific versioned build
docker pull ghcr.io/cybergavin/alpine:3.22.1-b01-2025.07

# Run with non-root user
docker run --user appuser ghcr.io/cybergavin/alpine:latest
```

### Dockerfile Example

```dockerfile
FROM ghcr.io/cybergavin/alpine:latest

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



## Security Verification

### Signature Verification
```bash
# Verify image signature
cosign verify ghcr.io/cybergavin/alpine:latest \
  --certificate-identity-regexp="^https://github.com/cybergavin/" \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com
```

### SBOM Access
```bash
# Extract SBOM for compliance
docker buildx imagetools inspect ghcr.io/cybergavin/alpine:latest \
  --format "{{ json .SBOM }}" > alpine-sbom.json
```

### Vulnerability Reports
Each build includes a security scan summary showing vulnerability counts by severity level. Reports are available in the GitHub Actions workflow summaries.

## Build System

### Automated Schedule
- **Frequency**: Every 14 days at 10 AM UTC
- **Trigger**: Scheduled via GitHub Actions cron
- **Manual Builds**: Available via workflow_dispatch

### Build Artifacts
Each successful build produces:
- Container image with security attestations
- SBOM in SPDX JSON format
- SLSA provenance attestation
- Vulnerability scan report

### Quality Gates
Builds must pass all stages to be published:
1. Dockerfile linting (Hadolint)
2. Successful image build
3. Functional testing (startup, package guardrails, user validation)
4. Security scanning (Trivy)
5. Signature verification


## Compliance

### Standards Alignment
- **NIST Cybersecurity Framework**: Implements detect, protect, and respond controls
- **CIS Benchmarks**: Follows container security best practices
- **SLSA Level 3**: Provides build provenance and verification
- **SPDX 2.3**: Standard SBOM format for license and dependency tracking

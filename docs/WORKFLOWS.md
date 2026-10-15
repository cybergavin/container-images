# CI/CD Workflows

This document describes the automated GitHub Actions pipelines powering image testing, security scanning, publishing, and supply chain attestation across this repository.

---

## Workflow Overview

| Workflow | File | Triggers | Scope | Key Outputs |
| :--- | :--- | :--- | :--- | :--- |
| **PR Validation** | [`.github/workflows/validate.yml`](../.github/workflows/validate.yml) | `pull_request` (targeting `main`), `workflow_dispatch` | `baseOS/**`, `runtimes/**`, `.github/workflows/**`, `.agents/**` | Static checks, local test builds, functional smoke tests, Trivy scan |
| **BaseOS Release** | [`.github/workflows/baseos-build-publish.yml`](../.github/workflows/baseos-build-publish.yml) | Schedule (`0 10 1 * *`), `workflow_dispatch` | `baseOS/**` | `ghcr.io/cybergavin/<base>` + Docker Hub mirror, Cosign signatures, SBOM/provenance artifacts |
| **Runtime Release** | [`.github/workflows/runtime-build-publish.yml`](../.github/workflows/runtime-build-publish.yml) | Schedule (`0 10 1 * *`), `workflow_dispatch` | `runtimes/**` | `ghcr.io/cybergavin/<runtime>-<variant>` + Docker Hub mirror, Cosign signatures, SBOM/provenance artifacts |

---

## 1. PR Validation Workflow (`validate.yml`)

Runs strictly with read-only permissions (`contents: read`) to validate changes before merging into `main`. No secrets, image pushes, or remote mutations occur.

```mermaid
sequenceDiagram
    autonumber
    actor Dev as Contributor / PR
    participant GHA as GitHub Actions Runner
    participant Docker as Local Docker Engine
    participant Trivy as Trivy Scanner

    Dev->>GHA: Open / Update Pull Request to main
    rect rgb(240, 248, 255)
        Note over GHA: Job 1: Static Lint & Syntax
        GHA->>GHA: Validate Workflow YAML syntax
        GHA->>GHA: Verify Actions 40-char SHA pinning
        GHA->>GHA: Validate Agent Skills YAML frontmatter
    end
    rect rgb(255, 250, 240)
        Note over GHA: Job 2: Detect Target Dockerfiles
        GHA->>GHA: git diff origin/main vs HEAD (filtered to Dockerfiles)
    end
    opt Dockerfiles Changed
        rect rgb(245, 255, 250)
            Note over GHA,Trivy: Job 3: Test Matrix
            GHA->>GHA: Hadolint Dockerfile linting
            GHA->>Docker: Build local image (push=false, load=true)
            GHA->>Docker: Run smoke tests (startup, UID 1001, guardrails)
            opt Runtime Image
                GHA->>Docker: Verify runtime version and package tools
            end
            GHA->>Trivy: Scan local image for CVEs (Advisory)
            Trivy-->>GHA: Report CVE summary in Step Summary
        end
    end
    GHA-->>Dev: PR Quality Gate Result (Pass / Fail)
```

---

## 2. BaseOS Build & Publish Workflow (`baseos-build-publish.yml`)

Builds hardened BaseOS images (`almalinux`, `alpine`, `amazonlinux`, `wolfi`), signs them with keyless Cosign via OIDC, and mirrors them to Docker Hub.

```mermaid
sequenceDiagram
    autonumber
    actor Cron as Schedule (Monthly) / Dispatch
    participant Runner as GitHub Actions Runner
    participant Docker as Docker Daemon / Buildx
    participant Sec as Trivy & Grype Scanners
    participant GHCR as ghcr.io (Primary Registry)
    participant DHub as Docker Hub (Mirror Registry)
    participant Cosign as Sigstore / Cosign

    Cron->>Runner: Trigger Workflow
    Runner->>Runner: Discover Dockerfiles under baseOS/
    Runner->>Runner: Derive BASE and compute BUILD_TAG via skopeo
    Runner->>Runner: Hadolint validation

    rect rgb(245, 255, 250)
        Note over Runner,Sec: Pre-Publish Verification
        Runner->>Docker: Build LOCAL image (no push)
        Runner->>Docker: Run functional checks (startup, UID 1001, upgrade guardrails)
        Runner->>Sec: Trivy vulnerability scan
        alt High or Critical CVEs found
            Sec-->>Runner: Exit with Error (Build Aborted)
        else Scan clean
            Sec-->>Runner: Scan Passed
        end
        Runner->>Sec: Grype scan (Generate HTML report)
        Runner->>Runner: Upload scan reports as artifacts
    end

    rect rgb(240, 248, 255)
        Note over Runner,DHub: Publishing & Mirroring
        Runner->>GHCR: Login with GHCR_PAT
        Runner->>Docker: Build & Push with SBOM + Provenance
        Docker->>GHCR: Push :BUILD_TAG and :latest
        Runner->>DHub: Login with DOCKER_HUB_PAT (best effort)
        Runner->>DHub: Mirror :BUILD_TAG and :latest via skopeo copy
    end

    rect rgb(255, 245, 245)
        Note over Runner,Cosign: Signing & Attestations
        Runner->>Cosign: Sign GHCR image digest via GitHub OIDC
        Runner->>Cosign: Verify signature with repo identity
        Runner->>Runner: Extract standalone SBOM (.spdx.json) & Provenance (.json)
        Runner->>Runner: Upload attestation artifacts
    end
```

---

## 3. Runtime Build & Publish Workflow (`runtime-build-publish.yml`)

Builds language runtime images (e.g. `python-wolfi`) inheriting from published Cybergavin BaseOS images.

```mermaid
sequenceDiagram
    autonumber
    actor Cron as Schedule (Monthly) / Dispatch
    participant Runner as GitHub Actions Runner
    participant Docker as Docker Daemon / Buildx
    participant Sec as Trivy & Grype Scanners
    participant GHCR as ghcr.io (Primary Registry)
    participant DHub as Docker Hub (Mirror Registry)
    participant Cosign as Sigstore / Cosign

    Cron->>Runner: Trigger Workflow
    Runner->>Runner: Discover Dockerfiles under runtimes/
    Runner->>Runner: Parse RUNTIME, VERSION_ID, and VARIANT
    Runner->>Runner: Set BASE_IMAGE (ghcr.io/cybergavin/variant:latest)
    Runner->>Runner: Compute BUILD_TAG via skopeo list-tags
    Runner->>Runner: Hadolint validation

    rect rgb(245, 255, 250)
        Note over Runner,Sec: Pre-Publish Verification
        Runner->>GHCR: Login to GHCR (to pull base image)
        Runner->>Docker: Build LOCAL image (no push)
        Runner->>Docker: Functional checks (startup, UID 1001, guardrails)
        Runner->>Docker: Verify primary CLI (e.g. python --version)
        Runner->>Docker: Verify secondary CLI (e.g. pip --version)
        Runner->>Sec: Trivy vulnerability scan (blocks on CRITICAL/HIGH)
        Runner->>Sec: Grype scan (Generate HTML report)
        Runner->>Runner: Upload scan reports as artifacts
    end

    rect rgb(240, 248, 255)
        Note over Runner,DHub: Publishing & Mirroring
        Runner->>Docker: Build & Push with SBOM + Provenance
        Docker->>GHCR: Push :BUILD_TAG, :VERSION_ID, and :latest
        Runner->>DHub: Login with DOCKER_HUB_PAT (best effort)
        Runner->>DHub: Mirror :BUILD_TAG, :VERSION_ID, :latest via skopeo copy
    end

    rect rgb(255, 245, 245)
        Note over Runner,Cosign: Signing & Attestations
        Runner->>Cosign: Sign GHCR image digest via GitHub OIDC
        Runner->>Cosign: Verify signature with repo identity
        Runner->>Runner: Extract standalone SBOM & Provenance artifacts
        Runner->>Runner: Upload attestation artifacts
    end
```

---

## Security Invariants Across Pipelines

1. **Gate before Push**: No remote image is ever published or mirrored without first passing local container smoke tests and Trivy vulnerability gates.
2. **Keyless Signing**: Images are signed using GitHub Actions short-lived OIDC tokens through Sigstore Cosign—no long-lived private signing keys are stored in repository secrets.
3. **Supply Chain Attestation**: Every published image embeds both an in-toto provenance attestation (`mode=max`) and an SPDX Software Bill of Materials (SBOM), and publishes standalone JSON artifacts to the workflow run.
4. **Action Pinning**: All third-party GitHub Actions are pinned to full 40-character commit SHAs to protect against upstream supply chain compromise.

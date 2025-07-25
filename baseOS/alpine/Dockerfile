# ============================================================================================
# Copyright (c) 2025 Cybergavin (https://github.com/cybergavin/container-images) - MIT License
# See the LICENSE file in the root of this repository for full license details.
# ============================================================================================
# Do not override the default base image
# The value of BASE_IMAGE is set in the GitHub Actions workflow to ensure consistency across builds and tests.
# If you need to change the base image, update the workflow file instead.

# Set the base image
ARG BASE_IMAGE=alpine
FROM ${BASE_IMAGE}

# Add metadata
LABEL cg.image.maintainer="cybergavin" \
      cg.image.description="Validated Alpine base OS image with essential tools" \
      cg.image.support="cybergavin@gmail.com" \
      org.opencontainers.image.source="https://github.com/cybergavin/container-images" \
      org.opencontainers.image.description="Validated Alpine base OS image with essential tools"

# Install essential tools without cache
RUN apk update && apk upgrade && apk add --no-cache \
    curl \
    openssl \
    bash \
    jq \
    unzip \
    tzdata

# Create standardized user (consistent across all base images)
RUN addgroup -g 1001 appgroup && \
    adduser -D -u 1001 -G appgroup -s /bin/bash appuser && \
    mkdir -p /app && \
    chown appuser:appgroup /app

# Copy the original package manager binary to a new location
# This is to allow us to create a wrapper script for package management
RUN cp /sbin/apk /sbin/apk.real

# Create wrapper script as a guardrail to prevent package upgrades
# This is to ensure the base image remains stable and consistent
RUN cat > /sbin/apk <<'EOF'
#!/bin/bash
# Check for blocked commands
for arg in "$@"; do
    case "$arg" in
        upgrade)
            echo "ERROR: 'apk $arg' is not allowed. Use 'apk add' to install new packages."
            echo "Contact the Cloud Platform Services team if you need package updates."
            exit 1
            ;;
    esac
done
exec /sbin/apk.real "$@"
EOF

# Set working directory
WORKDIR /app

# Set user to non-root for security
USER appuser
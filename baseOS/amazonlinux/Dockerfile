FROM public.ecr.aws/amazonlinux/amazonlinux:minimal

# Add metadata
LABEL cg.image.maintainer="cybergavin" \
      cg.image.description="Validated AmazonLinux base OS image with essential tools" \
      cg.image.support="cybergavin@gmail.com" \
      org.opencontainers.image.source="https://github.com/cybergavin/container-images"

# Install essential tools and clean cache
RUN microdnf update -y && microdnf install -y --nodocs \
    shadow-utils \
    openssl \
    jq \
    unzip \
    bind-utils \
    tzdata \
    && microdnf clean all

# Create standardized user (consistent across all base images)
RUN groupadd -g 1001 appgroup && \
    useradd -m -u 1001 -g appgroup -s /bin/bash appuser && \
    mkdir -p /app && \
    chown appuser:appgroup /app

# Copy the original package manager binary to a new location
# This is to allow us to create a wrapper script for package management
RUN cp /bin/microdnf /bin/microdnf.real

# Create wrapper script as a guardrail to prevent package upgrades
# This is to ensure the base image remains stable and consistent
RUN cat > /bin/microdnf <<'EOF'
#!/bin/bash
# Check for blocked commands
for arg in "$@"; do
    case "$arg" in
        update|upgrade)
            echo "ERROR: 'microdnf $arg' is not allowed. Use 'microdnf install' to install new packages."
            echo "Contact the Cloud Platform Services team if you need package updates."
            exit 1
            ;;
    esac
done
exec /bin/microdnf.real "$@"
EOF

# Set working directory
WORKDIR /app

# Set user to non-root for security
USER appuser
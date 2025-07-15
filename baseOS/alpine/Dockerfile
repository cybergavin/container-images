ARG VERSION_ID
FROM alpine:${VERSION_ID}

# Add metadata
LABEL cg.image.maintainer="cybergavin" \
      cg.image.description="Validated Alpine base OS image with essential tools" \
      cg.image.support="cybergavin@gmail.com" \
      org.opencontainers.image.source="https://github.com/cybergavin/container-images"

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
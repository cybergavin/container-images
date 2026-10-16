# Copyright (c) 2025-2026 Cybergavin - MIT License
# https://github.com/cybergavin/container-images
# Refresh releases with --pull --no-cache. The workflow controls BASE_IMAGE.

ARG BASE_IMAGE=cgr.dev/chainguard/wolfi-base:latest
FROM ${BASE_IMAGE}

USER 0:0

ENV HOME=/home/appuser \
    XDG_CACHE_HOME=/var/cache/app \
    PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Centralize inherited OS-package updates in this base build.
# Keep APK and /bin/sh for downstream language-runtime image builds.
RUN set -eu; \
    umask 022; \
    apk upgrade --no-cache; \
    apk add --no-cache ca-certificates-bundle; \
    addgroup -g 1001 appgroup; \
    adduser -D -H -u 1001 -G appgroup \
        -h /home/appuser -s /bin/sh appuser; \
    mkdir -p /app /home/appuser /var/lib/app /var/cache/app; \
    chown 0:0 /app /home/appuser; \
    chmod 0755 /app /home/appuser; \
    chown 1001:1001 /var/lib/app /var/cache/app; \
    chmod 0750 /var/lib/app /var/cache/app

# Release workflows supply these values; defaults support local builds.
ARG IMAGE_VERSION=dev
ARG VCS_REF=local
ARG BUILD_LABEL=local
ARG SOURCE_URL=https://github.com/cybergavin/container-images

LABEL org.opencontainers.image.title="cg-wolfi-runtime" \
      org.opencontainers.image.description="Minimal Wolfi OS base for downstream language runtimes" \
      org.opencontainers.image.version="${IMAGE_VERSION}" \
      org.opencontainers.image.revision="${VCS_REF}" \
      org.opencontainers.image.source="${SOURCE_URL}" \
      cg.image.maintainer="cybergavin" \
      cg.image.support="cybergavin@gmail.com" \
      cg.image.description="Minimal Wolfi OS base for downstream language runtimes" \
      cg.image.os="wolfi" \
      cg.image.variant="runtime" \
      build_label="${BUILD_LABEL}"

WORKDIR /app
USER 1001:1001

# Downstream applications supply their own command.
CMD ["/bin/sh"]

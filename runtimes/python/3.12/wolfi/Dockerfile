# Refresh the upstream base for each runtime release with --pull --no-cache.
# Publish each validated release under a new immutable tag or digest.
ARG BASE_IMAGE=ghcr.io/cybergavin/wolfi:latest
FROM ${BASE_IMAGE}

USER 0:0

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONNOUSERSITE=1 \
    PATH="/opt/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Policy: resolve current Python 3.12 packages when this layer is rebuilt.
# Package names keep Python within the 3.12 family while allowing updates.
# APK --no-cache does not disable Docker's build-layer cache.
RUN set -eu; \
    umask 022; \
    apk add --no-cache python-3.12; \
    python3.12 -m venv --without-pip /opt/venv; \
    chmod -R go-w /opt/venv

# Supplied by the release build; defaults allow standalone local developer builds.
ARG IMAGE_VERSION=dev
ARG VCS_REF=local
ARG SOURCE_URL=https://github.com/cybergavin/container-images
ARG BUILD_LABEL=local

LABEL org.opencontainers.image.title="cg-python-runtime" \
      org.opencontainers.image.description="Non-root Python 3.12 runtime base" \
      org.opencontainers.image.version="${IMAGE_VERSION}" \
      org.opencontainers.image.revision="${VCS_REF}" \
      org.opencontainers.image.source="${SOURCE_URL}" \
      cg.image.runtime="python" \
      cg.image.runtime.version="3.12" \
      cg.image.variant="runtime" \
      build_label="${BUILD_LABEL}"

WORKDIR /app
USER 1001:1001

CMD ["/opt/venv/bin/python", "--version"]

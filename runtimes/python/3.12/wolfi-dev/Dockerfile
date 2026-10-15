# Development and builder variant for Python 3.12 on Wolfi
ARG BASE_IMAGE=cgr.dev/chainguard/wolfi-base:latest
FROM ${BASE_IMAGE}

USER 0:0

ENV HOME=/home/appuser \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_NO_CACHE_DIR=1 \
    PATH="/home/appuser/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

RUN set -eu; \
    umask 022; \
    apk add --no-cache \
        python-3.12 \
        py3.12-pip \
        py3.12-setuptools \
        py3.12-wheel; \
    if ! grep -q '^appgroup:' /etc/group; then addgroup -g 1001 appgroup; fi; \
    if ! grep -q '^appuser:' /etc/passwd; then adduser -D -u 1001 -G appgroup -h /home/appuser -s /bin/sh appuser; fi; \
    ln -sf /usr/bin/python3 /usr/bin/python; \
    ln -sf /usr/bin/pip3 /usr/bin/pip; \
    mkdir -p /home/appuser/.local /app /opt/venv; \
    chown -R 1001:1001 /home/appuser /app /opt/venv

ARG IMAGE_VERSION=dev
ARG VCS_REF=local
ARG SOURCE_URL=https://github.com/cybergavin/container-images
ARG BUILD_LABEL=local

LABEL org.opencontainers.image.title="cg-python-dev" \
      org.opencontainers.image.description="Hardened Python 3.12 development and build image" \
      org.opencontainers.image.version="${IMAGE_VERSION}" \
      org.opencontainers.image.revision="${VCS_REF}" \
      org.opencontainers.image.source="${SOURCE_URL}" \
      cg.image.runtime="python" \
      cg.image.runtime.version="3.12" \
      cg.image.variant="dev" \
      build_label="${BUILD_LABEL}"

WORKDIR /app
USER 1001:1001

CMD ["python", "--version"]

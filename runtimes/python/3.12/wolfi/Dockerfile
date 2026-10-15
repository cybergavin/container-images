# This runtime defaults to the public Wolfi base image so local builds do not require GHCR auth.
# A dedicated runtime workflow can override BASE_IMAGE to the published Cybergavin Wolfi base image later.

ARG BASE_IMAGE=cgr.dev/chainguard/wolfi-base:latest
FROM ${BASE_IMAGE}
ARG BUILD_LABEL

USER root

LABEL org.opencontainers.image.title="cg-python-runtime" \
      org.opencontainers.image.description="Cybergavin Hardened Python 3.12 Runtime built on Wolfi" \
      org.opencontainers.image.url="https://github.com/cybergavin/container-images" \
      org.opencontainers.image.source="https://github.com/cybergavin/container-images" \
      org.opencontainers.image.version="3.12" \
      cg.image.maintainer="Cybergavin" \
      cg.image.description="Cybergavin Hardened Python 3.12 Runtime built on Wolfi" \
      cg.image.runtime="python" \
      cg.image.runtime.version="3.12" \
      build_label=${BUILD_LABEL}

ENV HOME=/home/appuser \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_NO_CACHE_DIR=1 \
    PATH="/home/appuser/.local/bin:${PATH}"

RUN apk add --no-cache \
    python-3.12 \
    py3.12-pip \
    py3.12-setuptools \
    py3.12-wheel \
    && if ! grep -q '^appgroup:' /etc/group; then addgroup -g 1001 appgroup; fi \
    && if ! grep -q '^appuser:' /etc/passwd; then adduser -D -u 1001 -G appgroup -h /home/appuser -s /bin/sh appuser; fi \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && ln -sf /usr/bin/pip3 /usr/bin/pip \
    && mkdir -p /home/appuser/.local /app \
    && chown -R appuser:appgroup /home/appuser /app

WORKDIR /app
USER appuser

CMD ["python", "--version"]

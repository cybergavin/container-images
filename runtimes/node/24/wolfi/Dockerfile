# This runtime defaults to the public Wolfi base image so local builds do not require GHCR auth.
# A dedicated runtime workflow can override BASE_IMAGE to the published Cybergavin Wolfi base image later.

ARG BASE_IMAGE=cgr.dev/chainguard/wolfi-base:latest
FROM ${BASE_IMAGE}
ARG BUILD_LABEL

USER root

LABEL org.opencontainers.image.title="cg-node-runtime" \
      org.opencontainers.image.description="Cybergavin Hardened Node.js 24 Runtime built on Wolfi" \
      org.opencontainers.image.url="https://github.com/cybergavin/container-images" \
      org.opencontainers.image.source="https://github.com/cybergavin/container-images" \
      org.opencontainers.image.version="24" \
      cg.image.maintainer="Cybergavin" \
      cg.image.description="Cybergavin Hardened Node.js 24 Runtime built on Wolfi" \
      cg.image.runtime="node" \
      cg.image.runtime.version="24" \
      build_label=${BUILD_LABEL}

ENV HOME=/home/appuser \
    NODE_ENV=production \
    NPM_CONFIG_UPDATE_NOTIFIER=false \
    PATH="/home/appuser/.local/bin:${PATH}"

RUN apk add --no-cache \
    nodejs-24 \
    npm \
    && if ! grep -q '^appgroup:' /etc/group; then addgroup -g 1001 appgroup; fi \
    && if ! grep -q '^appuser:' /etc/passwd; then adduser -D -u 1001 -G appgroup -h /home/appuser -s /bin/sh appuser; fi \
    && mkdir -p /home/appuser/.local /app \
    && chown -R appuser:appgroup /home/appuser /app

WORKDIR /app
USER appuser

CMD ["node", "--version"]

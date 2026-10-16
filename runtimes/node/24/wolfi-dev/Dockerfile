# Development and builder variant for Node.js 24 on Wolfi
ARG BASE_IMAGE=ghcr.io/cybergavin/wolfi:latest
FROM ${BASE_IMAGE}

USER 0:0

ENV NODE_ENV=development \
    NPM_CONFIG_UPDATE_NOTIFIER=false \
    PATH="/home/appuser/.local/bin:/app/node_modules/.bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

RUN set -eu; \
    umask 022; \
    apk add --no-cache \
        nodejs-24 \
        npm; \
    mkdir -p /home/appuser/.npm /home/appuser/.local /app; \
    chown -R 1001:1001 /home/appuser /app

ARG IMAGE_VERSION=dev
ARG VCS_REF=local
ARG SOURCE_URL=https://github.com/cybergavin/container-images
ARG BUILD_LABEL=local

LABEL org.opencontainers.image.title="cg-node-dev" \
      org.opencontainers.image.description="Hardened Node.js 24 development and build image" \
      org.opencontainers.image.version="${IMAGE_VERSION}" \
      org.opencontainers.image.revision="${VCS_REF}" \
      org.opencontainers.image.source="${SOURCE_URL}" \
      cg.image.runtime="node" \
      cg.image.runtime.version="24" \
      cg.image.variant="dev" \
      build_label="${BUILD_LABEL}"

WORKDIR /app
USER 1001:1001

CMD ["node", "--version"]

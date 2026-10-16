# Refresh the upstream base for each runtime release with --pull --no-cache.
# Publish each validated release under a new immutable tag or digest.
ARG BASE_IMAGE=ghcr.io/cybergavin/wolfi:latest
FROM ${BASE_IMAGE}

USER 0:0

ENV NODE_ENV=production \
    PATH="/app/node_modules/.bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Policy: resolve current Node.js 24 packages when this layer is rebuilt.
# APK --no-cache does not disable Docker's build-layer cache.
RUN set -eu; \
    umask 022; \
    apk add --no-cache nodejs-24

# Supplied by the release build; defaults allow standalone local developer builds.
ARG IMAGE_VERSION=dev
ARG VCS_REF=local
ARG SOURCE_URL=https://github.com/cybergavin/container-images
ARG BUILD_LABEL=local

LABEL org.opencontainers.image.title="cg-node-runtime" \
      org.opencontainers.image.description="Non-root Node.js 24 runtime base" \
      org.opencontainers.image.version="${IMAGE_VERSION}" \
      org.opencontainers.image.revision="${VCS_REF}" \
      org.opencontainers.image.source="${SOURCE_URL}" \
      cg.image.runtime="node" \
      cg.image.runtime.version="24" \
      cg.image.variant="runtime" \
      build_label="${BUILD_LABEL}"

WORKDIR /app
USER 1001:1001

CMD ["node", "--version"]

# SPDX-License-Identifier: MIT
# Copyright (c) 2025-2026 Keelson contributors (Fred Cooke)

FROM debian:trixie-20260803-slim
# Always use the most specific tag possible

# Required plain semantic version tag - X.Y eg 1.0
ARG VERSION

# Grab the image name so that if the image is ever separated from its name people can tell what it is
ARG TARGET_IMAGE_NAME

# Architecture from build system (used for binary downloads)
ARG DOCKER_PLATFORM_CPU_ARCHITECTURE

# Set the tool version for use below
ENV YQ_VERSION=4.53.2

# Strong checksums to match the above (per architecture)
ENV YQ_SHA512_amd64=95d5755266c77627e6facc9b443174f17b94ab3b2c427b860232c7b79f56f1f534ed2e0b5a79b41dfcc8246a05db987d9aeb39913f36563784258ebe1d6be763
ENV YQ_SHA512_arm64=a4ffc2dc4f8c1c43e1ee687205248d361a16d0a0c6a7ceab17c443746b9d5d014dbccff5c34883bca9115979037ee2d992aa0e6d9a485a7184136df275f6729f

# Validate VERSION is provided
RUN if [ -z "${VERSION}" ]; then \
        echo "ERROR: VERSION must be provided" && exit 42; \
    fi

# Set labels for metadata
LABEL description="Keelson - Test Image (BATS + shellcheck + yq; kubectl/skopeo/curl-to-network/ECR helper are all mocked at the bats layer so the image stays minimal)"
LABEL version="${VERSION}"
LABEL image.name="${TARGET_IMAGE_NAME}"
LABEL yq.version="${YQ_VERSION}"

# Install required packages and clean up.
# curl is build-time only (fetches yq); ca-certificates is needed for that.
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update
RUN apt-get dist-upgrade -y
RUN apt-get install -y --no-install-recommends \
    bash \
    bats \
    shellcheck \
    curl \
    ca-certificates
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

# Install yq 4
RUN curl -fsSL "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_${DOCKER_PLATFORM_CPU_ARCHITECTURE}" -o /usr/local/bin/yq
RUN sha512sum /usr/local/bin/yq
RUN eval CHECKSUM=\$YQ_SHA512_${DOCKER_PLATFORM_CPU_ARCHITECTURE} && \
    echo "${CHECKSUM}  /usr/local/bin/yq" | sha512sum -c -
RUN chmod +x /usr/local/bin/yq

# Runtime user matching the base image so tests can run unprivileged against the same UID and home
ENV KEELSON_USER_ID=4242
RUN mkdir -p /home/keelson
RUN groupadd -g ${KEELSON_USER_ID} keelson
RUN useradd -u ${KEELSON_USER_ID} -g keelson -d /home/keelson -s /bin/bash keelson
RUN chown keelson:keelson /home/keelson

# Deterministic time and text handling for every process in the image
ENV TZ=UTC
ENV LC_ALL=C

# Test layout: mount the consumer repo at /workspace, bats walks src/tests/
RUN mkdir -p /workspace
RUN chown keelson:keelson /workspace
WORKDIR /workspace

ENV PS1="\u@\h:\w\\$ "

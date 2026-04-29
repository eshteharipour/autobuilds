ARG BASE_IMAGE=debian:12-slim
FROM ${BASE_IMAGE} AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    libsodium-dev \
    autoconf \
    libtool \
    libprotobuf-dev \
    protobuf-compiler \
    libutempter-dev \
    libcurl4-openssl-dev \
    build-essential \
    ninja-build \
    cmake \
    git \
    unzip \
    curl \
    zip \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /src
RUN git clone --recurse-submodules https://github.com/MisterTea/EternalTerminal.git /src

WORKDIR /src/build

# Dynamically calculate the version
# 1. git describe gets the closest tag + commit hash
# 2. sed 's/^v//' removes the leading 'v' (e.g. v6.2.9 -> 6.2.9)
# 3. sed 's/-/+git/' replaces the first dash so Debian sorts it correctly (e.g. 6.2.9-12-gabc -> 6.2.9+git12-gabc)
RUN PKG_VERSION=$(git describe --tags --always | sed 's/^v//' | sed 's/-/+git/') && \
    echo "Building version: ${PKG_VERSION}" && \
    cmake ../ \
    -DCPACK_PACKAGE_VERSION="${PKG_VERSION}" \
    -DCPACK_DEBIAN_PACKAGE_DEPENDS="libprotobuf32" \
    -DCPACK_GENERATOR="DEB" && \
    make -j$(nproc) && \
    make package

FROM scratch AS exporter
COPY --from=builder /src/build/*.deb /

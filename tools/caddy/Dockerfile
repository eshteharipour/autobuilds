FROM caddy:builder AS builder

ARG CADDY_VERSION="v2.7.6"
ARG PLUGINS=""

# Convert comma/space separated list into --with arguments
RUN set -e; \
    ARGS=""; \
    for plugin in $(echo $PLUGINS | tr ',' ' '); do \
        ARGS="$ARGS --with $plugin"; \
    done; \
    echo "Building with: $ARGS"; \
    xcaddy build $CADDY_VERSION $ARGS

FROM debian:bullseye-slim AS packager

ARG CADDY_VERSION="v2.7.6"
ARG PKG_REVISION="1"
ARG PROFILE_NAME="custom"

RUN apt-get update && apt-get install -y git dpkg-dev fakeroot binutils

# Clone the official dist repo to grab the systemd, sysusers, and scripts
RUN git clone https://github.com/caddyserver/dist.git /caddy-dist

# Prepare Debian package structure
WORKDIR /build/caddy-pkg
RUN mkdir -p \
    usr/bin \
    etc/caddy \
    usr/share/caddy \
    lib/systemd/system \
    usr/lib/sysusers.d \
    DEBIAN

# Copy binary from builder
COPY --from=builder /usr/bin/caddy usr/bin/caddy

# Set permissions
RUN chmod 755 usr/bin/caddy

# Copy Official Dist files
RUN cp /caddy-dist/config/Caddyfile etc/caddy/Caddyfile && \
    cp /caddy-dist/init/caddy.service lib/systemd/system/ && \
    cp /caddy-dist/init/caddy-api.service lib/systemd/system/ && \
    cp /caddy-dist/init/caddy.sysusers usr/lib/sysusers.d/caddy.conf && \
    cp /caddy-dist/welcome/index.html usr/share/caddy/index.html

# Copy Debian maintainer scripts
RUN cp /caddy-dist/scripts/postinstall.sh DEBIAN/postinst && \
    cp /caddy-dist/scripts/preremove.sh DEBIAN/prerm && \
    cp /caddy-dist/scripts/postremove.sh DEBIAN/postrm && \
    chmod 755 DEBIAN/postinst DEBIAN/prerm DEBIAN/postrm

# Generate the control file
RUN ARCH=$(dpkg --print-architecture) && \
    VERSION=$(echo $CADDY_VERSION | sed 's/^v//') && \
    cat <<EOF > DEBIAN/control
Package: caddy-${PROFILE_NAME}
Version: ${VERSION}-${PKG_REVISION}
Architecture: ${ARCH}
Maintainer: Custom Build <shayaneshteharipour@gmail.com>
Section: web
Priority: optional
Depends: systemd, adduser
Conflicts: caddy
Provides: caddy
Description: Caddy web server with custom plugins (${PROFILE_NAME})
EOF

# Build the .deb file
WORKDIR /build
RUN fakeroot dpkg-deb --build caddy-pkg "caddy-${PROFILE_NAME}_${CADDY_VERSION}-${PKG_REVISION}_amd64.deb"

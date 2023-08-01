# docker build -t arm32v7/opensesame . --build-arg VARIANT=bullseye
ARG VARIANT=bullseye
ARG RUST_VERSION=1.67.0
FROM rust:${RUST_VERSION}-slim-${VARIANT}
ENV DEBIAN_FRONTEND=noninteractive \
    PKG_CONFIG_PATH="/usr/lib/arm-linux-gnueabihf/pkgconfig" \
    PKG_CONFIG_ALLOW_CROSS="true" \
    GETTEXT_SYSTEM=1

# Set locale
RUN apt-get update && apt-get upgrade -y  && \
    apt-get install -y --no-install-recommends sudo tzdata locales ssh software-properties-common apt-transport-https debhelper d-shlibs
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Install libelektra armhf
RUN dpkg --add-architecture armhf && apt-get update
RUN echo "deb [trusted=yes] https://thesis.maxirlinger.at/repo/ armhf/" > /etc/apt/sources.list.d/elektra-armhf.list

RUN set -eux; \
    apt-get update && \
    apt-get install -y --no-install-recommends --allow-unauthenticated \
    gcc \
    curl \
    wget \
    clang \
    pkg-config \
    build-essential \
    ca-certificates \
    libssl-dev:armhf \
    libelektra-dev:armhf \
    apt-transport-https \
    software-properties-common \
    gettext

# Elektra armhf symlinks
RUN ln -s /usr/lib/arm-linux-gnueabi /usr/lib/pkgconfig/arm-linux-gnueabi
RUN ln -s /usr/lib/arm-linux-gnueabihf /usr/lib/pkgconfig/arm-linux-gnueabihf
RUN ln -s /usr/lib/arm-none-eabi /usr/lib/pkgconfig/arm-none-eabi

# For cross compilation
RUN apt install -y --no-install-recommends \
    openssl \
    libc6-dev \
    libclang-dev \
    libc6-dev-i386 \
    libc6-armhf-cross \
    libc6-dev-armhf-cross \
    gcc-arm-linux-gnueabihf \
    binutils-arm-linux-gnueabihf && \
    rm -rf /var/lib/apt/lists/*

# Install deps
RUN cargo install cargo-deb
RUN rustup component add clippy rustfmt
RUN rustup target add armv7-unknown-linux-gnueabihf
RUN rustup toolchain install stable-armv7-unknown-linux-gnueabihf 
RUN chown 47110:47110 -R /usr/local/cargo

# docker build -t opensesame . --build-arg
# docker build -t opensesame:armhf . --build-arg CROSS=armhf --build-arg VARIANT=bullseye
# docker build -t opensesame:armhf . --build-arg CROSS=armhf --build-arg VARIANT=buster
ARG CROSS=amd64
ARG VARIANT=bullseye
ARG RUST_VERSION=1.67.0

FROM rust:${RUST_VERSION}-slim-${VARIANT} AS builder
ENV DEBIAN_FRONTEND=noninteractive
ARG CROSS
ARG VARIANT
ARG RUST_VERSION

# Set locale
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends sudo tzdata locales ssh gnupg software-properties-common apt-transport-https debhelper d-shlibs
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Install libelektra based on architecture
RUN set -eux; \
    case "${CROSS}" in \
        amd64) \
            # Install libelektra for amd64
            apt-key adv --keyserver keyserver.ubuntu.com --recv-keys F26BBE02F3C315A19BF1F791A9A25CC1CC83E839 && \
            echo "deb https://debs.libelektra.org/$VARIANT $VARIANT main" | sudo tee /etc/apt/sources.list.d/elektra.list; \
            ;; \
        armhf) \
            # Install libelektra for armhf
            dpkg --add-architecture armhf && apt-get update && \
            echo "deb [trusted=yes] https://thesis.maxirlinger.at/repo/ armhf/" > /etc/apt/sources.list.d/elektra-armhf.list; \
            # Set important env variables for cross compilation
            export PKG_CONFIG_PATH="/usr/lib/arm-linux-gnueabihf/pkgconfig"; \
            export PKG_CONFIG_ALLOW_CROSS="true"; \
            ;; \
        *) \
            echo >&2 "unsupported cross compilation: ${CROSS}"; \
            exit 1; \
            ;; \
    esac

RUN if [ "${CROSS}" = "amd64" ]; then \
    set -eux; \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    curl \
    wget \
    clang \
    pkg-config \
    build-essential \
    ca-certificates \
    libssl-dev \
    apt-transport-https \
    software-properties-common \
    gettext \
    libelektra5-all; \
fi

# For cross compilation (applicable for armhf)
RUN if [ "${CROSS}" = "armhf" ]; then \
    set -eux; \
    apt-get update && \
    apt-get install -y --no-install-recommends --allow-unauthenticated \
    openssl \
    libssl-dev:armhf \
    libc6-dev \
    libclang-dev \
    libc6-dev-i386 \
    libc6-armhf-cross \
    libc6-dev-armhf-cross \
    libelektra-dev:armhf  \
    gcc-arm-linux-gnueabihf \
    binutils-arm-linux-gnueabihf && \
    rm -rf /var/lib/apt/lists/*; \
fi

# Elektra armhf symlinks (applicable for armhf)
RUN if [ "${CROSS}" = "armhf" ]; then \
    ln -s /usr/lib/arm-linux-gnueabi /usr/lib/pkgconfig/arm-linux-gnueabi; \
    ln -s /usr/lib/arm-linux-gnueabihf /usr/lib/pkgconfig/arm-linux-gnueabihf; \
    ln -s /usr/lib/arm-none-eabi /usr/lib/pkgconfig/arm-none-eabi; \
fi

# Install additional Rust dependencies and set up permissions
RUN cargo install cargo-deb
RUN rustup component add clippy rustfmt
RUN rustup target add armv7-unknown-linux-gnueabihf

ENV PROJECT_PATH /home/batman/
WORKDIR $PROJECT_PATH
COPY . $PROJECT_PATH

# Second stage: Final image with only the necessary artifacts
FROM rust:${RUST_VERSION}-slim-${VARIANT}

# Copy the installed dependencies from the builder stage
COPY --from=builder /usr/local/cargo /usr/local/cargo
COPY --from=builder /usr/bin/kdb /usr/bin/kdb

# Set environment variables
ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH \
    RUST_VERSION=${RUST_VERSION}

# Change ownership of cargo directories
RUN chown 47110:47110 -R /usr/local/cargo

# Verify installation
RUN rustup --version && \
    cargo --version && \
    rustc --version

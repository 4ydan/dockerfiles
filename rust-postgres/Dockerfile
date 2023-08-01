FROM rust:1.67.1-slim-bookworm

ENV DEBIAN_FRONTEND=noninteractive \
    DIESEL_CLI_VERSION=2.0.1 \
    TYPESHARE_CLI_VERSION=1.6.0 \
    RUSTFLAGS='-D warnings' \
    RUSTDOCFLAGS='-D warnings' \
    DATABASE_URL=postgres://ci:ci@db/ci \
    BIND_ADDRESS_HOST=127.0.0.1 \
    BIND_ADDRESS_PORT=8080 \
    AUTH_DISCOVERY_URI=unused \
    AUTH_CLIENT_ID=unused

RUN apt update && apt install -y --no-install-recommends \
    make \
    git \
    libpq-dev \
    libssl-dev \
    pkg-config \
    postgresql-client \
    pre-commit && \
    rm -rf /var/lib/apt/lists/*

RUN cargo install diesel_cli@$DIESEL_CLI_VERSION --no-default-features --features postgres && \
    cargo install typeshare-cli@$TYPESHARE_CLI_VERSION && \
    rustup component add clippy rustfmt && \
    chown 47110:47110 -R /usr/local/cargo

ENV PROJECT_PATH /home/batman/
WORKDIR $PROJECT_PATH
COPY . $PROJECT_PATH

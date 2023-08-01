# docker build -t my_image .
FROM debian:bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive \
    RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH \
    RUST_VERSION=1.67.1 \
    DIESEL_VERSION=2.0.2 \
    MDBOOK_VERSION=0.4.32 \
    MDBOOK_MERMAID_VERSION=0.12.6 \
    TYPESHARE_CLI_VERSION=1.0.0

# Create folders
RUN mkdir -p $RUSTUP_HOME && mkdir -p $CARGO_HOME

# Set locale to en_US.UTF-8
RUN apt-get update && \
    apt-get install -y --no-install-recommends sudo tzdata locales ssh && \
    rm -rf /var/lib/apt/lists/*
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Install packages
RUN set -eux; \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    make \
    coreutils \
    build-essential \
    libpq-dev \
    libssl-dev \
    pkg-config \
    postgresql-client \
    ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Install rust
RUN curl --proto '=https' --tlsv1.2 -sSf -y https://sh.rustup.rs | sh && \
    curl https://sh.rustup.rs -sSf | sh -s -- -y \
    --profile=minimal \
    --default-toolchain $RUST_VERSION -v \
    --component clippy rustfmt && \
    # Install cargo packages
    cargo install diesel_cli@$DIESEL_VERSION --no-default-features --features postgres && \
    cargo install --git https://github.com/ElektraInitiative/mdbook-generate-summary mdbook-generate-summary --locked

# Install mdbook pre-compiled binary
RUN curl -LO https://github.com/rust-lang/mdBook/releases/download/v$MDBOOK_VERSION/mdbook-v$MDBOOK_VERSION-x86_64-unknown-linux-gnu.tar.gz && \
tar -xzf mdbook-v$MDBOOK_VERSION-x86_64-unknown-linux-gnu.tar.gz && \
mv mdbook /usr/local/bin && \
mdbook --version

# Install mdbook-mermaid pre-compiled binary
RUN curl -LO https://github.com/badboy/mdbook-mermaid/releases/download/v$MDBOOK_MERMAID_VERSION/mdbook-mermaid-v$MDBOOK_MERMAID_VERSION-x86_64-unknown-linux-gnu.tar.gz && \
tar -xzf mdbook-mermaid-v$MDBOOK_MERMAID_VERSION-x86_64-unknown-linux-gnu.tar.gz && \
mv mdbook-mermaid /usr/local/bin && \
mdbook-mermaid --version

# Install typeshare pre-compiled binary
RUN curl -LO https://github.com/1Password/typeshare/releases/download/v$TYPESHARE_CLI_VERSION/typeshare-cli-v$TYPESHARE_CLI_VERSION-x86_64-unknown-linux-gnu.tar.xz && \
tar -xJf typeshare-cli-v$TYPESHARE_CLI_VERSION-x86_64-unknown-linux-gnu.tar.xz && \
cd typeshare-cli-v$TYPESHARE_CLI_VERSION-x86_64-unknown-linux-gnu && \
mv typeshare /usr/local/bin && \
typeshare --version

# One day install diesel_cli binary https://github.com/diesel-rs/diesel/issues/2379

# Sanity check
RUN rustup --version \
    cargo --version \
    rustc --version

# Set permissions
RUN chmod -R a+w $RUSTUP_HOME $CARGO_HOME && chown 47110:47110 -R $CARGO_HOME

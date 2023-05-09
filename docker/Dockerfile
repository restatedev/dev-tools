FROM --platform=$BUILDPLATFORM rust:1.67.1-bookworm
# prepopulate cargo index
RUN cargo search --limit=0
RUN apt update && apt upgrade -y
RUN apt -y install \
    g++-x86-64-linux-gnu \
    g++-aarch64-linux-gnu \
    protobuf-compiler \
    libprotoc-dev \
    clang \
    llvm \
    ca-certificates
RUN dpkg --add-architecture amd64 && dpkg --add-architecture arm64
RUN apt update
RUN apt -y install \
    musl-dev:arm64 \
    musl-dev:amd64
RUN curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /usr/local/bin
RUN rustup target add \
    x86_64-unknown-linux-musl \
    aarch64-unknown-linux-musl \
    x86_64-unknown-linux-gnu \
    aarch64-unknown-linux-gnu
RUN rustup component add clippy rustfmt
ENV CC_x86_64_unknown_linux_musl="clang"
ENV AR_x86_64_unknown_linux_musl="llvm-ar"
ENV CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_RUSTFLAGS="-Clink-self-contained=yes -Clinker=rust-lld"
ENV CC_aarch64_unknown_linux_musl="clang"
ENV AR_aarch64_unknown_linux_musl="llvm-ar"
ENV CARGO_TARGET_AARCH64_UNKNOWN_LINUX_MUSL_RUSTFLAGS="-Clink-self-contained=yes -Clinker=rust-lld"
ENV CARGO_TARGET_X86_64_UNKNOWN_LINUX_GNU_LINKER=x86_64-linux-gnu-gcc
ENV CARGO_TARGET_AARCH64_UNKNOWN_LINUX_GNU_LINKER=aarch64-linux-gnu-gcc
ENV BINDGEN_EXTRA_CLANG_ARGS_x86_64_unknown_linux_gnu='--sysroot=/usr/x86_64-linux-gnu'
ENV BINDGEN_EXTRA_CLANG_ARGS_aarch64_unknown_linux_gnu='--sysroot=/usr/aarch64-linux-gnu'
RUN cargo install cargo-chef@0.1.59 cargo-license@0.5.1
WORKDIR /restate

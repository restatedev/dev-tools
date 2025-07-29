FROM ghcr.io/rust-cross/rust-musl-cross:x86_64-musl AS musl_x86_64
FROM ghcr.io/rust-cross/rust-musl-cross:aarch64-musl AS musl_aarch64

FROM rust:1.88.0-slim-bookworm AS build-base

# Prepopulate cargo index and install dependencies
RUN cargo search --limit=1 && \
    apt update && apt upgrade -y && \
    apt -y install --no-install-recommends \
    g++-x86-64-linux-gnu \
    g++-aarch64-linux-gnu \
    protobuf-compiler \
    libprotoc-dev \
    clang \
    cmake \
    llvm \
    liburing-dev \
    ca-certificates \
    build-essential \
    git \
    curl && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Install rustup components and global tools in a single layer
RUN rustup target add \
    x86_64-unknown-linux-musl \
    aarch64-unknown-linux-musl \
    x86_64-unknown-linux-gnu \
    aarch64-unknown-linux-gnu && \
    rustup component add clippy rustfmt && \
    cargo install cargo-chef@0.1.72 cargo-license@0.6.1 cargo-hakari@0.9.36

# Install `just`
RUN curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /usr/local/bin

# Install `sccache`, `buf`, and `parca-debuginfo` in a single layer
ARG TARGETARCH
RUN arch=$(echo "$TARGETARCH" | sed s/arm64/aarch64/ | sed s/amd64/x86_64/) && \
    curl -sSLf https://github.com/mozilla/sccache/releases/download/v0.10.0/sccache-v0.10.0-${arch}-unknown-linux-musl.tar.gz -o sccache.tar.gz && \
    tar -xvf sccache.tar.gz && \
    rm sccache.tar.gz && \
    cp sccache-v0.10.0-${arch}-unknown-linux-musl/sccache /usr/bin/sccache && \
    rm -rf sccache-v0.10.0-${arch}-unknown-linux-musl && \
    curl -sSLf "https://github.com/bufbuild/buf/releases/download/v1.55.1/buf-Linux-${arch}" -o "/usr/bin/buf" && \
    chmod +x "/usr/bin/buf" && \
    curl -sSLf https://github.com/parca-dev/parca-debuginfo/releases/download/v0.12.2/parca-debuginfo_0.12.2_Linux_${arch} -o parca-debuginfo && \
    mv parca-debuginfo /usr/bin/parca-debuginfo

# Copy musl from the prebuilt x86_64 and aarch64 images
COPY --from=musl_x86_64 "/usr/local/musl" /usr/local/musl-x86_64/
COPY --from=musl_aarch64 "/usr/local/musl" /usr/local/musl-aarch64/

# Set environment variables for cross-compilation
ENV PATH="${PATH}:/usr/local/musl-x86_64/bin:/usr/local/musl-aarch64/bin"
ENV SCCACHE_DIR=/usr/local/sccache
ENV RUSTC_WRAPPER="sccache"

# linker script forcing static compilation of libstdc++
RUN echo 'GROUP ( libstdc++.a AS_NEEDED( -lgcc -lc -lm ) )' > $(readlink -f $(x86_64-unknown-linux-musl-g++ --print-file-name libstdc++.so))
RUN echo 'GROUP ( libstdc++.a AS_NEEDED( -lgcc -lc -lm ) )' > $(readlink -f $(aarch64-unknown-linux-musl-g++ --print-file-name libstdc++.so))

# x86_64 musl
ENV CC_x86_64_unknown_linux_musl="sccache x86_64-unknown-linux-musl-gcc"
ENV CXX_x86_64_unknown_linux_musl="sccache x86_64-unknown-linux-musl-g++"
ENV AR_x86_64_unknown_linux_musl="x86_64-unknown-linux-musl-ar"
ENV CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_LINKER="x86_64-unknown-linux-musl-gcc"
ENV BINDGEN_EXTRA_CLANG_ARGS_x86_64_unknown_linux_musl="--sysroot=/usr/x86_64-linux-musl"

# aarch64 musl
ENV CC_aarch64_unknown_linux_musl="sccache aarch64-unknown-linux-musl-gcc"
ENV CXX_aarch64_unknown_linux_musl="sccache aarch64-unknown-linux-musl-g++"
ENV AR_aarch64_unknown_linux_musl="aarch64-unknown-linux-musl-ar"
ENV CARGO_TARGET_AARCH64_UNKNOWN_LINUX_MUSL_LINKER="aarch64-unknown-linux-musl-gcc"
ENV BINDGEN_EXTRA_CLANG_ARGS_aarch64_unknown_linux_musl="--sysroot=/usr/aarch64-linux-musl"

# x86_64 gnu
ENV CC_x86_64_unknown_linux_gnu="sccache x86_64-linux-gnu-gcc"
ENV CXX_x86_64_unknown_linux_gnu="sccache x86_64-linux-gnu-g++"
ENV AR_x86_64_unknown_linux_gnu="x86_64-linux-gnu-ar"
ENV CARGO_TARGET_X86_64_UNKNOWN_LINUX_GNU_LINKER="x86_64-linux-gnu-gcc"
ENV BINDGEN_EXTRA_CLANG_ARGS_x86_64_unknown_linux_gnu="--sysroot=/usr/x86_64-linux-gnu"

# aarch64 gnu
ENV CC_aarch64_unknown_linux_gnu="sccache aarch64-linux-gnu-gcc"
ENV CXX_aarch64_unknown_linux_gnu="sccache aarch64-linux-gnu-g++"
ENV AR_aarch64_unknown_linux_gnu="aarch64-linux-gnu-ar"
ENV CARGO_TARGET_AARCH64_UNKNOWN_LINUX_GNU_LINKER="aarch64-linux-gnu-gcc"
ENV BINDGEN_EXTRA_CLANG_ARGS_aarch64_unknown_linux_gnu="--sysroot=/usr/aarch64-linux-gnu"

# Set up working directory
WORKDIR /restate

# Make git work if different owner runs commands
RUN git config --global --add safe.directory /restate

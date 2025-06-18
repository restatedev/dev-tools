FROM ghcr.io/rust-cross/rust-musl-cross:aarch64-musl AS aarch64-musl

FROM ghcr.io/rust-cross/rust-musl-cross:x86_64-musl AS x86_64-musl

FROM rust:1.87.0-bookworm
# prepopulate cargo index
RUN cargo search --limit=1
RUN apt update && apt upgrade -y
RUN apt -y install \
    g++-x86-64-linux-gnu \
    g++-aarch64-linux-gnu \
    protobuf-compiler \
    libprotoc-dev \
    clang \
    cmake \
    llvm \
    liburing-dev \
    ca-certificates
RUN curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /usr/local/bin
RUN rustup target add \
    x86_64-unknown-linux-musl \
    aarch64-unknown-linux-musl \
    x86_64-unknown-linux-gnu \
    aarch64-unknown-linux-gnu
RUN rustup component add clippy rustfmt
RUN cargo install cargo-chef@0.1.67 cargo-license@0.6.1

COPY --from=x86_64-musl "/usr/local/musl" /usr/local/musl-x86_64/
COPY --from=aarch64-musl "/usr/local/musl" /usr/local/musl-aarch64/
ENV PATH="${PATH}:/usr/local/musl-x86_64/bin:/usr/local/musl-aarch64/bin"

# linker script forcing static compilation of libstdc++
RUN echo 'GROUP ( libstdc++.a AS_NEEDED( -lgcc -lc -lm ) )' > $(readlink -f $(x86_64-unknown-linux-musl-g++ --print-file-name libstdc++.so))
RUN echo 'GROUP ( libstdc++.a AS_NEEDED( -lgcc -lc -lm ) )' > $(readlink -f $(aarch64-unknown-linux-musl-g++ --print-file-name libstdc++.so))

ARG TARGETARCH
RUN arch=$(echo "$TARGETARCH" | sed s/arm64/aarch64/ | sed s/amd64/x86_64/) && \
    curl -LSfs https://github.com/mozilla/sccache/releases/download/v0.7.4/sccache-v0.7.4-${arch}-unknown-linux-musl.tar.gz -o sccache.tar.gz && \
    tar -xvf sccache.tar.gz && \
    rm sccache.tar.gz && \
    cp sccache-v0.7.4-${arch}-unknown-linux-musl/sccache /usr/bin/sccache && \
    rm -rf sccache-v0.7.4-${arch}-unknown-linux-musl

ENV SCCACHE_DIR=/usr/local/sccache

RUN arch=$(echo "$TARGETARCH" | sed s/arm64/aarch64/ | sed s/amd64/x86_64/) && \
    curl -sSL "https://github.com/bufbuild/buf/releases/download/v1.29.0/buf-Linux-${arch}" -o "/usr/bin/buf" && \
    chmod +x "/usr/bin/buf"

ENV CC_x86_64_unknown_linux_musl="sccache x86_64-unknown-linux-musl-gcc"
ENV CXX_x86_64_unknown_linux_musl="sccache x86_64-unknown-linux-musl-g++"
ENV AR_x86_64_unknown_linux_musl="x86_64-unknown-linux-musl-ar"
ENV CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_LINKER="x86_64-unknown-linux-musl-gcc"
ENV BINDGEN_EXTRA_CLANG_ARGS_x86_64_unknown_linux_musl="--sysroot=/usr/x86_64-linux-musl"

ENV CC_aarch64_unknown_linux_musl="sccache aarch64-unknown-linux-musl-gcc"
ENV CXX_aarch64_unknown_linux_musl="sccache aarch64-unknown-linux-musl-g++"
ENV AR_aarch64_unknown_linux_musl="aarch64-unknown-linux-musl-ar"
ENV CARGO_TARGET_AARCH64_UNKNOWN_LINUX_MUSL_LINKER="aarch64-unknown-linux-musl-gcc"
ENV BINDGEN_EXTRA_CLANG_ARGS_aarch64_unknown_linux_musl="--sysroot=/usr/aarch64-linux-musl"

ENV CC_x86_64_unknown_linux_gnu="sccache x86_64-linux-gnu-gcc"
ENV CXX_x86_64_unknown_linux_gnu="sccache x86_64-linux-gnu-g++"
ENV AR_x86_64_unknown_linux_gnu="x86_64-linux-gnu-ar"
ENV CARGO_TARGET_X86_64_UNKNOWN_LINUX_GNU_LINKER="x86_64-linux-gnu-gcc"
ENV BINDGEN_EXTRA_CLANG_ARGS_x86_64_unknown_linux_gnu="--sysroot=/usr/x86_64-linux-gnu"

ENV CC_aarch64_unknown_linux_gnu="sccache aarch64-linux-gnu-gcc"
ENV CXX_aarch64_unknown_linux_gnu="sccache aarch64-linux-gnu-g++"
ENV AR_aarch64_unknown_linux_gnu="aarch64-linux-gnu-ar"
ENV CARGO_TARGET_AARCH64_UNKNOWN_LINUX_GNU_LINKER="aarch64-linux-gnu-gcc"
ENV BINDGEN_EXTRA_CLANG_ARGS_aarch64_unknown_linux_gnu="--sysroot=/usr/aarch64-linux-gnu"

ENV RUSTC_WRAPPER="sccache"

WORKDIR /restate
# Make git work if different owner runs commands
RUN git config --global --add safe.directory /restate

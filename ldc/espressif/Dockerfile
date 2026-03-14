ARG IMAGE=alpine
ARG BASE_VERSION=latest
ARG LLVM_VERSION="release/esp_21.x"
ARG LDC_VERSION="master"

# Build llvm-xtensa
FROM ${IMAGE}:${BASE_VERSION} AS builder-llvm-xtensa
ARG LLVM_VERSION

RUN apk update && apk add --no-cache \
        git \
        gcc \
        g++ \
        binutils \
        cmake \
        make \
        ninja \
        autoconf \
        python3 \
        bison \
        flex-dev \
        llvm-dev \
        python3-dev \
        libxml2-static \
        zstd-static \
        zlib-static

WORKDIR /build
RUN git clone \
        --recurse-submodules \
        --depth 1 \
        --branch "${LLVM_VERSION}" \
        "https://github.com/espressif/llvm-project.git" \
        llvm-source

RUN cd llvm-source \
    && cmake -B build \
    -S llvm \
    -GNinja \
    -DCMAKE_INSTALL_PREFIX=/opt/llvm-xtensa \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLVM_DEFAULT_TARGET_TRIPLE=x86_64-alpine-linux-musl \
    -DLLVM_HOST_TRIPLE=x86_64-alpine-linux-musl \
    -DLLVM_ENABLE_BINDINGS=OFF \
    -DLLVM_ENABLE_LIBEDIT=OFF \
    -DLLVM_ENABLE_LIBPFM=OFF \
    -DLLVM_ENABLE_LIBXML2=OFF \
    -DLLVM_ENABLE_OCAMLDOC=OFF \
    -DLLVM_ENABLE_PLUGINS=OFF \
    -DLLVM_ENABLE_PROJECTS="lld;clang" \
    -DLLVM_ENABLE_Z3_SOLVER=OFF \
    -DLLVM_ENABLE_ZSTD=OFF \
    -DLLVM_INCLUDE_UTILS=OFF \
    -DLLVM_INCLUDE_TESTS=OFF \
    -DLLVM_INCLUDE_EXAMPLES=OFF \
    -DLLVM_INCLUDE_BENCHMARKS=OFF \
    -DLLVM_INCLUDE_DOCS=OFF \
    -DLLVM_TOOL_LLVM_LTO2_BUILD=OFF \
    -DLLVM_TOOL_LLVM_LTO_BUILD=OFF \
    -DLLVM_TOOL_LTO_BUILD=OFF \
    -DLLVM_TOOL_REMARKS_SHLIB_BUILD=OFF \
    -DCLANG_BUILD_TOOLS=OFF \
    -DCLANG_INCLUDE_DOCS=OFF \
    -DCLANG_INCLUDE_TESTS=OFF \
    -DCLANG_TOOL_CLANG_IMPORT_TEST_BUILD=OFF \
    -DCLANG_TOOL_CLANG_LINKER_WRAPPER_BUILD=OFF \
    -DCLANG_TOOL_C_INDEX_TEST_BUILD=OFF \
    -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=Xtensa \
    -DCLANG_TOOL_LIBCLANG_BUILD=OFF \
    && cmake --build build --parallel \
    && ninja -C build install


# Build ldc-xtensa using system ldc for bootstrap
FROM ${IMAGE}:${BASE_VERSION} AS builder-ldc-xtensa
ARG LDC_VERSION

COPY --from=builder-llvm-xtensa /opt/llvm-xtensa /opt/llvm-xtensa
ENV PATH="/opt/llvm-xtensa/bin:${PATH}"

RUN apk update && apk add --no-cache \
        xz \
        tar \
        git \
        clang \
        lld \
        cmake \
        make \
        ninja-is-really-ninja \
        autoconf \
        libxml2-static \
        zstd-static \
        zlib-static \
        compiler-rt \
        curl

WORKDIR /build
RUN curl -sLO --fail https://github.com/ldc-developers/ldc/releases/download/v1.42.0/ldc2-1.42.0-alpine-$(uname -m).tar.xz \
    && tar -xJf ldc2-1.42.0-alpine-$(uname -m).tar.xz --directory /opt \
    && ln -s /opt/ldc2-1.42.0-alpine-$(uname -m)/bin/ldc2 /usr/bin/ldc2 \
    && ln -s /opt/ldc2-1.42.0-alpine-$(uname -m)/bin/ldmd2 /usr/bin/ldmd2 \
    && ln -s /opt/ldc2-1.42.0-alpine-$(uname -m)/bin/dub /usr/bin/dub \
    && ln -s /opt/ldc2-1.42.0-alpine-$(uname -m)/bin/ldc-build-plugin /usr/bin/ldc-build-plugin \
    && ln -s /opt/ldc2-1.42.0-alpine-$(uname -m)/bin/ldc-build-runtime /usr/bin/ldc-build-runtime

RUN git clone \
        --recurse-submodules \
        --depth 1 \
        --branch "${LDC_VERSION}" \
        "https://github.com/ldc-developers/ldc.git" \
        ldc-source

RUN cd ldc-source \
    && cmake -B build \
    -GNinja \
    -DCMAKE_INSTALL_PREFIX=/opt/ldc-xtensa \
    -DCMAKE_C_COMPILER=clang \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DCMAKE_BUILD_TYPE=Release \
    -DD_COMPILER=ldmd2 \
    -DLLVM_ROOT_DIR=/opt/llvm-xtensa \
    -DLDC_FULLY_STATIC=ON \
    -DLDC_INSTALL_LTOPLUGIN=OFF \
    -DCOMPILER_RT_LIBDIR_OS=linux \
    -DRT_SUPPORT_SANITIZERS=ON \
    -DD_COMPILER_FLAGS="-O -link-defaultlib-shared=false" \
    -DLDC_ENABLE_PLUGINS=OFF \
    -DBUILD_SHARED_LIBS=OFF \
    -DLDC_BUNDLE_LLVM_TOOLS=OFF \
    -DLDC_INSTALL_LLVM_RUNTIME_LIBS_ARCH=$(uname -m) \
    -DCMAKE_EXE_LINKER_FLAGS=-static-libstdc++ \
    -DJITRT_EXTRA_LDFLAGS=-static-libstdc++ \
    -DLLVM_IS_SHARED=OFF \
    -DLDC_DYNAMIC_COMPILE=OFF \
    -DBUILD_LTO_LIBS=OFF \
    && cmake --build build --parallel \
    && ninja -C build install


# Build D tools
FROM ${IMAGE}:${BASE_VERSION} AS builder-tools
ARG LDC_VERSION

COPY --from=builder-llvm-xtensa /opt/llvm-xtensa /opt/llvm-xtensa
COPY --from=builder-ldc-xtensa /opt/ldc-xtensa /opt/ldc-xtensa

ENV PATH="/opt/llvm-xtensa/bin:/opt/ldc-xtensa/bin:${PATH}"

RUN apk update && apk add --no-cache \
        git \
        gcc \
        g++ \
        dub \
        llvm-libunwind-dev

# Build dlang tools
RUN git clone --recursive https://github.com/dlang/tools -b stable --depth=1 dlang-tools \
    && cd dlang-tools \
    && mkdir bin \
    && /opt/ldc-xtensa/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 rdmd.d -of=bin/rdmd \
    && /opt/ldc-xtensa/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 ddemangle.d -of=bin/ddemangle \
    && /opt/ldc-xtensa/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 DustMite/dustmite.d DustMite/splitter.d DustMite/polyhash.d -of=bin/dustmite \
    && cp bin/* /opt/ldc-xtensa/bin \
    && cd .. \
    && rm -rf dlang-tools

# Build dub
RUN git clone --recursive https://github.com/dlang/dub --depth=1 \
    && cd dub \
    && dub build -b release --compiler=/opt/ldc-xtensa/bin/ldc2 \
    && chmod +x bin/dub \
    && mv bin/dub /opt/ldc-xtensa/bin/dub \
    && cd .. \
    && rm -rf dub


# Trim llvm directory
FROM builder-llvm-xtensa AS builder-llvm-xtensa-trimmed
RUN cd /opt/llvm-xtensa && \
    rm -rf \
        include libexec share \
        bin/c-index-test bin/llvm-exegesis \
        lib/*.a \
        lib/cmake lib/libear lib/libscanbuild 2>/dev/null || true


# Final stage with ESP-IDF
FROM ${IMAGE}:${BASE_VERSION}
ARG ESP_IDF_VERSION

# COPY --from=builder-llvm-xtensa-trimmed /opt/llvm-xtensa /opt/llvm-xtensa
# COPY --from=builder-ldc-xtensa /opt/ldc-xtensa /opt/ldc-xtensa
COPY --from=builder-tools /opt/ldc-xtensa /opt/ldc-xtensa

RUN apk update && apk add --no-cache \
        sudo

RUN ln -s /opt/ldc-xtensa/bin/ldc2 /usr/bin/ldc2 \
    && ln -s /opt/ldc-xtensa/bin/ldmd2 /usr/bin/ldmd2 \
    && ln -s /opt/ldc-xtensa/bin/ldc-build-plugin /usr/bin/ldc-build-plugin \
    && ln -s /opt/ldc-xtensa/bin/ldc-build-runtime /usr/bin/ldc-build-runtime

    # && ln -s /opt/ldc-xtensa/bin/dub /usr/bin/dub \
    # && ln -s /opt/ldc-xtensa/bin/rdmd /usr/bin/rdmd \
    # && ln -s /opt/ldc-xtensa/bin/ddemangle /usr/bin/ddemangle \
    # && ln -s /opt/ldc-xtensa/bin/dustmite /usr/bin/dustmite

ENV PATH="/opt/llvm-xtensa/bin:/opt/ldc-xtensa/bin:${PATH}" \
    DC=ldc2

WORKDIR /app

ARG UNAME=ldc
ARG UID=1000
ARG GID=1000
RUN addgroup -S $UNAME \
    && adduser \
    --disabled-password \
    --gecos "" \
    --home "$(pwd)" \
    --ingroup "$UNAME" \
    --no-create-home \
    --uid "$UID" \
    "$UNAME" \
    && echo "$UNAME ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$UNAME \
    && chmod 0440 /etc/sudoers.d/$UNAME \
    && chown -R $UNAME:$UNAME /app

USER $UNAME

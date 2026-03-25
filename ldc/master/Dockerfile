ARG IMAGE=alpine
ARG BASE_VERSION=latest

FROM ${IMAGE}:${BASE_VERSION} AS builder

RUN apk update && apk add --no-cache \
        xz \
        tar \
        git \
        clang \
        lld \
        cmake \
        ninja \
        libxml2-static \
        zstd-static \
        zlib-static \
        llvm-dev \
        llvm-static \
        compiler-rt \
        curl

RUN apkArch="$(apk --print-arch)"; \
    case "$apkArch" in \
        x86_64|amd64) \
            LDC_URL="https://github.com/ldc-developers/ldc/releases/download/v1.42.0/ldc2-1.42.0-alpine-x86_64.tar.xz"; \
            LDC_DIR="ldc2-1.42.0-alpine-x86_64"; \
            ;; \
        arm64|aarch64) \
            LDC_URL="https://github.com/ldc-developers/ldc/releases/download/v1.42.0/ldc2-1.42.0-alpine-aarch64.tar.xz"; \
            LDC_DIR="ldc2-1.42.0-alpine-aarch64"; \
            ;; \
        riscv64) \
            LDC_URL="https://archive.org/download/ldc2-riscv64-linux-musl.tar/ldc2-riscv64-linux-musl.tar.xz"; \
            LDC_DIR="ldc2-riscv64-linux-musl"; \
            ;; \
        *) echo >&2 "Unsupported architecture: $apkArch"; exit 1 ;; \
    esac; \
    \
    curl -sLO --fail "$LDC_URL" && \
    tar xf "$(basename $LDC_URL)" --directory /opt && \
    ln -s /opt/$LDC_DIR/bin/ldc2          /usr/bin/ldc2 && \
    ln -s /opt/$LDC_DIR/bin/ldmd2         /usr/bin/ldmd2 && \
    ln -s /opt/$LDC_DIR/bin/dub           /usr/bin/dub && \
    ln -s /opt/$LDC_DIR/bin/ldc-build-plugin /usr/bin/ldc-build-plugin && \
    ln -s /opt/$LDC_DIR/bin/ldc-build-runtime /usr/bin/ldc-build-runtime

RUN git clone --recursive https://github.com/ldc-developers/ldc --depth=1
RUN ln -sf /usr/bin/ld.lld /usr/$(uname -m)-alpine-linux-musl/bin/ld
RUN if [ "$(apk --print-arch)" = "riscv64" ]; then \
        cp /opt/ldc2-riscv64-linux-musl/include/d/core/stdc/fenv.d \
           ldc/runtime/druntime/src/core/stdc/fenv.d; \
    fi
RUN cd ldc && \
    if [ "$(apk --print-arch)" = "riscv64" ]; then \
        cmake -B build \
            -DCMAKE_INSTALL_PREFIX="/opt/ldc-master" \
            -GNinja \
            -DCMAKE_C_COMPILER=clang \
            -DCMAKE_CXX_COMPILER=clang++ \
            -DCMAKE_BUILD_TYPE=Release \
            -DLDC_FULLY_STATIC=ON \
            -DLDC_INSTALL_LTOPLUGIN=OFF \
            -DCOMPILER_RT_LIBDIR_OS=linux \
            -DRT_SUPPORT_SANITIZERS=ON \
            -DLDC_ENABLE_PLUGINS=OFF \
            -DBUILD_SHARED_LIBS=OFF \
            -DLDC_BUNDLE_LLVM_TOOLS=OFF \
            -DLDC_INSTALL_LLVM_RUNTIME_LIBS_ARCH=$(uname -m) \
            -DCMAKE_EXE_LINKER_FLAGS=-static-libstdc++ \
            -DJITRT_EXTRA_LDFLAGS=-static-libstdc++ \
            -DLLVM_IS_SHARED=OFF \
            -DLDC_DYNAMIC_COMPILE=OFF \
            -DD_COMPILER_FLAGS=-O \
            -DBUILD_LTO_LIBS=OFF; \
    else \
        cmake -B build \
            -DCMAKE_INSTALL_PREFIX="/opt/ldc-master" \
            -GNinja \
            -DCMAKE_C_COMPILER=clang \
            -DCMAKE_CXX_COMPILER=clang++ \
            -DCMAKE_BUILD_TYPE=Release \
            -DLDC_FULLY_STATIC=ON \
            -DLDC_INSTALL_LTOPLUGIN=OFF \
            -DCOMPILER_RT_LIBDIR_OS=linux \
            -DRT_SUPPORT_SANITIZERS=ON \
            -DLDC_ENABLE_PLUGINS=OFF \
            -DBUILD_SHARED_LIBS=OFF \
            -DLDC_BUNDLE_LLVM_TOOLS=OFF \
            -DLDC_INSTALL_LLVM_RUNTIME_LIBS_ARCH=$(uname -m) \
            -DCMAKE_EXE_LINKER_FLAGS=-static-libstdc++ \
            -DJITRT_EXTRA_LDFLAGS=-static-libstdc++ \
            -DLLVM_IS_SHARED=OFF \
            -DLDC_DYNAMIC_COMPILE=OFF \
            -DD_COMPILER_FLAGS="-O -flto=full -defaultlib=phobos2-ldc-lto,druntime-ldc-lto" \
            -DEXTRA_CXXFLAGS=-flto=full \
            -DBUILD_LTO_LIBS=ON; \
    fi && \
    cmake --build build --parallel "$(nproc)" && \
    ninja -C build install

RUN apk add llvm-libunwind-dev
RUN git clone --recursive https://github.com/dlang/tools -b stable --depth=1 dlang-tools \
    && cd dlang-tools \
    && mkdir bin \
    && /opt/ldc-master/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 rdmd.d -of=bin/rdmd \
    && /opt/ldc-master/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 ddemangle.d -of=bin/ddemangle \
    && /opt/ldc-master/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 DustMite/dustmite.d DustMite/splitter.d DustMite/polyhash.d -of=bin/dustmite \
    && cp bin/* /opt/ldc-master/bin \
    && cd .. \
    && rm -fr dlang-tools
RUN git clone --recursive https://github.com/dlang/dub --depth=1 \
    && cd dub \
    && dub build -b release --compiler=/opt/ldc-master/bin/ldc2 \
    && chmod +x bin/dub \
    && mv bin/dub /opt/ldc-master/bin/dub \
    && cd .. \
    && rm -fr dub ldc

FROM ${IMAGE}:${BASE_VERSION}

RUN apk update && apk add --no-cache \
        sudo \
        lld \
        gcc

COPY --from=builder /opt/ldc-master /opt/ldc-master

RUN ln -sf /usr/bin/ld.lld /usr/$(uname -m)-alpine-linux-musl/bin/ld \
    && ln -s /opt/ldc-master/bin/ldc2 /usr/bin/ldc2 \
    && ln -s /opt/ldc-master/bin/ldmd2 /usr/bin/ldmd2 \
    && ln -s /opt/ldc-master/bin/dub /usr/bin/dub \
    && ln -s /opt/ldc-master/bin/ldc-build-plugin /usr/bin/ldc-build-plugin \
    && ln -s /opt/ldc-master/bin/ldc-build-runtime /usr/bin/ldc-build-runtime

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

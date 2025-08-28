ARG IMAGE=alpine
ARG BASE_VERSION=latest

FROM ${IMAGE}:${BASE_VERSION} AS builder

RUN apk update && apk add --no-cache \
        ldc \
        git \
        clang \
        lld \
        dub \
        cmake \
        ninja \
        libxml2-static \
        zstd-static \
        zlib-static \
        llvm-dev \
        llvm-static \
        compiler-rt \
        curl \
        llvm-libunwind-static

RUN git clone --recursive https://github.com/ldc-developers/ldc --depth=1
RUN ln -sf /usr/bin/ld.lld /usr/$(uname -m)-alpine-linux-musl/bin/ld
RUN cd ldc \
    && cmake -B build \
    -DCMAKE_INSTALL_PREFIX="/opt" \
    -GNinja \
    -DCMAKE_C_COMPILER=clang \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DCMAKE_BUILD_TYPE=Release \
    -DLDC_FULLY_STATIC=ON \
    -DLDC_INSTALL_LTOPLUGIN=OFF \
    -DCOMPILER_RT_LIBDIR_OS=linux \
    -DRT_SUPPORT_SANITIZERS=ON \
    -DD_COMPILER_FLAGS="-O -flto=full -link-defaultlib-shared=false" \
    -DEXTRA_CXXFLAGS="-flto=full" \
    -DCONF_INST_DIR=/etc/ldc \
    -DLDC_ENABLE_PLUGINS=OFF \
    -DBUILD_SHARED_LIBS=OFF \
    -DLDC_BUNDLE_LLVM_TOOLS=OFF \
    -DLDC_INSTALL_LLVM_RUNTIME_LIBS_ARCH=$(uname -m) \
    -DCMAKE_EXE_LINKER_FLAGS=-static-libstdc++ \
    -DLLVM_IS_SHARED=OFF \
    -DLDC_DYNAMIC_COMPILE=OFF \
    -DBUILD_LTO_LIBS=ON \
    && cmake --build build --parallel \
    && ninja -C build install

RUN apk del ldc && apk add llvm-libunwind-dev
RUN git clone --recursive https://github.com/dlang/tools -b stable --depth=1 dlang-tools \
    && cd dlang-tools \
    && mkdir bin \
    && /opt/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 rdmd.d -of=bin/rdmd \
    && /opt/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 ddemangle.d -of=bin/ddemangle \
    && /opt/bin/ldc2 -O -static -cleanup-obj -w -de -dip1000 DustMite/dustmite.d DustMite/splitter.d DustMite/polyhash.d -of=bin/dustmite \
    && cp bin/* /opt/bin \
    && cd .. \
    && rm -fr dlang-tools
RUN git clone --recursive https://github.com/dlang/dub --depth=1 \
    && cd dub \
    && dub build -b release --compiler=/opt/bin/ldc2 \
    && chmod +x bin/dub \
    && apk del dub \
    && mv bin/dub /usr/bin/dub \
    && cd .. \
    && rm -fr dub ldc

FROM ${IMAGE}:${BASE_VERSION}

RUN apk update && apk add --no-cache \
        sudo \
        clang \
        lld \
        compiler-rt \
        llvm-libunwind-static \
        llvm-libunwind-dev \
        musl-dev

COPY --from=builder /opt /opt
COPY --from=builder /usr/bin/dub /usr/bin/dub

RUN ln -sf /usr/bin/ld.lld /usr/$(uname -m)-alpine-linux-musl/bin/ld \
    && ln -s /opt/bin/ldc2 /usr/bin/ldc2 \
    && ln -s /opt/bin/ldmd2 /usr/bin/ldmd2 \
    && ln -s /opt/bin/ldc-build-plugin /usr/bin/ldc-build-plugin \
    && ln -s /opt/bin/ldc-build-runtime /usr/bin/ldc-build-runtime

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

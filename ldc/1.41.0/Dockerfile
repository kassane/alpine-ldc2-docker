ARG IMAGE=alpine
ARG BASE_VERSION=latest

FROM ${IMAGE}:${BASE_VERSION}

ENV LDC_VERSION=1.41.0

RUN apk update && apk add --no-cache \
        sudo \
        ldc \
        git \
        clang \
        dub \
        cmake \
        ninja \
        lld \
        llvm-dev \
        libxml2-static \
        zstd-static \
        zlib-static \
        llvm-static \
        llvm-libunwind-static \
        compiler-rt \
        curl \
        curl-dev

RUN git clone --recursive https://github.com/ldc-developers/ldc -b v${LDC_VERSION} --depth=1
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
    -DCMAKE_EXE_LINKER_FLAGS=-static-libstdc++ \
    -DLDC_ENABLE_PLUGINS=OFF \
    -DBUILD_SHARED_LIBS=OFF \
    -DLDC_BUNDLE_LLVM_TOOLS=OFF \
    -DLDC_INSTALL_LLVM_RUNTIME_LIBS_ARCH=$(uname -m) \
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
    && cp $PWD/bin/* /opt/bin \
    && cd .. \
    && rm -fr dlang-tools

RUN git clone --recursive https://github.com/dlang/dub --depth=1 \
    && cd dub \
    && dub build -b release --compiler=/opt/bin/ldc2 \
    && chmod +x $PWD/bin/dub \
    && apk del dub \
    && mv $PWD/bin/dub /usr/bin/dub

RUN apk del git llvm-dev llvm-static cmake ninja curl curl-dev libxml2-static zlib-static zstd-static \
    && ln -s /opt/bin/ldc2 /usr/bin/ldc2 \
    && ln -s /opt/bin/ldmd2 /usr/bin/ldmd2 \
    && ln -s /opt/bin/ldc-build-plugin /usr/bin/ldc-build-plugin \
    && ln -s /opt/bin/ldc-build-runtime /usr/bin/ldc-build-runtime \
    && apk add musl-dev \
    && rm -fr ldc dub


RUN apk del cmake ninja \
    && rm -fr \
    ldc-${LDC_VERSION}-src \
    *.tar.gz \
    dub-${LDC_VERSION}

WORKDIR /app

ARG UNAME=ldc
ARG UID=1000
ARG GID=1000
RUN addgroup -S $UNAME
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "$(pwd)" \
    --ingroup "$UNAME" \
    --no-create-home \
    --uid "$UID" \
    "$UNAME"
RUN echo "$UNAME ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$UNAME \
        && chmod 0440 /etc/sudoers.d/$UNAME
RUN chown -R $UNAME:$UNAME /app
USER $UNAME

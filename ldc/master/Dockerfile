ARG IMAGE=alpine
ARG BASE_VERSION=latest

FROM ${IMAGE}:${BASE_VERSION}

RUN apk update && apk add --no-cache \
        sudo \
        ldc \
        git \
        clang \
        dub \
        cmake \
        ninja \
        llvm-dev \
        libxml2-static \
        zstd-static \
        zlib-static \
        llvm-static \
        llvm-libunwind-static

RUN git clone --recursive https://github.com/ldc-developers/ldc --depth=1
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
    -DD_COMPILER_FLAGS="-O -flto=full -link-defaultlib-shared=false" \
    -DRT_SUPPORT_SANITIZERS=ON \
    -DLDC_ENABLE_PLUGINS=OFF \
    -DBUILD_SHARED_LIBS=BOTH \
    -DLDC_INSTALL_LLVM_RUNTIME_LIBS_ARCH=$(uname -m) \
    -DCMAKE_EXE_LINKER_FLAGS=-static-libstdc++ \
    -DLLVM_IS_SHARED=OFF \
    -DLDC_DYNAMIC_COMPILE=OFF \
    -DBUILD_LTO_LIBS=ON \
    && cmake --build build --parallel \
    && ninja -C build install

RUN git clone --recursive https://github.com/dlang/tools --depth=1 dlang-tools \
    && cd dlang-tools \
    && mkdir bin \
    && /opt/bin/ldc2 -O -static -w -de -dip1000 rdmd.d -of=bin/rdmd \
    && /opt/bin/ldc2 -O -static -w -de -dip1000 ddemangle.d -of=bin/ddemangle \
    && /opt/bin/ldc2 -O -static -w -de -dip1000 DustMite/dustmite.d DustMite/splitter.d DustMite/polyhash.d -of=bin/dustmite \
    && cp $PWD/bin/* /opt/bin \
    && cd .. \
    && rm -fr dlang-tools

RUN git clone --recursive https://github.com/dlang/dub --depth=1 \
    && cd dub \
    && dub build -b release --compiler=/opt/bin/ldc2 \
    && chmod +x $PWD/bin/dub \
    && apk del dub \
    && mv $PWD/bin/dub /usr/bin/dub

RUN apk del ldc llvm-static libxml2-static zstd-static zlib-static clang cmake ninja git \
    && ln -s /opt/bin/ldc2 /usr/bin/ldc2 \
    && ln -s /opt/bin/ldmd2 /usr/bin/ldmd2 \
    && ln -s /opt/bin/ldc-build-plugin /usr/bin/ldc-build-plugin \
    && ln -s /opt/bin/ldc-build-runtime /usr/bin/ldc-build-runtime \
    && apk add llvm-libunwind-dev musl-dev gcc \
    && rm -fr ldc dub

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

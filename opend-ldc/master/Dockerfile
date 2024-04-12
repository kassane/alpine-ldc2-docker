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
        llvm-static \
        llvm-libunwind-static

RUN git clone --recursive https://github.com/opendlang/opend --depth=1
RUN cd opend/ldc \
    && cmake -B build \
    -DCMAKE_INSTALL_PREFIX="/opt" \
    -GNinja \
    -DCMAKE_C_COMPILER=clang \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DCMAKE_BUILD_TYPE=Release \
    -DRT_SUPPORT_SANITIZERS=ON \
    -DCMAKE_EXE_LINKER_FLAGS=-static-libstdc++ \
    -DLLVM_IS_SHARED=OFF \
    -DD_COMPILER_FLAGS=-link-defaultlib-shared=false \
    -DBUILD_SHARED_LIBS=BOTH \
    -DBUILD_LTO_LIBS=ON \
    && cmake --build build --parallel \
    && ninja -C build install

RUN git clone --recursive https://github.com/dlang/dub --depth=1 \
    && cd dub \
    && dub build -b release --compiler=/opt/bin/ldc2 \
    && chmod +x $PWD/bin/dub \
    && apk del dub \
    && mv $PWD/bin/dub /usr/bin/dub

RUN apk del ldc llvm-static clang cmake ninja git \
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

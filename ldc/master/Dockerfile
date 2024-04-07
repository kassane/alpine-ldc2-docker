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

RUN git clone --recursive https://github.com/kassane/ldc -b musl_fix --depth=1
RUN cd ldc \
    && cmake -B build \
    -DCMAKE_INSTALL_PREFIX="/opt" \
    -GNinja \
    -DCMAKE_C_COMPILER=clang \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_EXE_LINKER_FLAGS=-static-libstdc++ \
    -DLLVM_IS_SHARED=OFF \
    && cmake --build build --parallel

RUN ninja -C ldc/build install \
    && apk del ldc llvm-static llvm-libunwind-static clang \
    && ln -s /opt/bin/ldc2 /usr/bin/ldc2 \
    && ln -s /opt/bin/ldmd2 /usr/bin/ldmd2

# RUN git https://github.com/dlang/dub --depth=1 \
#     && cd dub \
#     && dub build -b release --compiler=/opt/bin/ldc2 \
#     && chmod +x $PWD/bin/dub \
#     && apk del dub \
#     && mv $PWD/bin/dub /usr/bin/dub

RUN apk del cmake ninja git \
    && apk add llvm-libunwind-dev musl-dev gcc \
    && rm -fr ldc

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

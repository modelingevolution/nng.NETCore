# Build NNG native shared library for: linux-x64, linux-arm, linux-arm64
#
# USAGE:
# docker run --rm --privileged multiarch/qemu-user-static:register
# docker build -t jeikabu/build-nng dockerfiles/build_nng
# docker run -i -t --rm -v "$PWD/nng.NET/runtimes:/runtimes" jeikabu/build-nng

ARG SRC=/usr/src

# Git NNG source
FROM debian:buster AS nng

RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    git

ARG SRC
ARG NNG_BRANCH=v1.4.0
ARG MBEDTLS_BRANCH=mbedtls-2.28
WORKDIR ${SRC}

RUN git clone https://github.com/nanomsg/nng.git \
    && cd nng \
    && git checkout ${NNG_BRANCH}

RUN git clone https://github.com/Mbed-TLS/mbedtls.git \
    && cd mbedtls \
    && git checkout ${MBEDTLS_BRANCH}

# Build x64 Linux
FROM python:3.8-buster AS linux-x64
    
RUN apt-get update && apt-get install -y \
    build-essential \
    clang \
    cmake \
	perl \
	&& pip install Jinja2

ARG SRC
ARG RUNTIME_NATIVE=${SRC}/runtimes/linux-x64/native

WORKDIR ${SRC}
COPY --from=nng ${SRC}/mbedtls .
RUN mkdir -p build && cd build \
    && cmake -G "Unix Makefiles" -D CMAKE_BUILD_TYPE=Release -DUSE_SHARED_MBEDTLS_LIBRARY=On -DCMAKE_INSTALL_PREFIX=${SRC}/mbedtls/install .. \
    && mkdir -p install \
	&& make -j2 \
	&& make install
	
WORKDIR ${SRC}
COPY --from=nng ${SRC}/nng .
RUN mkdir -p build && cd build \
    && cmake -G "Unix Makefiles" -DBUILD_SHARED_LIBS=ON -DNNG_ENABLE_TLS=ON -DCMAKE_BUILD_TYPE=Release -DNNG_ELIDE_DEPRECATED=ON -DNNG_TESTS=OFF -DNNG_TOOLS=OFF -DMBEDTLS_ROOT_DIR=${SRC}/mbedtls/install .. \
    && make -j2 \
    && mkdir -p ${RUNTIME_NATIVE} \
    && cp libnng.so ${RUNTIME_NATIVE}

# Build ARM32
FROM arm32v7/python:3.8-buster AS arm32v7

RUN apt-get update && apt-get install -y \
    build-essential \
    clang \
    cmake \
	perl \
	&& pip install Jinja2
	
ARG SRC
ARG RUNTIME_NATIVE=${SRC}/runtimes/linux-arm/native

WORKDIR ${SRC}
COPY --from=nng ${SRC}/mbedtls .
RUN mkdir -p build && cd build \
    && cmake -G "Unix Makefiles" -D CMAKE_BUILD_TYPE=Release -DUSE_SHARED_MBEDTLS_LIBRARY=On -DCMAKE_INSTALL_PREFIX=${SRC}/mbedtls/install .. \
    && mkdir -p install \
	&& make -j2 \
	&& make install
	
WORKDIR ${SRC}
COPY --from=nng ${SRC}/nng .
RUN mkdir -p build && cd build \
    && cmake -G "Unix Makefiles" -DBUILD_SHARED_LIBS=ON -DNNG_ENABLE_TLS=ON -DCMAKE_BUILD_TYPE=Release -DNNG_ELIDE_DEPRECATED=ON -DNNG_TESTS=OFF -DNNG_TOOLS=OFF -DMBEDTLS_ROOT_DIR=${SRC}/mbedtls/install .. \
    && make -j2 \
    && mkdir -p ${RUNTIME_NATIVE} \
    && cp libnng.so ${RUNTIME_NATIVE}

# Build ARM64
FROM arm64v8/python:3.8-buster AS arm64v8

RUN apt-get update && apt-get install -y \
    build-essential \
    clang \
    cmake \
	perl \
	&& pip install Jinja2

ARG SRC
ARG RUNTIME_NATIVE=${SRC}/runtimes/linux-arm64/native

WORKDIR ${SRC}
COPY --from=nng ${SRC}/mbedtls .
RUN mkdir -p build && cd build \
    && cmake -G "Unix Makefiles" -D CMAKE_BUILD_TYPE=Release -DUSE_SHARED_MBEDTLS_LIBRARY=On -DCMAKE_INSTALL_PREFIX=${SRC}/mbedtls/install .. \
    && mkdir -p install \
	&& make -j2 \
	&& make install
	
WORKDIR ${SRC}
COPY --from=nng ${SRC}/nng .
RUN mkdir -p build && cd build \
    && cmake -G "Unix Makefiles" -DBUILD_SHARED_LIBS=ON -DNNG_ENABLE_TLS=ON -DCMAKE_BUILD_TYPE=Release -DNNG_ELIDE_DEPRECATED=ON -DNNG_TESTS=OFF -DNNG_TOOLS=OFF -DMBEDTLS_ROOT_DIR=${SRC}/mbedtls/install .. \
    && make -j2 \
    && mkdir -p ${RUNTIME_NATIVE} \
    && cp libnng.so ${RUNTIME_NATIVE}

# Copy binaries to host
FROM debian:buster AS copy

ARG SRC
WORKDIR ${SRC}
COPY --from=linux-x64 ${SRC}/runtimes ./runtimes
COPY --from=arm32v7 ${SRC}/runtimes ./runtimes
COPY --from=arm64v8 ${SRC}/runtimes ./runtimes
CMD cp -rf ./runtimes/* /runtimes

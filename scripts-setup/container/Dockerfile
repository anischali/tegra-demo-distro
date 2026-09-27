FROM debian:trixie-slim AS builder

RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        build-essential \
        cmake \
        libssl-dev \
        libseccomp-dev \
        linux-libc-dev \
        git \
    && rm -rf /var/lib/apt/lists/*

RUN git clone --depth=1 https://github.com/SUNET/pkcs11-proxy.git /src
# We use -Wno-incompatible-pointer-types because trixy
# promotes the incompatible pointers to error so for now
# we ignore the issue but maybe need to fix it in the code
RUN cd /src && \
    cmake . \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/install \
        -DCMAKE_C_FLAGS="-Wno-incompatible-pointer-types" && \
    make -j"$(nproc)" && \
    make install

FROM ghcr.io/siemens/kas/kas:latest
USER root

RUN apt update -y
RUN apt install -y make bash-completion \
    inkscape tmux bash curl texinfo
    
RUN apt install -y gawk make wget tar bzip2 \
    gzip python3 unzip perl patch diffutils \
    diffstat git cpp gcc chrpath \
    socat python3-pexpect findutils \
    file cpio python3-pip python3-jinja2 \
    zstd lz4 hostname xz-utils s3cmd

# Add tools for image signing and packaging
# these tools are passed to yocto via hosttools
# this approach permits a to reduce yocto infrastructure
# maintenance because once a feature is configured and
# developed for one product all the others has it.
RUN apt install -y opensc openssl pkcs11-provider \
    libengine-pkcs11-openssl device-tree-compiler \
    u-boot-tools ima-evm-utils

# Add the pkcs11-proxy, Only the client library is needed — the daemon stays in the proxy container
COPY --from=builder /install/lib/libpkcs11-proxy.so /usr/lib/libpkcs11-proxy.so
COPY openssl.cnf    /etc/pkcs11/openssl.cnf
# Clean the image to reduce its size.
RUN  apt-get clean && rm -rf /var/lib/apt/lists/*

ENV OPENSSL_CONF=/etc/pkcs11/openssl.cnf
ENV PKCS11_MODULE_PATH=/usr/lib/libpkcs11-proxy.so
ENV OPENSSL_ENGINES=/usr/lib/x86_64-linux-gnu/engines-3
ENV SHELL=bash
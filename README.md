# -static

A set of repositories which build useful tools - staticly.

# Requirements

A Dockerfile and source you wish to build.

An example `Dockerfile`:

```Dockerfile
FROM alpine:3.21 AS base

RUN apk add --no-cache \
        bash \
        build-base \
        linux-headers \
        e2fsprogs-dev \
        libaio-dev \
        build-base \
        musl-dev \
        musl-utils \
        linux-headers \
        autoconf \
        automake \
        libtool \
        pkgconf \
        cmake \
        git \
        curl \
        tar \
        gzip \
        flex \
        bison \
        xz \
        file \
        perl \
        python3 \
        gtk-doc \
        gettext-dev \
        gettext-static \
        zlib-dev \
        zlib-static

ENV PREFIX=/opt/static
ENV LDFLAGS="-pie -static"
ENV CXXFLAGS="-fPIE --static"

RUN printf '#!/bin/sh\nexec gcc -static -no-pie "$@"\n' \
        > /usr/local/bin/staticgcc \
    && chmod +x /usr/local/bin/staticgcc

ENV CC=/usr/local/bin/staticgcc

# =============================================================================
# build busybox 
# =============================================================================

FROM base AS build-busybox

COPY ./busybox_mirror /src

COPY .config /src

RUN cd /src && make -j `nproc`

RUN ldd "/src/busybox" && echo "failed to build" || true

# =============================================================================
# collect the artifacts for output
#
# usage: docker build --target dashstatic --output type=local,dest=./dashstatic .
#
# =============================================================================
FROM scratch AS dashstatic

COPY --from=build-busybox  /src/busybox /busybox
```

Once the above exists, something like so will produce assets in the folder `./dashstatic`

```bash
BUILDKIT_PROGRESS=plain docker build --target dashstatic --output type=local,dest=./dashstatic
```

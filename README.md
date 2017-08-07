# Cross build files for multiple architectures

## Intro

This image builds files for multiple architectures that can be referenced in multi-stage
builds to enable cross build capable images following the principles outlined in
https://resin.io/blog/building-arm-containers-on-any-x86-machine-even-dockerhub/.  The image
is not intended to be used to run anything directly but is designed as a repository to
"COPY --from" the qemu and cross build binaries.

## Using the container

There are files for arm32 and arm64 architectures in /cross-build/< arm32 | arm64 >/usr/bin.  Dummy
files for x86_64 are also included so that parameterized Dockerfiles can be built
that are identical for all architectures.  The following example demonstrates:

```
ARG ARCH_DIR=x86_64
ARG ARCH_IMG=docker.io/alpine:latest

FROM docker.io/monsonnl/qemu-wrap-build-files as build_files

FROM ${ARCH_IMG}
COPY --from=build_files /cross-build/${ARCH_DIR}/usr/bin /usr/bin
RUN [ "cross-build-start" ]
RUN echo "Hello from " `uname -m`
RUN [ "cross-build-end" ]
```
Now you can build the above hello world image for each of the architectures as:

```
# for x86_64
docker build Dockerfile .
# or
docker build --build-arg ARCH_DIR=x86_64 --build-arg ARCH_IMG=docker.io/alpine:latest Dockerfile .

# for arm32
docker build --build-arg ARCH_DIR=arm32 --build-arg ARCH_IMG=docker.io/arm32v7/alpine:latest Dockerfile .

# for arm64
docker build --build-arg ARCH_DIR=arm64 --build-arg ARCH_IMG=docker.io/arm64v8/alpine:latest Dockerfile .
```

## Reference

[1] Foundational article - Build ARM container on x86: https://resin.io/blog/building-arm-containers-on-any-x86-machine-even-dockerhub/

[2] Resin.io project that originated the cross build go scripts: https://github.com/resin-io-projects/armv7hf-debian-qemu

[3] Qemu building based on build script in: https://github.com/KurtStam/add-aarch64-qemu

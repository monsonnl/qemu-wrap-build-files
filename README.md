# Cross-build files for arm architectures

This image builds files for arm32 and arm64 architectures that can be referenced
to enable cross build capable images following the principles outlined in
https://resin.io/blog/building-arm-containers-on-any-x86-machine-even-dockerhub/.  
The image is not intended to be used to run anything directly but is designed as
a source in  multi-stage builds to "COPY --from" the qemu and cross build binaries.

## Using the container

There are files for arm32 and arm64 architectures in /cross-build/< arm32 | arm64 >/usr/bin.  Dummy
files for x86_64 are also included so that parameterized Dockerfiles can be built
that are identical for all architectures.  The following example demonstrates:

```
ARG ARCH_DIR=x86_64
ARG ARCH_IMG=debian:stretch

FROM monsonnl/qemu-wrap-build-files as build_files

ARG ARCH_IMG

FROM ${ARCH_IMG}

ARG ARCH_DIR

COPY --from=build_files /cross-build/${ARCH_DIR}/usr/bin /usr/bin
RUN [ "cross-build-start" ]

RUN echo "Hello from" `uname -m`

RUN [ "cross-build-end" ]
```
Now you can build the above hello world image for each of the architectures as:

```
# for x86_64
docker build Dockerfile .
# or
docker build --build-arg ARCH_DIR=x86_64 --build-arg ARCH_IMG=docker.io/debian:stretch .

# for arm32
docker build --build-arg ARCH_DIR=arm32 --build-arg ARCH_IMG=docker.io/arm32v7/debian:stretch .

# for arm64
docker build --build-arg ARCH_DIR=arm64 --build-arg ARCH_IMG=docker.io/arm64v8/debian:stretch .
```

## Extending to additional architectures

The cross-build go file and qemu static binary are built in individual stages in the multi-stage
Dockerfile and then each architecture's files are combined into a final directory tree in the complete
image.  Additional target architectures should be able to be added simply by adding an additional
qemu build stage following the current examples and adding to the final image.  Similarly this should
also work as a basis to build an image for the necessary binaries to build cross-builds on different
host architectures if desired though these haven't been tried.


## Reference

[1] Foundational article - Build ARM container on x86: https://resin.io/blog/building-arm-containers-on-any-x86-machine-even-dockerhub/

[2] Resin.io project that originated the cross build go scripts: https://github.com/resin-io-projects/armv7hf-debian-qemu

[3] Qemu building based on build script in: https://github.com/KurtStam/add-aarch64-qemu

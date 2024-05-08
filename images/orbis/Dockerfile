# BASE STAGE: Minimal install for what is required for the SDK to run
FROM ubuntu:20.04 AS base

ENV DEBIAN_FRONTEND=noninteractive

# Install needed applications for running the SDK
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      git \
      build-essential=12.8ubuntu1 \
      clang=1:10.0-50~exp1 \
      libicu66=66.1-2ubuntu2 \
      libssl1.1 \
      lld=1:10.0-50~exp1 && \
    rm -rf /var/lib/apt/lists/*

# SETUP STAGE: Minimal install for what is required to download/setup the SDK
FROM ubuntu:20.04 as setup

# Install needed applications for downloading/setting up the SDK
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    tar \
    unzip && \
    rm -rf /var/lib/apt/lists/*

# Set the OO_PS4_TOOLCHAIN environmental variable for later use vs using copy/paste
ENV OO_PS4_TOOLCHAIN=/lib/OpenOrbisSDK

# Built by https://github.com/OpenOrbis/OpenOrbis-PS4-Toolchain/actions/runs/8022770949
# https://github.com/OpenOrbis/OpenOrbis-PS4-Toolchain/blob/cd480230b946239f758333ef0dff40d67c5f0c20/.github/workflows/toolchain.yml
# Using an asset on my local machine is not ideal, but setting up a build
# pipeline is expensive given there's no way to iterate quickly on a github
# workflow which publishs a docker image (no local developement).
ADD toolchain-llvm-10.zip /tmp/toolchain-llvm-10.zip

WORKDIR /tmp

RUN unzip toolchain-llvm-10.zip
RUN mkdir -p $OO_PS4_TOOLCHAIN
RUN tar -xz -C $OO_PS4_TOOLCHAIN -f toolchain-llvm-10.tar.gz

FROM golang as create_fself

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    patch \
    unzip && \
    rm -rf /var/lib/apt/lists/*

ADD https://codeload.github.com/OpenOrbis/create-fself/zip/3dce1170125bf93ebca2b19236691359f8753d2f /tmp/create-fself.zip

WORKDIR /tmp
RUN mkdir -p /workspace && unzip create-fself.zip && mv /tmp/create-fself*/* /workspace

ADD create-fself.diff /tmp/patch.diff
WORKDIR /workspace
RUN patch --silent --strip=1 --force < /tmp/patch.diff

WORKDIR /workspace/cmd/create-fself
RUN go build -o /workspace/out/create-fself -modfile=go-linux.mod

# RUNTIME STAGE: The final stage where the magic happens
FROM base as runtime

# Set the environmental variables for the SDK location
ENV OO_PS4_TOOLCHAIN=/lib/OpenOrbisSDK
ENV PATH=$OO_PS4_TOOLCHAIN:$OO_PS4_TOOLCHAIN/bin/linux:$PATH

# Copy the SDK from the setup stage to this stage
COPY --from=setup ${OO_PS4_TOOLCHAIN}/OpenOrbis/PS4Toolchain ${OO_PS4_TOOLCHAIN}
COPY --from=create_fself /workspace/out/create-fself ${OO_PS4_TOOLCHAIN}/bin/linux/create-fself

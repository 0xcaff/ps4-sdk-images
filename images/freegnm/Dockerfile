ARG BASE_IMAGE_TAG
FROM ghcr.io/0xcaff/ps4-sdk-images/orbis:${BASE_IMAGE_TAG} as builder_base

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates \
      wget \
      gnupg \
      libgl1-mesa-dev \
      unzip && \
    rm -rf /var/lib/apt/lists/*

RUN wget -qO - https://packages.lunarg.com/lunarg-signing-key-pub.asc | apt-key add -
RUN wget -qO /etc/apt/sources.list.d/lunarg-vulkan-focal.list http://packages.lunarg.com/vulkan/lunarg-vulkan-focal.list

RUN apt-get update && \
    apt-get install -y --no-install-recommends vulkan-sdk && \
    rm -rf /var/lib/apt/lists/*

FROM builder_base as builder_freegnm

ADD https://github.com/recp/cglm/archive/refs/tags/v0.9.2.zip /tmp/cglm.zip
RUN cd /tmp && unzip cglm.zip && cp -r cglm-0.9.2/include/cglm /usr/include/cglm

# Version 38254450858db2e00aba2a00c63d0e415b7dcf3c
ADD https://gitgud.io/gluesniffer/freegnm/-/archive/38254450858db2e00aba2a00c63d0e415b7dcf3c/freegnm-38254450858db2e00aba2a00c63d0e415b7dcf3c.zip /tmp/freegnm.zip
RUN cd /tmp && unzip freegnm.zip && mv freegnm-*/ /workspace

ADD freegnm.diff /tmp/patch.diff

WORKDIR /workspace
RUN patch --silent --strip=1 --force < /tmp/patch.diff

FROM builder_freegnm as builder_freegnm_library

RUN mv config.orbis.mak config.mak
RUN make

FROM builder_freegnm as builder_freegnm_bin

RUN mv config.generic.mak config.mak
RUN make

FROM builder_base as builder_psbc

RUN apt-get update && apt-get install -y --no-install-recommends \
      python3 \
      python3-mako && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder_freegnm /workspace/gnm /usr/local/include/gnm

ADD https://gitgud.io/gluesniffer/psbc/-/archive/751555965cef6ea44b6d2dd9edac332b9bc93868/psbc-751555965cef6ea44b6d2dd9edac332b9bc93868.zip /tmp/psbc.zip
RUN cd /tmp && unzip psbc.zip && cp -r /tmp/psbc-*/ /workspace
WORKDIR /workspace

ADD psbc.diff /tmp/patch.diff
RUN patch --silent --strip=1 --force < /tmp/patch.diff
RUN make

FROM builder_base as builder_shaderc

ADD https://storage.googleapis.com/shaderc/artifacts/prod/graphics_shader_compiler/shaderc/linux/continuous_clang_release/443/20231201-072117/install.tgz /tmp/install.tgz

RUN cd /tmp && tar -xzvf install.tgz

FROM ghcr.io/0xcaff/ps4-sdk-images/orbis:${BASE_IMAGE_TAG}

RUN apt-get update && \
    apt-get install -y --no-install-recommends software-properties-common && \
    add-apt-repository ppa:ubuntu-toolchain-r/test && \
    apt-get update && \
    apt-get install -y --no-install-recommends gcc-11 g++-11 && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder_freegnm_bin /workspace/gcn-dis /workspace/gnf-conv /workspace/gnf-mk /workspace/psb-dis /bin/
COPY --from=builder_freegnm_library /workspace/libgnm.a $OO_PS4_TOOLCHAIN/lib/
COPY --from=builder_freegnm_library /workspace/gnm $OO_PS4_TOOLCHAIN/include/gnm
COPY --from=builder_freegnm_library /usr/include/cglm $OO_PS4_TOOLCHAIN/include/cglm
COPY --from=builder_psbc /workspace/psbc /usr/local/bin/

COPY --from=builder_shaderc /tmp/install/bin /bin
COPY --from=builder_shaderc /tmp/install/include /include
COPY --from=builder_shaderc /tmp/install/lib /lib

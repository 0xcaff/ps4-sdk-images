FROM orbis as setup

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    unzip && \
    rm -rf /var/lib/apt/lists/*

ADD https://codeload.github.com/GoldHEN/GoldHEN_Plugins_SDK/zip/bcea3c7ef01dac6d7f9f49ebf9e90fe66d86f5f7 /tmp/sdk.zip

WORKDIR /tmp

RUN mkdir -p /opt/GoldHEN_Plugins_SDK && unzip sdk.zip && mv /tmp/GoldHEN_Plugins_SDK*/* /opt/GoldHEN_Plugins_SDK

WORKDIR /opt/GoldHEN_Plugins_SDK

ADD sdk.diff /tmp/sdk.diff
RUN patch --silent --strip=1 --force < /tmp/sdk.diff

RUN make

FROM orbis as runtime

ENV GOLDHEN_SDK=/opt/GoldHEN_Plugins_SDK

COPY --from=setup /opt/GoldHEN_Plugins_SDK $GOLDHEN_SDK
COPY --from=setup /opt/GoldHEN_Plugins_SDK/libGoldHEN_Hook.a ${OO_PS4_TOOLCHAIN}/lib/libGoldHEN_Hook.a

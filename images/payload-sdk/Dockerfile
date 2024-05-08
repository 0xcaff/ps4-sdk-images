FROM orbis AS setup

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    unzip && \
    rm -rf /var/lib/apt/lists/*

ADD https://codeload.github.com/Scene-Collective/ps4-payload-sdk/zip/ffcd53412e40d9b5d80775912ce4d071faa68d96 /tmp/ps4sdk.zip

WORKDIR /tmp

RUN mkdir -p /opt/ps4sdk && unzip ps4sdk.zip && mv /tmp/ps4-payload-sdk*/* /opt/ps4sdk

WORKDIR /opt/ps4sdk/libPS4

RUN make

FROM orbis as runtime

ENV PS4SDK=/opt/ps4sdk

COPY --from=setup /opt/ps4sdk $PS4SDK

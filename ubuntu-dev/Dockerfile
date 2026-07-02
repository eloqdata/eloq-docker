FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH=/home/eloq/.local/bin:$PATH

# Force IPv4 for apt to avoid flaky IPv6 mirrors.
RUN echo 'Acquire::ForceIPv4 "true";' > /etc/apt/apt.conf.d/99force-ipv4

# Development toolchain and libraries — everything here is installable via apt.
RUN apt-get update && apt-get install -y --no-install-recommends \
      jq sudo vim wget curl gnupg apt-transport-https apt-utils \
      python3 python3-dev python3-pip python3-venv \
      gdb lcov ccache rsync \
      build-essential gcc g++ make cmake ninja-build m4 bison pkg-config patchelf \
      git openssh-client openssh-server \
      openjdk-11-jdk golang nodejs npm \
      libprotobuf-dev protobuf-compiler \
      libcurl4-openssl-dev libssl-dev openssl gnutls-dev \
      libncurses5-dev ncurses-dev libreadline-dev \
      zlib1g-dev libbz2-dev liblz4-dev libzstd-dev liblzma-dev \
      libuv1-dev libgflags-dev libleveldb-dev libsnappy-dev libjsoncpp-dev \
      libboost-context-dev libboost-stacktrace-dev \
      libc-ares-dev libc-ares2 libsqlite3-dev libffi-dev \
      ca-certificates tar redis tcl \
    && rm -rf /var/lib/apt/lists/*

# Google Cloud CLI (installed from Google's apt repository).
RUN curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg \
      | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
      > /etc/apt/sources.list.d/google-cloud-sdk.list \
    && apt-get update && apt-get install -y --no-install-recommends google-cloud-cli \
    && rm -rf /var/lib/apt/lists/*

# Python 3.8 + the log-replay test dependencies used by the eloqkv CI, baked
# into a venv so the CI never has to apt-install python or pip-install at runtime.
# Consumers activate it via $LOG_REPLAY_VENV/bin/activate.
RUN apt-get update && apt-get install -y --no-install-recommends software-properties-common \
    && add-apt-repository -y ppa:deadsnakes/ppa \
    && apt-get update && apt-get install -y --no-install-recommends \
      python3.8 python3.8-venv python3.8-dev \
    && python3.8 -m venv /opt/eloq/log-replay-venv \
    && /opt/eloq/log-replay-venv/bin/pip install --no-cache-dir \
      async-timeout==4.0.3 psutil==5.9.5 redis==5.0.3 \
    && chown -R root:sudo /opt/eloq && chmod -R g+rwX /opt/eloq \
    && rm -rf /var/lib/apt/lists/*
ENV LOG_REPLAY_VENV=/opt/eloq/log-replay-venv

# Python test/runtime tooling used by the eloqkv CI (formerly installed by
# scripts/dep/ubuntu/02-python.sh): awscli, boto3, cassandra-driver, grpcio,
# etc. Baked into a venv so the build/test scripts never pip-install at runtime.
# Consumers activate it via $ELOQ_TEST_VENV/bin/activate.
RUN python3 -m venv /opt/eloq/test-venv \
    && /opt/eloq/test-venv/bin/pip install --no-cache-dir \
      setuptools==45.2.0 \
      cassandra-driver==3.29.2 \
      awscli==1.29.44 \
      boto3==1.28.36 \
      botocore==1.31.44 \
      mysql-connector-python==8.1.0 \
      psutil==5.9.5 \
      grpcio==1.60.0 \
      grpcio-tools==1.60.0 \
    && chown -R root:sudo /opt/eloq && chmod -R g+rwX /opt/eloq
ENV ELOQ_TEST_VENV=/opt/eloq/test-venv

# Non-root user with passwordless sudo (no password is baked into the image).
RUN useradd -rm -s /bin/bash -g sudo eloq \
    && echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && mkdir -p /var/crash && chown -R eloq /var/crash

USER eloq
WORKDIR /home/eloq
RUN mkdir -p /home/eloq/workspace

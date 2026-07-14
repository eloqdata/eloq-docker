FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHON2_HOME=/opt/eloq/python2
ENV PATH=$PYTHON2_HOME/bin:/home/eloq/.local/bin:$PATH

# Force IPv4 for apt to avoid flaky IPv6 mirrors, and retry transient mirror
# sync failures seen on Ubuntu ports.
RUN echo 'Acquire::ForceIPv4 "true";' > /etc/apt/apt.conf.d/99force-ipv4 \
    && echo 'Acquire::Retries "5";' > /etc/apt/apt.conf.d/80-retries \
    && printf '%s\n' \
      '#!/bin/sh' \
      'set -eu' \
      'status=1' \
      'for attempt in 1 2 3 4 5; do' \
      '  rm -rf /var/lib/apt/lists/*' \
      '  apt-get update' \
      '  if apt-get install -y --no-install-recommends "$@"; then' \
      '    exit 0' \
      '  fi' \
      '  status=$?' \
      '  echo "apt install failed on attempt ${attempt}; retrying..." >&2' \
      '  sleep $((attempt * 10))' \
      'done' \
      'exit "$status"' \
      > /usr/local/bin/apt-install-retry \
    && chmod +x /usr/local/bin/apt-install-retry

# Development toolchain and libraries — everything here is installable via apt.
RUN apt-install-retry \
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
    && apt-install-retry google-cloud-cli \
    && rm -rf /var/lib/apt/lists/*

# Python 3.8 + the log-replay test dependencies used by the eloqkv CI, baked
# into a venv so the CI never has to apt-install python or pip-install at runtime.
# Consumers activate it via $LOG_REPLAY_VENV/bin/activate.
RUN apt-install-retry software-properties-common \
    && add-apt-repository -y ppa:deadsnakes/ppa \
    && apt-install-retry \
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

# Python 2.7.18 + the MongoDB 4.0 build/resmoke dependencies used by EloqDoc.
# Python 2 cannot reliably build against Ubuntu 24.04's OpenSSL 3, so build a
# private OpenSSL 1.1 runtime for the Python 2 ssl module.
RUN set -eux; \
    apt-install-retry \
      build-essential ca-certificates curl libbz2-dev libffi-dev liblzma-dev \
      libncursesw5-dev libreadline-dev libsqlite3-dev libssl-dev make tk-dev \
      wget xz-utils zlib1g-dev; \
    mkdir -p /tmp/python2-build; \
    cd /tmp/python2-build; \
    curl -fsSLO https://www.openssl.org/source/openssl-1.1.1w.tar.gz; \
    tar -xzf openssl-1.1.1w.tar.gz; \
    cd openssl-1.1.1w; \
    ./config --prefix=/opt/eloq/openssl-1.1 --openssldir=/opt/eloq/openssl-1.1 shared zlib; \
    make -j"$(nproc)"; \
    make install_sw; \
    cd /tmp/python2-build; \
    curl -fsSLO https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz; \
    tar -xzf Python-2.7.18.tgz; \
    cd Python-2.7.18; \
    ./configure \
      --prefix="$PYTHON2_HOME" \
      --enable-shared \
      --enable-unicode=ucs4 \
      --with-system-ffi \
      CPPFLAGS="-I/opt/eloq/openssl-1.1/include" \
      LDFLAGS="-Wl,-rpath,/opt/eloq/openssl-1.1/lib -L/opt/eloq/openssl-1.1/lib"; \
    make -j"$(nproc)"; \
    make install; \
    echo "$PYTHON2_HOME/lib" > /etc/ld.so.conf.d/python2.conf; \
    echo "/opt/eloq/openssl-1.1/lib" > /etc/ld.so.conf.d/openssl-1.1.conf; \
    ldconfig; \
    curl -fsSL https://bootstrap.pypa.io/pip/2.7/get-pip.py -o /tmp/get-pip.py; \
    python2 /tmp/get-pip.py 'pip<21' 'setuptools<45' 'wheel<0.38'; \
    python2 -m pip install --no-cache-dir \
      'Cheetah3==3.0.0' \
      'Jinja2==2.10' \
      'MarkupSafe<2.0' \
      'PyYAML==3.11' \
      'mock==2.0.0' \
      'pymongo>=3.0,<4' \
      'regex==2021.11.10' \
      'requests>=2.16.1,<2.28' \
      'subprocess32>=3.2.7' \
      'typing==3.6.1' \
      'unittest-xml-reporting==2.1.0'; \
    ln -sf "$PYTHON2_HOME/bin/python2.7" /usr/local/bin/python2; \
    ln -sf "$PYTHON2_HOME/bin/pip2.7" /usr/local/bin/pip2; \
    chown -R root:sudo /opt/eloq && chmod -R g+rwX /opt/eloq; \
    rm -rf /tmp/python2-build /tmp/get-pip.py /root/.cache

# Non-root user with passwordless sudo (no password is baked into the image).
RUN useradd -rm -s /bin/bash -g sudo eloq \
    && echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && mkdir -p /var/crash && chown -R eloq /var/crash

USER eloq
WORKDIR /home/eloq
RUN mkdir -p /home/eloq/workspace

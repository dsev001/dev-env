# Base image of the workspaces: the tools every project needs. A project image puts
# FROM ghcr.io/dsev001/dev-env-base in front and adds its own tools. There is no sudo:
# every tool comes in through a Dockerfile, so a second user gets the same image.
# The FROM names a tag and a digest; the digest decides. Change both together.
FROM docker.io/library/ubuntu:26.04@sha256:6a06111c3b0a8600feadb270aec771dc239bf407b25625f93f44ab00e11f4d3c

ENV DEBIAN_FRONTEND=noninteractive LANG=C.UTF-8

# Tools from Ubuntu, then gh from the apt repository of GitHub, Claude Code from the
# apt repository of Anthropic (channel stable) and the Docker client from the apt
# repository of Docker. The fingerprint checks stop the build when a key is not the one
# the docs name. apt installs do not update themselves: a new version comes with a new
# image. fd-find installs fdfind; the link gives it the usual name.
# Docker: only client, buildx and compose; the daemon runs as a sidecar of the workspace
# pod (DOCKER_HOST from the template). The link docker-compose serves Makefiles that
# call compose as its own command.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        bash-completion bind9-dnsutils bzip2 ca-certificates curl fd-find file gettext-base git \
        git-lfs gnupg htop iproute2 jq less lsof make man-db moreutils nano \
        ncdu netcat-openbsd openssh-client procps psmisc python3 python3-pip python3-venv \
        ripgrep rsync shellcheck sqlite3 tmux tree tzdata unzip vim wget xz-utils zip \
    && ln -s /usr/bin/fdfind /usr/local/bin/fd \
    && install -d -m 0755 /etc/apt/keyrings \
    && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        -o /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
        > /etc/apt/sources.list.d/github-cli.list \
    && curl -fsSL https://downloads.claude.ai/keys/claude-code.asc -o /etc/apt/keyrings/claude-code.asc \
    && gpg --show-keys --with-colons /etc/apt/keyrings/claude-code.asc \
        | grep -q '^fpr:::::::::31DDDE24DDFAB679F42D7BD2BAA929FF1A7ECACE:$' \
    && echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/stable stable main" \
        > /etc/apt/sources.list.d/claude-code.list \
    && curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc \
    && gpg --show-keys --with-colons /etc/apt/keyrings/docker.asc \
        | grep -q '^fpr:::::::::9DC858229FC7DD38854AE2D88D81803C0EBFCD88:$' \
    && echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu resolute stable" \
        > /etc/apt/sources.list.d/docker.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends gh claude-code \
        docker-ce-cli docker-buildx-plugin docker-compose-plugin \
    && ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/local/bin/docker-compose \
    && rm -rf /var/lib/apt/lists/*

# actionlint and yq (mikefarah) as release binaries, checked against the sha256 sums the
# releases publish. New version: tag and sum together.
ARG ACTIONLINT_VERSION=1.7.12
ARG ACTIONLINT_SHA256=8aca8db96f1b94770f1b0d72b6dddcb1ebb8123cb3712530b08cc387b349a3d8
ARG YQ_VERSION=4.54.2
ARG YQ_SHA256=44c7759059b4910827a0059a2757b22001e59360cdecdb2ef8dedc4b6c7e9f32
RUN curl -fsSL -o /tmp/actionlint.tar.gz \
        "https://github.com/rhysd/actionlint/releases/download/v${ACTIONLINT_VERSION}/actionlint_${ACTIONLINT_VERSION}_linux_amd64.tar.gz" \
    && echo "${ACTIONLINT_SHA256}  /tmp/actionlint.tar.gz" | sha256sum -c - \
    && tar -xzf /tmp/actionlint.tar.gz -C /usr/local/bin actionlint \
    && rm /tmp/actionlint.tar.gz \
    && curl -fsSL -o /usr/local/bin/yq \
        "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_amd64" \
    && echo "${YQ_SHA256}  /usr/local/bin/yq" | sha256sum -c - \
    && chmod 0755 /usr/local/bin/yq

# Codex as the static binary of a GitHub release, checked against the sha256 of that
# release. New version: tag and sum together. Login happens in the workspace with
# codex login --device-auth; the state lives in ~/.codex on the home volume.
ARG CODEX_VERSION=0.167.0
ARG CODEX_SHA256=9d64836106d5b374e90ee4454c7ebcde7ca7106558bd7e41a34b9f26ebda251a
RUN curl -fsSL -o /tmp/codex.tar.gz \
        "https://github.com/openai/codex/releases/download/rust-v${CODEX_VERSION}/codex-x86_64-unknown-linux-musl.tar.gz" \
    && echo "${CODEX_SHA256}  /tmp/codex.tar.gz" | sha256sum -c - \
    && tar -xzf /tmp/codex.tar.gz -C /usr/local/bin codex-x86_64-unknown-linux-musl \
    && mv /usr/local/bin/codex-x86_64-unknown-linux-musl /usr/local/bin/codex \
    && rm /tmp/codex.tar.gz

# Coder runs the workspace as UID 1000 with the home /home/coder on a volume. Ubuntu
# ships a user "ubuntu" with that UID; it goes, so the UID is free for coder.
RUN userdel -r ubuntu \
    && groupadd -g 1000 coder \
    && useradd -m -u 1000 -g 1000 -s /bin/bash coder

USER coder
WORKDIR /home/coder

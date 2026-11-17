# Base image of the workspaces: the tools every project needs. A project image puts
# FROM ghcr.io/dsev001/dev-env-base in front and adds its own tools. There is no sudo:
# every tool comes in through a Dockerfile, so a second user gets the same image.
# Each build takes the latest version of each tool; a version gets pinned only when it
# causes trouble. The FROM follows the tag 26.04, the LTS line.
FROM docker.io/library/ubuntu:26.04

ENV DEBIAN_FRONTEND=noninteractive LANG=C.UTF-8

# Tools from Ubuntu, then gh from the apt repository of GitHub, Claude Code from the
# apt repository of Anthropic (channel latest) and the Docker client from the apt
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
    && echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/latest latest main" \
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

# actionlint and yq (mikefarah) from their latest releases, checked against the sha256
# sums the releases publish. actionlint has the version in the file name; the redirect
# of releases/latest names it.
RUN mkdir /tmp/dl && cd /tmp/dl \
    && url=$(curl -fsSL -o /dev/null -w '%{url_effective}' https://github.com/rhysd/actionlint/releases/latest) \
    && v=${url##*/v} \
    && base="https://github.com/rhysd/actionlint/releases/download/v${v}" \
    && curl -fsSLO "${base}/actionlint_${v}_linux_amd64.tar.gz" \
    && curl -fsSL "${base}/actionlint_${v}_checksums.txt" | grep " actionlint_${v}_linux_amd64.tar.gz\$" | sha256sum -c - \
    && tar -xzf "actionlint_${v}_linux_amd64.tar.gz" -C /usr/local/bin actionlint \
    && curl -fsSLO https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 \
    && curl -fsSL https://github.com/mikefarah/yq/releases/latest/download/checksums-bsd \
        | grep '^SHA256 (yq_linux_amd64) = ' | sha256sum -c - \
    && install -m 0755 yq_linux_amd64 /usr/local/bin/yq \
    && cd / && rm -rf /tmp/dl \
    && actionlint -version && yq --version

# Codex as the package of the latest GitHub release, checked against the sha256 sums of
# that release.
# The bare binary does not start: it needs the package around it (codex-package.json,
# exec server, rg, bwrap for the sandbox). The package goes to /opt/codex, not into the
# home volume; the link finds it, as with the official install.sh. Login happens in the
# workspace with codex login --device-auth; the state lives in ~/.codex on the home volume.
RUN mkdir /tmp/dl && cd /tmp/dl \
    && base=https://github.com/openai/codex/releases/latest/download \
    && curl -fsSLO "${base}/codex-package-x86_64-unknown-linux-musl.tar.gz" \
    && curl -fsSL "${base}/codex-package_SHA256SUMS" \
        | grep ' codex-package-x86_64-unknown-linux-musl.tar.gz$' | sha256sum -c - \
    && install -d -m 0755 /opt/codex \
    && tar -xzf codex-package-x86_64-unknown-linux-musl.tar.gz -C /opt/codex --no-same-owner \
    && ln -s /opt/codex/bin/codex /usr/local/bin/codex \
    && cd / && rm -rf /tmp/dl \
    && codex --version

# Coder runs the workspace as UID 1000 with the home /home/coder on a volume. Ubuntu
# ships a user "ubuntu" with that UID; it goes, so the UID is free for coder.
RUN userdel -r ubuntu \
    && groupadd -g 1000 coder \
    && useradd -m -u 1000 -g 1000 -s /bin/bash coder

USER coder
WORKDIR /home/coder

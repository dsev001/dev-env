# Image of the project cls: the base image plus JDK 25, Node 26, Pillow and psql.
# The Coder template "cls" starts it; cls-setup clones clsroot on the first start.
# The base image is ours and moves with every build, so latest is on purpose.
FROM ghcr.io/dsev001/dev-env-base:latest

USER root

# JDK 25 copied from the Temurin image. The tag names it, the digest decides;
# change both together. Version inside: jdk-25.0.5+8, the same as on macOS.
COPY --from=docker.io/library/eclipse-temurin:25-jdk-noble@sha256:95e350ad1eb738f943e141640f7e3d797e483359a3b33c3864255bfd1ad71f8a \
    /opt/java/openjdk /opt/java/openjdk
ENV JAVA_HOME=/opt/java/openjdk PATH=/opt/java/openjdk/bin:$PATH

# Node 26 from nodejs.org, checked against the SHASUMS256.txt of the release.
# New version: version and sum together.
ARG NODE_VERSION=26.13.0
ARG NODE_SHA256=d28a96af6c38596f6d9d97c0ee531c13d238d757742387d1928de822d959e997
RUN curl -fsSL -o /tmp/node.tar.xz \
        "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" \
    && echo "${NODE_SHA256}  /tmp/node.tar.xz" | sha256sum -c - \
    && tar -xJf /tmp/node.tar.xz -C /usr/local --strip-components=1 \
        --exclude='*/CHANGELOG.md' --exclude='*/LICENSE' --exclude='*/README.md' \
    && rm /tmp/node.tar.xz

# Pillow for the asset checks of cls, psql for the database sidecar.
RUN apt-get update \
    && apt-get install -y --no-install-recommends python3-pil postgresql-client \
    && rm -rf /var/lib/apt/lists/*

COPY --chmod=0755 cls-setup /usr/local/bin/cls-setup

USER coder

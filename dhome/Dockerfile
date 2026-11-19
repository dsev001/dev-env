# Image for dhome: the base image plus the exact tool versions of dhome, JDK 25.0.5+8,
# Node 26.13.1, npm 12.2.3 and Chromium for Playwright 1.65.1, and two Claude Code plugins.
# Unlike the other images this one pins on purpose: the versions follow
# scripts/install-toolchain.sh and src/testsuite/browser/package.json of dhome. When dhome
# changes a version, change it here too, version and checksum together.
# The base image is ours and moves with every build, so latest is on purpose.
FROM ghcr.io/dsev001/dev-env-base:latest

USER root

# JDK and Node from their releases, checked against the sha256 sums that
# install-toolchain.sh of dhome names.
RUN mkdir -p /tmp/dl /opt/java/openjdk && cd /tmp/dl \
    && curl -fsSLo jdk.tar.gz https://github.com/adoptium/temurin25-binaries/releases/download/jdk-25.0.5%2B8/OpenJDK25U-jdk_x64_linux_hotspot_25.0.5_8.tar.gz \
    && echo "68982529869257d0ee6b824ec5aa270ff8b118bb438670da0b97ebdab178d7fb  jdk.tar.gz" | sha256sum -c - \
    && tar -xzf jdk.tar.gz -C /opt/java/openjdk --strip-components=1 \
    && curl -fsSLo node.tar.xz https://nodejs.org/dist/v26.13.1/node-v26.13.1-linux-x64.tar.xz \
    && echo "5c063e46bab0fbc1c17cc445d7ff129cbdcd0471be6ed83b6ab864f8143bc864  node.tar.xz" | sha256sum -c - \
    && tar -xJf node.tar.xz -C /usr/local --strip-components=1 \
        --exclude='*/CHANGELOG.md' --exclude='*/LICENSE' --exclude='*/README.md' \
    && cd / && rm -rf /tmp/dl
ENV JAVA_HOME=/opt/java/openjdk PATH=/opt/java/openjdk/bin:$PATH

# npm and playwright-core from the npm registry; the sha512 sums are the integrity values
# of the registry in hex, the one of playwright-core also stands in the package-lock.json
# of the browser tests. The npm tarball brings its dependencies along. playwright-core
# only installs Chromium with its libraries from Ubuntu (--with-deps) and goes again: the
# browser tests bring their own Playwright through npm ci. The browsers lie in the image,
# not in a cache in the home.
ENV PLAYWRIGHT_BROWSERS_PATH=/opt/playwright
RUN mkdir /tmp/dl && cd /tmp/dl \
    && curl -fsSLO https://registry.npmjs.org/npm/-/npm-12.2.3.tgz \
    && echo "a2f98d4517bbe89bdca7513a88e78f43a1fd66be1730eaea81f69e18d86d46f3f75d29762e8fdaf42d4de7f8d194b2ebb2a09e9bfa268067591e96796baca7d5  npm-12.2.3.tgz" | sha512sum -c - \
    && npm install --global --no-fund --no-audit ./npm-12.2.3.tgz \
    && [ "$(npm --version)" = 12.2.3 ] \
    && curl -fsSLO https://registry.npmjs.org/playwright-core/-/playwright-core-1.65.1.tgz \
    && echo "86a3e9cfb6e548c3f1595768784d728814b35ec2a254e77693e9230af994cb00f618f3b60ddf477dc5a7abd34dd061db3590c5819f497c116bf97e7f04ba34ab  playwright-core-1.65.1.tgz" | sha512sum -c - \
    && tar -xzf playwright-core-1.65.1.tgz \
    && node package/cli.js install --with-deps chromium \
    && cd / && rm -rf /tmp/dl /var/lib/apt/lists/* /root/.npm

# Claude Code plugins superpowers and mattpocock-skills in a seed directory: Claude Code
# loads them from there in place and never writes to it, so they need nothing in the
# home and a new workspace has them too. Each build takes the default branch of the
# official marketplace, which names a commit per plugin. managed-settings.json turns the
# plugins on.
ENV CLAUDE_CODE_PLUGIN_SEED_DIR=/opt/claude-seed
RUN export HOME=/tmp/claude-build CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed \
    && mkdir -p "$HOME" \
    && claude plugin marketplace add anthropics/claude-plugins-official \
    && claude plugin install superpowers@claude-plugins-official \
    && claude plugin install mattpocock-skills@claude-plugins-official \
    && rm -rf /tmp/claude-build
COPY managed-settings.json /etc/claude-code/managed-settings.json

USER coder

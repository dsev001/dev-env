# Image for Java and Node projects: the base image plus JDK 25, Node 26, Pillow, psql,
# Playwright with Chromium and two Claude Code plugins.
# The Coder template "cls" starts it; cls-setup clones clsroot on the first start.
# The base image is ours and moves with every build, so latest is on purpose.
FROM ghcr.io/dsev001/dev-env-base:latest

USER root

# JDK 25 copied from the Temurin image; the tag follows the latest build of JDK 25.
COPY --from=docker.io/library/eclipse-temurin:25-jdk-noble \
    /opt/java/openjdk /opt/java/openjdk
ENV JAVA_HOME=/opt/java/openjdk PATH=/opt/java/openjdk/bin:$PATH

# Node 26 from nodejs.org, the latest release of the line 26, checked against its
# SHASUMS256.txt; the file name in it names the version.
RUN mkdir /tmp/dl && cd /tmp/dl \
    && base=https://nodejs.org/dist/latest-v26.x \
    && curl -fsSL "${base}/SHASUMS256.txt" | grep -- '-linux-x64.tar.xz$' > sums \
    && curl -fsSLO "${base}/$(awk '{print $2}' sums)" \
    && sha256sum -c sums \
    && tar -xJf node-v*-linux-x64.tar.xz -C /usr/local --strip-components=1 \
        --exclude='*/CHANGELOG.md' --exclude='*/LICENSE' --exclude='*/README.md' \
    && cd / && rm -rf /tmp/dl

# Pillow for image checks, psql for a database sidecar. 12.1.1-2ubuntu1.5 broke resize
# with NEAREST (LP: #2174424), 1.6 fixes it. The check stops the build when a package
# breaks it again; then pin the last good version here.
RUN apt-get update \
    && apt-get install -y --no-install-recommends python3-pil postgresql-client \
    && rm -rf /var/lib/apt/lists/* \
    && python3 -I -c "from PIL import Image; [Image.new(m, (4, 4)).resize((8, 8), Image.Resampling.NEAREST) for m in ('RGBA', 'RGB', 'L', 'P', '1')]"

# Playwright with Chromium for screenshots without a display, the latest release at
# build time; each Playwright version brings its own Chromium build. The browsers lie
# in the image, not in a cache in the home; --with-deps installs their libraries from
# Ubuntu. NODE_PATH lets a script outside a project require('playwright').
ENV PLAYWRIGHT_BROWSERS_PATH=/opt/playwright NODE_PATH=/usr/local/lib/node_modules
RUN npm install -g playwright@latest \
    && playwright install --with-deps chromium \
    && rm -rf /var/lib/apt/lists/* /root/.npm

# Claude Code plugins codex and pixijs-skills in a seed directory: Claude Code loads them
# from there in place and never writes to it, so they need nothing in the home and a new
# workspace has them too. Each build takes the default branch of the marketplaces; a
# pin goes on as owner/repo@<commit>. managed-settings.json turns the plugins on.
ENV CLAUDE_CODE_PLUGIN_SEED_DIR=/opt/claude-seed
RUN export HOME=/tmp/claude-build CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed \
    && mkdir -p "$HOME" \
    && claude plugin marketplace add openai/codex-plugin-cc \
    && claude plugin install codex@openai-codex \
    && claude plugin marketplace add pixijs/pixijs-skills \
    && claude plugin install pixijs-skills@pixijs-skills \
    && rm -rf /tmp/claude-build
COPY managed-settings.json /etc/claude-code/managed-settings.json

COPY --chmod=0755 cls-setup /usr/local/bin/cls-setup

USER coder

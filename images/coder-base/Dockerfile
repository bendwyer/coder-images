# coder-base: foundational multi-arch image for Coder Kubernetes workspaces.
# Provides a non-root `coder` user (UID/GID 1000, passwordless sudo) and minimal
# tooling. The Coder agent and code-server are downloaded at workspace start, not
# baked in; domain images build FROM this and add their toolchain.

FROM debian:13-slim

ARG USERNAME=coder
ARG USER_UID=1000
ARG USER_GID=1000

ENV DEBIAN_FRONTEND=noninteractive

# apt resolves the arch, so this layer is multi-arch with no per-arch handling.
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    bash \
    ca-certificates \
    curl \
    git \
    gnupg \
    jq \
    locales \
    openssh-client \
    sudo \
    unzip \
    wget \
  && sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen \
  && locale-gen \
  && rm -rf /var/lib/apt/lists/*

# Claude Code CLI, latest apt channel (stable lags too far for current models).
# A system path so the home-volume mount can't shadow it. Renovate bumps the pin.
ARG CLAUDE_CODE_VERSION=2.1.185-1
RUN install -d -m 0755 /etc/apt/keyrings \
  && curl -fsSL https://downloads.claude.ai/keys/claude-code.asc \
       -o /etc/apt/keyrings/claude-code.asc \
  && echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/latest latest main" \
       > /etc/apt/sources.list.d/claude-code.list \
  && apt-get update \
  && apt-get install -y --no-install-recommends "claude-code=${CLAUDE_CODE_VERSION}" \
  && rm -rf /var/lib/apt/lists/*

# mise: lightweight tool-version manager. Tools are NOT baked here; the workspace
# installs them per-project (.tool-versions/mise.toml) or on demand (`mise use`)
# into the home PVC. Bare static binary at a system path so the home mount can't
# shadow it. Renovate bumps the pin (jdx/mise releases).
ARG MISE_VERSION=v2026.6.11
RUN arch="$(dpkg --print-architecture)" \
  && case "${arch}" in \
       amd64) march=x64 ;; \
       arm64) march=arm64 ;; \
       *) echo "unsupported arch: ${arch}" >&2; exit 1 ;; \
     esac \
  && curl -fsSL "https://github.com/jdx/mise/releases/download/${MISE_VERSION}/mise-${MISE_VERSION}-linux-${march}" \
       -o /usr/local/bin/mise \
  && chmod 0755 /usr/local/bin/mise \
  && mise --version

# Non-root `coder` user with passwordless sudo; UID/GID 1000 matches the template.
RUN groupadd --gid ${USER_GID} ${USERNAME} \
  && useradd --uid ${USER_UID} --gid ${USER_GID} --create-home --shell /bin/bash ${USERNAME} \
  && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
  && chmod 0440 /etc/sudoers.d/${USERNAME}

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# mise shims + ~/.local/bin on PATH so non-interactive shells (e.g. Claude Code's
# `bash -c`) resolve mise-managed and on-demand tools, not just interactive ones.
ENV PATH="/home/${USERNAME}/.local/share/mise/shims:/home/${USERNAME}/.local/bin:${PATH}"

USER ${USERNAME}
WORKDIR /home/${USERNAME}

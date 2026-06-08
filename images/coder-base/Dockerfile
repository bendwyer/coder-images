# coder-base: foundational multi-arch image for Coder Kubernetes workspaces.
#
# Provides exactly what the Coder template (coder-config) assumes:
#   - a non-root `coder` user (UID/GID 1000) with home at /home/coder
#   - passwordless sudo
#   - a download tool (curl) so the workspace can fetch the Coder agent at start
#   - a minimal, sane base (bash, git, ca-certificates, locale)
#
# The Coder agent and code-server are NOT baked in: the agent is downloaded at
# workspace start by the template's init script, and code-server by its startup
# script. Domain images build FROM this and add their toolchain.

FROM debian:13-slim

ARG USERNAME=coder
ARG USER_UID=1000
ARG USER_GID=1000

ENV DEBIAN_FRONTEND=noninteractive

# apt resolves the correct architecture automatically, so this layer is
# multi-arch with no per-arch handling.
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

# Non-root `coder` user with passwordless sudo, matching the template's
# run_as_user / fs_group of 1000 and its /home/coder mount.
RUN groupadd --gid ${USER_GID} ${USERNAME} \
  && useradd --uid ${USER_UID} --gid ${USER_GID} --create-home --shell /bin/bash ${USERNAME} \
  && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
  && chmod 0440 /etc/sudoers.d/${USERNAME}

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

USER ${USERNAME}
WORKDIR /home/${USERNAME}

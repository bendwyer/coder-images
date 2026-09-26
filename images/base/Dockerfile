# base: foundational multi-arch image for Coder Kubernetes workspaces.
# Provides a non-root `coder` user (UID/GID 1000, passwordless sudo) and the
# minimum needed to bootstrap one. Developer tooling belongs in the images
# built on this one, not here.

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

# Non-root `coder` user with passwordless sudo; UID/GID 1000 matches the template.
RUN groupadd --gid ${USER_GID} ${USERNAME} \
  && useradd --uid ${USER_UID} --gid ${USER_GID} --create-home --shell /bin/bash ${USERNAME} \
  && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
  && chmod 0440 /etc/sudoers.d/${USERNAME}

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

USER ${USERNAME}
WORKDIR /home/${USERNAME}

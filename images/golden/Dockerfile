# golden: the general-purpose workspace image. base plus the tools every
# workspace uses regardless of which repos it clones, including mise, which
# installs what each repo declares in its mise.toml.

# Pinned by digest so a base publish reaches this image through a
# Renovate bump, not by whichever base happened to be current at build time.
FROM ghcr.io/bendwyer/coder-images/base:2026.9.28@sha256:9d6b45abca17723a320e127a37baf8325f8c23c7be9446f7a78f1ac4e1261147

ARG TARGETARCH

# renovate: datasource=github-releases depName=cli/cli
ARG GH_VERSION=2.102.0
# renovate: datasource=github-releases depName=mikefarah/yq
ARG YQ_VERSION=4.54.1
# renovate: datasource=github-releases depName=getsops/sops
ARG SOPS_VERSION=3.13.3
# renovate: datasource=github-releases depName=FiloSottile/age
ARG AGE_VERSION=1.3.2
# renovate: datasource=custom.onepassword-cli depName=op
ARG OP_VERSION=2.40.0
# renovate: datasource=github-releases depName=jdx/mise
ARG MISE_VERSION=2026.10.0

USER root

# tmux lets a Claude Code Remote Control session outlive the terminal that
# started it, which ends the session otherwise.
RUN apt-get update \
  && apt-get install -y --no-install-recommends tmux \
  && rm -rf /var/lib/apt/lists/*

# Claude Code CLI, latest apt channel (stable lags too far for current models).
# A system path so the home-volume mount can't shadow it. Renovate bumps the pin.
ARG CLAUDE_CODE_VERSION=2.1.288-1
RUN install -d -m 0755 /etc/apt/keyrings \
  && curl -fsSL https://downloads.claude.ai/keys/claude-code.asc \
       -o /etc/apt/keyrings/claude-code.asc \
  && echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/latest latest main" \
       > /etc/apt/sources.list.d/claude-code.list \
  && apt-get update \
  && apt-get install -y --no-install-recommends "claude-code=${CLAUDE_CODE_VERSION}" \
  && rm -rf /var/lib/apt/lists/*

# Every upstream here but mise spells the arch the same way Docker's TARGETARCH does.
RUN set -eux; \
  cd /tmp; \
  case "${TARGETARCH}" in amd64) mise_arch=x64 ;; *) mise_arch="${TARGETARCH}" ;; esac; \
  curl -fsSL -o /usr/local/bin/mise "https://github.com/jdx/mise/releases/download/v${MISE_VERSION}/mise-v${MISE_VERSION}-linux-${mise_arch}"; \
  curl -fsSL "https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_${TARGETARCH}.tar.gz" \
    | tar -xz --strip-components=2 -C /usr/local/bin "gh_${GH_VERSION}_linux_${TARGETARCH}/bin/gh"; \
  curl -fsSL -o /usr/local/bin/yq "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_${TARGETARCH}"; \
  curl -fsSL -o /usr/local/bin/sops "https://github.com/getsops/sops/releases/download/v${SOPS_VERSION}/sops-v${SOPS_VERSION}.linux.${TARGETARCH}"; \
  curl -fsSL "https://github.com/FiloSottile/age/releases/download/v${AGE_VERSION}/age-v${AGE_VERSION}-linux-${TARGETARCH}.tar.gz" \
    | tar -xz --strip-components=1 -C /usr/local/bin age/age age/age-keygen; \
  curl -fsSL -o op.zip "https://cache.agilebits.com/dist/1P/op2/pkg/v${OP_VERSION}/op_linux_${TARGETARCH}_v${OP_VERSION}.zip"; \
  unzip -q op.zip op -d /usr/local/bin; \
  rm op.zip; \
  chmod 0755 /usr/local/bin/gh /usr/local/bin/yq /usr/local/bin/sops /usr/local/bin/age /usr/local/bin/age-keygen /usr/local/bin/op /usr/local/bin/mise; \
  gh --version; yq --version; sops --version; age --version; op --version; mise --version

USER coder

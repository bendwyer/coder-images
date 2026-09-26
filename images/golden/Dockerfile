# golden: the general-purpose workspace image. coder-base plus the tools every
# workspace uses regardless of which repos it clones. Anything a particular
# repo needs belongs in that repo's mise.toml, not here.

# Pinned by digest so a coder-base publish reaches this image through a
# Renovate bump, not by whichever base happened to be current at build time.
FROM ghcr.io/bendwyer/coder-images/coder-base:2026.9.26@sha256:546024e95f81a7aa0727dc802fa793e253d0b4611178e4e690cf79c3442e1e23

ARG TARGETARCH

# renovate: datasource=github-releases depName=cli/cli
ARG GH_VERSION=2.101.0
# renovate: datasource=github-releases depName=mikefarah/yq
ARG YQ_VERSION=4.53.6
# renovate: datasource=github-releases depName=getsops/sops
ARG SOPS_VERSION=3.13.3
# renovate: datasource=github-releases depName=FiloSottile/age
ARG AGE_VERSION=1.3.2
# renovate: datasource=custom.onepassword-cli depName=op
ARG OP_VERSION=2.39.0

USER root

# Every upstream here spells the arch the same way Docker's TARGETARCH does.
RUN set -eux; \
  cd /tmp; \
  curl -fsSL "https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_${TARGETARCH}.tar.gz" \
    | tar -xz --strip-components=2 -C /usr/local/bin "gh_${GH_VERSION}_linux_${TARGETARCH}/bin/gh"; \
  curl -fsSL -o /usr/local/bin/yq "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_${TARGETARCH}"; \
  curl -fsSL -o /usr/local/bin/sops "https://github.com/getsops/sops/releases/download/v${SOPS_VERSION}/sops-v${SOPS_VERSION}.linux.${TARGETARCH}"; \
  curl -fsSL "https://github.com/FiloSottile/age/releases/download/v${AGE_VERSION}/age-v${AGE_VERSION}-linux-${TARGETARCH}.tar.gz" \
    | tar -xz --strip-components=1 -C /usr/local/bin age/age age/age-keygen; \
  curl -fsSL -o op.zip "https://cache.agilebits.com/dist/1P/op2/pkg/v${OP_VERSION}/op_linux_${TARGETARCH}_v${OP_VERSION}.zip"; \
  unzip -q op.zip op -d /usr/local/bin; \
  rm op.zip; \
  chmod 0755 /usr/local/bin/gh /usr/local/bin/yq /usr/local/bin/sops /usr/local/bin/age /usr/local/bin/age-keygen /usr/local/bin/op; \
  gh --version; yq --version; sops --version; age --version; op --version

USER coder

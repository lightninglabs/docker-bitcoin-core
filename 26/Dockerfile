FROM debian:bookworm-slim

ARG UID=101
ARG GID=101

LABEL maintainer.0="João Fonseca (@joaopaulofonseca)" \
  maintainer.1="Pedro Branco (@pedrobranco)" \
  maintainer.2="Rui Marinho (@ruimarinho)"

RUN groupadd --gid ${GID} bitcoin \
  && useradd --create-home --no-log-init -u ${UID} -g ${GID} bitcoin \
  && apt-get update -y \
  && apt-get --no-install-recommends -y install jq curl gnupg gosu ca-certificates \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ARG TARGETPLATFORM
ENV BITCOIN_VERSION=26.0
ENV BITCOIN_DATA=/home/bitcoin/.bitcoin
ENV PATH=/opt/bitcoin-${BITCOIN_VERSION}/bin:$PATH

SHELL ["/bin/bash", "-c"]
RUN set -ex \
  && if [ "${TARGETPLATFORM}" = "linux/amd64" ]; then export TARGETPLATFORM=x86_64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm64" ]; then export TARGETPLATFORM=aarch64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm/v7" ]; then export TARGETPLATFORM=arm-linux-gnueabihf; fi \
  && SIGNERS=($(curl -sS "https://api.github.com/repos/bitcoin-core/guix.sigs/contents/${BITCOIN_VERSION}" | jq -r '.[] | select(.type=="dir") | .name' | tr "\n" " ")) \
  && for name in "${SIGNERS[@]}"; do \
  curl -sSL "https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/${name}.gpg" | gpg --import - ; \
  done \
  && curl -SLO https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/bitcoin-${BITCOIN_VERSION}-${TARGETPLATFORM}.tar.gz \
  && curl -SLO https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS \
  && curl -SLO https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS.asc \
  && gpg --verify SHA256SUMS.asc SHA256SUMS \
  && grep " bitcoin-${BITCOIN_VERSION}-${TARGETPLATFORM}.tar.gz" SHA256SUMS | sha256sum -c - \
  && tar -xzf ./*.tar.gz -C /opt \
  && rm ./*.tar.gz ./*.asc \
  && rm -rf "/opt/bitcoin-${BITCOIN_VERSION}/bin/bitcoin-qt" \
  && rm -rf "/opt/bitcoin-${BITCOIN_VERSION}/bin/test_bitcoin"
  
COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/bitcoin/.bitcoin"]

EXPOSE 8332 8333 18332 18333 18443 18444 38333 38332

ENTRYPOINT ["/entrypoint.sh"]

RUN bitcoind -version | grep "Bitcoin Core version v${BITCOIN_VERSION}"

CMD ["bitcoind"]

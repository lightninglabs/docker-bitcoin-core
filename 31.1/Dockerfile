FROM debian:bookworm-slim

ARG UID=101
ARG GID=101

LABEL maintainer.0="João Fonseca (@joaopaulofonseca)" \
  maintainer.1="Pedro Branco (@pedrobranco)" \
  maintainer.2="Rui Marinho (@ruimarinho)"

RUN groupadd --gid ${GID} bitcoin \
  && useradd --create-home --no-log-init -u ${UID} -g ${GID} bitcoin \
  && apt-get update -y \
  && apt-get install -y curl gnupg gosu \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Special variables just for release candidates, both should be empty for stable
# releases.
ARG RC
ARG RC_DIR

# Bitcoin Core release-signing (builder) keys are imported from the bitcoin-core/guix.sigs
# repository rather than public keyservers, which do not reliably carry every builder's key
# (e.g. keys.openpgp.org strips User IDs until the owner verifies their email, and GnuPG then
# refuses the UID-less key). Pinned to a specific commit so the trusted key set is reproducible
# and auditable; bump this when a release is signed by a key added after this commit.
ARG GUIX_SIGS_COMMIT=c69655407bb9d47d6e1cae6a30b7d4c1ca78445e

ARG TARGETPLATFORM
ENV BITCOIN_VERSION=31.1
ENV BITCOIN_DATA=/home/bitcoin/.bitcoin
ENV PATH=/opt/bitcoin-${BITCOIN_VERSION}${RC}/bin:$PATH

RUN set -ex \
  && if [ "${TARGETPLATFORM}" = "linux/amd64" ]; then export TARGETPLATFORM=x86_64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm64" ]; then export TARGETPLATFORM=aarch64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm/v7" ]; then export TARGETPLATFORM=arm-linux-gnueabihf; fi \
  && curl -SL "https://github.com/bitcoin-core/guix.sigs/archive/${GUIX_SIGS_COMMIT}.tar.gz" | tar -xz \
  && gpg --batch --import "guix.sigs-${GUIX_SIGS_COMMIT}"/builder-keys/* \
  && rm -rf "guix.sigs-${GUIX_SIGS_COMMIT}" \
  && curl -SLO https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/${RC_DIR}bitcoin-${BITCOIN_VERSION}${RC}-${TARGETPLATFORM}.tar.gz \
  && curl -SLO https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/${RC_DIR}SHA256SUMS \
  && curl -SLO https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/${RC_DIR}SHA256SUMS.asc \
  && gpg --verify SHA256SUMS.asc SHA256SUMS \
  && grep " bitcoin-${BITCOIN_VERSION}${RC}-${TARGETPLATFORM}.tar.gz" SHA256SUMS | sha256sum -c - \
  && tar -xzf *.tar.gz -C /opt \
  && rm *.tar.gz *.asc \
  && rm -rf /opt/bitcoin-${BITCOIN_VERSION}${RC}/bin/bitcoin-qt

COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/bitcoin/.bitcoin"]

EXPOSE 8332 8333 18332 18333 18443 18444 38333 38332

ENTRYPOINT ["/entrypoint.sh"]

RUN bitcoind -version | grep "Bitcoin Core daemon version v${BITCOIN_VERSION}"

CMD ["bitcoind"]

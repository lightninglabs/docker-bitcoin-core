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

ARG TARGETPLATFORM
ENV BITCOIN_VERSION=29.0
ENV BITCOIN_DATA=/home/bitcoin/.bitcoin
ENV PATH=/opt/bitcoin-${BITCOIN_VERSION}${RC}/bin:$PATH

RUN set -ex \
  && if [ "${TARGETPLATFORM}" = "linux/amd64" ]; then export TARGETPLATFORM=x86_64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm64" ]; then export TARGETPLATFORM=aarch64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm/v7" ]; then export TARGETPLATFORM=arm-linux-gnueabihf; fi \
  && for key in \
  101598DC823C1B5F9A6624ABA5E0907A0380E6C3 \
  6A8F9C266528E25AEB1D7731C2371D91CB716EA7 \
  2840EAABF4BC9F0FFD716AFAFBAFCC46DE2D3FE2 \
  E86AE73439625BBEE306AAE6B66D427F873CB1A3 \
  A0083660F235A27000CD3C81CE6EC49945C17EA6 \
  F19F5FF2B0589EC341220045BA03F4DBE0C63FB4 \
  637DB1E23370F84AFF88CCE03152347D07DA627C \
  ED9BDF7AD6A55E232E84524257FF9BDBCC301009 \
  CFB16E21C950F67FA95E558F2EEB9F5CC09526C1 \
  152812300785C96444D3334D17565732E08E5E41 \
  C388F6961FB972A95678E327F62711DBDCA8AE56 \
  9DEAE0DC7063249FB05474681E4AED62986CD25D \
  D1DBF2C4B96F2DEBF4C16654410108112E7EA81F \
  F4FC70F07310028424EFC20A8E4256593F177720 \
  E61773CD6E01040E2F1BD78CE7E2984B6289C93A \
  9D3CC86A72F8494342EA5FD10A41BDC3F4FAFF1C \
  33C103B4B2794170546CCF7BCFB2C83C66CD792A \
  0CCBAAFD76A2ECE2CCD3141DE2FFD5B1D88CA97D \
  F2CFC4ABD0B99D837EEBB7D09B79B45691DB4173 \
  ; do \
  gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" || \
  gpg --batch --keyserver keys.openpgp.org --recv-keys "$key" || \
  gpg --batch --keyserver pgp.mit.edu --recv-keys "$key" || \
  gpg --batch --keyserver keyserver.pgp.com --recv-keys "$key" || \
  gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" || \
  gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" ; \
  done \
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

# Build stage for Bitcoin Core
FROM alpine as bitcoin-core

ENV GNUPGHOME=/tmp/gnupg

RUN sed -i 's/http\:\/\/dl-cdn.alpinelinux.org/https\:\/\/alpine.global.ssl.fastly.net/g' /etc/apk/repositories
RUN apk --no-cache add cmake
RUN apk --no-cache add boost-dev
RUN apk --no-cache add build-base
RUN apk --no-cache add chrpath
RUN apk --no-cache add file
RUN apk --no-cache add gnupg
RUN apk --no-cache add libevent-dev
RUN apk --no-cache add libressl
RUN apk --no-cache add libtool
RUN apk --no-cache add linux-headers
RUN apk --no-cache add sqlite-dev
RUN apk --no-cache add zeromq-dev
RUN mkdir -p ${GNUPGHOME}

# Bitcoin Core release-signing (builder) keys are imported from the bitcoin-core/guix.sigs
# repository rather than public keyservers, which do not reliably carry every builder's key
# (e.g. keys.openpgp.org strips User IDs until the owner verifies their email, and GnuPG then
# refuses the UID-less key). Pinned to a specific commit so the trusted key set is reproducible
# and auditable; bump this when a release is signed by a key added after this commit.
ARG GUIX_SIGS_COMMIT=c69655407bb9d47d6e1cae6a30b7d4c1ca78445e
RUN set -ex \
  && wget -qO - "https://github.com/bitcoin-core/guix.sigs/archive/${GUIX_SIGS_COMMIT}.tar.gz" | tar -xz \
  && gpg --batch --import "guix.sigs-${GUIX_SIGS_COMMIT}"/builder-keys/* \
  && rm -rf "guix.sigs-${GUIX_SIGS_COMMIT}"

# Special variables just for release candidates, both should be empty for stable
# releases.
ARG RC
ARG RC_DIR

ENV BITCOIN_VERSION=31.1
ENV BITCOIN_PREFIX=/opt/bitcoin-${BITCOIN_VERSION}${RC}

RUN wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/${RC_DIR}SHA256SUMS
RUN wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/${RC_DIR}SHA256SUMS.asc
RUN wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/${RC_DIR}bitcoin-${BITCOIN_VERSION}${RC}.tar.gz
RUN gpg --verify SHA256SUMS.asc SHA256SUMS
RUN grep " bitcoin-${BITCOIN_VERSION}${RC}.tar.gz\$" SHA256SUMS | sha256sum -c -
RUN tar -xzf *.tar.gz

WORKDIR /bitcoin-${BITCOIN_VERSION}${RC}

RUN cmake -B build \
    -DWITH_QRENCODE=OFF \
    -DBUILD_TESTS=OFF \
    -DWITH_ZMQ=ON \
    -DENABLE_IPC=OFF \
    -DCMAKE_INSTALL_PREFIX=${BITCOIN_PREFIX}
RUN cmake --build build -j4
RUN cmake --install build
RUN strip ${BITCOIN_PREFIX}/bin/bitcoin-cli
RUN strip ${BITCOIN_PREFIX}/bin/bitcoind

# Build stage for compiled artifacts
FROM alpine

ARG UID=100
ARG GID=101
ARG RC
ARG RC_DIR

LABEL maintainer.0="João Fonseca (@joaopaulofonseca)" \
  maintainer.1="Pedro Branco (@pedrobranco)" \
  maintainer.2="Rui Marinho (@ruimarinho)"

RUN addgroup bitcoin --gid ${GID} --system
RUN adduser --uid ${UID} --system bitcoin --ingroup bitcoin
RUN sed -i 's/http\:\/\/dl-cdn.alpinelinux.org/https\:\/\/alpine.global.ssl.fastly.net/g' /etc/apk/repositories
RUN apk --no-cache add \
  boost-filesystem \
  boost-system \
  boost-thread \
  gosu \
  libevent \
  libzmq \
  shadow \
  sqlite-dev \
  su-exec

# Special variables just for release candidates, both should be empty for stable
# releases.
ENV RC=${RC}
ENV RC_DIR=${RC_DIR}

ENV BITCOIN_DATA=/home/bitcoin/.bitcoin
ENV BITCOIN_VERSION=31.1
ENV BITCOIN_PREFIX=/opt/bitcoin-${BITCOIN_VERSION}${RC}
ENV PATH=${BITCOIN_PREFIX}/bin:$PATH

COPY --from=bitcoin-core /opt /opt
COPY docker-entrypoint.sh /entrypoint.sh
RUN sed -i '1s|/bin/bash|/bin/sh|' /entrypoint.sh

VOLUME ["/home/bitcoin/.bitcoin"]

EXPOSE 8332 8333 18332 18333 18444

ENTRYPOINT ["/entrypoint.sh"]

RUN bitcoind -version | grep "Bitcoin Core daemon version v${BITCOIN_VERSION}"

CMD ["bitcoind"]

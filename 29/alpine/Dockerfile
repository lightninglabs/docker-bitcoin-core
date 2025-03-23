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
RUN set -ex \
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
  gpg --batch --keyserver keyserver.pgp.com --recv-keys "$key" || \
  gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" || \
  gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" ; \
  done

# Special variables just for release candidates, both should be empty for stable
# releases.
ARG RC
ARG RC_DIR

ENV BITCOIN_VERSION=29.0
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
    -DCMAKE_INSTALL_PREFIX=${BITCOIN_PREFIX}
RUN cmake --build build -j4
RUN cmake --install build
RUN strip ${BITCOIN_PREFIX}/bin/bitcoin-cli
RUN strip ${BITCOIN_PREFIX}/bin/bitcoind

# Build stage for compiled artifacts
FROM alpine

ARG UID=100
ARG GID=101

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
  libevent \
  libzmq \
  shadow \
  sqlite-dev \
  su-exec

# Special variables just for release candidates, both should be empty for stable
# releases.
# ENV RC=
# ENV RC_DIR=

ENV BITCOIN_DATA=/home/bitcoin/.bitcoin
ENV BITCOIN_VERSION=29.0
ENV BITCOIN_PREFIX=/opt/bitcoin-${BITCOIN_VERSION}${RC}
ENV PATH=${BITCOIN_PREFIX}/bin:$PATH

COPY --from=bitcoin-core /opt /opt
COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/bitcoin/.bitcoin"]

EXPOSE 8332 8333 18332 18333 18444

ENTRYPOINT ["/entrypoint.sh"]

RUN bitcoind -version | grep "Bitcoin Core daemon version v${BITCOIN_VERSION}"

CMD ["bitcoind"]

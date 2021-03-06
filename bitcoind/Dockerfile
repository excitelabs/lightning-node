# builder image for extracting and downloading bitcoin binaries
FROM debian:stretch-slim as builder

RUN set -ex \
    && apt-get update \
    && apt-get install -qq --no-install-recommends ca-certificates dirmngr gpg wget \
    && rm -rf /var/lib/apt/lists/*

ENV BITCOIN_VERSION 0.15.1
ENV BITCOIN_URL https://bitcoin.org/bin/bitcoin-core-0.15.1/bitcoin-0.15.1-x86_64-linux-gnu.tar.gz
ENV BITCOIN_SHA256 387c2e12c67250892b0814f26a5a38f837ca8ab68c86af517f975a2a2710225b
ENV BITCOIN_ASC_URL https://bitcoin.org/bin/bitcoin-core-0.15.1/SHA256SUMS.asc
ENV BITCOIN_PGP_KEY 01EA5486DE18A882D4C2684590C8019E36C2E964

RUN set -ex \
    && cd /tmp \
    && gpg --keyserver keyserver.ubuntu.com --recv-keys "$BITCOIN_PGP_KEY" \
    && wget -qO bitcoin.tar.gz "$BITCOIN_URL" \
    && echo "$BITCOIN_SHA256 bitcoin.tar.gz" | sha256sum -c - \
    && wget -qO bitcoin.asc "$BITCOIN_ASC_URL" \
    && gpg --verify bitcoin.asc \
    && tar -xzvf bitcoin.tar.gz -C /usr/local --strip-components=1 --exclude=*-qt \
    && rm -rf /tmp/*

# image for running bitcoin binaries
FROM debian:stretch-slim

ENV BITCOIN_DATA /data

RUN set -ex \
    && apt-get update \
    && apt-get install -qq --no-install-recommends gosu \
    && rm -rf /var/lib/apt/lists/* \
    && groupadd -r bitcoin && useradd -r -m -g bitcoin bitcoin \
    && mkdir "$BITCOIN_DATA" \
    && chown -R bitcoin:bitcoin "$BITCOIN_DATA" \
    && ln -sfn "$BITCOIN_DATA" /home/bitcoin/.bitcoin \
    && chown -h bitcoin:bitcoin /home/bitcoin/.bitcoin

COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /usr/local/lib /usr/local/lib
COPY docker-entrypoint.sh /entrypoint.sh

VOLUME /data
ENTRYPOINT ["/entrypoint.sh"]
EXPOSE 8332 8333 18332 18333
CMD ["bitcoind"]
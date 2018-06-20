FROM alpine:3.7

LABEL maintainer="a.v.galushko86@gmail.com"
RUN set -xe \
    && apk add --update --no-cache \
    ca-certificates \
    curl \
    openssh-client \
    libmemcached-libs \
    libevent \
    libssl1.0 \
    musl \
    yaml \
    supervisor \
    nodejs \
    nodejs-npm \
    git \
    && apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing/ --allow-untrusted \
    gnu-libiconv
RUN mkdir -p /opt/parsoid \
    && git clone https://gerrit.wikimedia.org/r/p/mediawiki/services/parsoid /opt/parsoid \
    && cd /opt/parsoid \
    && npm install \
    && npm install -g pm2
COPY config.example.yaml /opt/parsoid/config.yaml
WORKDIR /opt/parsoid

ENTRYPOINT /usr/bin/node /opt/parsoid/bin/server.js
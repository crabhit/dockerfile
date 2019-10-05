FROM golang:1.13.1-alpine3.10 AS builder

ENV GO111MODULE on
ENV GOPROXY https://goproxy.io
ENV GOSUMDB sum.golang.google.cn

RUN apk upgrade \
    && apk add git \
    && go get github.com/shadowsocks/go-shadowsocks2

FROM alpine:3.10 AS dist

LABEL maintainer="mritd <mritd@linux.com>"

ARG TZ="Asia/Shanghai"

ENV TZ ${TZ}
ENV OBFS_DOWNLOAD_URL https://github.com/shadowsocks/simple-obfs.git

RUN apk upgrade --update \
    && apk add tzdata \
    && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && rm -rf /var/cache/apk/* \
    && apk add --virtual .build-deps \
        asciidoc \
        autoconf \
        automake \
        g++ \
        gcc \
        libev-dev \
        libpcre32 \
        libtool \
        linux-headers \
        make \
        openssl \
        xmlto \
        zlib-dev \
        git \
    && git clone ${OBFS_DOWNLOAD_URL} \
    && (cd simple-obfs \
    && git submodule update --init --recursive \
    && ./autogen.sh && ./configure --disable-documentation\
    && make && make install) \
    && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo $TZ > /etc/timezone \
    && runDeps="$( \
        scanelf --needed --nobanner /usr/local/bin/obfs-* \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | xargs -r apk info --installed \
            | sort -u \
        )" \
    && apk add --virtual .run-deps $runDeps \
    && apk del .build-deps \
    && rm -rf simple-obfs \
        /var/cache/apk/*

COPY --from=builder /go/bin/go-shadowsocks2 /usr/bin/shadowsocks
COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

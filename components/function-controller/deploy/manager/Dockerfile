# image builder base on golang:1.19.1-alpine3.16
FROM eu.gcr.io/kyma-project/external/golang@sha256:ca4f0513119dfbdc65ae7b76b69688f0723ed00d9ecf9de68abbf6ed01ef11bf as builder

ENV BASE_APP_DIR=/workspace/go/src/github.com/kyma-project/kyma/components/function-controller \
    CGO_ENABLED=1 \
    GOOS=linux \
    GOARCH=amd64 \
    LIBGIT2_VERSION=1.1.0-r2

RUN apk add --no-cache gcc libc-dev
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.14/community libgit2-dev=${LIBGIT2_VERSION}

WORKDIR ${BASE_APP_DIR}

#
# copy files allowed in .dockerignore
#
COPY . ${BASE_APP_DIR}/

RUN go build -ldflags "-s -w" -a -o manager cmd/manager/main.go \
    && mkdir /app \
    && mv ./manager /app/manager

# get latest CA certs from alpine:3.16.2
FROM eu.gcr.io/kyma-project/external/alpine@sha256:1304f174557314a7ed9eddb4eab12fed12cb0cd9809e4c28f29af86979a3c870 as certs

RUN apk add --no-cache ca-certificates

# result container base on alpine:3.16.2
FROM eu.gcr.io/kyma-project/external/alpine@sha256:1304f174557314a7ed9eddb4eab12fed12cb0cd9809e4c28f29af86979a3c870

ENV LIBGIT2_VERSION=1.1.0-r2

RUN apk update --no-cache && apk upgrade --no-cache
RUN apk add --no-cache --update --repository=http://dl-cdn.alpinelinux.org/alpine/edge/main openssl
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.14/community libgit2=${LIBGIT2_VERSION}

LABEL source = git@github.com:kyma-project/kyma.git

COPY --from=builder /app /app
COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt

ENTRYPOINT ["/app/manager"]


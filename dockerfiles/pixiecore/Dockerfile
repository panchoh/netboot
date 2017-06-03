FROM alpine:3.20 AS build

LABEL MAINTAINER David Anderson <dave@natulte.net>

ARG PIXIECORE_SANDBOX=/tmp/sandbox
ARG PIXIECORE_CONTEXT="$PIXIECORE_SANDBOX"/context

COPY . "$PIXIECORE_CONTEXT"

# https://docs.docker.com/build/building/best-practices/
# https://docs.docker.com/build/building/multi-stage/
# TODO: go get can no longer be used outside a module, must rething the build logic.
RUN set -uxe;                                                                   \
                                                                                \
    apk add --no-cache --virtual .build-deps                                    \
        git                                                                     \
        go                                                                      \
        musl-dev;                                                               \
                                                                                \
    # Pixiecore assets                                                          \
    NAMESPACE=go.universe.tf;                                                   \
    REPO=netboot;                                                               \
    PKG=cmd/pixiecore;                                                          \
                                                                                \
    export GOPATH="$PIXIECORE_SANDBOX"/go;                                      \
                                                                                \
    NAMESPACE_PATH="$GOPATH/src/$NAMESPACE";                                    \
    REPO_PATH="$NAMESPACE_PATH/$REPO";                                          \
    PKG_PATH="$REPO_PATH/$PKG";                                                 \
                                                                                \
    if [[ -d "$PIXIECORE_CONTEXT"/"$PKG" ]]; then                               \
        echo "Building from local dev copy";                                    \
        mkdir -p "$NAMESPACE_PATH";                                             \
        mv -v "$PIXIECORE_CONTEXT" "$REPO_PATH";                                \
    else                                                                        \
        echo "Building from upstream git repo";                                 \
    fi;                                                                         \
                                                                                \
    # go get -v -d "$NAMESPACE/$REPO/$PKG@latest";                              \
    cd "$REPO_PATH";                                                            \
    GOBIN=/ go install -ldflags -s ./"$PKG";                                    \
    apk del --purge .build-deps;

FROM alpine:3.20

RUN apk add --no-cache ca-certificates

COPY --from=build /pixiecore /usr/local/bin

ENTRYPOINT ["/usr/local/bin/pixiecore"]

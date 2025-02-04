VERSION 0.6
FROM alpine
ARG BUNDLE
ARG VERSION="latest"
ARG IMAGE_REPOSITORY=quay.io/kairos/community-bundles

version:
    FROM alpine
    RUN apk add git

    COPY . ./

    RUN echo $(git describe --exact-match --tags || echo "v0.0.0-$(git log --oneline -n 1 | cut -d" " -f1)") > VERSION

    SAVE ARTIFACT VERSION VERSION

build:
    COPY +version/VERSION ./
    ARG VERSION=$(cat VERSION)
    FROM DOCKERFILE -f ${BUNDLE}/Dockerfile ./${BUNDLE}
    SAVE IMAGE --push $IMAGE_REPOSITORY:${BUNDLE}_${VERSION}

rootfs:
    FROM +build
    SAVE ARTIFACT /. rootfs

test:
    FROM quay.io/kairos/core-alpine-opensuse-leap
    RUN apk add go
    ENV GOPATH=/go
    ENV PATH=$PATH:$GOPATH/bin
    COPY (+rootfs/rootfs --BUNDLE=$BUNDLE) /bundle
    COPY . .
    RUN cd tests && \ 
        go install -mod=mod github.com/onsi/ginkgo/v2/ginkgo && \
        ginkgo --label-filter="${BUNDLE}" -v ./...

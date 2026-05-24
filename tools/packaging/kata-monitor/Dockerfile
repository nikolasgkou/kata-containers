# Copyright (c) 2020 Eric Ernst
# Copyright (c) 2026 Red Hat, Inc.
# SPDX-License-Identifier: Apache-2.0
#
# Build the kata-monitor image from the repository root:
#
#   docker buildx build \
#       --platform linux/amd64,linux/arm64,linux/s390x,linux/ppc64le \
#       -f tools/packaging/kata-monitor/Dockerfile \
#       -t <registry>/kata-monitor:<tag> .
#
# The build is fully driven by the in-tree sources under src/runtime, so it
# does not depend on any pre-built kata-static tarball.
#
# Note: `kata-monitor` indirectly imports `virtcontainers`, which contains
# CGO-using files for arm64/s390x. The builder stage therefore deliberately
# does NOT pin `--platform=$BUILDPLATFORM`: buildx + QEMU run each target
# platform's build natively under emulation, ensuring the right per-arch
# files are compiled against the matching kernel headers and toolchain.

ARG GOLANG_IMAGE=golang:1.25-alpine
ARG ALPINE_IMAGE=alpine:3.22

FROM ${GOLANG_IMAGE} AS builder

# Optional: passed in by CI so that `kata-monitor --version` reports the
# commit it was built from. When unset the resulting binary will simply
# report an empty commit, matching the pre-existing behaviour of building
# from a context without a git tree.
ARG GIT_COMMIT=""

RUN apk add --no-cache bash git make build-base linux-headers

WORKDIR /go/src/github.com/kata-containers/kata-containers/src/runtime

# Only copy what is actually needed to build the kata-monitor binary so
# that the build context stays small and changes to unrelated files do
# not invalidate the layer cache.
COPY src/runtime /go/src/github.com/kata-containers/kata-containers/src/runtime

RUN \
	set -eux; \
	mkdir -p /out; \
	go build \
		-buildmode=pie \
		-ldflags "-X main.GitCommit=${GIT_COMMIT}" \
		-o /out/kata-monitor ./cmd/kata-monitor

FROM ${ALPINE_IMAGE}

RUN apk add --no-cache ca-certificates

COPY --from=builder /out/kata-monitor /usr/bin/kata-monitor

EXPOSE 8090

ENTRYPOINT ["/usr/bin/kata-monitor"]
CMD ["--help"]

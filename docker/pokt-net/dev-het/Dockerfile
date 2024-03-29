# Prepares a dockerfile to run a pocket node whose source code will be mounted
# from the host for local development and hot reloading.

# Dynamically pull Go-lang version for the image
ARG GOLANG_IMAGE_VERSION=golang:1.16.2-alpine3.13

# First build step to build the app binary
FROM ${GOLANG_IMAGE_VERSION} AS builder

ARG POCKET_E2E_STACK_RELATIVE_PATH
ARG POCKET_CORE_REPO_RELATIVE_PATH
ARG POCKET_NETWORK_TENDERMINT_RELATIVE_PATH

# Install dependencies
RUN apk -v --update --no-cache add \
	curl \
	git \
	groff \
	less \
	mailcap \
	gcc \
	libc-dev \
	bash  \
	leveldb-dev && \
	rm /var/cache/apk/* || true

# Environment and system dependencies setup
ENV POCKET_ROOT_TEMP=/go/src/github.com/temp/pokt-network
ENV POCKET_ROOT=/go/src/github.com/pokt-network
ENV POCKET_PATH=${POCKET_ROOT}/pocket-core
ENV TENDERMINT_PATH=${POCKET_ROOT}/tendermint
ENV GO111MODULE="on"
ENV EXECCOMMAND="start --keybase=false --datadir=/home/app/.pocket"
ENV POCKET_CORE_SEEDS="notparsed"
ENV POCKET_CORE_KEY="nokeyisparsed"
ENV POCKET_CORE_ACCOUNT_ADDRESS="noaddressisparsed"

# Create directories for the source code.
RUN mkdir -p ${POCKET_ROOT_TEMP}
RUN mkdir -p ${POCKET_ROOT}
RUN mkdir -p ${POCKET_PATH}
RUN mkdir -p ${TENDERMINT_PATH}
WORKDIR $POCKET_ROOT

# Copy the source code from the host.
COPY ${POCKET_CORE_REPO_RELATIVE_PATH} ${POCKET_PATH}
COPY ${POCKET_NETWORK_TENDERMINT_RELATIVE_PATH} ${TENDERMINT_PATH}

# Copying deps and hot reloading/watch script
COPY ${POCKET_E2E_STACK_RELATIVE_PATH}/watch.sh ./watch.sh
COPY ${POCKET_E2E_STACK_RELATIVE_PATH}/prepare-tendermint.sh ./
COPY ${POCKET_E2E_STACK_RELATIVE_PATH}/entrypoint-dev.sh /home/app/entrypoint.sh
COPY ${POCKET_E2E_STACK_RELATIVE_PATH}/command.sh /home/app/command.sh
COPY ${POCKET_E2E_STACK_RELATIVE_PATH}/local_command_dev.sh /home/app/local_command_dev.sh

RUN apk add --update --no-cache expect bash leveldb-dev
RUN addgroup -S app \
	&& adduser -S -G app app
RUN mkdir -p /home/app/.pocket/config
RUN chown -R app /home/app/.pocket
RUN chown -R app /go

RUN cp -rT ${POCKET_ROOT_TEMP} ${POCKET_ROOT}

RUN cd ${POCKET_PATH} \
	&& go build -tags cleveldb -o ${GOPATH}/bin/pocket app/cmd/pocket_core/main.go

# Overwrite go.mod files for local development.
COPY ${POCKET_E2E_STACK_RELATIVE_PATH}/go.mod.local $POCKET_ROOT/go.mod.core
RUN chmod 654 $POCKET_ROOT/go.mod.core

COPY ${POCKET_E2E_STACK_RELATIVE_PATH}/go.mod.tendermint $POCKET_ROOT/go.mod.tendermint
RUN chmod 654 $POCKET_ROOT/go.mod.tendermint

# Install reflex and dlv
RUN go get github.com/cespare/reflex
RUN go get github.com/go-delve/delve/cmd/dlv@latest

ENTRYPOINT ["/home/app/entrypoint.sh"]

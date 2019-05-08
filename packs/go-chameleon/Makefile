SHELL := /bin/bash
GO := GO15VENDOREXPERIMENT=1 go
NAME := REPLACE_ME_APP_NAME
OS := $(shell uname)
ROOT_PACKAGE := $(GIT_PROVIDER)/REPLACE_ME_ORG/$(NAME)
GO_VERSION := $(shell $(GO) version | sed -e 's/^[^0-9.]*\([0-9.]*\).*/\1/')
GOOS := linux
GOARCH := amd64
PKG := github.com/virtualinstruments/${NAME}
MAIN_GO := ${PKG}/cmd/${NAME}/...
PACKAGE_DIRS := $(shell $(GO) list ./... | grep -v /vendor/)
PKGS := $(shell go list ./... | grep -v /vendor | grep -v generated)
PKGS := $(subst  :,_,$(PKGS))
BUILDFLAGS := '-s -a -installsuffix cgo -i'
CGO_ENABLED = 0

# docker vars
DOCKER_REGISTRY := 189688312005.dkr.ecr.us-west-2.amazonaws.com/
IMAGE_NAME := ${NAME}
IMAGE_TAG := $(shell git branch | sed -n -e 's/^\* \(.*\)/\1/p')
SERVICE_PORT := 8000

all: build

check: fmt build test

build:
	@protoc -I api/ \
		-I${GOPATH}/src \
		-I${GOPATH}/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
		--go_out=plugins=grpc:api \
		api/*.proto
	CGO_ENABLED=$(CGO_ENABLED) GOOS=${GOOS} GOARCH=${GOARCH} $(GO) build -ldflags $(BUILDFLAGS) -o bin/$(NAME) ${MAIN_GO}
	@if ! [ -d ${MOCK_DIR} ]; then \
		mkdir ${MOCK_DIR}; \
	fi
	@go generate ${PKG}/...

test:
	CGO_ENABLED=$(CGO_ENABLED) $(GO) test -cover ${PKG}/...

install:
	GOBIN=${GOPATH}/bin $(GO) install -ldflags $(BUILDFLAGS) $(MAIN_GO)

fmt:
	@FORMATTED=`$(GO) fmt $(PACKAGE_DIRS)`
	@([[ ! -z "$(FORMATTED)" ]] && printf "Fixed unformatted files:\n$(FORMATTED)") || true

clean:
	rm -rf build release

linux:
	CGO_ENABLED=$(CGO_ENABLED) GOOS=linux GOARCH=amd64 $(GO) build -ldflags $(BUILDFLAGS) -o bin/$(NAME) $(MAIN_GO)

.PHONY: release clean

docker:
	docker build -t ${DOCKER_REGISTRY}${IMAGE_NAME}:${IMAGE_TAG} -f Dockerfile .

run-docker-it:
	docker run -it \
		--name ${NAME} \
		--env-file env.test \
		-p ${SERVICE_PORT}:${SERVICE_PORT} \
		${DOCKER_REGISTRY}${IMAGE_NAME}:${IMAGE_TAG}

run-docker:
	docker run -d \
		--name ${NAME} \
		--env-file env \
		-p ${SERVICE_PORT}:${SERVICE_PORT} \
		${DOCKER_REGISTRY}${IMAGE_NAME}:${IMAGE_TAG}



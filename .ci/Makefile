# This makefile contains targets used by the CI
include common.mk

ABS_PATH = $(abspath $(shell pwd)/..)

#
# Create a container image image used to build the go code.
#
# We're building a hash of that list of dependencies (Go,...). That should give us a deterministic
# version identifier for any particular list of dependencies -- if, with a future agent version,
# the dependencies change, then that hash will change. We can use that to minimize downloads
# from external sources, and maximise re-use of the "dependencies" image for as long as dependencies
# remain stable.
#

BASE_IMAGE_BUILDER = ubuntu:20.04

# Compute hash to detect any changes and rebuild /push the image
DEPEND_HASH = $(shell echo "${BASE_IMAGE_BUILDER}-DOCKERFILE:$(shell md5sum Dockerfile.depend)" | md5sum | cut -f1 -d' ')
DEPEND_IMAGE = ${DEPEND_BASE}:${DEPEND_HASH}

ifdef CI_BUILD
        PUSH_IMAGE = docker image push ${DEPEND_IMAGE}
else
        PUSH_IMAGE = echo not pushing image
endif


.PHONY: depend_image
depend-image: ## Make dependencies image. (Not required normally; is implied in making any other container image).
	# Try to pull an existing dependencies image; it's OK if none exists yet.
	@echo Building depend image
	docker image pull ${DEPEND_IMAGE} || /bin/true
	docker image inspect ${DEPEND_IMAGE} >/dev/null 2>/dev/null \
          || ( docker image build \
               -f ./Dockerfile.depend \
               --build-arg BASE_IMAGE=${BASE_IMAGE_BUILDER} \
               --tag ${DEPEND_IMAGE} \
	       ${ABS_PATH}/ \
	       && ${PUSH_IMAGE} )
	docker tag ${DEPEND_IMAGE} ${DEPEND_BASE}:latest

#
# CI targets
#
.PHONY: go-check ci-images install
go-check: depend-image
	@echo Checking go code
	docker run -t --rm --name build_temp -v ${ABS_PATH}:/vpp-dataplane ${DEPEND_IMAGE} make -C /vpp-dataplane go-check

ci-install:
	git config --global user.email "calicoci@cisco.com"
	git config --global user.name "Calico CI"

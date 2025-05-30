#!/usr/bin/make -f

include ../scripts/check.mk

PACKER ?= packer
PACKER_LOG ?= 0
ISO ?= rhel-baseos-8.0-x86_64-dvd.iso
TIMEOUT ?= 1h
ARCH ?= x86_64

# Detect if running on ARM host
ifeq ($(shell uname -m),aarch64)
    HOST_IS_ARM = true
else
    HOST_IS_ARM = false
endif

ifeq ($(wildcard /usr/share/OVMF/OVMF_CODE.fd),)
	OVMF_SFX ?= _4M
else
	OVMF_SFX ?=
endif

export PACKER_LOG

# Fallback
ifeq ($(strip $(ARCH)),amd64)
	ARCH = x86_64
endif

.PHONY: all clean

all: rhel8.tar.gz

$(eval $(call check_packages_deps))

lint:
	packer validate .
	packer fmt -check -diff .

format:
	packer fmt .

OVMF_VARS.fd: /usr/share/OVMF/OVMF_VARS${OVMF_SFX}.fd
	cp -v $< ${ARCH}_VARS.fd

SIZE_VARS.fd:
ifeq ($(strip $(ARCH)),aarch64)
	truncate -s 64m ${ARCH}_VARS.fd
else
	truncate -s 2m ${ARCH}_VARS.fd
endif

rhel8.tar.gz: check-deps clean OVMF_VARS.fd SIZE_VARS.fd
	${PACKER} init rhel8.pkr.hcl && ${PACKER} build \
							-var architecture=${ARCH} \
							-var host_is_arm=${HOST_IS_ARM} \
							-var ovmf_suffix=${OVMF_SFX} \
							-var "rhel8_iso_path=${ISO}" \
							-var timeout=${TIMEOUT} \
							rhel8.pkr.hcl

clean:
	${RM} -rf *.fd output-rhel8 rhel8.tar.gz

TOOLS_TOP_DIR := $(shell pwd)
TOOL_NAME := busybox-1.31.1
TOOL_TAR_BALL := $(TOOL_NAME).tar.bz2
TOOL_BIN := bin
ARCH ?= arm
OSDRV_DIR ?= ../../osdrv

BUSYBOX_CFG ?=
OSDRV_CROSS ?=arm-linux-gnueabi

all:
	-rm $(TOOLS_TOP_DIR)/$(TOOL_NAME) -rf;
	tar xjf $(TOOLS_TOP_DIR)/$(TOOL_TAR_BALL) -C $(TOOLS_TOP_DIR)
	pushd $(TOOLS_TOP_DIR)/$(TOOL_NAME)/;patch -p1 < ../0001-networking.patch;patch -p1 < ../0002-Makefile.flags.patch;popd
	cp $(TOOLS_TOP_DIR)/$(BUSYBOX_CFG) $(TOOLS_TOP_DIR)/$(TOOL_NAME)/.config
	pushd $(TOOLS_TOP_DIR)/$(TOOL_NAME)/; \
	make ARCH=$(ARCH) CROSS_COMPILE=$(OSDRV_CROSS)- -j 20 || exit "$$?" ;popd
	make CROSS_COMPILE=$(OSDRV_CROSS)- -C $(TOOLS_TOP_DIR)/$(TOOL_NAME) install || exit "$$?"
ifeq ($(OSDRV_DIR)/pub/$(PUB_ROOTFS), $(wildcard $(OSDRV_DIR)/pub/$(PUB_ROOTFS)))
	cp -af $(TOOLS_TOP_DIR)/$(TOOL_NAME)/_install/* $(OSDRV_DIR)/pub/$(PUB_ROOTFS)
endif

clean:
	-rm $(TOOLS_TOP_DIR)/$(TOOL_NAME) -rf;

distclean: clean



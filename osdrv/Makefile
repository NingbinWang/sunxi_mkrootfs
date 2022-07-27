##########################################################################################
#	prepare param
##########################################################################################
export OSDRV_DIR=$(shell pwd)

ifeq ($(OSDRV_DIR)/../open_source/, $(wildcard $(OSDRV_DIR)/../open_source/))
OPEN_SOURCE=$(OSDRV_DIR)/../open_source/
else
$(warning "---------------------------------------------------------------------")
$(warning "   Cannot found the open_source Contents!!       ")
$(warning "---------------------------------------------------------------------")
$(error )
endif

MAKE=make
export CHIP?=f1s200c

export ARCH=arm
export ARCH_TYPE = $(ARCH)
export OSDRV_CROSS?=arm-buildroot-linux-gnueabi
export CROSS_COMPILE=$(OSDRV_CROSS)-
export OSDRV_CROSS_CFLAGS=
export AMP_TYPE ?= linux
export MP_TYPE=smp
export MAKE=make

LIB_TYPE:=glibc
BUSYBOX_CFG:=config_sunxi_busybox
TOOLCHAIN_DIR:=
TOOLCHAINI_VERSION:=
TOOLCHAIN_LIB:=lib
TOOLCHAIN_LIBC:=libc
TOOLCHAIN_SYSROOT:=sysroot
CROSS_SPECIFIED:=y
TOOLCHAIN_DIRNAME:=gcc-arm-buildroot-linux-gnueabi


BUSYBOX_VER:=busybox-1.34.1
BUSYBOX_DIR := $(OPEN_SOURCE)/busybox
TOOLCHAIN_RUNTIME_LIB_C:=lib.tgz

TOOLCHAIN_RUNTIME_LIB:=
OSDRV_CROSS_CFLAGS:=

export PUB_BOARD:=board_$(LIB_TYPE)
export PUB_ROOTFS:=rootfs_$(LIB_TYPE)


XZ_DIR:= $(OPEN_SOURCE)/xz

LZMA_DIR:=$(OPEN_SOURCE)/lzma

EUDEV_VER:= eudev-3.2.9
EUDEV_DIR:=$(OPEN_SOURCE)/eudev
EUDEV_TAR:=$(EUDEV_VER).tar.gz

MTD_DIR:=$(OPEN_SOURCE)/mtd-utils/
MTD_VER:= mtd-utils-2.1.1
MTD_TAR:=$(MTD_VER).tar.bz2

ZLIB_VER:= zlib-1.2.11
ZLIB_DIR:= $(OPEN_SOURCE)/zlib
ZLIB_TAR:=$(ZLIB_VER).tar.gz

SQUASHFS_VER:= squashfs4.4
SQUASHFS_DIR:=$(OPEN_SOURCE)/squashfs
SQUASHFS_TAR:=$(SQUASHFS_VER).tar.gz

CRAMFS_VER:= util-linux-2.35.1
CRAMFS_DIR:=$(OPEN_SOURCE)/util-linux
CRAMFS_TAR:=$(CRAMFS_VER).tar.gz


TOOLCHAIN_FILE:= $(shell which $(OSDRV_CROSS)-gcc )
TOOLCHAIN_DIR:=$(shell dirname $(shell dirname $(TOOLCHAIN_FILE)))
TOOLCHAINLIB_DIR=$(shell dirname $(TOOLCHAIN_DIR))/$(TOOLCHAIN_DIRNAME)/$(OSDRV_CROSS)/$(TOOLCHAIN_LIB)
TOOLCHAINLIBC_DIR=$(shell dirname $(TOOLCHAIN_DIR))/$(TOOLCHAIN_DIRNAME)/$(OSDRV_CROSS)/$(TOOLCHAIN_LIBC)
TOOLCHAINSYSROOT_DIR=$(shell dirname $(TOOLCHAIN_DIR))/$(TOOLCHAIN_DIRNAME)/$(OSDRV_CROSS)/$(TOOLCHAIN_SYSROOT)
##########################################################################################
#	set task
##########################################################################################
ifeq ($(CROSS_SPECIFIED),y)
ifeq ($(MP_TYPE),amp)
all: prepare hiboot hisecureboot hikernel hiliteos hirootfs_prepare hiliteos_sample hibusybox \
	hipctools hiboardtools hirootfs_build
clean: hiboot_clean hikernel_clean hisecureboot_clean\
       hibusybox_clean hipctools_clean hiboardtools_clean hirootfs_clean hiliteos_dirclean 
else
all: prepare hiboot hisecureboot hikernel hirootfs_prepare hibusybox \
	hipctools hiboardtools hirootfs_build
clean: hiboot_clean hikernel_clean hisecureboot_clean\
       hibusybox_clean hipctools_clean hiboardtools_clean hirootfs_clean
endif
notools: hiboot hikernel hinotools_prepare hirootfs_notools_build
distclean: clean hibusybox_clean pub_clean hiboot_clean hikernel_clean 
endif

a:=$(shell $(OSDRV_CROSS)-gcc --version)
b:=$(findstring $(TOOLCHAINI_VERSION),$(a))
c:= $(word 2, $(a))
##########################################################################################
#task [0]	prepare
##########################################################################################
prepare:
	mkdir $(OSDRV_DIR)/pub/bin/$(PUB_BOARD) -p
	mkdir $(OSDRV_DIR)/pub/bin/pc -p


##########################################################################################
#task [1]	prepare rootfs
##########################################################################################
rootfs_prepare: prepare
	@echo "---------task [1] prepare rootfs "
	rm $(OSDRV_DIR)/pub/$(PUB_ROOTFS)* -rf
	tar xzf $(OSDRV_DIR)/rootfs_scripts/rootfs.tgz -C $(OSDRV_DIR)/pub
	mv $(OSDRV_DIR)/pub/rootfs $(OSDRV_DIR)/pub/$(PUB_ROOTFS)
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);popd
	cp $(TOOLCHAINLIB_DIR)/*so* $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/lib -d
	cp $(TOOLCHAIN_DIR)/lib/*so* $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/lib -d
	cp $(TOOLCHAINSYSROOT_DIR)/lib/*so* $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/lib -d
	#cp $(TOOLCHAINLIBC_DIR)/usr/lib/*so* $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/usr/lib -d
	#tar xzf $(RUNTIMELIB_DIR)/$(TOOLCHAIN_RUNTIME_LIB)/$(TOOLCHAIN_RUNTIME_LIB_C) -C $(OSDRV_DIR)/pub/$(PUB_ROOTFS)
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);$(CROSS_COMPILE)strip -d ./lib/*.so*;$(CROSS_COMPILE)strip -d ./usr/lib/*.so*;popd
	
##########################################################################################
#task [2]	build busybox
##########################################################################################
busybox: prepare
	@echo "---------task [2] build busybox "
	$(MAKE) -C $(BUSYBOX_DIR) LIB_TYPE=$(LIB_TYPE) \
				ARCH_TYPE=$(ARCH_TYPE) BUSYBOX_CFG=$(BUSYBOX_CFG) all

busybox_clean:
	$(MAKE) -C $(BUSYBOX_DIR) clean

##########################################################################################
#task [3]	build pc tools
##########################################################################################
pctools: prepare
	@echo -e $(GREEN)"---------task [3] build tools which run on pc"  $(DONE)
	$(MAKE) -C $(ZLIB_DIR) zlib_pc
	$(MAKE) -C $(MTD_DIR) mtd_pc
	$(MAKE) -C $(LZMA_DIR)

	cp $(MTD_DIR)/out/pc/mkfs.jffs2 $(OSDRV_DIR)/pub/bin/pc
	cp $(MTD_DIR)/out/pc/mkfs.ubifs $(OSDRV_DIR)/pub/bin/pc
	cp $(MTD_DIR)/out/pc/ubinize $(OSDRV_DIR)/pub/bin/pc

	$(MAKE) -C $(CRAMFS_DIR)
	cp $(CRAMFS_DIR)/out/pc/sbin/mkfs.cramfs $(OSDRV_DIR)/pub/bin/pc
	$(MAKE) -C $(XZ_DIR)
	$(MAKE) -C $(SQUASHFS_DIR)
	cp $(SQUASHFS_DIR)/out/pc/mksquashfs $(OSDRV_DIR)/pub/bin/pc
	cp $(LZMA_DIR)/bin/lzma $(OSDRV_DIR)/pub/bin/pc

pctools_clean:
	$(MAKE) -C $(ZLIB_DIR) distclean
	$(MAKE) -C $(MTD_DIR) distclean
	$(MAKE) -C $(SQUASHFS_DIR) distclean
	$(MAKE) -C $(CRAMFS_DIR) distclean
	$(MAKE) -C $(XZ_DIR) distclean
	$(MAKE) -C $(LZMA_DIR) distclean

##########################################################################################
#task [4]	build board tools
##########################################################################################
boardtools: rootfs_prepare
	@echo "---------task [4] build tools which run on board "

	$(MAKE) -C $(ZLIB_DIR) OSDRV_CROSS=$(OSDRV_CROSS) zlib_board
	$(MAKE) -C $(EUDEV_DIR) OSDRV_CROSS=$(OSDRV_CROSS)
	cp -af $(EUDEV_DIR)/rootfs/. $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/

	$(MAKE) -C $(MTD_DIR) OSDRV_CROSS=$(OSDRV_CROSS) mtd_board
	cp $(MTD_DIR)/out/board/bin/* $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)


boardtools_clean:
	make -C $(EUDEV_DIR) clean
	make -C $(ZLIB_DIR) clean
	make -C $(MTD_DIR) distclean

##########################################################################################
#task [5]	build rootfs
##########################################################################################
rootfs_build: rootfs_prepare busybox boardtools  pctools rootfs_notools_build
rootfs_notools_build:
	@echo "---------task [5] build rootfs"
	rm $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/dev/* -rf
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);ln -s sbin/init init;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod go-x * -R;chmod go-w * -R;chmod go-r * -R;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 4755 bin/su;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 4555 bin/login usr/bin/passwd;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 2555 usr/bin/wall;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 1777 tmp;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 755 bin etc lib opt proc usr usr/bin usr/bin/crontab sbin sbin/arp usr/lib var;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 750 boot etc/init.d etc/init.d/*;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 644 etc/group etc/inittab etc/mtab etc/passwd etc/profile etc/protocols etc/services etc/shadow root/.profile;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 640 var/run/utmp;popd;
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);chmod 550 bin/mount bin/netstat sbin/route;popd;

	pushd $(OSDRV_DIR)/pub;tar czf $(PUB_ROOTFS).tgz $(PUB_ROOTFS);popd
	@echo "---------finish osdrv work"

rootfs_clean:boardtools_clean pctools_clean busybox_clean
ifeq ($(OSDRV_DIR)/pub/$(PUB_ROOTFS), $(wildcard $(OSDRV_DIR)/pub/$(PUB_ROOTFS)))
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS); chmod +w usr/bin -R; chmod +w usr/sbin -R; chmod +w sbin -R; popd
endif
	rm $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/ -rf

##########################################################################################
#task [8]	clean pub
##########################################################################################
pub_clean:
	rm $(OSDRV_DIR)/pub -rf

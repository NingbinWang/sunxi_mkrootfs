TOOL_BIN := bin
OSDRV_CROSS ?= arm-himix410-linux
TOOL_NAME :=mtd-utils-2.1.1
TOOL_TAR_BALL := mtd-utils-2.1.1.tar.bz2
TOOL_DIR := $(shell pwd)

ZLIB_NAME:=zlib-1.2.11
ZLIB_TAR_BALL:=zlib-1.2.11.tar.gz
BOARD_ZLIB_LIB_PATH :=$(TOOL_DIR)/../zlib/out/board/lib/libz.a

PC_DIR := $(shell pwd)/$(TOOL_NAME)_pc
BOARD_DIR := $(shell pwd)/$(TOOL_NAME)_board
ZLIB_PC_DIR := $(TOOL_DIR)/../zlib/out/pc

OSDRV_CROSS_CFLAGS := -fPIC
OSDRV_CROSS_LDFLAGS := -Wl,-z,relro -Wl,-z,now -Wl,-z,noexecstack

INCLUDE += -I$(TOOL_DIR)/../zlib/out/board/include

LOCAL_CFLAGS := -fPIC
LOCAL_LDFLAGS := -Wl,-z,relro -Wl,-z,now -Wl,-z,noexecstack
GCC_VERSION_GE49 := $(shell echo `gcc -dumpversion | cut -f1-2 -d.` \>= 4.9 | sed -e 's/\./*100+/g' | bc)
ifeq ($(GCC_VERSION_GE49),1)
LOCAL_CFLAGS += -fstack-protector-strong
OSDRV_CROSS_CFLAGS += -fstack-protector-strong
else
LOCAL_CFLAGS += -fstack-protector-all
OSDRV_CROSS_CFLAGS += -fstack-protector-all
endif

all:  mtd_pc mtd_board

prepare:
ifneq ($(TOOL_DIR)/out, $(wildcard $(TOOL_DIR)/out))
	rm $(TOOL_DIR)/out -rf
	mkdir $(TOOL_DIR)/out
	mkdir $(TOOL_DIR)/out/pc
	mkdir $(TOOL_DIR)/out/board
endif

mtd_board : prepare
ifneq ($(BOARD_ZLIB_LIB_PATH), $(wildcard $(BOARD_ZLIB_LIB_PATH)))
	$(warning "---------------------------------------------------------------------")
	$(warning "     Cannot found the $(BOARD_ZLIB_LIB_PATH) file                    ")
	$(warning "   	Please to make $(ZLIB_NAME) page                                ")
	$(warning "---------------------------------------------------------------------")
	$(error )
endif
	rm $(BOARD_DIR) -rf;
	mkdir $(BOARD_DIR)
	tar -xf $(TOOL_TAR_BALL) -C $(BOARD_DIR)
	mkdir -p $(BOARD_DIR)/$(TOOL_NAME)/$(TOOL_BIN)/;
	mkdir -p $(BOARD_DIR)/$(TOOL_BIN)/;
	pushd $(BOARD_DIR)/$(TOOL_NAME)/;\
		./configure --without-ubifs --without-xattr --without-lzo --without-zstd --without-crypto \
		--host=$(OSDRV_CROSS) CFLAGS="$(OSDRV_CROSS_CFLAGS)" LDFLAGS="$(OSDRV_CROSS_LDFLAGS)" \
		--prefix=$(BOARD_DIR)/$(TOOL_NAME)/$(TOOL_BIN) \
		ZLIB_LIBS=$(BOARD_ZLIB_LIB_PATH) --disable-tests; \
		make CFLAGS=--static INCLUDES=$(INCLUDE) -j 20 > /dev/null || exit "$$?"; \
		make install > /dev/null || exit "$$?";\
	popd
	cp -rf $(BOARD_DIR)/$(TOOL_NAME)/$(TOOL_BIN)/sbin/* $(BOARD_DIR)/$(TOOL_BIN)/
	cp $(BOARD_DIR)/$(TOOL_BIN)/  $(TOOL_DIR)/out/board -r
	rm $(BOARD_DIR) -rf

mtd_pc : prepare
	rm $(PC_DIR) -rf
	mkdir $(PC_DIR)
	tar -xf $(TOOL_TAR_BALL) -C $(PC_DIR)
	pushd $(PC_DIR)/$(TOOL_NAME) && \
		./configure --prefix=$(PC_DIR) CC=gcc \
		--without-xattr --without-crypto --without-lzo --without-zstd \
		    CFLAGS="$(LOCAL_CFLAGS) -I$(ZLIB_PC_DIR)/include"  \
			LDFLAGS="$(LOCAL_LDFLAGS) -L$(ZLIB_PC_DIR)/lib" \
			ZLIBCPPFLAGS="-I$(ZLIB_PC_DIR)/include" \
			ZLIBLDFLAGS="-L$(ZLIB_PC_DIR)/lib" && \
		make || exit "$$?" && \
		make install || exit "$$?" ; \
	popd
	cp $(PC_DIR)/sbin/mkfs.ubifs $(TOOL_DIR)/out/pc/
	cp $(PC_DIR)/sbin/ubinize $(TOOL_DIR)/out/pc/
	cp $(PC_DIR)/sbin/mkfs.jffs2 $(TOOL_DIR)/out/pc/
	rm $(PC_DIR) -rf
clean:
	rm $(PC_DIR) -rf;
	rm $(BOARD_DIR) -rf;
	rm $(TOOL_DIR)/out -rf

distclean: clean

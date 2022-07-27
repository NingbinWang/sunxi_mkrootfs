ZLIB_NAME:=zlib-1.2.11
ZLIB_TAR_BALL:=zlib-1.2.11.tar.gz

PREFIX := $(shell pwd)

PC_DIR := $(shell pwd)/$(ZLIB_NAME)_pc
BOARD_DIR := $(shell pwd)/$(ZLIB_NAME)_board

OSDRV_CROSS ?= arm-himix410-linux
OSDRV_CROSS_CFLAGS := -fPIC
OSDRV_CROSS_LDFLAGS := -Wl,-z,relro -Wl,-z,now -Wl,-z,noexecstack

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

all: zlib_pc  zlib_board

prepare:
ifneq ($(PREFIX)/out, $(wildcard $(PREFIX)/out))
	rm $(PREFIX)/out -rf
	mkdir $(PREFIX)/out
	mkdir $(PREFIX)/out/pc
	mkdir $(PREFIX)/out/board
endif

zlib_pc: $(PC_DIR)/lib/libz.a
$(PC_DIR)/lib/libz.a : prepare
	rm $(PC_DIR) -rf
	mkdir $(PC_DIR)
	tar -xf $(ZLIB_TAR_BALL) -C $(PC_DIR)
	cd $(PC_DIR)/$(ZLIB_NAME)/ && \
		CFLAGS="$(LOCAL_CFLAGS)" LDFLAGS="$(LOCAL_LDFLAGS)" ./configure --prefix="$(PC_DIR)" && \
		make -j 16 CC=gcc || exit "$$?" && \
		make install CC=gcc || exit "$$?" && \
	cd -
	cp $(PC_DIR)/include $(PREFIX)/out/pc -r
	cp $(PC_DIR)/lib $(PREFIX)/out/pc -r
	cp $(PC_DIR)/share $(PREFIX)/out/pc -r
	rm $(ZLIB_NAME)_* -rf

zlib_board: $(BOARD_DIR)/lib/libz.a
$(BOARD_DIR)/lib/libz.a : prepare
	rm $(BOARD_DIR) -rf
	mkdir $(BOARD_DIR)
	tar -xf $(ZLIB_TAR_BALL) -C $(BOARD_DIR)
	pushd $(BOARD_DIR)/$(ZLIB_NAME)/;\
		prefix=zlib_install CC=$(OSDRV_CROSS)-gcc CFLAGS="$(OSDRV_CROSS_CFLAGS)" \
		LDFLAGS="$(OSDRV_CROSS_LDFLAGS)" ./configure; \
		make -j 20 > /dev/null || exit "$$?";\
		make install > /dev/null || exit "$$?";\
	popd
	cp $(BOARD_DIR)/$(ZLIB_NAME)/zlib_install/* $(PREFIX)/out/board -r
	rm $(ZLIB_NAME)_* -rf

clean:
	rm out -rf;
	rm $(ZLIB_NAME)_* -rf
distclean: clean

TOOLS_TOP_DIR ?= $(shell pwd)
TOOL_TAR_BALL := lzma1900.7z
TOOL_NAME := lzma1900
TOOL_BIN := bin

all:
	rm $(TOOLS_TOP_DIR)/$(TOOL_BIN) -rf
	mkdir $(TOOLS_TOP_DIR)/$(TOOL_BIN)/

	-rm $(TOOLS_TOP_DIR)/$(TOOL_NAME) -rf;
	7z x $(TOOL_TAR_BALL) -o$(TOOL_NAME)
	pushd $(TOOLS_TOP_DIR)/$(TOOL_NAME)/C/Util/Lzma/;\
	make -f makefile.gcc || exit "$$?";\
	cp lzma $(TOOLS_TOP_DIR)/$(TOOL_BIN)/ -rf; \
	popd;

	-rm $(TOOLS_TOP_DIR)/$(TOOL_NAME) -rf;

.PHONY: clean
clean:
	-rm $(TOOLS_TOP_DIR)/$(TOOL_NAME) -rf;
	-rm $(TOOLS_TOP_DIR)/lzma -rf;

.PHONY: distclean
distclean:
	-rm $(TOOLS_TOP_DIR)/$(TOOL_NAME) -rf;
	-rm $(TOOLS_TOP_DIR)/$(TOOL_BIN) -rf;


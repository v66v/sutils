# Copyright 2024 v66v
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

SRC_DIR ?= src
BUILDDIR := build
DESTDIR ?= $(BUILDDIR)

INC_DIRS = $(SRC_DIR)/
INCS := $(addprefix -I,$(INC_DIRS))
SRC_FILES := $(shell find $(SRC_DIR)/ -name *.c)

OBJS_DIR = $(BUILDDIR)/objects
OBJS := $(SRC_FILES:$(SRC_DIR)/%.c=$(OBJS_DIR)/%.o)
DEPS := $(OBJS:.o=.d)
-include $(DEPS)

PKGCONFIGS_CFLAGS :=
PKGCONFIGS_LIBS :=

ifneq ($(PKGCONFIGS_CFLAGS),)
	PKGCONFIG_CFLAGS = $(shell pkg-config --cflags $(PKGCONFIGS_CFLAGS))
endif

ifneq ($(PKGCONFIGS_LIBS),)
	PKGCONFIG_LIBS = $(shell pkg-config --libs $(PKGCONFIGS_LIBS))
endif

LIB_NAMES :=
LIBS := $(addprefix -l,$(LIB_NAMES))
LIBS += $(PKGCONFIG_LIBS)

CFLAGS = -MMD -MP -flto -O2 -std=gnu11
CFLAGS += $(PKGCONFIG_CFLAGS)

CC = gcc
BIN_NAME := main

.DEFAULT_GOAL := all
all: build

debug: CFLAGS += -DDEBUG -ggdb -Wall -Wextra -Wswitch-enum -Wswitch-default
debug: debug_info build

build: build_info dirs makefile $(BUILDDIR)/bin/$(BIN_NAME)

$(BUILDDIR)/bin/$(BIN_NAME): $(OBJS)
	$(info [CC] Linking object files)
	@$(CC) $(OBJS) -o $(BUILDDIR)/bin/$(BIN_NAME) $(LIBS)

$(OBJS_DIR)/%.o: $(SRC_DIR)/%.c makefile
	$(info [CC] $<)
	@$(CC) $(CFLAGS) $(INCS) -c $< -o $@

install: install_info build
	@mkdir -p $(DESTDIR)/
ifneq ($(BUILDDIR), $(DESTDIR))
	@cp -r $(BUILDDIR)/bin/ $(DESTDIR)/
else
	$(info [WRN] BUILDDIR($(BUILDDIR)) == DESTDIR($(DESTDIR)))
endif

dirs: $(BUILDDIR)/etc/ $(BUILDDIR)/bin/ $(dir $(OBJS))

.PRECIOUS: $(BUILDDIR)/. $(BUILDDIR)/%/.
$(BUILDDIR)/:
	$(info [INFO] Creating directory $@)
	@mkdir -p $@

$(BUILDDIR)/%/:
	$(info [INFO] Creating directory $@)
	@mkdir -p $@

.PHONY: clean ccls debug_info build_info install_info
build_info:
	$(info [INFO] Building Executable [$(BUILDDIR)/bin/$(BIN_NAME)])

debug_info:
	$(info [INFO] Compiling In Debug Mode)

install_info:
	$(info [INFO] Installing Executable [$(DESTDIR)/bin/$(BIN_NAME)])

clean:
	$(info [INFO] Cleaning)
	@rm -rf $(BUILDDIR)/

ccls:
	$(info [INFO] Creating .ccls file)
	@printf "%s\\n" $(CC) $(subst ' ','\n',$(LIBS)) $(CFLAGS) \
			$(subst ' ','\n',$(INCS)) -I$(subst :, -I,$(C_INCLUDE_PATH)) > .ccls

# used by Linux kernel
obj-y = mymain.o myinterrupt.o


KERNEL = linux-4.1

ROOT_DIR=$(shell pwd)
DOWNLOAD_DIR = $(ROOT_DIR)/downloads
KERNEL_DIR = $(ROOT_DIR)/kernel
KERNEL_PATCH_DIR = $(KERNEL_DIR)/$(KERNEL).new
KERNEL_SRC_DIR = $(KERNEL_DIR)/$(KERNEL).src


PATCH_DIR = $(ROOT_DIR)/patches
PATCH_FILE = linux-4_1-mykernel.patch

CONFIG_DIR = $(ROOT_DIR)/configs

# the generic parts
LINUX_URL = https://www.kernel.org/pub/linux/kernel/v4.x
LINUX_FILE = downloads/$(KERNEL).tar.xz
LINUX_MD5 = "fe9dc0f6729f36400ea81aa41d614c37"
#LINUX_MD5 = "8c7499c672af2e7adb88c3ea3ad03fbb"




OUT = $(PWD)/.out

ALL = .stamps/downloads .stamps/setup .stamps/build
.PHONY: all
all: $(ALL)

TMPFILE := $(shell mktemp)

.stamps:
	@mkdir -p $@

to-md5 = $1 $(addsuffix .md5,$1)
%.md5: FORCE
	@$(if $(filter-out $(shell cat $@ 2>/dev/null),$(shell md5sum $*)),md5sum $* > $@)
FORCE:

$(LINUX_FILE):
	mkdir -p $(DOWNLOAD_DIR)
	(cd downloads; wget -c $(LINUX_URL)/$(KERNEL).tar.xz)

.stamps/downloads: $(call to-md5,$(LINUX_FILE)) .stamps
	@echo $(LINUX_MD5) > $(TMPFILE)
	@cmp -n 32 $(TMPFILE) $<.md5 >/dev/null || (echo "File checksum mismatch!"; exit 1)
	@touch $@

.stamps/setup: .stamps/extract .stamps/patch .stamps/config
	@touch $@

.stamps/extract: $(LINUX_FILE)
	mkdir -p $(KERNEL_DIR)
	tar Jxf $< -C $(KERNEL_DIR)
	(cd $(KERNEL_DIR);								\
	 mv $(KERNEL) $(KERNEL).src;	\
	 cp -rf $(KERNEL).src $(KERNEL).new)
	(cd $(KERNEL_PATCH_DIR); ln -s ../../src mysrc)
	@touch $@

.stamps/patch:
	(cd $(KERNEL_PATCH_DIR); \
	 patch --dry-run -f -p1 < $(PATCH_DIR)/$(PATCH_FILE) >/dev/null && \
	 patch -p1 < $(PATCH_DIR)/$(PATCH_FILE)) || touch $@

.stamps/config:
	@mkdir -p $(OUT)
	(cd $(KERNEL_PATCH_DIR); \
	 cp -f $(CONFIG_DIR)/mini-x86.config $(OUT)/.config; \
	 make O=$(OUT) oldconfig)
	@touch $@

# number of CPUs
ifndef CPUS
CPUS := $(shell grep -c ^processor /proc/cpuinfo 2>/dev/null || \
          sysctl -n hw.ncpu)
endif

.stamps/build: $(KERNEL_PATCH_DIR)/Makefile \
               src/interrupt.c src/scheduler.c src/main.c src/pcb.h
	(cd $(KERNEL_PATCH_DIR); $(MAKE) O=$(OUT) -j $(CPUS))
	@touch $@

.PHNOY	:	run
run: $(OUT)/arch/x86/boot/bzImage
	qemu -kernel $<

#	---------------------------
#	generate patch and patch it
#	---------------------------

#	generate your kernel patch
.PHNOY	:	genpatch
genpatch:	.stamps/extract
	cd $(KERNEL_DIR)
	diff -Naur linux-4.1.src linux-4.1.new/ >linux-4.1-mykernel.patch

#	patch your kernel
.PHNOY	:	patch
patch	:	.stamps/extract
	(cd $(KERNEL_PATCH_DIR); \
	 patch --dry-run -f -p1 < $(PATCH_DIR)/$(PATCH_FILE) >/dev/null && \
	 patch -p1 < $(PATCH_DIR)/$(PATCH_FILE)) || touch $@


#	----------------------------------------------------
#	link and rmlink ./src to /kernel/linux-4.1.new/mysrc
#	----------------------------------------------------

#	remove your links
.PHNOY	:	rmlink
rmlink	:
	rm $(KERNEL_PATCH_DIR)/mysrc

#	relink ./src to /kernel/linux-4.1.new/mysrc
.PHNOY	:	link
link	:
	(cd $(KERNEL_PATCH_DIR); ln -s ../../src mysrc)


#	------------
#	build kernel
#	------------
.PHNOY	:	build
build	:
	(cd $(KERNEL_PATCH_DIR); $(MAKE) O=$(OUT))


#	-------------------
#	clean and distclean
#	-------------------

#	clean your work
.PHONY	:	clean
clean:
	$(MAKE) -C $(KERNEL_PATCH_DIR) O=$(OUT) clean
	rm -f .stamps/build

#	distclean all your work
.PHONY	:	distclean
distclean: clean
	rm -rf .stamps $(OUT)
	rm -rf $(KERNEL_DIR)

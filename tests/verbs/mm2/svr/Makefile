KFI_TOP ?=$(shell pwd)/../../../..
KFI=$(KFI_TOP)/kfi

MODULE_NAME=kfit_verbs_mm2_srv

ifneq ("","$(wildcard /usr/src/compat-rdma)")
  #for OFED-3.12
  OFED_DIR:=/usr/src/compat-rdma
else ifneq ("","$(wildcard /usr/src/ofa_kernel)")
  #for OFED 1.5.4.1
  OFED_DIR:=/usr/src/ofa_kernel
else
  OFED_DIR:=
endif

KDIR ?= /lib/modules/`uname -r`/build
autoconf_h=$(shell /bin/ls -1 $(KDIR)/include/*/autoconf.h | head -1)
kconfig_h=$(shell /bin/ls -1 $(KDIR)/include/*/kconfig.h 2>/dev/null | head -1)

KSYM := $(KFI)/Module.symvers

ifeq ("$(OFED_DIR)","")
 INCLUDES:=
else
 KSYM += $(OFED_DIR)/Module.symvers

 INCLUDES:= -include $(autoconf_h)

 ifneq ($(kconfig_h),)
   INCLUDES+= -include $(kconfig_h)
 endif

 INCLUDES+= -I$(KFI_TOP)/include \
	    -I$(OFED_DIR)/include \
	    -I$$(srctree)/arch/$$(SRCARCH)/include \
	    -Iarch/$$(SRCARCH)/include/generated \
	    $$(if $$(KBUILD_SRC), -I$$(srctree)/include) \
	    -Iinclude \
	    -I$$(srctree)/arch/$$(SRCARCH)/include/uapi \
	    -Iarch/$$(SRCARCH)/include/generated/uapi \
	    -I$$(srctree)/include/uapi \
	    -Iinclude/generated/uapi
endif

INCLUDES+=-DCONFIG_KFI_DEBUG

KBUILD_SRC=$(KDIR)

obj-m += $(MODULE_NAME).o

$(MODULE_NAME)-y := test.o
$(MODULE_NAME)-y += main.o

ccflags-y += -I$(src)/..

all: $(KSYM)
	@echo $(PWD)
	@make -C $(KDIR) M=$(PWD) modules \
		LINUXINCLUDE='$(INCLUDES)' KBUILD_EXTRA_SYMBOLS+='$(KSYM)'

clean:
	@make -C $(KDIR) M=$(PWD) clean 2>&1 >/dev/null
	@echo Done.

main.o: main.c Makefile
test.o: test.c Makefile

LOC:=$(shell ip addr show ib0 | fgrep 'inet ' | cut -f 6 -d' ' | cut -f 1 -d '/')
LEN ?=128	# payload length
NM ?=1		# total num msgs
PD ?=1		# post-depth
ARGS=local_ipaddr=$(LOC) msg_len=$(LEN) num_msgs=$(NM) post_depth=$(PD) verify=0 debug=0

load:
	@echo loading $(MODULE_NAME).ko $(ARGS)
	@sudo insmod $(MODULE_NAME).ko $(ARGS)

unload:
	@sudo rmmod ../cli/kfit_ib_mm_cli.ko 2>/dev/null || echo > /dev/null
	@sudo rmmod $(MODULE_NAME).ko 2>/dev/null || echo > /dev/null

test:
	@make load NM=4 PD=2

regress:
	@make load NM=327680 PD=25 LEN=32768

regress4:
	@make load NM=4 PD=4

regress20:
	@make load NM=20 PD=20

regress24:
	@make load NM=20 PD=4

test1:
	@make load NM=2000 PD=25 LEN=8192

check:
	@lsmod | fgrep  $(MODULE_NAME); if [ $$? -eq 1 ] ; then echo "$(MODULE_NAME) not loaded" ; fi

kcheck:
	@lsmod | fgrep  kfi; if [ $$? -eq 1 ] ; then echo "kfi* not loaded" ; fi

reset:
	-@sudo rmmod $(MODULE_NAME) 2> /dev/null || echo > /dev/null
	-@dmesg -c &>/dev/null

creset:
	-@sudo rmmod $(MODULE_NAME) 2> /dev/null || echo > /dev/null
	-@dmesg -c &>/dev/null
	clear

help:
	@echo "make all"
	@echo "make load"
	@echo "make unload"
	@echo "make kfi_load"
	@echo "make kfi_unload"
	@echo "make check # is $(MODULE_NAME) loaded?"
	@echo "make kcheck # is kfi* module loaded?"
	@echo "make reset # unload $(MODULE_NAME) + dmesg -c"

kfi_load: kfi_unload
	@if [ `lsmod | grep -c rdma` -eq 0 ] ; then sudo modprobe rdma_cm ; fi
	@echo loading kfi.ko
	@sudo insmod $(KFI_TOP)/kfi/kfi.ko
	@echo loading kfip_verbs.ko
	@sudo insmod $(KFI_TOP)/providers/verbs/*.ko

kfi_unload:
	@sudo rmmod kfip_verbs.ko 2>/dev/null || echo > /dev/null
	@sudo rmmod kfi.ko 2>/dev/null || echo > /dev/null

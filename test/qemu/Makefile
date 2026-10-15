# Runs tests.c on emulated Cortex-M4 (mps2-an386) and Cortex-M33 (mps2-an505) under QEMU.
#
#   make                 default config: CPU=m4 FPU=1 DCACHE=0 MULSQR=0 FASTBASE=1
#   make CPU=m33 FPU=0   any single config
#   make matrix          all 32 combinations of CPU/FPU/DCACHE/MULSQR/FASTBASE
#
# Requires arm-none-eabi-gcc and qemu-system-arm.

CPU      ?= m4
FPU      ?= 1
DCACHE   ?= 0
MULSQR   ?= 0
FASTBASE ?= 1

CC   := arm-none-eabi-gcc
QEMU := qemu-system-arm
ROOT := ../..

ifeq ($(CPU),m4)
MACHINE := mps2-an386
MCPU    := cortex-m4
FPUNAME := fpv4-sp-d16
else ifeq ($(CPU),m33)
MACHINE := mps2-an505
MCPU    := cortex-m33
FPUNAME := fpv5-sp-d16
else
$(error CPU must be m4 or m33)
endif

ifeq ($(FPU),1)
ARCHFLAGS := -mcpu=$(MCPU) -mfpu=$(FPUNAME) -mfloat-abi=hard
else
ARCHFLAGS := -mcpu=$(MCPU)+nofp -mfloat-abi=soft
endif

CONFIG := $(CPU)-fpu$(FPU)-dcache$(DCACHE)-mulsqr$(MULSQR)-fastbase$(FASTBASE)
OUT    := build/$(CONFIG)
DEFS   := -Dhas_fpu=$(FPU) -Dhas_d_cache=$(DCACHE) -Duse_mul_for_sqr=$(MULSQR) -Duse_fast_p256_basemult=$(FASTBASE)
CFLAGS := $(ARCHFLAGS) -mthumb -O2 -g -ffreestanding -fno-tree-loop-distribute-patterns -Wall $(DEFS) -Iinclude -I$(ROOT)
LDFLAGS := $(ARCHFLAGS) -mthumb -nostdlib -L. -T $(MACHINE).ld -Wl,--gc-sections

SRCS := $(ROOT)/p256-cortex-m4.c $(ROOT)/p256-cortex-m4-asm-gcc.S $(ROOT)/tests.c startup.c
HDRS := $(ROOT)/p256-cortex-m4.h $(ROOT)/p256-cortex-m4-config.h include/string.h

.PHONY: test matrix clean

test: $(OUT)/tests.elf
	@printf '%-40s ' $(CONFIG); $(QEMU) -M $(MACHINE) -nographic -semihosting-config enable=on,target=native -kernel $<

$(OUT)/tests.elf: $(SRCS) $(HDRS) $(MACHINE).ld sections.ld Makefile
	@mkdir -p $(OUT)
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $(SRCS) -lgcc

matrix:
	@fail=0; \
	for cpu in m4 m33; do for fpu in 0 1; do for dc in 0 1; do for ms in 0 1; do for fb in 0 1; do \
		$(MAKE) -s test CPU=$$cpu FPU=$$fpu DCACHE=$$dc MULSQR=$$ms FASTBASE=$$fb || fail=$$((fail+1)); \
	done; done; done; done; done; \
	echo "failed configs: $$fail"; [ $$fail -eq 0 ]

clean:
	rm -rf build

# Copyright Statement:
#
# (C) 2005-2016  MediaTek Inc. All rights reserved.
#
# This software/firmware and related documentation ("MediaTek Software") are
# protected under relevant copyright laws. The information contained herein
# is confidential and proprietary to MediaTek Inc. ("MediaTek") and/or its licensors.
# Without the prior written permission of MediaTek and/or its licensors,
# any reproduction, modification, use or disclosure of MediaTek Software,
# and information contained herein, in whole or in part, shall be strictly prohibited.
# You may only use, reproduce, modify, or distribute (as applicable) MediaTek Software
# if you have agreed to and been bound by the applicable license agreement with
# MediaTek ("License Agreement") and been granted explicit permission to do so within
# the License Agreement ("Permitted User").  If you are not a Permitted User,
# please cease any access or use of MediaTek Software immediately.
# BY OPENING THIS FILE, RECEIVER HEREBY UNEQUIVOCALLY ACKNOWLEDGES AND AGREES
# THAT MEDIATEK SOFTWARE RECEIVED FROM MEDIATEK AND/OR ITS REPRESENTATIVES
# ARE PROVIDED TO RECEIVER ON AN "AS-IS" BASIS ONLY. MEDIATEK EXPRESSLY DISCLAIMS ANY AND ALL
# WARRANTIES, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE OR NONINFRINGEMENT.
# NEITHER DOES MEDIATEK PROVIDE ANY WARRANTY WHATSOEVER WITH RESPECT TO THE
# SOFTWARE OF ANY THIRD PARTY WHICH MAY BE USED BY, INCORPORATED IN, OR
# SUPPLIED WITH MEDIATEK SOFTWARE, AND RECEIVER AGREES TO LOOK ONLY TO SUCH
# THIRD PARTY FOR ANY WARRANTY CLAIM RELATING THERETO. RECEIVER EXPRESSLY ACKNOWLEDGES
# THAT IT IS RECEIVER'S SOLE RESPONSIBILITY TO OBTAIN FROM ANY THIRD PARTY ALL PROPER LICENSES
# CONTAINED IN MEDIATEK SOFTWARE. MEDIATEK SHALL ALSO NOT BE RESPONSIBLE FOR ANY MEDIATEK
# SOFTWARE RELEASES MADE TO RECEIVER'S SPECIFICATION OR TO CONFORM TO A PARTICULAR
# STANDARD OR OPEN FORUM. RECEIVER'S SOLE AND EXCLUSIVE REMEDY AND MEDIATEK'S ENTIRE AND
# CUMULATIVE LIABILITY WITH RESPECT TO MEDIATEK SOFTWARE RELEASED HEREUNDER WILL BE,
# AT MEDIATEK'S OPTION, TO REVISE OR REPLACE MEDIATEK SOFTWARE AT ISSUE,
# OR REFUND ANY SOFTWARE LICENSE FEES OR SERVICE CHARGE PAID BY RECEIVER TO
# MEDIATEK FOR SUCH MEDIATEK SOFTWARE AT ISSUE.
#

SOURCE_DIR = ../../../../..

PWD= $(shell pwd)
SDK_PATH    = $(abspath $(PWD)/$(SOURCE_DIR))

FEATURE ?= feature.mk
include $(FEATURE)
# Gloabl Config
include $(SOURCE_DIR)/.config
# IC Config
include $(SOURCE_DIR)/config/chip/$(IC_CONFIG)/chip.mk
# Board Config
include $(SOURCE_DIR)/config/board/$(BOARD_CONFIG)/board.mk
# EPT Config
include $(SOURCE_DIR)/driver/board/mt25x3_hdk/ept/module.mk


RAM_BOOTING=0

DEBUG = 0
FLOAT_TYPE = hard
BUILD_DIR = $(PWD)/Build

# Project name
PROJ_NAME       = $(notdir $(abspath $(dir $(PWD))))
PROJ_PATH = $(PWD)
OUTPATH = $(PWD)/Build

# HAL driver files
include $(SOURCE_DIR)/driver/board/mt25x3_hdk/module.mk
include $(SOURCE_DIR)/driver/chip/mt2523/module.mk

# NVDM files
include $(SOURCE_DIR)/middleware/MTK/nvdm/module.mk
# Main APP files
APP_PATH        = $(patsubst $(SDK_PATH)/%,%,$(abspath $(dir $(PWD))))
APP_PATH_SRC = project/mt2523_hdk/apps/bt_spp_transfer_data/src
APP_FILES = $(APP_PATH_SRC)/main.c \
            $(APP_PATH_SRC)/regions_init.c \
			$(APP_PATH)/GCC/syscalls.c
C_FILES += $(APP_FILES)
C_FILES += $(APP_PATH_SRC)/system_mt2523.c
C_FILES += $(APP_PATH_SRC)/hci_log.c
C_FILES += $(APP_PATH_SRC)/ept_gpio_var.c
C_FILES += $(APP_PATH_SRC)/ept_eint_var.c
C_FILES += $(APP_PATH_SRC)/bt_init.c
C_FILES += $(APP_PATH_SRC)/bt_spp_main.c
C_FILES += $(APP_PATH_SRC)/bt_spp_server.c
C_FILES += $(APP_PATH_SRC)/bt_spp_client.c
C_FILES += $(APP_PATH_SRC)/sys_init.c
# add startup file to build
S_FILES += $(APP_PATH)/GCC/startup_mt2523.s

###################################################
# Check for valid float argument
# NOTE that you have to run make clan after
# changing these as hardfloat and softfloat are not
# binary compatible
ifneq ($(FLOAT_TYPE), hard)
ifneq ($(FLOAT_TYPE), soft)
override FLOAT_TYPE = hard
#override FLOAT_TYPE = soft
endif
endif

###################################################
# CC Flags
ifeq ($(DEBUG), 1)
ALLFLAGS = -g -O0
else
ALLFLAGS = -g -Os
endif
ALLFLAGS += -Wall -mlittle-endian -mthumb -mcpu=cortex-m4
CFLAGS += $(ALLFLAGS) -flto -ffunction-sections -fdata-sections -fno-builtin

ifeq ($(FLOAT_TYPE), hard)
FPUFLAGS = -fsingle-precision-constant -Wdouble-promotion
FPUFLAGS += -mfpu=fpv4-sp-d16 -mfloat-abi=hard
#CFLAGS += -mfpu=fpv4-sp-d16 -mfloat-abi=softfp
else
FPUFLAGS = -msoft-float
endif

# Definitions
CFLAGS += -D_REENT_SMALL
CFLAGS += -DARM_MATH_CM4
CFLAGS += -D__FPU_PRESENT
CFLAGS += $(FPUFLAGS)

# LD Flags
LDFLAGS = $(ALLFLAGS) $(FPUFLAGS) --specs=nano.specs -lnosys -nostartfiles
LDFLAGS += -Wl,-wrap=malloc -Wl,-wrap=calloc -Wl,-wrap=realloc -Wl,-wrap=free
ifeq ($(RAM_BOOTING), 1)
LDFLAGS += -Wl,-Tsram.ld -Wl,--gc-sections
else
LDFLAGS += -Wl,-Tflash.ld -Wl,--gc-sections
endif
ifeq ($(DEBUG), 1)
LDFLAGS += --specs=rdimon.specs -lrdimon
endif
#LDFLAGS = $(CFLAGS)

###################################################
# include path
CFLAGS += -I$(SOURCE_DIR)/$(APP_PATH)/inc -I$(SOURCE_DIR)/driver/CMSIS/Include
CFLAGS += -I$(SOURCE_DIR)/driver/CMSIS/Device/MTK/mt2523/Include
CFLAGS += -I$(SOURCE_DIR)/driver/chip/inc
CFLAGS += -I$(SOURCE_DIR)/driver/board/mt25x3_hdk/ept/inc
CFLAGS += -I$(SOURCE_DIR)/middleware/MTK/bluetooth/inc
CXXFLAGS = $(CFLAGS) -fno-rtti -fno-exceptions

###################################################
###################################################
# Sources or Lib
# RTOS source files
include $(SOURCE_DIR)/kernel/rtos/FreeRTOS/module.mk

# mems driver files
include $(SOURCE_DIR)/driver/board/component/mems/module.mk

# kernel service files
include $(SOURCE_DIR)/kernel/service/module.mk

# Bluetooth module
include $(SOURCE_DIR)/middleware/MTK/bluetooth/module.mk

C_OBJS = $(C_FILES:%.c=$(BUILD_DIR)/%.o)
CXX_OBJS        = $(CXX_FILES:%.cpp=$(BUILD_DIR)/%.o)
S_OBJS = $(S_FILES:%.s=$(BUILD_DIR)/%.o)
###################################################
# Rules
.PHONY: proj clean $(MODULE_PATH)

all: proj
	@mkdir -p $(BUILD_DIR)
	@$(SIZE) $(OUTPATH)/$(PROJ_NAME).elf
	@$(SOURCE_DIR)/tools/scripts/build/copy_firmware.sh $(SOURCE_DIR) $(OUTPATH) $(IC_CONFIG) $(BOARD_CONFIG) $(PROJ_NAME).bin $(PWD)

MOD_EXTRA = BUILD_DIR=$(BUILD_DIR) OUTPATH=$(OUTPATH) PROJ_PATH=$(PROJ_PATH)

$(LIBS): $(MODULE_PATH)

$(MODULE_PATH):
	@+make -C $@ $(MOD_EXTRA) $($@_EXTRA)

proj: $(OUTPATH)/$(PROJ_NAME).elf


$(OUTPATH)/$(PROJ_NAME).elf: $(C_OBJS) $(CXX_OBJS) $(S_OBJS) $(LIBS)
	@echo Linking...
	@if [ -e "$@" ]; then rm -f "$@"; fi
	@if [ -e "$(OUTPATH)/$(PROJ_NAME).map" ]; then rm -f "$(OUTPATH)/$(PROJ_NAME).map"; fi
	@if [ -e "$(OUTPATH)/$(PROJ_NAME).dis" ]; then rm -f "$(OUTPATH)/$(PROJ_NAME).dis"; fi
	@if [ -e "$(OUTPATH)/$(PROJ_NAME).hex" ]; then rm -f "$(OUTPATH)/$(PROJ_NAME).hex"; fi
	@if [ -e "$(OUTPATH)/$(PROJ_NAME).bin" ]; then rm -f "$(OUTPATH)/$(PROJ_NAME).bin"; fi
	@echo $(patsubst $(SDK_PATH)/%,$(SOURCE_DIR)/%, $(LDFLAGS) -Wl,--start-group $^ -Wl,--end-group -Wl,-Map=$(patsubst $(SDK_PATH)/%,$(SOURCE_DIR)/%,$(OUTPATH)/$(PROJ_NAME).map) -lm -o $@) > $(OUTPATH)/link_option.tmp
	@$(CC) @$(OUTPATH)/link_option.tmp 2>>$(ERR_LOG)
	@rm -f $(OUTPATH)/link_option.tmp
	@$(OBJDUMP) -D $(OUTPATH)/$(PROJ_NAME).elf > $(OUTPATH)/$(PROJ_NAME).dis
	@$(OBJCOPY) -O ihex $(OUTPATH)/$(PROJ_NAME).elf $(OUTPATH)/$(PROJ_NAME).hex
	@$(OBJCOPY) -O binary $(OUTPATH)/$(PROJ_NAME).elf $(OUTPATH)/$(PROJ_NAME).bin
	@echo Done

include $(SOURCE_DIR)/.rule.mk

clean:
	rm -rf $(OUTPATH)

ifneq ($(MAKECMDGOALS),clean)
-include $(C_OBJS:.o=.d)
endif

################################################################################
# Automatically-generated file. Do not edit!
################################################################################

-include ../makefile.init

RM := rm -rf

# All of the sources participating in the build are defined here
-include sources.mk
-include Startup/subdir.mk
-include Src/subdir.mk
-include Middlewares/ST/STM32_USB_Device_Library/Core/Src/subdir.mk
-include Middlewares/ST/STM32_USB_Device_Library/Class/CustomHID/Src/subdir.mk
-include Drivers/STM32F1xx_HAL_Driver/Src/subdir.mk
-include subdir.mk
-include objects.mk

ifneq ($(MAKECMDGOALS),clean)
ifneq ($(strip $(C_DEPS)),)
-include $(C_DEPS)
endif
endif

-include ../makefile.defs

# Add inputs and outputs from these tool invocations to the build variables 
EXECUTABLES += \
USB\ Custom\ HID.elf \

SIZE_OUTPUT += \
default.size.stdout \

OBJDUMP_LIST += \
USB\ Custom\ HID.list \

OBJCOPY_HEX += \
USB\ Custom\ HID.hex \


# All Target
all: USB\ Custom\ HID.elf secondary-outputs

# Tool invocations
USB\ Custom\ HID.elf: $(OBJS) $(USER_OBJS) L:\Documents\GitHub\STM32F103C8T6-Familiarization\USB\ Custom\ HID\Firmware\USB\ Custom\ HID\STM32F103C8TX_FLASH.ld
	arm-none-eabi-gcc -o "USB Custom HID.elf" @"objects.list" $(USER_OBJS) $(LIBS) -mcpu=cortex-m3 -T"L:\Documents\GitHub\STM32F103C8T6-Familiarization\USB Custom HID\Firmware\USB Custom HID\STM32F103C8TX_FLASH.ld" --specs=nosys.specs -Wl,-Map="USB Custom HID.map" -Wl,--gc-sections -static --specs=nano.specs -mfloat-abi=soft -mthumb -Wl,--start-group -lc -lm -Wl,--end-group
	@echo 'Finished building target: $@'
	@echo ' '

default.size.stdout: $(EXECUTABLES)
	arm-none-eabi-size  $(EXECUTABLES)
	@echo 'Finished building: $@'
	@echo ' '

USB\ Custom\ HID.list: $(EXECUTABLES)
	arm-none-eabi-objdump -h -S $(EXECUTABLES) > "USB Custom HID.list"
	@echo 'Finished building: $@'
	@echo ' '

USB\ Custom\ HID.hex: $(EXECUTABLES)
	arm-none-eabi-objcopy  -O ihex $(EXECUTABLES) "USB Custom HID.hex"
	@echo 'Finished building: $@'
	@echo ' '

# Other Targets
clean:
	-$(RM) *
	-@echo ' '

secondary-outputs: $(SIZE_OUTPUT) $(OBJDUMP_LIST) $(OBJCOPY_HEX)

.PHONY: all clean dependents
.SECONDARY:

-include ../makefile.targets

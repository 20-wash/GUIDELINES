# STM32 Initial Guide

## Software Install

-----------------------------------------------------------------------------
| Software            | Description                                 | Install Guide                                                                      |
| ------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------- |
| STM32CubeMx         | STM32 configuration tool                    | Install from [here](https://www.st.com/en/development-tools/stm32cubemx.html)      |
| STM32CubeProgrammer | Program and debug STM32 devices             | Install from [here](https://www.st.com/en/development-tools/stm32cubeprog.html)    |
| STM32CubeMonitor    | Plot graphs for monitoring                  | Install from [here](https://www.st.com/en/development-tools/stm32cubemonitor.html) |
| stlink-tools        | Command line tools for STM32 flash          | sudo apt install stlink-tools                                                      |
| arm-none-eabi-gcc   | GCC compiler for ARM-based microcontrollers | sudo apt install gcc-arm-none-eabi                                                 |
-----------------------------------------------------------------------------

## Generate Basic Code

### Pinout & Configuration
-  Clear default pinouts by pressing `Ctrl + P` and `Enter`.
-  Go to `RCC > High Speed Clock (HSE)` and select `Crystal Ceramic Resonator`.
-  Go to `SYS > Debug` and select `Serial Wire`.

    Now you can select pins for LED, UART, Timers and many more.

### Clock Configuration
- Specify `HCLK (MHx)`. We prefer to use maximum clock frequency.

### Project Manager
**Project**:
- Write `Project Name`.
- Select `Toolchanin/IDE`. We prefer to choose `Makefile`. `Cmake` was not available before.

**Code Generator**:
- Select copy all used libraries in to the project folder.
- Tick:
    - Generate eripheral initialization as a pair '.c/.h' file per peripheral.
    - Keep user code when regenerating.
    - Delete priviously generated file when not in use.

Generate code by clicking `GENERATE CODE`. Open your project folder, open terminal at your project folder and run `make` command in your terminal.

If you have selected `Cmake` for your project.
```bash
mkdir build
cd build
cmake ..
make
```
But this does not generate `.bin` file which is needed for flashing you microcontroller. To generate, you need to use `arm-none-eabi-objcopy`.
```bash
arm-none-eabi-objcopy -O binary <your_project_name.elf> <your_project_name.bin>
```

## Edit Makefile for Flash

For `stlink`, add these lines below your `Makefile`.
```Makefile
flash:
    st-flash write $(BUILD_DIR)/$(TARGET).bin 0x8000000
```

For `jlink`, add these lines  below you makefile.
```Makefile
device = STM32F103C8
$(BUILD_DIR)/jflash: $(BUILD_DIR)/$(TARGET).bin
	@touch $@
	@echo device $(device) > $@
	@echo -e si 1'\n'speed 4000 >> $@
	@echo loadbin $< 0x8000000 >> $@
	@echo -e r'\n'g'\n'qc >> $@

jflash: $(BUILD_DIR)/jflash
	JLinkExe -commanderscript $<
```

Use `make flash` or `make jflash` commands in order to flash.


## Debugging Setup

1. Install `Cortex-Debug` by `marus25` in VSCODE.
2. Extract and copy arm-none-eabi-gdb inside `/usr/bin`.
3. Click `debug` button in VSCODE and create `lanch.json` inside `.vscode`.
4. Update `lauch.json` the json as below.
   ```json
   {
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Cortex Debug",
            "cwd": "${workspaceFolder}",
            "executable": "${workspaceFolder}/build/base-0.1.elf",
            "request": "launch",
            "type": "cortex-debug",
            "runToEntryPoint": "main",
            "servertype": "stlink",
            "device": "STM32F407VG",
            "interface": "swd",
            "gdbPath": "/usr/bin/gdb-multiarch"
        }
    ]
   }
   ```
5. Run `Coretex-Debug` from play `icon`.

   For `OpenOCD`, watch [this](https://www.youtube.com/watch?v=_1u7IOnivnM).


---
[home](../README.md)

# IoT Firmware Tools

This folder contains Windows executables and instructions for working with and loading IoT firmware reference design binaries to supported development platforms.

## Firmware Loading Tools

### fm_load.exe

The firmware loading tool **fm_load.exe** is "plug and load" and ready to program firmware binaries into internal flash on supported targets.    This tool currently supports programming the MSP432 MCU through the XDS110 debug interface present on newer LaunchPad development platforms for CC26xx, CC13xx and MSP432.  fm_load has been built and tested with Windows 10.

#### Usage

fm_load.exe is intended to be executed in a console.

Without any command-line options, executing fm_load.exe will show the following console output if a supported development platform is connected (e.g. MSP-EXP432P401R LaunchPad).  The MSP432 LaunchPad is correctly detected as board id "0000".   If no firmware binary image (.hex or .bin) is supplied nor any command-line argument, the message "No operation performed" is displayed and the program ends.

Firmware Module System Firmware Loader v1.1.56
Copyright (c) 2016 Firmware Modules Inc.
License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE

INFO:root:Board unique id 0000
INFO:root:DAP JTAG MODE initialised
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M4
INFO:root:FPU present
INFO:root:4 hardware watchpoints
No operation performed

##### Loading Firmware

Loading firmware onto a supported target device is as simple as adding the file name of the firmware binary image to load as a command-line argument as shown below.

c:\tools\fm_load OTA-MSP432-CC3100_1_0_15_MAN_0x1509FB65.bin
Firmware Module System Firmware Loader v1.1.56
Copyright (c) 2016 Firmware Modules Inc.
License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE

INFO:root:Board unique id 0000
INFO:root:DAP JTAG MODE initialised
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M4
INFO:root:FPU present
INFO:root:4 hardware watchpoints
[====================] 100%

In this instance a progress bar is displayed showing the status of firmware programming.  When programming is completed, the target is reset and begins executing out of internal flash at address 0x0.


# Release Notes

## fm_load

### Version 1.1.56

* First public release.
* Supported debuggers/programmers
..* XDS110
* Supported targets
..* MSP432P401R 
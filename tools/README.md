# IoT Firmware Tools

This folder contains Windows executables and instructions for working with and loading IoT firmware reference design binaries to supported development platforms.

## Firmware Loading Tools

### fm_load.exe

The firmware loading tool **fm_load.exe** is "plug and load" and ready to program firmware binaries into internal flash on supported targets.    This tool currently supports programming the MSP432, CC26XX and CC13XX MCUs from TI and the Nordic nRF52832 MCU through the XDS110 debug interface present on newer LaunchPad development platforms for CC26xx, CC13xx and MSP432.  fm_load has been built and tested with Windows 10, and is code-signed by Firmware Modules Inc.

#### Usage

fm_load.exe is intended to be executed in a console.

Without any command-line options, executing fm_load.exe will show the following console output if a supported development platform is connected (e.g. MSP-EXP432P401R LaunchPad).  The actual connected device is reported at the `Detecting target...` phase.   If no firmware binary image (.hex or .bin) is supplied nor any command-line argument, the message "No operation performed" is displayed and the program ends.
```
Firmware Module System Firmware Loader v1.2.68
Copyright (c) 2016 Firmware Modules Inc.
License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE
Firmware loader for XDS110 with TI MSP432, CC13xx, CC26xx devices; Nordic nRF52832

INFO:root:DAP JTAG MODE initialised
INFO:root:Detecting target... cc2650f128rgz
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M3
INFO:root:4 hardware watchpoints
No operation performed
```

In case multiple target development boards are connected, fm_load will enumerate them:
```
Firmware Module System Firmware Loader v1.2.68
Copyright (c) 2016 Firmware Modules Inc.
License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE
Firmware loader for XDS110 with TI MSP432, CC13xx, CC26xx devices; Nordic nRF52832

Multiple connected boards found.
Please specify desired board target with -t and/or -b option.
Listing connected boards:
Auto-detecting board target...
XDS110 (02.02.04.00) with CMSIS-DAP 00000000 [msp432p401ripz]
Auto-detecting board target...
XDS110 (02.03.00.02) with CMSIS-DAP L1000588 [cc2650f128rgz]
Auto-detecting board target...
XDS110 (02.02.05.01) with CMSIS-DAP L2000698 [cc1310f128rgz]
Auto-detecting board target...
XDS110 (02.02.05.01) with CMSIS-DAP L400A30V [cc1350f128rgz]
Auto-detecting board target...
XDS110 (02.02.04.00) with CMSIS-DAP 00000000 [nrf52832qfaa]
```
To select a specific board to connect to,  use the `-t` option to specify a target like this:
`fm_load -t cc1310f128`.  In this case, each board is examined closely for a match and released if the detected target does not match the supplied target string.

```
Firmware Module System Firmware Loader v1.2.68
Copyright (c) 2016 Firmware Modules Inc.
License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE
Firmware loader for XDS110 with TI MSP432, CC13xx, CC26xx devices; Nordic nRF52832

INFO:root:DAP JTAG MODE initialised
INFO:root:DAP SWD MODE initialised
INFO:root:Detecting target... nrf52832qfaa
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M4
INFO:root:FPU present
INFO:root:4 hardware watchpoints
INFO:root:DAP JTAG MODE initialised
INFO:root:Detecting target... msp432p401ripz
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M4
INFO:root:FPU present
INFO:root:4 hardware watchpoints
INFO:root:DAP JTAG MODE initialised
INFO:root:Detecting target... cc2650f128rgz
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M3
INFO:root:4 hardware watchpoints
INFO:root:DAP JTAG MODE initialised
INFO:root:Detecting target... cc1310f128rgz
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M3
INFO:root:4 hardware watchpoints
INFO:root:Selected board with target: cc1310f128
No operation performed
```


##### Loading Firmware

Loading firmware onto a supported target device is as simple as adding the file name of the firmware binary image to load as a command-line argument as shown below (with only the desired target development board connected).  When multiple supported target development boards are connected, the specific target must be set with the `-t` option.

```

C:\tools>fm_load OTA-CC2650-CC3100_1_0_21_MAN_0xB61214C5.bin -t cc2650f128
Firmware Module System Firmware Loader v1.2.68
Copyright (c) 2016 Firmware Modules Inc.
License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE
Firmware loader for XDS110 with TI MSP432, CC13xx, CC26xx devices; Nordic nRF52832

INFO:root:DAP JTAG MODE initialised
INFO:root:Detecting target... msp432p401ripz
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M4
INFO:root:FPU present
INFO:root:4 hardware watchpoints
INFO:root:DAP JTAG MODE initialised
INFO:root:Detecting target... cc2650f128rgz
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M3
INFO:root:4 hardware watchpoints
INFO:root:Selected board with target: cc2650f128
Writing 62176 bytes...
[====================] 100%
```

In this instance a progress bar is displayed showing the status of firmware programming.  When programming is completed, the target is reset and begins executing out of internal flash at address 0x0.


# Release Notes

## fm_load

### Version 1.2.68
* Adds support for the Nordic Semiconductor nRF52832 Cortex-M4F MCU target.  This target can be accessed through any XDS110 debug probe.
* Adds factory reset option to disable access port protection on nRF52832 devices.
* Adds soft reset option to initiate an MCU soft reset. This feature allows externally debugged targets without any reset control to be reset without power cycling.

### Version 1.2.65
* Executable is code-signed with Firmware Modules' globally recognized certificate.
* Adds support for connecting to and programming CC26XX and CC13XX devices.
* Adds support for selecting amongst multiple connected target development boards.

* Supported targets
  * MSP432P401x
  * CC26xx
  * CC13xx

### Version 1.1.56

* First public release.
* Supported debuggers/programmers
  * XDS110
* Supported targets
  * MSP432P401R 
  

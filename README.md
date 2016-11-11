# IoT Firmware
Secure and scalable firmware for the Internet of *Things*.  These firmware reference designs are easy to use and ready to work with popular development platforms for ARM Cortex-M MCUs.

## Supported Development Platforms

TI MCU and LaunchPad Ecosystem:
* MSP432 with CC3100  [*MSP-EXP432P401R + CC3100BOOST*]

## Interesting IoT Applications

Each project folder contains firmware reference design binaries that implement an interesting IoT application for each supported platform.  Instructions to use each binary on the supported development platform is provided in the folder.

A brief description of available IoT applications is provided below.

### OTA Update Demo */otaupdate*

Demonstrates a robust live over-the-air firmware application update.  The firmware update binary is securely pulled from this very repository.

## Loading Firmware

The firmware binary **.bin** files can be loaded to the target using any means of your choosing, including vendor-supplied programmers, on-chip bootloaders, or the firmware loading tool supplied in the /tools folder for Windows PCs.  Instructions for use are found with the tool.

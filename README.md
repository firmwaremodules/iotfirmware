# IoT Firmware

Secure, scalable and low-power firmware for the Internet of *Things*.  These firmware reference designs are easy to use and ready to work with popular development platforms for ARM Cortex-M MCUs.

## Supported Development Platforms

TI MCU and LaunchPad Ecosystem:
* MSP432 with CC3100  [*MSP-EXP432P401R + CC3100BOOST*]
* CC2650 with CC3100  [*LAUNCHXL-CC2650 + CC3100BOOST*]
* CC1350 with CC3100  [*LAUNCHXL-CC1350 + CC3100BOOST*]
* CC1310 with CC3100  [*LAUNCHXL-CC1310 + CC3100BOOST*]

The MSP432 is a more capable "gateway-class" device with more RAM and Flash but no internal radio.  The MSP432's low-power architecture means that it can be built into the design of battery-powered IoT gateways and edge-routers.

The CC26XX and CC13XX class of devices offer extremely low power consumption with on-chip radios and LPLAN network stacks making them ideal for implementing devices at the IoT's edge - connecting to sensors and actuators to interact with the physical world.

The CC3100 is a WiFi network processor hosting the entire secure WiFi network stack.  A simple interface allows mating to the MSP432 gateway MCU.

All of these devices have on-board security support allowing for the design of extremely secure end-to-end IoT networks.


## Interesting IoT Applications

Each project folder contains firmware reference design binaries that implement an interesting IoT application for each supported platform.  Instructions to use each binary on the supported development platform is provided in the folder.

A brief description of available IoT applications is provided below.

### OTA Update Demo */otaupdate*

Demonstrates a robust and secure over-the-air firmware application update.  The firmware update binary is securely pulled from this very repository.

## Loading Firmware

The firmware binary **.bin** files can be loaded to the target using any means of your choosing, including vendor-supplied programmers, on-chip bootloaders, or Firmware Modules' own firmware loading tool `fm_load` supplied in the /tools folder for Windows PCs.

## Customized Firmware

If you'd like to customize one of our reference designs for your application, or have an application in mind, get in touch at `contact@firmwaremodules.com`



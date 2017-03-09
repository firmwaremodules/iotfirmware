# IoT Firmware

Secure, scalable and low-power firmware for the Internet of *Things*.  These firmware reference design binaries are easy to use and ready to work with popular development platforms for ARM Cortex-M MCUs.

## Supported Development Platforms

TI MCU and LaunchPad Ecosystem:
* MSP432 with CC3100  [*MSP-EXP432P401R + CC3100BOOST*]
* CC2650 with CC3100  [*LAUNCHXL-CC2650 + CC3100BOOST*]
* CC1350 with CC3100  [*LAUNCHXL-CC1350 + CC3100BOOST*]
* CC1310 with CC3100  [*LAUNCHXL-CC1310 + CC3100BOOST*]

The MSP432 is a more capable "gateway-class" device with more RAM and Flash but no internal radio.  The MSP432's low-power architecture means that it can be built into the design of battery-powered IoT gateways and edge-routers.

The CC26XX and CC13XX class of devices offer extremely low power consumption with on-chip radios and LPLAN network stacks making them ideal for implementing devices at the IoT's edge - connecting to sensors and actuators to interact with the physical world.
* The CC26xx devices target 2.4 GHz phy LPLANs including BLE, 802.15.4
* The CC1310 device targets sub-GHz phy LPLANs including 802.15.4g and proprietary.
* The CC1350 device targets a hybrid phy network architecture with BLE (2.4) and 802.15.4 (sub-GHz) possible to support novel device provisioning schemes.

The CC3100 is a WiFi network processor hosting the entire secure WiFi network stack.  A simple interface allows mating to the MSP432 gateway MCU (and even directly to the CC26xx and CC13xx devices as demonstrated in the OTA Update demos).

All of these devices have on-board security support allowing for the design of extremely secure end-to-end IoT networks.


## Interesting IoT Applications

Each project folder contains firmware reference design binaries that implement an interesting IoT application for each supported platform.  Instructions to use each binary on the supported development platform is provided in the folder.

A brief description of available IoT applications is provided below.

### OTA Update Demo */otaupdate/*

Demonstrates a robust and secure over-the-air firmware application update.  The firmware update binary is securely pulled from this very repository.

### Cloud and Sensor Platforms ###
* Xively with Seeed Studio Grove  */xively/grove/*

## Loading Firmware

The firmware binary **.bin** files can be loaded to the target using any means of your choosing, including vendor-supplied programmers, on-chip bootloaders, or Firmware Modules' own firmware loading tool `fm_load` supplied in the /tools folder for Windows PCs.

## Customized Firmware

If you'd like to customize one of our reference designs for your product, or have an application in mind, get in touch at `contact@firmwaremodules.com`

## FAQ

### Where's the source code?
* We use a highy integrated build and development process that is comprised of Java, Javascript and Python (in addition to 'C' code!) and consequently it is infeasible to distribute and parcel off pieces of our system for general consumption. The output of this system is what you see here; we produce manufacturable and OTA updatable firmware binaries that include bootloaders, provisioning apps, integrity checking, embedded versioning and security considered throughout - all tailored for a particular hardware platform and use case (e.g. MSP432 MCU, with CC3100 WiFi network processor, and GitHub as the OTA update host, and Xively as the IoT data platform).



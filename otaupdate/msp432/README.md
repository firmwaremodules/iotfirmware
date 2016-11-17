# Secure OTA Firmware Update with MSP432


## About this project
Demonstrate a secure and robust over-the-air firmware update from a GitHub repository over WiFi with the MSP432 LaunchPad and CC3100 BoosterPack.

## Project info
Difficulty: Easy  
Estimated Time: 10 minutes

## Things Used in this project

### Hardware

* [TI MSP-EXP432P401R LaunchPad](http://www.ti.com/tool/MSP-EXP432P401R)
* [TI CC3100BOOST BoosterPack](http://www.ti.com/tool/CC3100BOOST)

### Software

* [OTA MSP432 MAN Firmware Binary]()

### Tools

* [Firmware Modules Firmware Loader](https://github.com/firmwaremodules/iotfirmware/raw/master/tools/fm_load.exe)

## Story
### Introduction
The purpose of this demonstration is to show how a firmware update image can be securely hosted and retrieved from a publicly accessible cloud server and installed in a high-availability IoT device.  The device's internal flash is partitioned into 4 sections:
* A simple non-updatable bootloader in the BOOT section.
* A completely separate provisioning application containing features and functions intended only to be used during manufacturing of the device in the MTA section.
* Two equally sized executable application partitions called APP1 and APP2.

There are two types of firmware image files used in this demo:
* The MAN image (short for manufacturing) is the combination of bootloader, MTA and APP binaries and is programmed onto the device's flash before it is sent to the field.
* The APP image is the main application implementing the device's main functions.  The APP contains the OTA update logic, network stacks and/or interfaces and OS, and thus all components critical to the execution of the device are updated together.  The APP is encrypted with AES-CBC.  The APP can be placed into either the APP1 or APP2 sections at the discretion of the OTA update logic.

When the device is first provisioned with the MAN firmware image, the application image resides in the APP1 section, and APP2 is empty.  The bootloader checks for a valid application in either section and boots the most recent (highest version).  

The demo application polls a fixed URL for the presence of newer firmware images.  When a new application image is found, it is downloaded through a TLS-secured link.  The application image itself is decrypted on-the-fly with the embedded AES hardware and stored into the APP2 section.  The update logic in this demo automatically and immediately reboots the device when the stored image has been verified.  The bootloader then determines that the new image is valid and jumps to it, completing the update process.   

If the update is interrupted at any time, for example mid-way during the download or during a flash write, or if the image could not otherwise be correctly written in its entirety to the APP2 section, the bootloader will not accept the content of APP2 and continue to boot the existing application. This is an extremely robust update process designed for IoT devices operating in real-world conditions requiring no external components.  However, it comes at a cost - the amount of internal flash available to applications is reduced by half.

### Steps to complete the demo

1. Assemble the MSP432 LaunchPad and CC3100 BoosterPack as shown in the cover image and connect to your PC with a USB cable.

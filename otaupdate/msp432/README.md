# Secure OTA Firmware Update with MSP432


## About this project
Demonstrate a secure and robust over-the-air firmware update from a GitHub repository over WiFi with the MSP432 Cortex-M4 MCU LaunchPad and the CC3100 WiFi BoosterPack.

## Project info
Difficulty: Easy  
Estimated Time: 15 minutes

## Things used in this project

### Hardware

* [TI MSP-EXP432P401R LaunchPad](http://www.ti.com/tool/MSP-EXP432P401R) x1
* [TI CC3100BOOST BoosterPack](http://www.ti.com/tool/CC3100BOOST) x1
* Jumper wire for 0.1" male headers x1

### Software

* [OTA MSP432 MAN Firmware Binary]()

### Software Tools

* [Firmware Modules Firmware Loader](https://github.com/firmwaremodules/iotfirmware/raw/master/tools/fm_load.exe)
* Terminal emulator such as [Tera Term](https://ttssh2.osdn.jp/index.html.en) or [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/)

## Story
### Introduction
The purpose of this demonstration is to show how a firmware update image can be securely hosted and retrieved from a publicly accessible cloud server and installed into the internal flash of a high-availability IoT MCU device.  The device's internal flash is partitioned into 4 sections:
* A simple non-updatable bootloader in the BOOT section.
* A completely separate provisioning application containing features and functions intended only to be used during manufacturing of the device in the MTA section.
* Two equally sized executable application sections called APP1 and APP2.

There are two types of firmware image files used in this demo:
* The MAN image (short for manufacturing) is the combination of bootloader, MTA and APP binaries and is programmed onto the device's internal flash before it is sent to the field.
* The APP image is the main application implementing the device's main functions.  The APP contains the OTA update logic, network stacks and/or interfaces and OS, and thus all components critical to the execution of the device are updated together.  The APP is encrypted with AES-CBC.  The APP can be placed into either the APP1 or APP2 sections at the discretion of the OTA update logic.

When the device is first provisioned with the MAN firmware image, the application image resides in the APP1 section, and APP2 is empty.  The bootloader checks for a valid application in either section and boots the most recent (highest version).  

The demo application polls a fixed URL for the presence of newer firmware images.  When a new application image is found, it is downloaded through a TLS-secured link.  The application image itself is decrypted on-the-fly with the embedded AES hardware and stored into the APP2 section.  The update logic in this demo automatically and immediately reboots the device when the stored image has been verified.  The bootloader then determines that the new image is valid and jumps to it, completing the update process.   

If the update is interrupted at any time, for example mid-way during the download or during a flash write, or if the image could not otherwise be correctly written in its entirety to the APP2 section, the bootloader will not accept the content of APP2 and continue to boot the existing application. This is an extremely robust update process designed for IoT devices operating in real-world conditions and requires no external components.  However, it comes at a cost - the amount of internal flash available to applications is reduced by half.

### Steps to complete the demo

1. **Assemble the hardware.** Assemble the MSP432 LaunchPad and CC3100 BoosterPack as shown in the cover image and connect to your PC with a USB cable.
2. **Get the firmware files.** Download the firmware loader tool `fm_load.exe` and the `OTA-MSP432-CC3100_x_x_x_MAN_x.bin` firmware binary programming file to a folder on your PC.
3. **Test the connection.** Open a console and run `fm_load.exe` to ensure your LaunchPad is connected.  You should see the following:

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
  
4. **Start the terminal.** Start a terminal emulator session/connection on the LaunchPad's "Application/User UART" at 9600 baud.  On some PCs in Tera Term this appears as "COM4: XDS110 Class Application/User UART (COM4)".  Choose the COM port that assigned by your PC.

5. **Load the firmware.** Load the MAN firmware binary with `fm_load OTA-MSP432_1_0_15_MAN_0x1509FB65.bin`.  Observe loading progress to completion (100%):  

  INFO:root:Board unique id 0000  
  INFO:root:DAP JTAG MODE initialised  
  INFO:root:6 hardware breakpoints, 4 literal comparators  
  INFO:root:CPU core is Cortex-M4  
  INFO:root:FPU present  
  INFO:root:4 hardware watchpoints 
  [====================] 100%  
  
  Observe the serial terminal window's output [link]() showing the need to setting the WiFi parameters which we'll do next.
  
7. **Provision the device's WiFi settings.**  To do this we must boot into the device's MTA (manufacturing test application) by pulling GPIO P4.0 high and resetting the board.  Connect a short jumper wire as shown [link]() between the BoosterPack connector pins P3-4 and P1-1 and press the reset button.  The serial terminal will show a simple menu requesting that you fill in 3 fields, your WiFi network's SSID, security type, and passkey.  Enter each in turn.  When complete, commit the settings to flash with menu option 4.  Remove the jumper wire from the 3.3V pin but leave the other end attached - you'll need to use it again later.

8. **Perform the update.** Reset the board (ensure the jumper is disconnected) to boot back into the main application.  This time it should connect to your local WiFi network and proceed to download and install the firmware update sourced from OTA-MSP432-CC3100_1_1_15_APP_0xA94C2B4C.fmu.  This update is identical in function to the previous version, except that the version 'minor' digit has been incremented from 0 to 1, and there is an extra message in the banner indicating that the updated app is running.  When the update is complete, the board automatically reboots and you should see the updated application now running out of the internal flash's APP2 section.

9. **Rollback the update.**

10. **Interrupt the update.**

### What's next

This IoT firmware update demo is just the tip of the iceberg when it comes to the complex world of IoT device management.  This demo could be enhanced to show:
* Updating the firmware on the external CC3100 network processor using the same transfer mechanism.  Instead of "APP" files the network processor updates could be "NWP" files and the OTA server running on the MSP432 would route those update file contents to the CC3100's SFLASH as appropriate.
* Provision the WiFi parameters using a mobile device.

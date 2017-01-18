# Secure OTA Firmware Update with CC2650

  ![](https://github.com/firmwaremodules/iotfirmware/raw/master/otaupdate/cc2650/resources/OTA-CC2650-Cover.jpg "CC2650 LaunchPad with CC3100 BoosterPack")

## About this project
Demonstrate a secure and robust over-the-air firmware update from a GitHub repository over WiFi with the CC2650 Cortex-M3 MCU LaunchPad and the CC3100 WiFi BoosterPack.
This demonstration deploys the OTA update technology previously developed for the MSP432 to the CC26XX and CC13XX family of devices.

## Project info
Difficulty: Easy  
Estimated Time: 15 minutes

## Things used in this project

### Hardware

* [TI LAUNCHXL-CC2650 LaunchPad](http://www.ti.com/tool/LAUNCHXL-CC2650) x1
* [TI CC3100BOOST BoosterPack](http://www.ti.com/tool/CC3100BOOST) x1
* Jumper wire x1 such as [these](http://www.digikey.com/product-search/en?keywords=1528-1159-ND).

### Software

* [OTA CC2650 MAN Firmware Binary](https://github.com/firmwaremodules/iotfirmware/raw/master/otaupdate/cc2650/OTA-CC2650-CC3100_1_0_21_MAN_0xB61214C5.bin)

### Software Tools

* [Firmware Modules Firmware Loader](https://github.com/firmwaremodules/iotfirmware/raw/master/tools/fm_load.exe)
* Terminal emulator such as [Tera Term](https://ttssh2.osdn.jp/index.html.en) or [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/)

## Story
### Introduction
The purpose of this demonstration is to show how a firmware update image can be securely hosted and retrieved from a publicly accessible cloud server and installed into the internal flash of a high-availability IoT MCU device.  The device's internal flash is partitioned into 4 sections:
* A simple non-updatable bootloader in the **BOOT** section.
* A completely separate provisioning application containing features and functions intended only to be used during manufacturing of the device in the **MTA** section.
* Two equally sized executable application sections called **APP1** and **APP2**.

There are two types of firmware image files used in this demo:
* The MAN image (short for manufacturing) is the combination of bootloader, MTA and APP binaries and is programmed onto the device's internal flash before it is sent to the field.
* The APP image is the main application implementing the device's main functions.  The APP contains the OTA update logic, network stacks and/or interfaces and OS, and thus all components critical to the execution of the device are updated together.  The APP is encrypted with AES-CBC.  The APP can be placed into either the APP1 or APP2 sections at the discretion of the OTA update logic.

When the device is first provisioned with the MAN firmware image, the application image resides in the APP1 section, and APP2 is empty.  The bootloader checks for a valid application in either section and boots the most recent (highest version).  

The demo application polls a fixed URL for the presence of newer firmware images.  When a new application image is found, it is downloaded through a TLS-secured link.  The application image itself is decrypted on-the-fly with the MCU's embedded AES hardware and stored into the APP2 section.  The update logic in this demo automatically and immediately reboots the device when the stored image has been verified.  The bootloader then determines that the new image is valid and jumps to it, completing the update process.   

If the update is interrupted at any time, for example mid-way during the download or during a flash write, or if the image could not otherwise be correctly written in its entirety to the APP2 section, the bootloader will not accept the content of APP2 and continue to boot the existing application. This is an extremely robust update process designed for IoT devices operating in real-world conditions and requires no external components.  However, it comes at a cost - the amount of internal flash available to applications is reduced by half.

### What will happen?

* The firmware application running on the CC2650 target MCU will be updated by downloading a new version of the same application from a folder on GitHub.

### Steps to complete the demo

1. **Assemble the hardware.** Assemble the CC2650 LaunchPad and CC3100 BoosterPack as shown in the image below and connect to your PC with a USB cable.  The CC3100 BoosterPack can be mounted either on top or below, however mounting it below affords better access to the LaunchPad header markings and jumpers.  Take note of the re-routing of the XDS110's UART to pins DIO29 and DIO30.  The default routing of the UART connects the XDS100 directly to the CC3100's UART and must be avoided.  The demo application routes TXD to DIO29 and RXD to DIO30. Remove the TXD and RXD jumpers on the LaunchPad and connect the XDS110 side to DIO29 and DIO30.  Connect a jumper from 3.3V to DIO26 to select the provisioning application (the MTA) for initial boot.

  ![](https://github.com/firmwaremodules/iotfirmware/raw/master/otaupdate/cc2650/resources/OTA-CC2650-Board-Setup.jpg "CC2650 LaunchPad with CC3100 BoosterPack and jumpers")
  
2. **Get the firmware files.** Download the firmware loader tool `fm_load.exe` and the `OTA-CC2650-CC3100_x_x_x_MAN_x.bin` firmware binary programming file to a folder on your PC.

3. **Test the connection.** Open a console and run `fm_load.exe` to ensure your LaunchPad is connected.  You should see the following:
  ```
  Firmware Module System Firmware Loader v1.2.65
  Copyright (c) 2016 Firmware Modules Inc.
  License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE
  Firmware loader for XDS110 with TI MSP432, CC13xx, CC26xx devices.

  INFO:root:DAP JTAG MODE initialised
  INFO:root:Detecting target... cc2650f128rgz
  INFO:root:6 hardware breakpoints, 4 literal comparators
  INFO:root:CPU core is Cortex-M3
  INFO:root:4 hardware watchpoints
  No operation performed  
  ```
4. **Start the terminal.** Start a terminal emulator session/connection on the LaunchPad's "Application/User UART" at 115200 baud.  On some PCs in Tera Term this appears as "COM4: XDS110 Class Application/User UART (COM4)".  Choose the COM port that is assigned by your PC.

5. **Load the firmware.** Load the MAN firmware binary with `fm_load OTA-CC2650-CC3100_1_0_21_MAN_0xB61214C5.bin`.  Observe loading progress to completion (100%):  
  ```
  INFO:root:DAP JTAG MODE initialised
  INFO:root:Detecting target... cc2650f128rgz
  INFO:root:6 hardware breakpoints, 4 literal comparators
  INFO:root:CPU core is Cortex-M3
  INFO:root:4 hardware watchpoints
  Writing 62176 bytes...
  [====================] 100%
  ```  
  Observe the serial terminal window's output showing the need to set the WiFi parameters which we'll do next.
  
7. **Provision the device's WiFi settings.**  To do this we must boot into the device's MTA (manufacturing test application) by pulling DIO26 high and resetting the board.  Connect a short jumper wire as shown between the BoosterPack connector pins J3-4 and J1-1 and press the reset button.  

  
  The serial terminal will show a simple menu requesting that you fill in 3 fields, your WiFi network's SSID, security type, and passkey.  Enter each in turn as shown.  
  
  ```
************************************************************************
 Firmware Modules OTA Update Demo MTA for LAUNCHXL-CC2650 + CC3100BOOST
************************************************************************

 WiFi Settings:
   
   SSID: not set
   Security Type: OPEN
   Passkey: not set

 WiFi Menu:
   
   1) Set SSID
   2) Set Security Type
   3) Set Passkey

  >
  ```

  When complete, commit the settings to flash with menu option `4) Save WiFi settings`.  Remove the jumper wire from the 3.3V pin but leave the other end attached - you'll need to use it again later.  
  
    
8. **Perform the update.** Reset the board (ensure the jumper is disconnected) to boot back into the main application.  This time it should connect to your local WiFi network and proceed to download and install the firmware update sourced from OTA-CC2650-CC3100_1_1_21_APP_0x99473585.fmu.  
  ```
************************************************************************
 Firmware Modules OTA Update Demo APP for LAUNCHXL-CC2650 + CC3100BOOST
************************************************************************
BOOT version: 1.0.21
MTA version: 1.0.21
APP1 version: 1.0.21
APP2 version: 0.0.0
Current version: 1.0.21
Starting network processor....success
CHIP version 67108864
MAC version 1.2.0.2
PHY version 1.0.3.23
NWP version 2.2.0.1
ROM version 13107
HOST version 1.0.0.1
MAC f4:b8:5e:04:61:9b
WiFi settings retrieved
SSID: ********
Security type: WPA_WPA2
Passkey: **********
Connecting to AP...success
Setting time...success
Unique ID is "FirmwareModules OTA DEMO 99 0"
Connecting to HTTP server...success
Checking for available firmware updates...
Polling https://api.github.com/repos/firmwaremodules/iotfirmware/contents/otaupdate/cc2650/updates ...
 Type   Current   Update
 -----------------------
 APP     1.0.21   1.1.21  updating..................................complete!



Demo ended.
System going down NOW
  ```
  This update is identical in function to the previous version, except that the version 'minor' digit has been incremented from 0 to 1, and there is an extra message in the banner indicating that the updated app is running.  When the update is complete, the board automatically reboots and you should see the updated application now running out of the internal flash's APP2 section.  

  ```
************************************************************************
 Firmware Modules OTA Update Demo APP for LAUNCHXL-CC2650 + CC3100BOOST
 Running the updated APP!
************************************************************************
BOOT version: 1.0.21
MTA version: 1.0.21
APP1 version: 1.0.21
APP2 version: 1.1.21
Current version: 1.1.21
Starting network processor....success
CHIP version 67108864
MAC version 1.2.0.2
PHY version 1.0.3.23
NWP version 2.2.0.1
ROM version 13107
HOST version 1.0.0.1
MAC f4:b8:5e:04:61:9b
WiFi settings retrieved
SSID: ********
Security type: WPA_WPA2
Passkey: **********
Connecting to AP...success
Setting time...success
Unique ID is "FirmwareModules OTA DEMO 99 1"
Connecting to HTTP server...success
Checking for available firmware updates...
Polling https://api.github.com/repos/firmwaremodules/iotfirmware/contents/otaupdate/cc2650/updates ...
 Type   Current   Update
 -----------------------
 no updates found.

Demo ended.
  ```

9. **Rollback the update.** Re-connect the jumper wire between DIO26 and 3.3V and reset the board to access the MTA.  The menu will have a new option `5) rollback a firmware update` as shown. Select this option and confirm with `y`. 

  ```
************************************************************************
 Firmware Modules OTA Update Demo MTA for LAUNCHXL-CC2650 + CC3100BOOST
************************************************************************

 WiFi Settings:
   
   SSID: *****
   Security Type: WPA_WPA2
   Passkey: *****

 WiFi Menu:
   
   1) Set SSID
   2) Set Security Type
   3) Set Passkey
   5) Rollback a firmware update

  > 5 rollback to version 1.0.21 from 1.1.21 y/n?
  ```
10. **Interrupt the update.**  Remove the jumper wire again from the 3.3V pin and reset the board to boot the main APP.  Now this time around, you want to have your finger hovering over the reset button as the demo progresses.  When the `updating.........` text begins to appear in the terminal window, press the reset button to interrupt the download mid-update.  At this stage, a portion of the image has been written to the APP2 section in internal flash, and a flash write operation may have been ongoing.  When the application boots up again, it remains untouched and is the same as before.  The update is detected and re-attempted.  You can also remove power to the LaunchPad during this update period.  If you miss the update window and it completes, just perform a rollback again. See the example terminal window output below.


### So how does this work?

The bootloader, manufacturing test application and main application are independently developed, compiled and linked with complete vector tables.  They are assembled into the combined MAN image in a post-build ("release") step.  At the same time, the application images are embedded into a firmware update container format (**.fmu**) and then encrypted.  You can confirm that the .fmu file is encrypted by noting that there is no evidence of a vector table anywhere in the file including the SP and PC initialization values typical at word offsets 0 and 1.

The bootloader is located at the beginning of the MAN image and after programming, it is placed at the location where the Cortex-M processor expects an application to reside on startup.  The bootloader performs self-tests, checks the status of the MTA selection pin (DIO26) and verifies the integrity of all application images to decide where to set the processor's VTOR (vector table offset register).  Once the VTOR is set, the PC (program counter) is moved to point to the target application's reset vector (specified at word offset 1 in the target application's vector table). The bootloader's job ends there and the application's startup procedures take over and reconfigures the device as required for the application.

If the bootloader encounters any fatal errors, it is designed to signal the error condition to port DIO6 - conveniently attached to the LaunchPad's red LED1.

The encryption key is stored with the application and is therefore vulnerable to inspection in the MAN image.  However the MAN image is not distributed and intended to be used only in a "trusted" product manufacturing environment (such as your workbench or lab).  Once the MAN image is programmed onto the device, and the JTAG/CC2650-bootloader access ports have been disabled by writing appropriately to the CCFG section, the encryption key and indeed the firmware content itself is no longer easily accessible to anyone that can gain access to your firmware update file or device.



### What's next

You've just experienced the result of significant behind-the-scenes effort to coordinate the production and consumption of multiple firmware entities for an MCU-based IoT device. This IoT firmware update demo is just the tip of the iceberg when it comes to the complex world of IoT device management. Other, more functional, IoT application demos will be built on top of this technology.  

This specific demo could be enhanced to show:
* Updating the firmware on the external CC3100 network processor using the same transfer mechanism.  Instead of "APP" files the network processor updates could be "NWP" files and the OTA server running on the CC2650 would route those update file contents to the CC3100's SFLASH as appropriate.
* Provision the WiFi parameters using a mobile device.
* Pulling update files from other content distribution networks such as Sync or Dropbox.
* Adding a Certificate Authority certificate to verify the content server (in this case GitHub).  As it stands, the device could be tricked into redirecting to an unauthorized content server to download the firmware update file.  However, due to the update file encryption and OTA design, it would be very difficult to create an unauthorized update file that the device would accept.
* Add the CC2650 as an OTA update "target", much like the CC3100, when hosted by an MSP432 in a gateway-style setup (CC3100 + MSP432 + CC2650 bridges end nodes over LPLAN networks to WiFi and the Internet).
* Utilize the LAUNCHXL-CC2650's on-board SPI flash to store larger update files that include the TI-RTOS + BLE or 802.15.4 stacks.  The current demonstration uses custom-designed drivers and operating context with SimpleLink and can fit within one-half of the CC2650's internal 128K SRAM to support a completely internal OTA update strategy as shown with this demo.  However, the addition of the BLE or 802.15.4 stacks will push the application towards the 128K limit requiring the OTA update system to leverage off-board storage.

### Errata

* Demo application sometimes hangs during connection to network processor (CC3100).  This is a possible "sync lockup" issue.  Workaround: Pressing the launchpad's reset button usually makes the connection on the next attempt.  Updating the CC3100's firmware may also resolve the issue.
* Demo application does not automatically reset after firmware update.  Workaround: manually reset the launchpad by pressing the reset button.

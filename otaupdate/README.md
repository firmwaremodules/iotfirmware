# OTA Firmware Update Demo
This folder contains OTA (over the air) firmware update demo applications for supported ARM MCU development platforms.

The OTA Update demo application showcases the Firmware Modules firmware deployment technology that combines a bootloader, manufacturing test application (MTA) and main application (APP) into a ready-to-load firmware binary (MAN), with simultaneous production of a secure encrypted firmware application update image for in-field OTA updating.

In this demonstration, the encrypted application update image is hosted on GitHub in the /updates folder.  The application running on the target polls GitHub for a newer version and proceeds to safely and securely (via TLS) download the encrypted firmware update image.  When completed, the demo application reboots and the target begins running the newly updated application.  

This update technology is extremely robust against network or power failures.  You may disconnect the network or reset the target at any time during the update process (you have to be quick to catch it however) and the currently running application always remains intact.  Only when the updated application has been fully placed into flash with verification will the switch-over occur. The user is encouraged to disrupt the update download and installation process at any time through any means (e.g. pressing reset button) to see just how robust this high-availablity OTA update system really is.  If the update is left to run to completion, a rollback can be performed by rebooting back into the MTA provisiong app and selecting the rollback option.

## How this works

The firmware application is decrypted then relocated at update time into one of two partitions in device internal flash.  The relocated application is run from exactly where it was placed.  The currently running application is never touched and the bootloader will boot the new application only when it is verified via checksum.  It is possible to rollback to the previous application should a situation require it.

The firmware update image is encrypted with AES cipher block chaining (CBC) and decrypted on-board the target using dedicated AES hardware.  The encrypted update firmware image can therefore be distributed "in the open" via untrusted channels if necessary, reducing the risks associated with IP theft, cloning or hijacking.

## General steps to run the OTA Update Demo

The OTA demo firmware application uses a serial port to communicate with a terminal program running on the host PC.  The exact steps for each development platform may be slightly different and are documented accordingly in the folders.  Generally, the process is:

1. Use a firmware loading tool to load the .bin file to the target.
2. Provision the network connection parameters SSID, type and password by invoking the MTA.
3. Reboot the target and observe serial terminal messages showing progress of the OTA update.

## Demo Specifics

OTA update demo binaries and detailed writeups are available for the following hardware platforms
* MSP432 + CC3100 in **msp432** folder




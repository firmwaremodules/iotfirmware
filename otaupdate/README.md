# OTA Update Demo
This folder contains OTA (over the air) firmware update demo applications for supported ARM MCU development platforms.

The OTA Update demo application showcases the Firmware Modules firmware deployment technology that combines a bootloader, manufacturing test application (MTA) and main application (APP) into a ready-to-load firmware binary (MAN), with simultaneous production of a secure firmware application update image for in-field OTA updating.

In this demonstration, the secure application update image is hosted on GitHub in the /updates folder.  The application running on the target polls GitHub for a newer version and proceeds to safely and securely download the firmware update image.  When completed, the demo application reboots and the target begins running the newly updated application.

This update technology is extremely robust against network or power failures.  You may disconnect the network or reset the target at any time during the update process (you have to be quick to catch it however) and the currently running application always remains intact.  Only when the updated application has been fully placed into flash with verification will the switch-over occur.

## General steps to run the OTA Update Demo

The OTA demo firmware application uses a serial port to communicate with a terminal program running on the host PC.  The exact steps for each development platform may be slightly different and are documented accordingly in the folders.  Generally, though:

1. Use a firmware loading tool to load the .bin file to the target.
2. Provision the network connection parameters by invoking the MTA.
3. Reboot the target and observe serial terminal messages showing progress of the OTA update.




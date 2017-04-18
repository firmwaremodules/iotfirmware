# Turn your mcModule120 into a Sensor Beacon with support for Eddystone!

  ![](https://github.com/firmwaremodules/iotfirmware/raw/master/mcthings/eddystone/resources/mcmod120_eddystone_cover.png "Mark up the world with mcThings mcModule 120 with Eddystone Firmware by Firmware Modules")

## About this project
Create and deploy an Eddystone compatible beacon to implement a vast array of Physical Web, proximity and context-aware applications! This project uses beacon-ready hardware from mcThings and firmware by Firmware Modules.

## Project info
Difficulty: Easy  
Estimated Time: 60 minutes

## Things used in this project

### Hardware

* [mcThings mcModule120](https://www.mcthings.com/products/mcmodule120/) x1
* [TI MSP-EXP432P401R LaunchPad](http://www.ti.com/tool/MSP-EXP432P401R) x1
* Jumper wire x1 such as [these](https://www.digikey.com/product-detail/en/twin-industries/TW-MP-10/438-1074-ND/2116120).

### Software

* [mcMod120 Eddystone Firmware](https://github.com/firmwaremodules/iotfirmware/raw/master/mcthings/eddystone/ble_app_eddystone_mcmod120_s132.hex)
* [Nordic Semiconductor SoftDevice S132 v4.0.2](https://www.nordicsemi.com/eng/Products/S132-SoftDevice)

### Software Tools

* [Firmware Modules Firmware Loader v1.2.68+](https://github.com/firmwaremodules/iotfirmware/raw/master/tools/fm_load.exe)

## Story
### Introduction

Get access to the Physical Web and a growing list of Sensor Beacon use cases with the powerful and low-power mcModule120 IoT sensor node from mcThings.  With your mcModule120 loaded with Sensor Beacon for Eddystone firmware, you can broadcast URL, UID and TLM (telemetry) frames. In many scenarios, the user's Android device will automatically notify the user of the presense of a nearby beacon device with the content of the URL - all without an installed app.  More advanced mobile device applications can be  built to utilize the UID and TLM frames for applications such as asset tracking, industrial process monitoring, smart city deployments, and many commercial applications such as proximity-driven experience enhancements.

The firmware you'll load onto the device consists of two parts:
- The Nordic Semiconductor "SoftDevice", a pre-built IoT firmware core containing BLE stack and device abstraction layers
- The Eddystone firmware application by Firmware Modules.

There are 4 steps to complete the project:
  1. **Wire the module to the debugger**
  2. **Unlock the module**
  3. **Program the firmware**
  4. **Test and configure the Eddystone application**

**!!Warning!!**
This project turns the mcModule120 into a powerful, general purpose, IoT hardware device that, with the proper firmware, could be deployed into the field.  This project requires that you unlock your mcModule120 device which will *permanently* remove the pre-loaded mcThings firmware.  There is no way to recover the device back into a mcThings development platform. If you're OK with exploring new applications with your mcModule120, then let's get to it!

#### Wire the module to the debugger and unlock the module
The first thing you'll need to do is to connect your mcModule120 to a low-cost CMSIS-DAP ARM debugger so that we can unlock then program the mcModule120.  Unlocking the mcModule120 will turn it into a general-purpose development platform for the nRF52832 MCU with an accurate temperature sensor, 3-axis accelerometer, 2 LEDs and 1 button.  Programming it will enable the Eddystone beacon functionality.

The CMSIS-DAP debugger we're using is the under-$20 TI MSP-EXP432P401R LaunchPad.  This LaunchPad can be turned into a general-purpose ARM debugger that is significantly more economical than the Segger J-Link setup that is typical for Nordic development, yet offers full access to the nRF52832 MCU.  You can use any CMSIS-DAP/DAPLink debugger supported by fm_load, which, at this time, is the XDS110 (from TI).  Most of the newer TI Launchpads contain the XDS110 debugger.

The fm_load application from Firmware Modules can unlock (factory reset) and program the mcModule120 devices.  There are other tools and methods out there to do these things, but this project will just describe the use of fm_load. 

The MSP432 Launchpad enables external connection to the XDS110 debugger through an unpopulated 7-pin header pad J103.  To configure the launchpad for external debug use, all jumpers must be removed, and the JTAG switch S101 must be set to "Ext Debug" to disconnect the on-board MCU from the debug path (see the image below for clarification).  Once configured, you can now connect the mcModule120 to the 7-pin header pads either by soldering wires, or using "machine pin jumper wires" that can be inserted into the holes on both the Launchpad and the mcModule120.  We'll be connecting these 7 signals as follows (however UART RX/TX are optional).  The signals are labeled conveniently on both PCBs and should be easy to follow as you connect the pins.  Note that it was easier to fit the pins together by alternating insertion from the top and bottom of the PCBs.

| mcModule120 | XDS110 |
|-------------|--------|
| VDD         | 3V3    |
| GND         | GND    |
| SWDIO       | SWDIO  |
| SWDCLK      | SWCLK  |
| #RESET	   | RST    |
| PIN0 A/D	   | TXD    | * Optional
| PIN1 A/D	   | RXD    | * Optional

* The optional TXD/RXD pins simply connect the mcModule's UART0 and the application's logging function to the XDS110's "Appication/User UART" at 115200 baud for a few informational messages that can be observed with any serial terminal program.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/mcthings/eddystone/resources/mcmod120_eddystone_programming_setup.png "Program the mcThings mcModule 120 with a TI LaunchPad and XDS110 debugger")

When you've completed the hookup, connect the Lauchpad's USB cable.  Next, download and run fm_load.  It'll try to detect the mcModule120 but should fail to do so because the module is locked. If the connections were made successfully, you should see the following output from fm_load when connecting with a locked mcModule120:
```
Firmware Module System Firmware Loader v1.2.68
Copyright (c) 2016 Firmware Modules Inc.
License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE
Firmware loader for XDS110 with TI MSP432, CC13xx, CC26xx devices; Nordic nRF52832

INFO:root:DAP JTAG MODE initialised
INFO:root:DAP SWD MODE initialised
WARNING:root:Unknown AHB IDR: 0x23000000
Unsupported SWD target
Traceback (most recent call last):
  File "<string>", line 273, in <module>
  File "<string>", line 166, in main
  File "pyOCD\board\mbed_board.py", line 375, in chooseBoard
  File "pyOCD\board\board.py", line 54, in init
  File "pyOCD\target\cortex_m.py", line 619, in init
Exception: Unsupported SWD target
flash_tool returned -1
```

Next we'll unlock the device.

Run fm_load with the options "--factory_reset" and "-t nrf52832" as shown:
```
fm_load -t nrf52832 --factory_reset

Firmware Module System Firmware Loader v1.2.68
Copyright (c) 2016 Firmware Modules Inc.
License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE
Firmware loader for XDS110 with TI MSP432, CC13xx, CC26xx devices; Nordic nRF52832

Attempting to factory reset board...
XDS110 (02.02.04.00) with CMSIS-DAP 00000000 []
INFO:root:DAP SWD MODE initialised
INFO:root:erasing flash...
INFO:root:reset complete.
INFO:root:DAP JTAG MODE initialised
INFO:root:DAP SWD MODE initialised
INFO:root:Detecting target... nrf52832qfaa
INFO:root:6 hardware breakpoints, 4 literal comparators
INFO:root:CPU core is Cortex-M4
INFO:root:FPU present
INFO:root:4 hardware watchpoints
Success!
```

Now the module's flash has been completely erased and it's ready for new firmware.

#### Load the Eddystone firmware

Place the following files in a directory with fm_load.

Step 1: Download the S132 version 4.0.2 SoftDevice firmware from Nordic Semiconductor's website at this URL: https://www.nordicsemi.com/eng/Products/S132-SoftDevice

Step 2: Download the Eddystone application firmware from GitHub: [ble_app_eddystone_mcmod120_s132.hex](https://github.com/firmwaremodules/iotfirmware/raw/master/mcthings/eddystone/ble_app_eddystone_mcmod120_s132.hex)

Step 3: Load the SoftDevice to mcModule120 flash:
```
fm_load s132_nrf52_4.0.2_softdevice.hex

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
Writing 123172 bytes...
[====================] 100%
```

Step 4: Load the Eddystone application firmware to mcModule120 flash:
```
fm_load ble_app_eddystone_mcmod120_s132.hex

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
Writing 55344 bytes...
[====================] 100%
```

When the last step completes, your mcModule120 should be pulsing the green LED - showing beacon activity!

After programming, only power and ground connections are required to operate the module from a fixed power supply.  Alternatively, insert a coin cell battery (e.g. CR2302 style) and place your module somewhere in the environment!

The default Physical Web URL is https://www.mcthings.com which you may want to change.  Read the next section for a discussion on how to do that.



#### Test and configure the Eddystone application

If you've got this far, congratulations!  Now you get to play with your Eddystone-compatible beacon.

Two Android apps are useful in testing, validating and configuring the beacons sensor: *nRF Connect*, and *nRF Beacon for Eddystone*.  Both apps are available for free from the Play store and should be installed on your mobile device.

When the beacon powers up for the first time, it immediately enters a default state that repeatedly broadcasts three beacon frame types in sequence at 1 second intervals:
- URL frame containing https://www.mcthings.com
- UID frame containing placeholder UID namespace, id and rfu fields: AAAABBBBCCCCDDDDEEEE, 123456, 00 respectively.
- TLM frame containing vbat derived from the on-chip SAADC's VDD input source, temperature derived from TMP102 sensor, advertising frame count since startup and seconds since startup.

You can view this activity with *nRF Connect* in the Scanner tab.  The mcModule120's green LED pulses briefly for each beacon transmission.  The device also outputs the temperature measurement to the UART channel at 115200 baud when it is requested to update the TLM (every 30 seconds).

The real fun is when you get to configure the device with the Eddystone Configuration Service and the *nRF Beacon for Eddystone* app (you can access the raw BLE characteristics with other methods but this app provides a nice GUI for you to use).

Launching the nRF Beacon for Eddystone app, enter the "Update" tab which will present you an option to connect with the device.  The mcMod120 with Eddystone application firmware is able to connect to the this app if you follow these instructions.  Press the button on the mcMod120 and both green and red LEDs should pulse rapidly, once for each connectable advertising beacon sent (100 ms period: much faster than the normal advertising rate).  This will last for 60 seconds before it times out and enters its normal, non-connectable mode again.  While both green and red LEDs are flashing, tap the "Connect" button in the app.  The device, called "mcMod120_Eddystone", should be detected.

Enter the unlock key.  The unlock key is simply FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF (16 hexadecimal 0xFF bytes).  The device remains unlocked (open for configuration) until the next power cycle.

There are 5 assignable beacon advertising slots available.  Each slot can be independently configured in terms of power level, period and frame type.  Each frame type has different configuration options; the URL frame type allows you to specify a URL that is presented to nearby Android devices.  Play around with the options to get a sense of what is possible here.  Settings do not persist until the app disconnects.

*Important Note*
To save settings made in nRF Beacon for Eddystone app to the device, you must exit ALL apps using BLE to generate the disconnect message that is required for the firmware to store its current state to persistent storage.  This means that you should close *nRF Connect* before running *nRF Beacon for Eddystone* to ensure the disconnect message is sent!




### What's Next
Configure a URL that is meaningful for you and place your beacon in the environment.  Any newer Android device should notify you of this beacon as you approach it.

Eddystone beacons have a great many applications and use cases.  But once configured and deployed, the content they broadcast is largely static.  The next step is to enable dynamic and real-time deployment of content to the beacons behind the scenes and bypass the use of the Eddystone Configuration Service to configure the devices.

### Design Notes
- Because this application makes use of the device's LEDs, the battery life is much shorter than one would expect normally (without the use of LEDs).  Therefore, this application is more useful for demonstration, evaluation and prototyping purposes.  The battery level (in terms of voltage) can be monitored in the TLM frame data.  A popular coin cell battery, the CR2302, provides 3V at the start of its life and approaches 2V fairly rapidly as it nears the end of its discharge cycle.
- The Firmware Modules Eddystone application is based on the Nordic NRF5 SDK v13.0.0 Eddystone example.  The TLM frame is populated with data from the mcModule120 on-board TMP102 temperature sensor, which is much more accurate and responsive than the nRF52832 internal die temperature sensor and thus more useful for environmental monitoring applications.
- The accelerometer is not used, but we envision applications where the device is put dormant and the accelerometer is configured to wake up the device or trigger activity when motion is detected.  Motion information could be encoded in the URL frame as well.
- Security features (EID, eTLM) are not enabled in this version of the application.

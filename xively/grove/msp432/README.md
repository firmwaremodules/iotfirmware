# Xively with Seeed Studio Grove on MSP432

  ![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/xively_grove_cover.png "Xively IoT Cloud Platform with Seeed Studio Grove Starter Kit on MSP432 LaunchPad with CC3100 BoosterPack")

## About this project
Demonstrate an IoT application utilizing all of the sensors and actuators in the Seeed Studio Grove Starter Kit for LaunchPad and connecting them to the Xively IoT Device and Data Management Platform.  This demonstration application works on the MSP432 MCU with the CC3100 WiFi BoosterPack.

## Project info
Difficulty: Easy  
Estimated Time: 45 minutes

## Things used in this project

### Hardware

* [TI MSP-EXP432P401R LaunchPad](http://www.ti.com/tool/MSP-EXP432P401R) x1
* [TI CC3100BOOST BoosterPack](http://www.ti.com/tool/CC3100BOOST) x1
* [Seeed Studio Grove Starter Kit for LaunchPad] (https://www.seeedstudio.com/Grove-Starter-Kit-for-LaunchPad-p-2178.html) x1
* Jumper wire x1 such as [these](http://www.digikey.com/product-search/en?keywords=1528-1159-ND).

### Software

* [Xively Grove MSP432 MAN Firmware Binary](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/XIVELY-GROVE-MSP432-CC3100_1_0_5_MAN_0x85FC24B5.bin)

### Software Tools

* [Xively IoT Cloud Platform Free Trial Account] (https://www.xively.com/xively-iot-platform)
* [Firmware Modules Firmware Loader](https://github.com/firmwaremodules/iotfirmware/raw/master/tools/fm_load.exe)
* Terminal emulator such as [Tera Term](https://ttssh2.osdn.jp/index.html.en) or [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/)

## Story
### Introduction
In this demo we'll be connecting a collection of sensors to the MSP432 LaunchPad and to the Xively IoT Platform.  You'll be shown how to setup a model of each sensor in the IoT platform and see how you can interact with each sensor in real time.  The firmware that interfaces to each sensor, and securely interacts with the Xively IoT platform, is provided in binary form.  All you are required to do is to provision your Xively account ID, device ID and device secret keys using the "manufacturing test mode" feature built into the firmware image.

There are two parts to this story: the sensor platform, and the cloud connectivity hub.  The sensor platform is provided by Seeed Studio with their Grove Starter Kit BoosterPack, and the cloud platform chosen for this demo is provided by Xively.  The cloud platform allows real-time measurement and control of each sensor connected to the MSP432 MCU through the Internet.

The Seeed Studio Grove Starter Kit contains 10 separate sensor and actuator modules and offers a total of 11 sensor measurements or actuator controls, as tabulated below.  All of these sensors can be simultaneously connected and sampled with the TI MCU LaunchPad ecosystem.

The details of how each sensor is connected is discussed below.

The Xively IoT platform offers a powerful collection of device management features, monitoring and application integration.  The platform is available to anyone to get started with for no cost. 

| Grove Sensor | Grove Port | MSP432 Interface            | 
|-------------|-------|-----------------------------------|
| Light       | J6    | ADC channel A13                   |
| Sound       | J7    | ADC channel A11                   |
| Rotary      | J8    | ADC channel A9                    |
| Moisture    | J9    | ADC channel A8                    |
| Display     | J10   | TM1637 on I2C B1                  |
| Temperature | J13   | DHT package (AM2302) on GPIO P2.7 |
| Humidity    | J13   | DHT package (AM2302) on GPIO P2.7 |
| Buzzer      | J14   | GPIO P2.6                         |
| Motion      | J15   | GPIO P2.4                         |
| Relay       | J16   | GPIO P5.6                         |
| Ranger      | J17   | TimerA configured for pulse width detection and 65 ms (full count) timeout. |

The analog channels are configured for 12-bit resolution (max reading is 4095) at 50 ksps for low-power mode.  Each analog sensor reading averages 4 samples before determining if a change needs to be reported.

The DHT sensor package (AM2302) is driven by a vendor-specific 1-wire protocol.  Among its requirements are that it should not be sampled less than 2 seconds apart.  It offers 16-bit fields for temperature and humidity to accuracy of .1 degrees C or .1% RH.  In this demo we have rounded the samples to nearest degree C or %RH.

The Ultrasonic Ranger provides its range data in the form of an active high pulse width period, offering 58 us / cm.  For maximum accuracy and precision, a TimerA timer resource is configured for capture mode and latches the time (count) of both edges of the pulse.  If both edges are not detected by the time the timer counts to its maximum count (65535), the timer overflow interrupt is used to signal a timeout.  The timer is driven at 1 MHz so as to easily handle the sensor vendor's stated accuracy of +/-1 cm (+/-58 us) and its maximum range of 400 cm (~23 ms pulse width). As a side effect, the vendor's recommended minimum sample interval (60 ms) is naturally met on timeout.  The ultrasonic ranger itself is sampled 3 times on 6 tries to get a more stable result.


### Start your free Xively trial

The first thing you need to do is to navigate to the Xively website https://www.xively.com/ and click their "Free Trial" button (top right of the page as of the time of writing).  Enter your info and select "Start Free Trial".

You'll be brought into your account dashboard and "Product Launcher".  On the left side of the page there are a series of options with headings "Devices, End-Users, Groups", etc.  We'll be creating a new device template for the Grove-outfitted LaunchPad to manage all of the sensors and actuators.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/xively_trial_signup_page.png "Xively IoT Platform Trial Account Signup Page")

** For further reference about how to setup a Xively account, please see this resource: https://xively-docs.readme.io/docs/ti-cc3200

### Create the device template

Select "Device templates" item in the Devices section of the left-hand side navigation pane.  If you have any existing templates, you'll see them listed here.  

We are going to create a new device template and add 16 new channels: 1 for each Grove Starter Kit sensor data item or actuator and 5 for LaunchPad LEDs and buttons.

Select "Create new device template" found at the top-right corner of the page.  Call it whatever you want; we called ours "CC3100 Grove"

Each channel will be of the "simple" type, which simply means that a data array is passed to and from the device over an MQTT topic.  We are limited to two types of visualizers at this time: toggle, or slider. However, we can easily send strings (encoded or otherwise) to each MQTT topic and sensor, and will be using this feature to send melodies to the buzzer and digits to the 7-segment display.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/xively_device_templates.png "Xively IoT Platform Device Templates Page")

### Setting up your first sensor device: the 7 segment display

In your device template, select the "+" icon in the channels pane.  Enter the channel name, "Display", and ensure channel type is "Simple" and Kinesis Bridge is "Do not send".  Select "Save Channel".

![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/xively_template_add_display.png "Xively IoT Platform Device Template Adding Channel")

### Setting up the remaining sensors

For all channels listed below, use the exact names (including spaces) as presented.

Setup an additional 10 channels for each Grove sensor or actuator.
* Buzzer
* Relay
* Temperature 
* Humidity
* Moisture
* Sound
* Light
* Rotary
* Ranger
* Motion

These additional channels are available on the MSP432 LaunchPad itself:
* Green LED
* Blue LED
* Red LED
* Button S1
* Button S2

After all is said and done, you should have 17 channels in your template (including the built-in "_log" channel which we don't use in this demo).

### Create a device instance from your template

Select the "Create new device" button in the top-right corner as shown.  Fill in any number as its serial number.  Select "Create".  Now comes an important step in provisioning your device.  You'll need to get your device credentials: a unique device ID (DEVICE_ID in the provisioning app) and a secret key (DEVICE_SECRET in the provisioning app) associated with only that device.  To get these two data items, select the "Get password" button.  If you choose to download the password, you'll get a key=value pair in a text file.  The key is the DEVICE_SECRET string, and the value is the DEVICE_ID.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/xively_create_new_device_from_template.png "Xively IoT Platform Creating New Device From Template")

You may also pull the DEVICE_ID field from the device page, along with the ACCOUNT_ID which you'll also need to properly provision your device.  These three data items should be recorded in a text file for easy cut and paste access when you provision your device with the provisioning firmware app (which we call the MTA).  See the image below for the device's credential (provisioning) data fields (blurred out to protect our account).

![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/xively_device_credentials.png "Xively IoT Platform Obtaining Device Credentials")

### Setting up the channel visualizers

A device represented in the Xively IoT platfrom can be simulated.  The simulated device can have a visualizer associated with each channel. Each channel can have one of two types of visualizers: 
* Slider, which allows you publish a range of values by moving the slider.
* Toggle, which simply shows the boolean state of a channel, either on or off.  Selecting the toggle visualizer will send either "0" (decimal 48) or "1" (decimal 49) to the topic.  If the device is subscribed to this channel, the data will be immediately recevied by the device.

For channels that offer a range of values, i.e. Rotary, Moisture, Light, Sound, Temperature, Humidity, Ranger, Display, use a slider to visualize the data.

For channels that offer one of two states, i.e.
Motion, Relay, Button S1, Button S2, Green LED, Blue LED, Red LED, use a toggle.

The buzzer channel works with strings and cannot be visualized.

Look at the section "Interacting with channels" later in this write-up to learn how to work with the channels once your device is provisioned and connected with the Xively IoT Platform.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/xively_grove_device_simulation.png "Xively IoT Platform Device Simulation and Channel Visualizers")




### Load the firmware onto the target and provision the Xively account and device IDs.


1. **Assemble the hardware.** 
Attach the Grove Starter Kit modules to the Grove Base as shown in the image and described in the introductory table.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/grove_sensors.png "Xively Grove MSP432 demo sensor attachment diagram")

Assemble the MSP432 LaunchPad and CC3100 BoosterPack as shown in the image below and connect to your PC with a USB cable.  The CC3100 BoosterPack can be mounted on top of the LaunchPad while the Grove Starter Kit can be mounted below to afford better access to the LaunchPad reset button.  Connect a jumper from 3.3V (J1-1) to P5.5 (J3-30) to select the provisioning application (the MTA) for initial boot.

  ![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/grove_launchpad_wifi_assembly.png "Xively Grove MSP432 demo BoosterPack stack assembly")
  
2. **Get the firmware files.** Download the firmware loader tool `fm_load.exe` and the `XIVELY-GROVE-MSP432-CC3100_1_0_5_MAN_0x85FC24B5.bin` firmware binary programming file to a folder on your PC.

3. **Test the connection.** Open a console and run `fm_load.exe` to ensure your LaunchPad is connected.  You should see the following:
  ```
  Firmware Module System Firmware Loader v1.2.65
  Copyright (c) 2016 Firmware Modules Inc.
  License: https://github.com/firmwaremodules/iotfirmware/blob/master/tools/LICENSE
  Firmware loader for XDS110 with TI MSP432, CC13xx, CC26xx devices.

  INFO:root:DAP JTAG MODE initialised
  INFO:root:Detecting target... msp432p401ripz
  INFO:root:6 hardware breakpoints, 4 literal comparators
  INFO:root:CPU core is Cortex-M4
  INFO:root:FPU present
  INFO:root:4 hardware watchpoints
  No operation performed  
  ```
4. **Start the terminal.** Start a terminal emulator session/connection on the LaunchPad's "Application/User UART" at 115200 baud.  On some PCs in Tera Term this appears as "COM4: XDS110 Class Application/User UART (COM4)".  Choose the COM port that is assigned by your PC.

5. **Load the firmware.** Load the MAN firmware binary with `fm_load XIVELY-GROVE-MSP432-CC3100_1_0_5_MAN_0x85FC24B5.bin`.  Observe loading progress to completion (100%):  
  ```
  INFO:root:DAP JTAG MODE initialised
  INFO:root:Detecting target... msp432p401ripz
  INFO:root:6 hardware breakpoints, 4 literal comparators
  INFO:root:CPU core is Cortex-M4
  INFO:root:FPU present
  INFO:root:4 hardware watchpoints
  INFO:root:Selected board with target: msp432p401r
  Writing 110240 bytes...
  [====================] 100%
  ```  
  Observe the serial terminal window's output showing the need to set the WiFi and Device parameters which we'll do next.
  
7. **Provision the device's WiFi settings and Xively device IDs.**  To do this we must boot into the device's MTA (manufacturing test application) by pulling P5.5 high and resetting the board.  Connect a short jumper wire as shown between the BoosterPack connector pins J3-30 and J1-1 and press the reset button.  

  
  The serial terminal will show a simple menu requesting that you fill in 6 fields, your WiFi network's SSID, security type, and passkey, and your Xively ACCOUND_ID, DEVICE_ID and DEVICE_SECRET.  Enter each in turn as shown. In Tera Term, you can copy from Notepad with Ctrl-C then paste the unique ID strings with Edit->Paste.  The best way to do this is to first assemble your Xively ID strings in a text editor such as Notepad.  
  
  ```
************************************************************************
 Firmware Modules Provisioning MTA for MSP-EXP432P401R + CC3100BOOST.
************************************************************************

 WiFi Settings:
   
   SSID: not set
   Security Type: OPEN
   Passkey: not set

 Device Settings:

   DEVICE_ID: not set
   DEVICE_SECRET: not set
   ACCOUNT_ID: not set

 WiFi Menu:

   1) Set SSID
   2) Set Security Type
   3) Set Passkey

 Device Menu:

   7) Set DEVICE_ID
   8) Set DEVICE_SECRET
   9) Set ACCOUNT_ID


  >
  ```


  When complete, commit the settings to flash with menu option `4) Save WiFi settings` which appears only when a change has been made.  Remove the jumper wire from the 3.3V pin.  
  
  Leave the terminal emulator running so that you can see the status of your device including the messages it publishes and receives in real time.

  After rebooting the board by pressing the LaunchPad's reset button, you should see output similar to the following.  Note that the GUIDs shown below have been obfuscated and do not represent any valid GUID in the Xively system.

```
************************************************************************
 Demo APP for Grove Starter Kit and Xively IoT Platform.
 By Firmware Modules for MSP-EXP432P401R + CC3100BOOST.
************************************************************************
Provisioning data retrieved.
Host Driver Version: 1.0.0.1
Build Version 2.2.0.1.31.1.2.0.2.1.0.3.23
Device is configured in default state
Device started as STATION
[WLAN EVENT] STA Connected to the AP: ********* , BSSID: *:*:*:*:*:*
[NETAPP EVENT] IP Acquired: IP=*********,Gateway=*********
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Temperature
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Green LED
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Blue LED
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Red LED
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Button S1
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Button S2
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Humidity
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Buzzer
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Relay
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Motion
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Display
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Ranger
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Light
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Sound
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Moisture
Initialized topic: xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Rotary
--- current time: 2017/03/06, 22:41:42
connected to broker.xively.com:8883
[1488840104] publishing [0] to [xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Buzzer]
[1488840105] publishing [0] to [xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Relay]
[1488840105] publishing [0] to [xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Display]
topic:xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Green LED. Subscription granted 0.
topic:xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Blue LED. Subscription granted 0.
topic:xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Red LED. Subscription granted 0.
topic:xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Buzzer. Subscription granted 0.
topic:xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Relay. Subscription granted 0.
topic:xi/blue/v1/add4f022-2f0d-4906-ac64-8729238abc23/d/dc16c3c6-3188-4f5d-936f-d9712875c29a/Display. Subscription granted 0.
```

### Using the sensor channels on the Xively IoT Platform.

In your Xively device's main page select the "Messaging" tab.  Here, you are connecting directly to the MQTT broker and effectively subscribing to each channel.  Data published by your device to each channel will scroll by, and you can also publish arbitrary data to each channel.  For the Display and Buzzer channels, you can publish data strings that are meaningful to these devices.  

For the Display channel, you can publish a 4-digit integer and have it appear on the display.  For example, publish "4567" to have that number displayed on the 4-segment display.

For the Buzzer channel, you can publish a string containing a sequence of notes.  Spaces are interpreted as 1 beat pauses.  Each beat is 250 ms.  For example, send the string "abcdefg" to attempt to play the associated tones on the buzzer.  Please note that the buzzer is not a very good audio reproduction device!



![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/xively_publish_display.png "Xively IoT Platform Publish Message to Display Channel")



### What's Next

* We'd like to follow this up with an OTA firmware update demo using the Xively IoT Platform and our firmware update technology.  This feature is not yet available from Xively but is described in their developer documentation.  Stay tuned!


### Design Notes
* The Xively client 'libXively' found on GitHub is used to connect with the IoT platform.  (See https://github.com/xively/xively-client-c).





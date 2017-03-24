# Losant with Seeed Studio Grove on MSP432

  ![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_grove_cover.png "Losant IoT Cloud Platform with Seeed Studio Grove Starter Kit on MSP432 LaunchPad with CC3100 BoosterPack")

## About this project
Demonstrate an IoT application utilizing all of the sensors and actuators in the Seeed Studio Grove Starter Kit for LaunchPad and connecting them to the Losant IoT developer platform.  This demonstration application works on the MSP432 MCU with the CC3100 WiFi BoosterPack.

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

* [Losant Grove MSP432 MAN Firmware Binary](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/LOSANT-GROVE-MSP432-CC3100_1_0_9_MAN_0xE6129B06.bin)

### Software Tools

* [Losant IoT Developer Platform Sandbox Account](https://accounts.losant.com/create-account)
* [Firmware Modules Firmware Loader](https://github.com/firmwaremodules/iotfirmware/raw/master/tools/fm_load.exe)
* Terminal emulator such as [Tera Term](https://ttssh2.osdn.jp/index.html.en) or [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/)

## Story
### Introduction
In this demo we'll be connecting a collection of sensors to the MSP432 LaunchPad and to the Losant IoT platform.  You'll be shown how to setup a model of each sensor in the IoT platform and see how you can interact with each sensor in real time.  The firmware that interfaces to each sensor, and securely interacts with the Losant IoT platform, is provided in binary form.  All you are required to do is to provision your Losant device ID, access key and access secret using the instructions below and the "manufacturing test and provisioning mode" feature built into the firmware image.

There are two parts to this story: the sensor platform, and the cloud connectivity hub.  The sensor platform is provided by Seeed Studio with their Grove Starter Kit BoosterPack, and the cloud platform chosen for this demo is provided by Losant.  The cloud platform allows real-time measurement and/or control of each sensor connected to the MSP432 MCU through the Internet.

The Seeed Studio Grove Starter Kit contains 10 separate sensor and actuator modules and offers a total of 11 sensor measurements or actuator controls, as tabulated below.  All of these sensors can be simultaneously connected and sampled with the TI MCU LaunchPad ecosystem.

The details of how each sensor is connected is discussed below.


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

**All sensors are sampled at 5 second intervals. Only changes in the readings are reported to the Losant IoT platform.**

### Sign up for your free Losant Sandbox account

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_website_frontpage.png "Losant IoT Developer Platform Front Page with Signup Button")

The first thing you need to do is to navigate to the Losant website https://www.losant.com/ and click their "Sign Up" button (top right of the page as of the time of writing).  Enter your info and select "Create Account".

You'll be brought into your account sandbox and prompted to create a new "application".  The application may contain Device Recipes (templates), device instances (simply called Devices), and workflows to process device data and interact with the device.  In this demo we'll be creating a new device and adding 16 attributes to represent each sensor and actuator/indicator connected to the LaunchPad. Then we'll be using the IoT platform's "Security" tab to generate the provisioning credentials: the access key and access secret.  Then we'll be creating a simple Dashboard containing gauges and charts to visually represent sensor data, and using the device's "Debug" feature to send command strings to control the Display, Buzzer, Relay and RGB LEDs.

Check out this resource describing how to setup a Losant account and use the platform: https://docs.losant.com/getting-started/walkthrough/

### Create the device

Once you've created your application (name it whatever you like, we called ours CC3100 Grove), create a new device by selecting "Devices" then "Add Device".  

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_create_device.png "Losant IoT Platform Device Creation Menu Item")

Select "Create Blank Device" in the "Create From Scratch" section.  Give the device a name and description.  We used "MSP432 LaunchPad with Grove Starter Kit" for both.  Ensure you select "Standalone" as the device type because the MSP432 LaunchPad is outfitted with a CC3100 WiFi network processor enabling direct Internet connection (no gateway required).  Ensure you record the "Device ID" located at the top right corner of your new device page.  This Device ID is part of a trio of data items along with the Access Key and Access Secret (described later) that forms the device credentials required for the provisioning step.
  
![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_new_device.png "Losant IoT Platform New Device Page")

Further down the page, add the attributes as listed and shown below.

| Type | Name |
| ---- | ---- |
| Number | Temperature |
| Number | Humidity |
| String | Buzzer |
| Boolean | Relay |
| Number | Moisture |
| Number | Sound |
| Number | Light |
| Number | Rotary |
| Number | Ranger |
| Boolean | Motion |
| Number | Display |
| Boolean | Green_LED |
| Boolean | Red_LED |
| Boolean | Blue_LED |
| Boolean | Button_S1 |
| Boolean | Button_S2 |

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_new_device_attributes.png "Losant IoT Platform New Device Attributes")

The attributes can be one of these types: Number, String, Boolean, GPS String. 

Don't forget to select "Save Device" at the bottom of the page!

We'll defer setting up the Dashboard until after the firmware has been programmed and the device is up and running and sending real data.


### Grab the provisioning credentials

Select the "Secuity" tab along the menu bar and create a new "Access Key" as shown in the following image.  It is best to "Download to File" the generated access key and access secret strings which will be used, along with the Device ID, in the provisioning step after the firmware is loaded.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_create_accesskey.png "Losant IoT Platform Device Security Access Key Page")

  




### Load the firmware onto the target and provision the Losant device ID, Access Key and Access Secret


1. **Assemble the hardware.** 
Attach the Grove Starter Kit modules to the Grove Base as shown in the image and described in the introductory table.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/grove_sensors.png "Losant Grove MSP432 demo sensor attachment diagram")

Assemble the MSP432 LaunchPad and CC3100 BoosterPack as shown in the image below and connect to your PC with a USB cable.  The CC3100 BoosterPack can be mounted on top of the LaunchPad while the Grove Starter Kit can be mounted below to afford better access to the LaunchPad reset button.  Connect a jumper from 3.3V (J1-1) to P5.5 (J3-30) to select the provisioning application (the MTA) for initial boot.  Note: you can run this application at any time to configure your device's connection credentials.

  ![](https://github.com/firmwaremodules/iotfirmware/raw/master/xively/grove/msp432/resources/grove_launchpad_wifi_assembly.png "Losant Grove MSP432 demo BoosterPack stack assembly")
  
2. **Get the firmware files.** Download the firmware loader tool `fm_load.exe` and the `LOSANT-GROVE-MSP432-CC3100_1_0_9_MAN_0xE6129B06.bin` firmware binary programming file to a folder on your PC.

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

5. **Load the firmware.** Load the MAN firmware binary with `fm_load LOSANT-GROVE-MSP432-CC3100_1_0_9_MAN_0xE6129B06.bin`.  Observe loading progress to completion (100%):  
  ```
  INFO:root:DAP JTAG MODE initialised
  INFO:root:Detecting target... msp432p401ripz
  INFO:root:6 hardware breakpoints, 4 literal comparators
  INFO:root:CPU core is Cortex-M4
  INFO:root:FPU present
  INFO:root:4 hardware watchpoints
  INFO:root:Selected board with target: msp432p401r
  Writing 89996 bytes...
  [====================] 100%
  ```  
  Observe the serial terminal window's output showing the need to set the WiFi and Device parameters which we'll do next.
  
7. **Provision the device's WiFi settings and Losant device credentials.**  To do this we must boot into the device's MTA (manufacturing test application) by pulling P5.5 high and resetting the board.  Connect a short jumper wire as shown between the BoosterPack connector pins J3-30 and J1-1 and press the reset button.  

  
  The serial terminal will show a simple menu requesting that you fill in 6 fields, your WiFi network's SSID, security type, and passkey, and your Losant DEVICE_ID, ACCESS_KEY and ACCESS_SECRET.  Enter each in turn as shown. In Tera Term, you can copy from Notepad with Ctrl-C then paste the unique ID strings with Edit->Paste.  The best way to do this is to ensure your device credentials are displayed in text editor such as Notepad.  
  
  ```
************************************************************************
 Firmware Modules Provisioning MTA for MSP-EXP432P401R + CC3100BOOST.
************************************************************************

 WiFi Settings:
   
   SSID: not set
   Security Type: OPEN
   Passkey: not set

 Device Settings:

   ACCESS_KEY: not set
   ACCESS_SECRET: not set
   DEVICE_ID: not set

 WiFi Menu:

   1) Set SSID
   2) Set Security Type
   3) Set Passkey

 Device Menu:

   7) Set ACCESS_KEY
   8) Set ACCESS_SECRET
   9) Set DEVICE_ID


  >
  ```


  When complete, commit the settings to flash with menu option `4) Save WiFi settings` which appears only when a change has been made.  **Remove the jumper wire from the 3.3V pin.**  You should see something like the following (the access keys and IDs have been altered and are unusable):

```
  (unsaved changes) > 4
 Settings written

************************************************************************

 WiFi Settings:

   SSID: TESTSSID
   Security Type: WPA_WPA2
   Passkey: TestPasskey

 Device Settings:

   ACCESS_KEY: f0b556cb-fa3b-9821-c5d8-5f4378cf9317
   ACCESS_SECRET: 887f8ce64f11d7e617b2a164f11d7e61c136c9a5a568b2a1f56598a25582afb2
   DEVICE_ID: 67d0491bc77add3321a21b79

 WiFi Menu:

   1) Set SSID
   2) Set Security Type
   3) Set Passkey

 Device Menu:

   7) Set ACCESS_KEY
   8) Set ACCESS_SECRET
   9) Set DEVICE_ID


  >
```
  
  Leave the terminal emulator running so that you can see the status of your device including the messages it publishes and receives in real time.

  After rebooting the board by pressing the LaunchPad's reset button, you should see output similar to the following. (Ensure you have removed the jumper wire).

```
************************************************************************
 Demo APP for Grove Starter Kit and Losant IoT Platform.
 By Firmware Modules for MSP-EXP432P401R + CC3100BOOST.
************************************************************************
Provisioning data retrieved.
Starting IoT platform.
Starting Wifi subsystem.
Host Driver Version: 1.0.0.1
Build Version 2.2.0.1.31.1.2.0.2.1.0.3.23
Device is configured in default state
Device started as STATION
[WLAN EVENT] STA Connected to the AP: ********* , BSSID: 4c:8b:30:3:26:1c
[NETAPP EVENT] IP Acquired: IP=**.**.**.** ,Gateway=192.168.1.254
Initialized topic: losant/67d0491bc77add3321a21b79/state
Initialized topic: losant/67d0491bc77add3321a21b79/command
SNTP: looking up current time... 3699282574
--- set current time: 2017/03/23, 18:29:34
connected to broker.losant.com:8883
Subscribing to losant/67d0491bc77add3321a21b79/command for event Green_LED
Subscribing to losant/67d0491bc77add3321a21b79/command for event Blue_LED
Subscribing to losant/67d0491bc77add3321a21b79/command for event Red_LED
Subscribing to losant/67d0491bc77add3321a21b79/command for event Buzzer
Subscribing to losant/67d0491bc77add3321a21b79/command for event Relay
Subscribing to losant/67d0491bc77add3321a21b79/command for event Display
[1490293774] publishing [0] to [Relay]... [JSON: {"data":{"Relay":"0"}}] success.
[1490293774] publishing [0] to [Display]... [JSON: {"data":{"Display":"0"}}] success.
[1490293774] publishing [1] to [Motion]... [JSON: {"data":{"Motion":"1"}}] success.
[1490293774] publishing [2480] to [Light]... [JSON: {"data":{"Light":"2480"}}] success.
[1490293774] publishing [600] to [Sound]... [JSON: {"data":{"Sound":"600"}}] success.
[1490293774] publishing [0] to [Moisture]... [JSON: {"data":{"Moisture":"0"}}] success.
[1490293774] publishing [2573] to [Rotary]... [JSON: {"data":{"Rotary":"2573"}}] success.
[1490293774] publishing [23] to [Temperature]... [JSON: {"data":{"Temperature":"23"}}] success.
[1490293774] publishing [31] to [Humidity]... [JSON: {"data":{"Humidity":"31"}}] success.
```


### Setting up the Dashboard

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_demo_dashboard.png "Losant IoT Platform MSP432 LaunchPad Grove Demo Dashboard")

With the MSP432 LaunchPad up and running and sending data, the Dashboard visualizers are easier to setup as they show data in real-time so you can see what your data would look like as you play with the settings.

Select "Dashboards" tab at the top of the page and then "Create Dashboard".  Give it a name and description (we used CC3100 Grove Demo Dashboard), and set "Fetch Data Every..." to 5 seconds for minimum latency.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_demo_dashboard_create.png "Losant IoT Platform Dashboard Creation Page")


We'll next create a Timeseries chart to visualize both temperature and humidity from the DHT22 sensor, and then a Gauge to visualize the rotary sensor data.

In your new dashboard, select the gear icon at the top-right of the page and then "Add Block".  Select "Customize" button in the "Time Series Graph" widget, as shown in the following image.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_dashboard_blocks.png "Losant IoT Platform Dashboard Visualizer Types")

Give the block header a name (we used "DHT Sensor"), choose your (only) application, select a duration (we used 60 minute range with 10 second intervals).  In the "Block Data" section, add two segments, one for Temperature and one for Humidity.  Select the appropriate device attribute for each (e.g. "Temperature" for Temperature) and fill out the rest like shown in the image below.  It's all pretty self-explanatory.


![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_dht_sensor_timeseries_setup.png "Losant IoT Platform Timeseries Setup for DHT22 Sensor")

Now do the same for the Rotary sensor, this time using a Gauge widget, as shown in the image below.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_rotary_sensor_guage.png "Losant IoT Platform Gauge Setup for Rotary Sensor")

After setting these two elements up, it should be easy to setup additional visualizers for other data items.



### Sending commands to the device from the Losant IoT platform.

You may have noticed in the terminal output that the MSP432 device subscribed to the command topic for the following device attributes: Display, Buzzer, Green_LED, Red_LED and Blue_LED.  The device is waiting to receive data on the command topic targeted to one of these device attributes.  The command JSON payload contains a field called "name" which must have a corresponding JSON string matching the targeted attribute, e.g. "Display".  Then, the command payload is explicity defined to contain the targeted attribute's value.  For the Display, it would be a number string, e.g. {"value":"1234"}.

To send commands, go back to your device (via Devices menu item).  On the left side of the page, select the "Debug" tab.  If you scroll down the page you'll find the "Send Device Command" section.  Set the Command Name field to one of Display, Buzzer, Green_LED, Blue_LED or Red_LED to target one of these 5 device attributes waiting for data.  Under Command Payload, use {"value":"<4digits>"} for Display, {"value":"1"} to turn on an LED or {"value":"0"} to turn it off, or {"value":"abcdefg"} to sound a melody on the Buzzer.  The following screenshots show examples of sending commands to each attribute.

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_send_command_display.png "Losant IoT Platform Sending Device Command for MSP432 LaunchPad Grove Demo Display Attribute")

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_send_command_buzzer.png "Losant IoT Platform Sending Device Command for MSP432 LaunchPad Grove Demo Buzzer Attribute")

![](https://github.com/firmwaremodules/iotfirmware/raw/master/losant/grove/msp432/resources/losant_send_command_LED.png "Losant IoT Platform Sending Device Command for MSP432 LaunchPad Grove Demo LED Attribute")


The Display accepts any 4-digit number (as a string).

The Buzzer accepts a string containing a sequence of notes.  Spaces are interpreted as 1 beat pauses.  Each beat is 500 ms.  For example, send the string "abcdefg" to attempt to play the associated tones on the buzzer.  Please note that the buzzer is not a very good audio reproduction device!



### What's Next

* Setup workflows to trigger alerts when the moisture sensor reports a dry plant or the motion sensor activates! With Losant's free-for-development "sandbox" account, you have a fully functional IoT platform at your fingertips for up to 10 devices and 1 million messages per month.

### IoT Platform Notes
* Devices interact with the Losant platform with just two MQTT topics: **command** and **state**.  
* Devices subscribe to the command topic and publish to the state topic.  
* Commands are application-specific JSON payloads that must be decoded by the device firmware. 
* All of the device's state could be published to the Losant platform in one JSON-encoded message, although we don't do that in this demo to accommodate other IoT platforms with one demo firmware implementation.
* The Losant platform's device ID, access key and access secret device provisioning items are placed into the MQTT connection requestion fields client ID, username and password respectively.

### Design Notes
* The Eclipse Paho MQTT client is used to connect with the IoT platform.
* All communications occur over a TLS secured socket (broker.losant.com:8883).
* The CC3100 built-in security stack (WolfSSL) handles the TLS security suite.
* SNTP is used to acquire the device's time.
* The client device is authenticated with the user-provisioned access key and access secret generated by the Losant platform. TLS is not used for client authentication with this platform.
* The demo application services the communication stacks at a rate of 10 Hz, thus there is up to 100 ms latency in servicing a command.

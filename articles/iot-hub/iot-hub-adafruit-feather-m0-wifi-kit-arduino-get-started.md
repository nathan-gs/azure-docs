---
title: M0 to cloud - Connect Feather M0 WiFi to Azure IoT Hub | Microsoft Docs
description: Explains how to connect an Arduino device, called Adafruit Feather M0 WiFi, to Azure IoT Hub, which is a Microsoft cloud service that helps manage your IoT assets.
services: iot-hub
documentationcenter: ''
author: shizn
manager: timtl
tags: ''
keywords: ''

ms.assetid: 51befcdb-332b-416f-a6a1-8aabdb67f283
ms.service: iot-hub
ms.devlang: arduino
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 6/17/2017
ms.author: xshi

---

# Connect Adafruit Feather M0 WiFi to Azure IoT Hub in the cloud
[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

![Connection between BME280, Feather M0 WiFi, and IoT Hub](media/iot-hub-adafruit-feather-m0-wifi-get-started/1_connection-m0-feather-m0-iot-hub.png)

In this tutorial, you begin by learning the basics of working with your Arduino board. You then learn how to seamlessly connect your devices to the cloud by using [Azure IoT Hub](iot-hub-what-is-iot-hub.md).

## What you do

Connect Adafruit Feather M0 WiFi to an IoT hub that you create. Then you run a sample application on M0 WiFi to collect the temperature and humidity data from a BME280. Finally, you send the sensor data to your IoT hub.



## What you learn

* How to create an IoT hub and register a device for Feather M0 WiFi
* How to connect Feather M0 WiFi with the sensor and your computer
* How to collect sensor data by running a sample application on Feather M0 WiFi
* How to send the sensor data to your IoT hub

## What you need

![Parts needed for the tutorial](media/iot-hub-adafruit-feather-m0-wifi-get-started/2_parts-needed-for-the-tutorial.png)

To complete this operation, you need the following parts from your Feather M0 WiFi Starter Kit:

* The Feather M0 WiFi board
* A Micro USB to Type A USB cable

You also need the following things for your development environment:

* Mac or PC that is running Windows or Ubuntu.
* Wireless network for Feather M0 WiFi to connect to.
* Internet connection to download the configuration tool.
* [Arduino IDE](https://www.arduino.cc/en/main/software) version 1.6.8 or later. Earlier versions don't work with the AzureIoT library.


The following items are optional in case you don’t have a sensor. You also have the option of using simulated sensor data.

* An BME280 temperature and humidity sensor
* A breadboard
* M/M jumper wires

[!INCLUDE [iot-hub-get-started-create-hub-and-device](../../includes/iot-hub-get-started-create-hub-and-device.md)]

## Connect Feather M0 WiFi with the sensor and your computer
In this section, you connect the sensors to your board. Then you plug in your device to your computer for further use.
### Connect a DHT22 temperature and humidity sensor to Feather M0 WiFi

Use the breadboard and jumper wires to make the connection as follows. If you don’t have a sensor, skip this section because you can use simulated sensor data instead.

![Connections reference](media/iot-hub-adafruit-feather-m0-wifi-get-started/3_connections_on_breadboard.png)


For sensor pins, use the following wiring:


| Start (Sensor)           | End (Board)            | Cable Color   |
| -----------------------  | ---------------------- | ------------: |
| VDD (Pin 27A)            | 3V (Pin 3A)            | Red cable     |
| GND (Pin 29A)            | GND (Pin 6A)           | Black cable   |
| SCK (Pin 30A)            | SCK (Pin 12A)          | Yellow cable  |
| SDO (Pin 31A)            | MI (Pin 14A)           | White cable   |
| SDI (Pin 32A)            | M0 (Pin 13A)           | Blue cable    |
| CS (Pin 33A)             | GPIO 5 (Pin 15J)       | Orange cable  |

For more information, see [Adafruit BME280 Humidity + Barometric Pressure + Temperature Sensor Breakout](https://learn.adafruit.com/adafruit-bme280-humidity-barometric-pressure-temperature-sensor-breakout/wiring-and-test?view=all) and [Adafruit Feather M0 WiFi Pinouts](https://learn.adafruit.com/adafruit-feather-m0-wifi-atwinc1500/pinouts).



Now your Feather M0 WiFi should be connected with a working sensor.

![Connect DHT22 with Feather Huzzah](media/iot-hub-adafruit-feather-m0-wifi-get-started/4_connect-bme280-feather-m0-wifi.png)

### Connect Feather M0 WiFi to your computer

As shown next, use the Micro USB to Type A USB cable to connect Feather M0 WiFi to your computer.

![Connect Feather Huzzah to your computer](media/iot-hub-adafruit-feather-m0-wifi-get-started/5_connect-feather-m0-wifi-computer.png)

### Add serial port permissions (Ubuntu only)

If you use Ubuntu, make sure you have the permissions to operate on the USB port of Feather M0 WiFi. To add serial port permissions, follow these steps:


1. Run the following commands at a terminal:

   ```bash
   ls -l /dev/ttyUSB*
   ls -l /dev/ttyACM*
   ```

   You get one of the following outputs:

   * crw-rw---- 1 root uucp xxxxxxxx
   * crw-rw---- 1 root dialout xxxxxxxx

   In the output, notice that `uucp` or `dialout` is the group owner name of the USB port.

1. Add the user to the group by running the following command:

   ```bash
   sudo usermod -a -G <group-owner-name> <username>
   ```

   `<group-owner-name>` is the group owner name you obtained in the previous step. `<username>` is your Ubuntu user name.

1. Sign out of Ubuntu, and then sign in again for the change to appear.

## Collect sensor data and send it to your IoT hub

In this section, you deploy and run a sample application on Feather M0 WiFi. The sample application blinks the LED on Feather M0 WiFi, and sends the temperature and humidity data collected from the BME280 sensor to your IoT hub.

### Get the sample application from GitHub and prepare Arduino IDE

The sample application is hosted on GitHub. Clone the sample repository that contains the sample application from GitHub. To clone the sample repository, follow these steps:

1. Open a command prompt or a terminal window.
1. Go to a folder where you want the sample application to be stored.
1. Run the following command:

   ```bash
   git clone https://github.com/Azure-Samples/iot-hub-Feather-M0-WiFi-client-app.git
   ```

Install the package for Feather M0 WiFi in the Arduino IDE:

1. Open the folder where the sample application is stored.
1. Open the app.ino file in the app folder in the Arduino IDE.

   ![Open the sample application in Arduino IDE](media/iot-hub-adafruit-feather-m0-wifi-get-started/6_arduino-ide-open-sample-app.png)

1. Click **Tools** > **Board** > **Boards Manager**, and then install the `Arduino SAMD Boards` version `1.6.2` or later. Then install `Adafruit SAMD` package to add the board file definitions.

   Boards Manager indicates that `Arduino SAMD Boards` with a version of `1.6.2` or later is installed. 

   ![The esp8266 package is installed](media/iot-hub-adafruit-feather-m0-wifi-get-started/7_arduino-ide-package-url.png)

1. Click **Tools** > **Board** > **Adafruit M0 WiFi**.

1. Install Drivers (Windows Only)
   When you plug in the Feather, you'll need to possibly install a driver
   Click [here](https://github.com/adafruit/Adafruit_Windows_Drivers/releases/download/1.1/adafruit_drivers.exe) to download the Driver Installer.
   Follow the steps to install the drivers you want to install.

### Install necessary libraries

1. In the Arduino IDE, click **Sketch** > **Include Library** > **Manage Libraries**.
1. Search for the following library names one by one. For each  library that you find, click **Install**.
   * `RTCZero`
   * `NTPClient`
   * `AzureIoTHub`
   * `AzureIoTUtility`
   * `AzureIoTProtocol_HTTP`
   * `ArduinoJson`
   * `Adafruit BME280 Library`
   * `Adafruit Unified Sensor`
1. Manually install `Adafruit_WINC1500`. Visit [this link](https://github.com/adafruit/Adafruit_WINC1500) and click the **Clone or download** button, then **Download ZIP**. Then in your Arduino IDE, go to **Sketch** -> **Include Library** -> **Add .zip Library** and add the zip file you just downloaded.

### Don’t have a real BME280 sensor?

The sample application can simulate temperature and humidity data in case you don’t have a real BME280 sensor. To set up the sample application to use simulated data, follow these steps:

1. Open the `config.h` file in the `app` folder.
1. Locate the following line of code and change the value from `false` to `true`:
   ```c
   define SIMULATED_DATA true
   ```
   ![Configure the sample application to use simulated data](media/iot-hub-adafruit-feather-m0-wifi-get-started/8_arduino-ide-configure-app-use-simulated-data.png)

1. Save the file with `Control-s`.

### Deploy the sample application to Feather M0 WiFi

1. In the Arduino IDE, click **Tool** > **Port**, and then click the serial port for Feather M0 WiFi.
1. Click **Sketch** > **Upload** to build and deploy the sample application to Feather M0 WiFi.

### Enter your credentials

After the upload completes successfully, follow these steps to enter your credentials:

1. In the Arduino IDE, click **Tools** > **Serial Monitor**.
1. In the serial monitor window, notice the two drop-down lists in the lower-right corner.
1. Select **No line ending** for the left drop-down list.
1. Select **115200 baud** for the right drop-down list.
1. In the input box located at the top of the serial monitor window, enter the following information if you are asked to provide them, and then click **Send**.
   * Wi-Fi SSID
   * Wi-Fi password
   * Device connection string

> [!Note]
> The credential information is stored in the EEPROM of Feather M0 WiFi. If you click the reset button on the Feather M0 WiFi board, the sample application asks if you want to erase the information. Enter `Y` to have the information erased. You are asked to provide the information a second time.

### Verify the sample application is running successfully

If you see the following output from the serial monitor window and the blinking LED on Feather M0 WiFi, the sample application is running successfully.

![Final output in Arduino IDE](media/iot-hub-adafruit-feather-m0-wifi-get-started/9_arduino-ide-final-output.png)

## Next steps

You have successfully connected a Feather M0 WiFi to your IoT hub, and sent the captured sensor data to your IoT hub. 

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]


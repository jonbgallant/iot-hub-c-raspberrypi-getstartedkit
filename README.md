---
services: iot-hub, stream-analytics, event-hubs, web-apps, documentdb, storage-accounts
platforms: c ,raspberry-pi
author: hegate
---

# Get Started with Microsoft Azure IoT Starter Kit - Raspberry Pi2 and Pi3

This page contains technical information to help you get familiar with Azure IoT using the Azure IoT Starter Kit - Raspberry Pi2 and Pi3. You will find two tutorials that will walk you through different scenarios: the first tutorial will show you how to connect your Azure IoT Starter kit to our Remote Monitoring preconfigured solution from Azure IoT Suite. In the second tutorial, you will leverage Azure IoT services to create your own IoT architecture.

You can choose to start with whichever tutorial you want to. If you've never worked with Azure IoT services before, we encourage you to start with the Remote Monitoring solution tutorial, because all of the Azure services will be provisioned for you in a built-in preconfigured solution. Then you can explore how each of the services work by going through the second tutorial.

We hope you enjoy the process. Please provide feedback if there's anything that we can improve.

***
**Don't have a kit yet?:** Click [here](http://azure.com/iotstarterkits)
***

- [Connecting your Raspberry Pi2 or Pi3 to the Azure IoT Suite Remote Monitoring Solution](#connecting-your-raspberry-pi)
- [Using Microsoft Azure IoT Services to Identify Temperature Anomalies](#identify-temperature-anomalies)

<a name="connecting-your-raspberry-pi" />
# Connecting your Raspberry Pi2 or Pi3 to the Azure IoT Suite Remote Monitoring Solution

This tutorial describes the process of taking your Microsoft Azure IoT Starter Kit for Raspberry Pi2 and Pi3, and using it to develop a temperature and humidity reader that can communicate with the cloud using the Raspbian OS and Microsoft Azure IoT SDK. For Windows 10 IoT Core samples, please visit [windowsondevices.com](http://www.windowsondevices.com/).

## Table of Contents

- [1.1 Tutorial Overview](#section1.1)
- [1.2 Before Starting](#section1.2)
  - [1.2.1 Required Software](#section1.2.1)
  - [1.2.2 Required Hardware](#section1.2.2)
- [1.3 Create a New Azure IoT Suite Remote Monitoring solution and Add Device](#section1.3)
- [1.4 Prepare the Device](#section1.4)
  - [1.4.1 Log in and Access the Terminal](#section1.4.1)
  - [1.4.2 Log in Using PuTTY](#section1.4.2)
    - [1.4.2.1 Log in using Linux or Mac OS](#section1.4.2.1)
- [1.5 Configure the Remote Monitoring device sample](#section1.5)
  - [1.5.1 Clone repositories](#section1.5.1)
  - [1.5.2 Update the Connection String](#section1.5.2)
- [1.6 Build the Modified Sample](#section1.6)
- [1.7 View the Sensor Data from the IoT Suite Portal](#section1.7)
- [1.8 Next steps](#section1.8)

<a name="section1.1" />
## 1.1 Tutorial Overview

In this tutorial, you'll be doing the following:

- Setting up your environment on Azure using the Microsoft Azure IoT Suite Remote Monitoring preconfigured solution, getting a large portion of the set-up that would be required done in one step.
- Setting your device and sensors up so that it can communicate with both your computer, and Azure IoT.
- Updating the device code sample to include our connection data and send it to Azure IoT to be viewed remotely.

<a name="section1.2" />
## 1.2 Before Starting

<a name="section1.2.1" />
### 1.2.1 Required Software

- An SSH client – This makes it so you can remotely access the Raspberry Pi’s command line remotely from your computer
  - Windows doesn’t have a built-in SSH client. We recommend using [PuTTY](http://www.putty.org/)
  - Many Linux distributions and Mac OS has SSH built into their terminal. If yours does not, we recommend OpenSSH
  - See also: [SSH Using Linux or Mac OS](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md)
  - Do this!
  
<a name="section1.2.2" />
### 1.2.2 Required Hardware

- Microsoft Azure IoT Starter Kit
  - 8GB MicroSD Card (Comes with the kit, flashed with Windows 10 IoT Core)
  - A USB Mini cable
  - An Ethernet cable or Wi-Fi dongle

<a name="section1.3" />
## 1.3 Create a New Azure IoT Suite Remote Monitoring solution and Add Device

- Log in to [Azure IoT Suite](https://www.azureiotsuite.com/)  with your Microsoft account and click **Create a New Preconfigured Solution**

***
**Note:** Creating an account is free. Click here to get your [Azure free trial](https://azure.microsoft.com/en-us/pricing/free-trial/)
***

- Click select in the **Remote Monitoring** option
- Type in a solution name. For this tutorial we’ll be using “PiSuite”

***
**Note:** Make sure to copy down the names and connection strings mentioned into a text document for reference later.
***

- Choose your subscription plan and geographic region, then click **Create Solution**
- Wait for Azure to finish provisioning your IoT suite (this process may take up to 10 minutes), and then click **Launch**

***
**Note:** You may be asked to log back in. This is to ensure your solution has proper permissions associated with your account.
***

- Open the link to your IoT Suite’s “Solution Dashboard.” You may have been redirected there already.
- This opens your personal remote monitoring site at the URL &lt;Your IoT Hub suite name>.azurewebsites.net (e.g. PiSuite.azurewebsites.net)
- Click **Add a Device** at the lower left hand corner of your screen
- Add a new **custom device**
- Enter your desired device ID. In this case we’ll use “RaspPi”, and then click **Create**
- Make note of your device ID, Device Key, and IoT Hub Hostname to enter into the code you’ll run on your device later

***
**Warning:** The Remote Monitoring solution provisions a set of Azure IoT Services in your Azure account. It is meant to reflect a real enterprise architecture and thus its Azure consumption is quite heavy. To avoid unnecessary Azure consumption, we recommend you delete the preconfigured solution in azureiotsuite.com once you are done with your work (since it is easy to recreate). Alternatively, if you want to keep it up and running you can do several things to reduce consumption:

1) Visit this [guide](https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Docs/configure-preconfigured-demo.md) to run the solution in demo mode and reduce the Azure consumption.  

2) Disable the simulated devices created with the solution (Go to Devices>>Select the device>> on the device details menu on the right, clich on Disable Device. Repeat with all the simulated devices).

3) **Stop** your remote monitoring solution while you are working on the next steps. (See: [Troubleshooting](#troubleshooting))
***

<a name="section1.4" />
## 1.4 Prepare the Device

If this is the first time you are using Raspberry Pi, now is the time to set it up. If you’ll be using Windows, please use [windowsondevices.com](http://www.windowsondevices.com/) for detailed
guidelines on how to get started with the Raspberry Pi. If you’re using Linux, Raspberry Pi and Adafruit have a set of tutorials and videos to help you get started.

Please visit the following links:

- [Raspberry Pi NOOBS setup page](https://www.raspberrypi.org/help/noobs-setup/)
- Using [this image](https://github.com/Azure-Samples/iot-hub-c-raspberrypi-getstartedkit/blob/master/img/rpi2_remote_monitoring.png?raw=true) as a reference, connect your BME280 to the breadboard and to the Raspberry Pi

***
**Note:** Column on the left corresponds to the sensor and on the right to the board. On the image, the BME280 is connected between pins 15A(VIN) and 21A(CS).
***

| Pi Pin                  | End                    | Cable Color   |
| ----------------------- | ---------------------- | ------------: |
| SPI_CE0 (Pin 24)        | CS (Pin 21A)           | Blue cable    |
| SPI_SCLK (Pin 23)       | SCK (18A)              | Yellow cable  |
| SPI_MISO (Pin 21)       | SDO (19A )             | White cable   |
| SPI_MOSI (Pin 19)       | SDI (Pin 20A)          | Green cable   |
| GND (Pin 6)             | GND (Pin 17A)          | Black cable   |
| 3.3V (Pin 1)            | 3Vo (Pin 16A)          | Red cable     |

**At the end of your work, your Raspberry Pi should be connected with a working sensor.**

<a name="section1.4.1" />
### 1.4.1 Log in and Access the Terminal

The default login for Raspbian is username `pi` with password `raspberry`. If you use the Raspbian interface directly, in the task bar up top, you can launch the Terminal using the 3rd icon from the left – The one that looks like a monitor.

<a name="section1.4.2" />
### 1.4.2 Log in Using PuTTY

- You need to discover the IP address of your Raspberry Pi before you can connect using PuTTY. Type `hostname -I` in a command prompt to discover your IP. For more information see: [https://www.raspberrypi.org/documentation/remote-access/ip-address.md](https://www.raspberrypi.org/documentation/remote-access/ip-address.md)
- With the Raspberry Pi on and running, open an SSH terminal program such as [PuTTY](http://www.putty.org/) on your desktop machine.
- Use the IP address from previous step as the Host Name, Port=22, and Connection type=SSH to complete the connection.
- When prompted, log in with username `pi`, and password `raspberry`.

<a name="section1.4.2.1" />
#### 1.4.2.1 Log in using Linux or Mac OS

- See: [SSH Using Linux of Mac OS](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md)

<a name="section1.5" />
## 1.5 Configure the Remote Monitoring device sample

Now we can download the Remote Monitoring device sample, input your device credentials (i.e. your "Connection String") and start sending data from the Pi to our Azure Remote Monitoring solution.

<a name="section1.5.1" />
### 1.5.1 Clone repositories

Clone the following repositories and combine them by entering the following commands on your Pi:

```
cd ~
git clone --recursive https://github.com/Azure-Samples/iot-hub-c-raspberrypi-getstartedkit.git
git clone --recursive https://github.com/WiringPi/WiringPi.git
```

Now, we need to update device connection string

<a name="section1.5.2" />
### 1.5.2 Update the Connection String

- Edit the file by entering the following command:

```
nano ~/iot-hub-c-raspberrypi-getstartedkit/samples/remote_monitoring/remote_monitoring.c
```

- Use the arrow keys to navigate to the following lines:

```
static const char* deviceId = "[Device Id]";
static const char* deviceKey = "[Device Key]";
static const char* hubName = "[IoTHub Name]";
static const char* hubSuffix = "[IoTHub Suffix, i.e. azure-devices.net]";
```
- Replace the placeholder with your device and IoT Hub information you created and saved at the beginning of this tutorial. In our example:
  - Device Id --> `RaspPi`
  - Device Key --> &lt;Azure device key ended with `==`>
  - IoTHub Name --> `PiSuite`
  - IoTHub Suffix, i.e. azure-devices.net --> `azure-devices.net`
- Save and exit with `Control-o, Enter, Control-x`

<a name="section1.6" />
## 1.6 Build the Modified Sample

Now that you’re exited out of Nano, install the prerequisite packages for the Microsoft Azure IoT Device SDK for C by issuing the following commands using the terminal (either on the device or through an SSH client like PuTTY):

```
sudo apt-get update
sudo apt-get install curl libcurl4-openssl-dev uuid-dev uuid g++ make cmake git unzip openjdk-7-jre libssl-dev libncurses-dev subversion gawk
```

- Now go ahead and build the updated sample solution by entering the following:

```
cd ~/iot-hub-c-raspberrypi-getstartedkit/
sudo ~/iot-hub-c-raspberrypi-getstartedkit/azure-iot-sdks/c/build_all/linux/setup.sh
chmod +x ~/iot-hub-c-raspberrypi-getstartedkit/samples/build.sh
~/iot-hub-c-raspberrypi-getstartedkit/samples/build.sh
```

- Before running the program we need to enable the SPI drive to install by default at boot.

```
sudo nano /boot/config.txt
```

- Scroll down and find the line:

```
#dtparam=spi=on
```

- Delete the `#` at the beginning of the line to uncomment it.
- Save and exit with `Control-o`, `Enter`, `Control-x`.
- Reboot the Raspberry Pi to enable the spi, it will disconnect the terminal, you will need to login again. Run the command:

```
sudo reboot
```

- Now that everything is compiled, it’s time to run the program. Enter the command:

```
sudo ~/cmake/samples/remote_monitoring/remote_monitoring
```

If all goes well, you will begin to see data streaming!
Press `ctrl-c` to exit at any time. Below is a sample of the expected command prompt output:

```
Humidity = 48.4% Temperature = 23.9*C
Sending sensor value Temperature = 23.9*C, Humidity = 48.4%
IoTHubClient accepted the message for delivery
```

<a name="section1.7" />
## 1.7 View the Sensor Data from the IoT Suite Portal

- Once you have the sample running, visit your dashboard by visiting azureiotsuite.com and clicking “Launch” on your solution
- Make sure the “Device to View” in the upper right is set to your device
- If the demo is running, you should see the graph change as your data updates in real time!

***
**Note:** Make sure you delete or **stop** your remote monitoring solution once you have completed this to avoid unnecessary Azure consumption!  (See: [Troubleshooting](#troubleshooting))
***

<a name="section1.8" />
## 1.8 Next steps

Please visit our [Azure IoT Dev Center](https://azure.microsoft.com/en-us/develop/iot/) for more samples and documentation on Azure IoT.

<a name="identify-temperature-anomalies" />
# Using Microsoft Azure IoT Services to Identify Temperature Anomalies

This tutorial describes the process of taking your Microsoft Azure IoT Starter Kit for Raspberry Pi 2 and Pi 3, and using it to develop a temperature and humidity reader that can communicate with Microsoft Azure IoT Services, process the data, detects abnormal data, and sends that back to the Pi for use. We will be using
the Raspbian OS and Microsoft Azure IoT SDK. For Windows 10 IoT Core samples,
please visit [windowsondevices.com](http://www.windowsondevices.com/).

## Table of Contents

- [2.1 Tutorial Overview](#section2.1)
- [2.2 Requirements](#section2.2)
  - [2.2.1 Required Software](#section2.2.1)
  - [2.2.2 Required Hardware](#section2.2.2)
- [2.3 Create a New Microsoft Azure IoT Hub and Add Device](#section2.3)
- [2.4 Prepare the Device](#section2.4)
  - [2.4.1 Log in and Access the Terminal](#section2.4.1)
  - [2.4.2 Log in Using PuTTY](#section2.4.2)
- [2.5 Create an Event Hub](#section2.5)
- [2.6 Create a Storage Account for Table Storage](#section2.6)
- [2.7 Create a Stream Analytics job to Save Sensor Data in Table Storage and Raise Alerts](#section2.7)
- [2.8 Node Application Setup](#section2.8)
- [2.9 Configure the Command Center device sample](#section2.9)
  - [2.9.1 Update the Connection Data](#section2.9.1)
- [2.10 Build the Modified Sample](#section2.10)
- [2.11 Next steps](#section2.11)

<a name="section2.1" />
## 2.1 Tutorial Overview

This tutorial has the following steps:
- Provision an IoT Hub instance on Microsoft Azure and adding your device.
- Prepare the device, get connected to the device, and set it up so that it can read sensor data.
- Configure your Microsoft Azure IoT services by adding Event Hub, Storage Account, and Stream Analytics resources.
- Prepare your local web solution for monitoring and sending commands to your device.
- Update the sample code to respond to commands and include the data from our sensors, sending it to Microsoft Azure to be viewed remotely.

Here is a breakdown of the data flow:
- The application running on the Raspberry Pi will get temperature data from the temperature sensor and it will send them to the IoT Hub
- A Stream Analytics job will read the data from IoT Hub and write them to an Azure Storage Table. Also, if an anomaly is detected, then this job will write data to an Event Hub
- The Node.js application that is running on your computers will read the data from the Azure Storage Table and the Event Hub and will present them to the user

The end result will be a functional command center where you can view the history of your device's sensor data, a history of alerts, and send commands back to the device.

<a name="section2.2" />
## 2.2 Requirements

<a name="section2.2.1" />
### 2.2.1 Required Software

- [Git](https://git-scm.com/downloads) - For cloning the required repositories
- Node.js - For the Node application, we will go over this later.
- An SSH client – This makes it so you can remotely access the Raspberry Pi’s command line remotely from your computer
  - Windows doesn’t have a built-in SSH client. We recommend using [PuTTY](http://www.putty.org/)
  - Many Linux distributions and Mac OS has SSH built into their terminal. If yours does not, we recommend OpenSSH
  - See also: [SSH Using Linux on Mac OS](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md)

<a name="section2.2.2" />
### 2.2.2 Required Hardware

- Microsoft Azure IoT Starter Kit
  - 8GB MicroSD Card (comes with the kit, flashed with Windows 10 IoT Core)
  - A USB Mini cable
  - An Ethernet cable or Wi-Fi dongle

<a name="section2.3" />
## 2.3 Create a New Microsoft Azure IoT Hub and Add Device

- To create your Microsoft Azure IoT Hub and add a device, follow the instructions outlined in the [Setup IoT Hub Microsoft Azure Iot SDK page](https://github.com/Azure/azure-iot-sdks/blob/master/doc/setup_iothub.md).
- We named our IoT Hub `raspPiIoT`

***
**Note:** Make sure to copy down the names and connection strings mentioned into a text document for reference later.
***

- After creating your device, make note of your connection string to enter into the code you’ll run on your device later

<a name="section2.4" />
## 2.4 Prepare the Device

If this is the first time you are using Raspberry Pi, now it’s the time to set it up. If you’ll be using Windows, please use [windowsondevices.com](http://www.windowsondevices.com/) for detail
guidelines on how to get started with the Raspberry Pi. If you’re using Linux, Raspberry Pi and Adafruit have a set of tutorials and videos to help you get started.

Please visit the following links:

- [Raspberry Pi NOOBS setup page](https://www.raspberrypi.org/help/noobs-setup/)
- Using [this image](https://github.com/Azure-Samples/iot-hub-c-raspberrypi-getstartedkit/blob/master/img/rpi2_command_center.png?raw=true) as a reference, connect your LEDs and your BME280 to the breadboard and the Raspberry Pi.

***
**Note:** Column on the left corresponds to the sensor and on the right to the board. On the image, the sensor is place between 15E(VIN) and 21E(CS). The - symbol refers to the blue row on the board, with the pins counting from the left starting at 15. See the diagram above for more reference.

**Note:** The resistor can change a little from one kit to another, e.g. it can be 330 Ohm (orange, orange, brown) or 560 Ohm (green, blue, brown). Both will work with success.
***

| Raspberry Pi Pins       | Breadboard ends        | Cable Color   |
| ----------------------- | ---------------------- | ------------: |
| SPI_CE0 (Pin 24)        | CS (Pin 21A)           | Blue cable    |
| SPI_SCLK (Pin 23)       | SCK (18A)              | Yellow cable  |
| SPI_MISO (Pin 21)       | SDO (19A )             | White cable   |
| SPI_MOSI (Pin 19)       | SDI (Pin 20A)          | Green cable   |
| GND (Pin 6)             | Pin 3-                 | Black cable   |
| 3.3V (Pin 1)            | 3Vo (Pin 16A)          | Red cable     |
| GPIO 24 (Pin 18)        | Pin 10D                | Green cable   |
| GPIO 23 (Pin 16)        | Pin 7D                 | Red cable     |
| Pin 13-                 | GND (Pin 17A)          | Black cable   |

| Start                   | End                    | Connector     |
| ----------------------- | ---------------------- | ------------: |
| Pin 4-                  | Pin 6B                 | Resistor      |
| Pin 6-                  | Pin 9B                 | Resistor      |
| Pin 6E                  | Pin 7E                 | Red LED (long leg) |
| Pin 9E                  | Pin 10E                | Green LED (long leg) |

**At the end of your work, your Raspberry Pi should be connected with a working sensor.**

<a name="section2.4.1" />
### 2.4.1 Log in and Access the Terminal

The default login for Raspbian is username `pi` with password `raspberry`. If you use the Raspbian interface directly, in the task bar up top, you can launch the Terminal using the 3rd icon from the left – The one that looks like a monitor.

<a name="section2.4.2" />
### 2.4.2 Log in Using PuTTY

- You need to discover the IP address of your Raspberry Pi before you can connect using PuTTY. Type `hostname -I` in a command prompt to discover your IP.For more information see: [https://www.raspberrypi.org/documentation/remote-access/ip-address.md](https://www.raspberrypi.org/documentation/remote-access/ip-address.md)
- With the Raspberry Pi on and running, open an SSH terminal program such as [PuTTY](http://www.putty.org/) on your desktop machine.
- Use the IP address from previous step as the Host Name, Port=22, and Connection type=SSH to complete the connection.
- When prompted, log in with username `pi`, and password `raspberry`.

<a name="section2.5" />
## 2.5 Create an Event Hub
Event Hub is an Azure IoT publish-subscribe service that can ingest millions of events per second and stream them into multiple applications, services or devices.

- Log on to the [Microsoft Azure Portal](https://portal.azure.com/)
- Click on **New** -&gt; **Internet of Things**-&gt; **Event Hub**
- Enter the following settings for the Event Hub Namespace (use a name of your choice for the event hub and the namespace):
    - Name: `Your choice` (we chose `PiSuite`)
    - Pricing Tier: `Basic`
    - Subscription: `Your choice`
    - Resource Group: `Your choice`
    - Location: `Your choice`
- Click on **Create**
- Wait until the Event Hub Namespace is created, and then create an Event Hub using the following steps:
    - Click on your `PiSuite` Event Hub Namespace (or pick any other name that you used)
    - Click the **Add Event Hub** 
    - Name: `piEventHub`
- Click on **Create**
- Wait until the new Event Bus is created
- Click on the **Event Hubs** arrow in the **Overview** tab (might require a few clicks, until the UI is updated)
- Select the `piEventHub` eventhub and go in the **Configure** tab in the **Shared Access Policies** section, add a new policy:
    - Name = `readwrite`
    - Permissions = `Send, Listen`
- Click **Save** at the bottom of the page, then click the **Dashboard** tab near the top and click on **Connection Information** at the bottom
- _Copy down the connection string for the `readwrite` policy you created._
- From the your IoT Hub Settings (The Resource that has connected dots) on the [Microsoft Azure Portal](https://portal.azure.com/), click the **Messaging blade** (found in your settings), write down the _Event Hub-compatible name_
- Look at the _Event-hub-compatible Endpoint_, and write down this part: sb://**thispart**.servicebus.windows.net/ we will call this one the _IoTHub EventHub-compatible namespace_

<a name="section2.6" />
## 2.6 Create a Storage Account for Table Storage
Now we will create a service to store our data in the cloud.
- Log on to the [Microsoft Azure Portal](https://portal.azure.com/)
- In the menu, click **New** and select **Data + Storage** then **Storage Account**
    - Name: `Your choice` (we chose `pistorage`)
    - Deployment model: `Classic`
    - Performance: `Standard`
    - Replication: `Read-access geo-redundant storage (RA-GRS)`
    - Subscription: `Your choice`
    - Resource Group: `Your choice`
    - Location: `Your choice`
- Once the account is created, find it in the **resources blade** or click on the **pinned tile**, go to **Settings**, **Keys**, and write down the _primary connection string_.

<a name="section2.7" />
## 2.7 Create a Stream Analytics job to Save Sensor Data in Table Storage and Raise Alerts
Stream Analytics is an Azure IoT service that streams and analyzes data in the cloud. We'll use it to process data coming from your device.

- Log on to the [Microsoft Azure Portal](https://portal.azure.com/)
- In the menu, click **New**, then click **Internet of Things**, and then click **Stream Analytics Job**
- Enter a name for the job (We chose “PiStorageJob”), a preferred region, then choose your subscription. At this stage you are also offered to create a new or to use an existing resource group. Choose the resource group you created earlier.
- Once the job is created, open your **Job’s blade** or click on the **pinned tile**, and find the section titled _“Job Topology”_ and click the **Inputs** tile. In the Inputs blade, click on **Add**
- Enter the following settings:
    - Input Alias = _`TempSensors`_
    - Source Type = _`Data Stream`_
    - Source = _`IoT Hub`_
    - IoT Hub = _`PiSuite`_ (use the name for the IoT Hub you create before)
    - Shared Access Policy Name = _`iothubowner`_
    - Shared Access Policy Key = _The `iothubowner` primary key can be found in your IoT Hub Settings -> Shared access policies_
    - IoT Hub Consumer Group = "" (leave it to the default empty value)
    - Event serialization format = _`JSON`_
    - Encoding = _`UTF-8`_

- Back to the **Stream Analytics Job blade**, click on the **Query tile** (next to the Inputs tile). In the Query settings blade, type in the below query and click **Save**:

```
SELECT
    DeviceId,
    EventTime,
    MTemperature as TemperatureReading
INTO
    TemperatureTableStorage
from TempSensors
WHERE
   DeviceId is not null
   and EventTime is not null

SELECT
    DeviceId,
    EventTime,
    MTemperature as TemperatureReading
INTO   
    TemperatureAlertToEventHub
FROM
    TempSensors
WHERE MTemperature>25
```

***
**Note:** You can change the `25` to `0` when you're ready to generate alerts to look at. This number represents the temperature in degrees celsius to check for when creating alerts. 25 degrees celsius is 77 degrees fahrenheit.
***

- Back to the **Stream Analytics Job blade**, click on the **Outputs** tile and in the **Outputs blade**, click on **Add**
- Enter the following settings then click on **Create**:
    - Output Alias = _`TemperatureTableStorage`_
    - Sink = _`Table Storage`_
    - Subscription = _`Provide table settings storage manually`_
    - Storage account = _`pistorage`_ (The storage account you created earlier)
    - Storage account key = _(The primary key for the storage account made earlier, can be found in Settings -&gt; Keys -&gt; Primary Access Key)_
    - Table Name = _`TemperatureRecords`_ (Your choice - If the table doesn’t already exist, Local Storage will create it)
    - Partition Key = _`DeviceId`_
    - Row Key = _`EventTime`_
    - Batch size = _`1`_
- Back to the **Stream Analytics Job blade**, click on the **Outputs tile**, and in the **Outputs blade**, click on **Add**
- Enter the following settings then click on **Create**:
    - Output Alias = _`TemperatureAlertToEventHub`_
    - Sink = _`Event Hub`_
    - Subscription = _`Provide table settings storage manually`_
    - Service Bus Namespace = _`PiSuite`_
    - Event Hub Name = _`piEventHub`_ (The Event Hub you made earlier)
    - Event Hub Policy Name = _`readwrite`_
    - Event Hub Policy Key = _`Primary Key for readwrite Policy name`_ (That's the one you wrote down after creating the event hub)
    - Partition Key Column = _`0`_
    - Event Serialization format = _`JSON`_
    - Encoding = _`UTF-8`_
    - Format = _`Line separated`_
- Back in the** Stream Analytics blade**, start the job by clicking on the **Start **button at the top

***
**Note:** Make sure to **stop** your Command Center jobs once you have when you take a break or finish to avoid unnecessary Azure consumption!  (See: [Troubleshooting](#troubleshooting))
***

<a name="section2.8" />
## 2.8 Node Application Setup

 - If you do not have it already, install Node.js and NPM.
   - Windows and Mac installers can be found here: https://nodejs.org/en/download/
     - Ensure that you select the options to install NPM and add to your PATH.
   - Linux users can use the commands:

```
sudo apt-get update
sudo apt-get install nodejs
sudo apt-get install npm
```

- Additionally, make sure you have cloned the project repository locally by issuing the following command in your desired directory:

```
git clone --recursive https://github.com/Azure-Samples/iot-hub-c-raspberrypi-getstartedkit.git
```

- Open the `command_center_node` folder in your command prompt (`cd path/to/command_center_node`) and install the required modules by running the following:

```
sudo npm install -g bower
sudo npm install
bower install
```

- Open the `config.json` file and replace the information with your project.  See the following for instructions on how to retrieve those values.

    - eventhubName:
        - Open the [Classic Azure Management Portal](https://manage.windowsazure.com)
        - Open the Service Bus namespace you created earlier
        - Switch to the **EVENT HUBS** page
        - You can see and copy the name of your event hub from that page
    - ehConnString:
        - Click on the name of the event hub from above to open it
        - Click on the "CONNECTION INFORMATION" button along the bottom.
        - From there, click the button to copy the readwrite shared access policy connection string.
    - deviceConnString:
        - Use the information on the [Manage IoT Hub](https://github.com/Azure/azure-iot-sdks/blob/master/doc/manage_iot_hub.md) to retrieve your device connection string using either the Device Explorer or iothub-explorer tools.
    - iotHubConnString:
        - In the [Azure Portal](https://portal.azure.com)
        - Open the IoT Hub you created previously.
        - Open the "Settings" blade
        - Click on the "Shared access policies" setting
        - Click on the "service" policy
        - Copy the primary connection string for the policy
    - storageAccountName:
        - In the [Azure Portal](https://portal.azure.com)
        - Open the classic Storage Account you created previously to copy its name
    - storageAccountKey:
        - Click on the name of the storage account above to open it
        - Click the "Settings" button to open the Settings blade
        - Click on the "Keys" setting
        - Click the button next to the "PRIMARY ACCESS KEY" top copy it
    - storageTableName:
        - This must match the name of the table that was used in the Stream Analytics table storage output above.
        - If you used the instructions above, you would have named it ***`TemperatureRecords`***
        - If you named it something else, enter the name you used instead.    

```
{
    "port": "3000",
    "eventHubName": "event-hub-name",
    "ehConnString": "Endpoint=sb://name.servicebus.windows.net/;SharedAccessKeyName=readwrite;SharedAccessKey=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa=",
    "deviceId": "iot-hub-device-name",
    "iotHubConnString": "HostName=iot-hub-name.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa=",
    "storageAcountName": "aaaaaaaaaaa",
    "storageAccountKey": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa==",
    "storageTable": "TemperatureRecords"
}
```

- Now it is time to run it! Enter the following command:

```
node server.js
```

- You should then see something similar to:

```
app running on http://localhost:3000
client connected
```

- Visit the url in your browser and you will see the Node app running!

To deploy this project to the cloud using Azure, you can reference [Creating a Node.js web app in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-nodejs-develop-deploy-mac/).

Next, we will update your device so that it can interact with all the things you just created.

<a name="section2.9" />
## 2.9 Configure the Command Center device sample

Now we can download the sample solution repos and incorporate the data from the sensors and send it up to our Microsoft Azure Command Center solution.

Clone the repositories by entering the following commands on your Pi:

```
cd ~
git clone --recursive https://github.com/Azure-Samples/iot-hub-c-raspberrypi-getstartedkit.git
git clone --recursive https://github.com/WiringPi/WiringPi.git
```

Now, we need to update three things:

- Include the required libraries (included in the git repository)
- Include the Device ID
- Update the connection string to use the string we saved above

<a name="section2.9.1" />
### 2.9.1 Update the Connection Data

To update the connection data, we need to update the source file. You can edit the file by entering the following command:

```
nano ~/iot-hub-c-raspberrypi-getstartedkit/samples/simplesample_amqp/simplesample_amqp.c
```

- Use the arrow keys to navigate to the following lines:

```
static const char* connectionString = "[device connection string]";
static char* deviceId = "[Device Name]";
```

- Replace the placeholder with your device connection information you gathered at the beginning of this tutorial. It must looks like `HostName=<host_name>.azure-devices.net;DeviceId=<device_id>;SharedAccessKey=<device_key>`.
- Save and exit with `Control-o, Enter, Control-x`

<a name="section2.10" />
## 2.10 Build the Modified Sample

Now that you’re exited out of Nano, install the prerequisite packages for the Microsoft Azure IoT Device SDK for C by issuing the following commands using the terminal (either on the device or through an SSH client like PuTTY):

```
sudo apt-get update
sudo apt-get install curl libcurl4-openssl-dev uuid-dev uuid g++ make cmake git unzip openjdk-7-jre libssl-dev libncurses-dev subversion gawk
```

- Now go ahead and build the updated sample solution by entering the following:

```
cd ~/iot-hub-c-raspberrypi-getstartedkit/
sudo ~/iot-hub-c-raspberrypi-getstartedkit/azure-iot-sdks/c/build_all/linux/setup.sh
chmod +x ~/iot-hub-c-raspberrypi-getstartedkit/samples/build.sh
~/iot-hub-c-raspberrypi-getstartedkit/samples/build.sh
```

- Before running the program we need to enable the SPI drive to install by default at boot.

```
sudo nano /boot/config.txt
```

- Scroll down and find the line:

```
#dtparam=spi=on
```

- Delete the `#` at the beginning of the line to uncomment it.
- Save and exit with `Control-o`, `Enter`, `Control-x`.
- Reboot the Raspberry Pi to enable the spi, it will disconnect the terminal, you will need to login again. Run the command:

```
sudo reboot
```

- Now that everything is compiled, it’s time to run the program. Enter the command:

```
sudo ~/cmake/samples/simplesample_amqp/simplesample_amqp
```

You will now see data being sent off at regular intervals to
Microsoft Azure. An alert has also been set to go off when it detects the temperature is above 25 degrees celsius (77'F). You can cup your hand around the sensor and blow warm air to raise the temperature and when the alert goes off, you will see the LED you’ve set up turn red!

***
**Note:** If you're in a particularly hot or cold room, you may need to adjust the alert temperature specified in [Create a Stream Analytics job to Save Sensor Data in Table Storage and Raise Alerts](#create-a-stream-analytics-job-to-save-iot-data-in-table-storage-and-raise-alerts).
***

Head back to your Node application and you will have a fully functional command center, complete with a history of sensor data, alerts that display when the temperature got outside a certain range, and commands that you can send to your device remotely.

***
**Note:** Make sure to **stop** your Command Center jobs once you have when you finish to avoid unnecessary Azure consumption!  (See: [Troubleshooting](#troubleshooting))
***

<a name="section2.11" />
## 2.11 Next steps

Please visit our [Azure IoT Dev Center](https://azure.microsoft.com/en-us/develop/iot/) for more samples and documentation on Azure IoT.

<a name="troubleshooting" />
# Troubleshooting

## Stopping Provisioned Services

- In the [Microsoft Azure Portal](https://portal.azure.com/)
    - Click on "All Resources"
    - For each Stream Analytics and Web App resource:
        - Click on the resource and click the "Stop" button in the new blade that appears
    - For each IoT Hub resource:
        - Click on the resource and click the "Devices" button in the new blade that appears
        - Click on each device in the list and click the "Disable" button that appears in the new blade at the bottom

## Data is not showing up in the Node.js application

In this section we will explain how to see the data flowing from the Arduino application to the Node.js application:
- Arduino application: In the Arduino IDE go to Tools -> Serial Monitor
- IoT Hub: Use [Device Explorer](https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/doc/how_to_use_device_explorer.md)
- Azure Storage Table: Use [Azure Storage Explorer](http://storageexplorer.com/)

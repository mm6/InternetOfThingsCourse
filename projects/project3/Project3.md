
# 95-733 Internet of Things  

# Project 3 Due: Thursday, October 10, 2024

### Topics: BLE, Node RED, MQTT, and the Web

:checkered_flag:**Submit to Canvas a single .pdf file named Your_Last_Name_First_Name_Project3.pdf. This single pdf will contain your responses to the questions marked with a checkered flag. It is important that you <em>clearly label</em> each answer with the project part number and question number and that you provide your name and email address at the top of the .pdf.**

### Overview

In class, we have reviewed the architecture of several IoT platforms: Google's Cloud IoT, AWS's IoT, IBM's Watson IoT, CMU's Sensor Andrew, and CMU's OpenChirp. Each of these included a constrained networking area (the edge), a gateway layer, a publish/subscribe broker, a persistence layer, analytics capabilities and a web front end.

In this project, you will build your own system using a similar architecture. The constrained area will use Bluetooth Low Energy (BLE). The gateway tier will use Node-RED. You will use MQTT as your broker and a browser for a web dashboard.

If you are using an Apple MAC, be sure that you have installed Xcode from Apple.

### Objective

The objective is to gain hands-on experience with a small implementation that includes several important pieces organized in a particular manner - the manner of real world IoT systems. It should be noted that you are not building a secure system - using physical hardening, encryption, and authentication logic.

### Part 1. Programming the Photon 2 to behave as a BLE peripheral device and installing LightBlue to behave as a BLE central device

A BLE peripheral device offering a service periodically announces its availability to the local environment (a distance of about 10 meters). A central device scans for these advertisements and may chooses to connect to the peripheral. After a connection is established, the peripheral stops advertising its capabilities and begins to behave as a service to the central device - acting as a client.

In Part 1, you will deploy code to the Photon 2 so that it behaves as a peripheral. You will use a phone application as a central device. You will connect to the Photon 2 and receive notifications that will be displayed on the phone.

The Photon 2 will provide two services but advertise only one. The services will each make available values (called BLE characteristics). Both services and characteristics are associated with unique UUID's.


1. Study the following BLE peripheral code and flash it to your Photon 2.

This code is adapted from the [Argon/BLE tutorial](https://docs.particle.io/tutorials/device-os/bluetooth-le/).

```
#include "Particle.h"

SerialLogHandler logHandler(LOG_LEVEL_TRACE);

const unsigned long UPDATE_INTERVAL_MS = 2000;
unsigned long lastUpdate = 0;

// in C++, declare functions before use
float getTempC();
uint32_t ieee11073_from_float(float temperature);

// The "Health Thermometer" service is 0x1809.
// See https://www.bluetooth.com/specifications/gatt/services/
BleUuid healthThermometerService(BLE_SIG_UUID_HEALTH_THERMONETER_SVC);

// We're using a well-known characteristics UUID. They're defined here:
// https://www.bluetooth.com/specifications/gatt/characteristics/
// The temperature-measurement is 16-bit UUID 0x2A1C
BleCharacteristic temperatureMeasurementCharacteristic("temp", BleCharacteristicProperty::NOTIFY, BleUuid(0x2A1C), healthThermometerService);

// The battery level service allows the battery level to be monitored
BleUuid batteryLevelService(BLE_SIG_UUID_BATTERY_SVC);

// The battery_level characteristic shows the battery level of
BleCharacteristic batteryLevelCharacteristic("bat", BleCharacteristicProperty::NOTIFY, BleUuid(0x2A19), batteryLevelService);


// We don't actually have a thermometer here, we just randomly adjust this value
float lastValue = 37.0; // 98.6 deg F;

uint8_t lastBattery = 100;

void setup() {
	(void)logHandler; // Does nothing, just to eliminate the unused variable warning

	BLE.on();

	// Add two characteristics - temperature and battery
	BLE.addCharacteristic(temperatureMeasurementCharacteristic);
	BLE.addCharacteristic(batteryLevelCharacteristic);

	// Make available the lastBattery variable
	batteryLevelCharacteristic.setValue(&lastBattery, 1);

	// We need an object to provide advertisements
	BleAdvertisingData advData;

	// While we support both the health thermometer service and the battery service, we
	// only advertise the health thermometer. The battery service will be found after
	// connecting.
	advData.appendServiceUUID(healthThermometerService);

	// Continuously advertise when not connected
	BLE.advertise(&advData);
}

void loop() {
	if (millis() - lastUpdate >= UPDATE_INTERVAL_MS) {
		lastUpdate = millis();

		if (BLE.connected()) {
			// a 6 byte buffer to transmit
			uint8_t buf[6];

			// The Temperature Measurement characteristic data is defined here:
			/*
			https://github.com/sputnikdev/bluetooth-gatt-parser/blob/master/src/main/resources/gatt/characteristic/org.bluetooth.characteristic.temperature_measurement.xml
      */
			// First byte is flags. We're using Celsius (bit 0b001 == 0), no timestamp (bit 0b010 == 0), with temperature type (bit 0b100), so the flags are 0x04.
			buf[0] = 0x04;

			// Value is a ieee11073 floating point number
			uint32_t value = ieee11073_from_float(getTempC());
			memcpy(&buf[1], &value, 4);

			// TempType is a constant for where the sensor is sensing:
			// <Enumeration key="1" value="Armpit" />
			// <Enumeration key="2" value="Body (general)" />
			// <Enumeration key="3" value="Ear (usually ear lobe)" />
			// <Enumeration key="4" value="Finger" />
			// <Enumeration key="5" value="Gastro-intestinal Tract" />
			// <Enumeration key="6" value="Mouth" />
			// <Enumeration key="7" value="Rectum" />
			// <Enumeration key="8" value="Toe" />
			// <Enumeration key="9" value="Tympanum (ear drum)" />

			// Let's go with mouth
			buf[5] = 6; // Mouth

			// Make available the 6 byte buf of the encoded 48 bits
			temperatureMeasurementCharacteristic.setValue(buf, sizeof(buf));

			// The battery starts at 100% and drops to 10% then will jump back up again
			batteryLevelCharacteristic.setValue(&lastBattery, 1);
			if (--lastBattery < 10) {
				lastBattery = 100;
			}
		}
	}
}

float getTempC() {
	// Adjust this by a little bit each check so we can see it change
	if (rand() > (RAND_MAX / 2)) {
		lastValue += 0.1;
	}
	else {
		lastValue -= 0.1;
	}

	return lastValue;
}

uint32_t ieee11073_from_float(float temperature) {
	// This code is from the ARM mbed temperature demo:
	// https://github.com/ARMmbed/ble/blob/master/ble/services/HealthThermometerService.h

	uint8_t  exponent = 0xFE; // Exponent is -2
	uint32_t mantissa = (uint32_t)(temperature * 100);

	return (((uint32_t)exponent) << 24) | mantissa;
}

```
<em>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Photon 2 code to behave as a BLE peripheral </em>

2. Note, in particular, how the array named "buf" is defined and used in specific places in the code:

```
uint8_t buf[6];   // create the array of six 8-bit bytes

buf[0] = 0x04;    // a standard code is placed in buf[0]

// compute the current temperature and convert the float to a
// 32 bit integer
uint32_t value = ieee11073_from_float(getTempC());

// copy the 32 bit integer (four bytes) into buf[1],buf[2],buf[3], and buf[4]
memcpy(&buf[1], &value, 4);

buf[5] = 6; // buf[5] is used to hold a standard code for body location

```

The buf array is used to hold the data that is passed over the BLE signal. The memcpy() call places 4 bytes from the "value" variable into the array starting at position 1. These four bytes will change often. The buf array at positions 0 and 5 are set according to standard settings for body temperature sensors. They will remain constant and are used for metadata, e.g., the temperature was taken in celsius and was monitored from the mouth. The receiver will have to know how to interpret the arriving bits.

3. Test your Argon's BLE code by [installing the LightBlue PunchThrough app](https://punchthrough.com/lightblue/) on your phone and making a BLE connection to your Argon. LightBlue will act as a BLE central device and will scan for peripheral advertisements.

Note that this may require a bit of browsing with LightBlue to find the right BLE source. The dBm value (decibels relative to one milliwatt) is used to define the strength of the BLE signal. Anything greater than -75 dBm is considered a reliable signal. The closer that you get to 0, the more reliable the signal. On LightBlue's PunchThrough application, you can use signal strength to help locate which signal is coming from your Argon.

To help you connect with LightBlue, you might also want to look at the BLE MAC address being used by your Argon. A device may have several different MAC addresses. To view your BLE MAC in a terminal, you can use this line of C++ code in your firmware:
```
Serial.printlnf("BLE MAC address: %s ",BLE.address().toString().c_str());

```

Within the firmware, there is code that defines the temperature characteristic as a "notify" characteristic. This means that a client does not have to perform explicit read requests. The client can sit back and receive the temperature values.

```
BleCharacteristicProperty::NOTIFY

```

Using LightBlue, you will need to use the "subscribe" or "listen" selection to receive these values.

:checkered_flag:**Take two screenshots showing LightBlue connected to your Photn 2. The first screenshot (named LightBlue1.jpg) will contain the Advertisement Data screen showing the raw advertisement packet. The second screenshot (named LightBlue2.jpg) will show the Temperature Measurement Properties screen and the Read/Indicated values of the Health Thermometer's Temperature Measurement - it is important that you display the values arriving on your phone from the Photon 2.**

### Part 2. Transmit text over BLE

0. Experiment by making modifications to the firmware in Part 1 so that the LightBlue BLE screen displays "Hello" followed by your name. In other words, we will not be receiving temperature updates but, instead, we will be receiving a personalized greeting over BLE.

1. Your changes to the code in Part 1 should be minimal. This can be done easily by assigning letters to the buf array. For example, your solution would assign the value 'H' to buf[0] and 'e' to buf[1], etc.

2. My solution looks like the following:

<img src="https://github.com/mm6/InternetOfThingsCourse/blob/master/images/Hello_Michael_LightBlue.jpg" alt="BLE Greeting" width="400" height="800"/>

<em>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Screen scrape of LightBlue on an Android Phone</em>

:checkered_flag: Submit two files. The first will be a copy of the modified firmware (named HelloCode.txt) and the second will be a screen capture (named LightBlue3.jpg) of your phone running LightBlue and showing your name in the hello greeting.

### Part 3. Using Node RED to act as a gateway device

In Parts 1 and 2 we used LightBlue as the central device. In Part 3, we will use a Node-RED node to behave as a central device.

0. Restore the firmware to its original state. That is, remove the code that we used to generate a personalized greeting over BLE.
We are back to generating random temperature values.

1. The goal is to establish a connection between Node-RED and the Photon 2 using BLE. Install two BLE Nodes in Node-RED with the following shell commands:
```
$cd ~/.node-red
$npm install node-red-contrib-generic-ble
```

2. Test if the two nodes were properly installed by running Node-RED from a shell:

```
$node-red

```
and with a browser visit:

```
http://127.0.0.1:1880/

```

Within Node-RED, expand the "Network" icon on the left and verify the presence of two BLE nodes: "Generic BLE In" and "Generic BLE out".  

3. Using the "+" sign, create a new flow entitled "BLE Test".
4. Add an "Inject node" to the palette.
5. In the "Inject node", set the message payload to JSON and set the message payload text to the following JSON message. This message will tell the "BLE In" node to listen for notify messages coming from the BLE connection.   

```
{"notify": true, "period": 0 }

```

6. Drag the "BLE In" node onto the palette. Create a wire from the "Inject" node in step 5 to the new "BLE In" node.

7. Doble click the "BLE In" node and select the two check boxes: "Stringify in payload" and "Emit notify events".

8. Double click the "BLE In" node and select the pencil symbol to edit the properties. Select the "BLE Scanning" check box and select "Apply".

9. Run the BLE firmware on the Photon 2 and select the device in the Properties box of the "Edit Generic BLE node" pane. Be sure that the "Emit Notify Events" check box is selected. Note that the "Edit Generic BLE node" pane populates the UUID field. Also note that since there may be many devices around, it may be hard to figure out which one is the Photon 2. In the Generic BLE In node, you can enter the mac address of the device, separated by colons. This will filter on the MAC address.

10. Drag a debug node onto the palette and connect it to the output of the "Generic BLE In" node. In this way, the data that is received over BLE will appear in the debug pane on the right.

11. Deploy the flow. To start listening for notifications, click on the far left side of the "Inject node" icon. With the Photon 2 running, you should see the publications (notifications) from the Photon 2 in the debug window.

12. Add a new "Inject node" and set the msg.topic to the value "disconnect" - without the quotes.

13. Add a new "Inject node" and set the msg.topic to "connect" - without the quotes.

14. See the following image and wire the two new "Inject nodes" to the input of the "Generic BLE In" node. The "BLE In" node should now have three inject nodes wired to it. Experiment. Start and stop the connection using these new nodes. After the connection is started, inject a listening request from the middle inject node. Watch the output from the debug node.

<img src="https://github.com/mm6/InternetOfThingsCourse/blob/master/images/ArgonToNodeREDBLEFlow.png" alt="BLE Greeting" width="400" height="400"/>
<em>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BLE connecting, listening, and disconnecting with Node-RED</em>


### Part 4. A BLE Exercise

In Part 4 you will first write code that communicates with the LightBlue phone application (for testing) and then write a Node-RED flow that displays the arriving values on a debug window. In Part 5, this flow will be extended to include an MQTT broker.

Note that the BLE standards body provides some UUID's that are fixed. In the example code of Part 1, the "Health Thermometer service" is represented as the value "0x1809". If you were to build a new health thermometer, you would likely use "0x1809" as your UUID so that other devices, perhaps not manufactured by you, would recognize the service that your new device provides. In Part 4, you will choose your own UUID's and use them to represent your service and its characteristics. This is necessary because the BLE standard body does not provide a unique code for a simple light value service. Since this will be a proprietary service, you will use a full 16 bytes to represent the service - not two bytes such as "0x1809". The UUID for the service that you offer will differ from the UUID that is used to refer to a characteristic.

0. Recall the LightMonitor firmware from Part 2 of Project 2. In that code, we transmitted light values via HTTP to Node-RED. Here, we will do the same but with BLE. Note that we want to transmit far fewer bytes (over BLE) than we did when using HTTP messages over TCP sockets.

1. We need to define our own UUID's for our proprietary light value service and light value characteristic. You should get your UUID's from [this web site.](https://www.uuidgenerator.net/)

2. In the C++ code, you can establish proprietary UUID's with a constructor like this (but use your own UUID's from the generator mentioned in step 1.) These are hexadecimal digits represented in the 4-2-2-2-6 format.

```
BleUuid lightValueService("beb5483e-36e1-4688-b7f5-ea07361b26a8");

```

3. Write the necessary firmware to communicate light values to the LightBlue application running on your phone. The hardware will be configured as in Project 2, Part 2.

:checkered_flag:**Submit a screenshot (named LightBlue4.jpg) showing the LightBlue application receiving the light values from the Photon 2.**

4. Develop a Node-RED flow that communicates with the Photon 2 over BLE and that displays the light values in the debug widow of the Node-RED palette. Again, the hardware will be configured as in Project 2, Part 2. You will use the same firmware as that developed in the previous question. The output displayed on the Node-RED debug window should look like the following (for various light values. You will use a function node to extract the data from msg.payload and create a new output value for msg.payload. The function node programming is left as a small exercise.

Your debug node and debug output will appear as in the following picture:

<img src="https://github.com/mm6/InternetOfThingsCourse/blob/master/images/DebugNodeAndOutput.png" alt="Debug Node" width="400" height="400"/>
<em>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Debug Node and Output</em>  

<p></p>

:checkered_flag:**Submit a screenshot showing Node-RED receiving the light values from the Argon. Be sure to capture the entire screen - showing the Node-RED flow and the debug window. Name the file "Node-RED1.jpg".**

:checkered_flag:**Submit a screenshot showing the Javascript code that you wrote for the function node of Node-RED. This is the function that extracts the light value and passes the JSON string onto the debug node. Name the file "Javascript1.jpg".**

### Part 5. Node RED publishes the data to MQTT for publish/subscribe

In Part 5 write a Node-RED flow so that Node-RED becomes a gateway device - taking in BLE signals and generating publications to MQTT. The publications to MQTT will be over TCP/IP. Thus, the MQTT service itself might reside anywhere on the internet. We are taking data from a constrained network (BLE - measured in meters) and moving it to the internet.

0. See Project 2 and run an instance of MQTT.

1. Using the Node-RED flow from Part 4, publish the light values read from the sensor on the Photon 2 to MQTT. Use an "mqtt out" node and publish to the topic "lightValues" with a quality of service equal to 0. Set the sever to “localhost:1883” on the "mqtt out" node. This assumes that mosquito is running on that port.

:checkered_flag:**Submit a screenshot showing Node-RED receiving the light values from the Argon and publishing JSON strings to MQTT. The MQTT window should be visible in your screenshot. Name the file "Node-RED2.jpg".**

### Part 6. Subscribe to MQTT with a browser and display the light values.

Finally, to complete our system, use a browser to subscribe to the light values being generated by the MQTT broker.

The design of the display is in your hands. GoogleCharts may be used but a real time text display is also sufficient.


:checkered_flag:**Submit a screenshot showing a Node-RED flow receiving the light values from the Photon 2 and publishing JSON strings to MQTT. Name the file "Node-RED3.jpg". In addition, provide a screenshot showing the light values displayed on a browser. Name this screenshot browser1.jpg.**

### Troubleshooting

[Troubleshooting BLE on Windows](https://github.com/mm6/InternetOfThingsCourse/blob/master/projects/project3/WindowsBLEHelp.pdf)

If unable to install the ble node in node-red, make sure your using the correct version of npm and node. The error message should show the proper version.

My node version looks like this:
```
node --version
v16.15.0
```
My npm version looks like this:
```
npm --version
8.5.5
```

On a MAC, you will need to have X-Code installed.

If your command prompt says (base), that will be a problem. Remove (base) from the command prompt.


# 95-733 Internet of Things Fall 2021

# Project 3 Due: 11:59 PM Fall 2021

### Topics: BLE, Node RED, MQTT, the Web, and InfluxDB

:checkered_flag: Submit to Canvas a single .pdf file named Your_Last_Name_First_Name_Project3.pdf. This single pdf will contain your responses to the questions marked with a checkered flag. It is important that you ***clearly label*** each answer with the project part number and that you provide your name and email address at the top of the .pdf.

### Objectives

### Overview and setup

### Part 1. Programming the Argon to behave as a BLE peripheral device and installing LightBlue to behave as a BLE central device

0. Notice that the Argon's antennae can be plugged into a Wi-Fi connection or a BLE connection. Both are available on the Argon. In my work, I found that I was able leave the antennae connected to Wi-Fi only.

1. Study the following BLE peripheral code and flash it to your Argon.

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

	BLE.addCharacteristic(temperatureMeasurementCharacteristic);

	BLE.addCharacteristic(batteryLevelCharacteristic);
	batteryLevelCharacteristic.setValue(&lastBattery, 1);

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
			uint8_t buf[6];

			// The Temperature Measurement characteristic data is defined here:
			// https://www.bluetooth.com/wp-content/uploads/Sitecore-Media-Library/Gatt/Xml/Characteristics/org.bluetooth.characteristic.temperature_measurement.xml

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
			buf[5] = 6; // Mouth

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
	// I'm pretty sure this only works for positive values of temperature, but that's OK for the health thermometer.
	uint8_t  exponent = 0xFE; // Exponent is -2
	uint32_t mantissa = (uint32_t)(temperature * 100);

	return (((uint32_t)exponent) << 24) | mantissa;
}

```
2. Test your Argon's BLE code by [installing the LightBlue PunchThrough app](https://punchthrough.com/lightblue/) on your phone and making a BLE connection to your Argon.

This may require a bit of browsing with LightBlue to find the right BLE source. The dBm value (decibels relative to one milliwatt) is used to define the strength of the BLE signal. Anything greater than -75 dBm is considered a reliable signal. The closer that you get to 0, the more reliable the signal. On the PunchThrough application, you can use signal strength to help locate which signal is coming from your Argon.

:checkered_flag: Take two screenshots showing LightBlue connected to your Argon. The first screenshot (named LightBlue1.jpg) will contain the Advertisement Data screen showing the raw advertisement packet. The second screenshot (named LightBlue2.jpg) will show the Temperature Measurement Properties screen and the Read/Indicated values of the Health Thermometer's Temperature Measurement - it is important that you display the values arriving on your phone from the Argon. You will need to subscribe to receive these values.

### Part 2. Transmit text over BLE

0. Experiment by making minimal modifications to the firmware in Part 1 so that the LightBlue BLE screen displays "Hello" followed by your name. In other words, we will not be receiving temperature updates but, instead, we will be receiving a personalized greeting over BLE.

1. My solution looks like the following:

<img src="https://github.com/mm6/InternetOfThingsCourse/blob/master/images/Hello_Michael_LightBlue.jpg" alt="BLE Greeting" width="400" height="800"/>

<b>Screen scrape of LightBlue on an Android Phone</b>

:checkered_flag: Submit two files. The first will be a copy of the modified firmware and the second will be a screen capture of your phone running LightBlue and showing your name in the hello greeting.

### Part 3. Using Node RED to act as a gateway device

0. Restore the firmaware to its original state. That is, remove the code that we used to generate a personalized greeting over BLE.
We are back to generating random temperature values.

1. The goal is to establish a connection between Node-RED and the Argon using BLE. Install two BLE Nodes in Node-RED with the following shell commands:
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
4. Add an inject node to the palette.
5. In the Inject Node, set the message payload to JSON and set the message payload text to the following JSON message. This message will tell the "BLE In" node to listen for notify messages coming from the BLE connection.  

```
{"notify": true, "period": 0 }

```

6. Drag the "BLE In" node onto the palette. Create a wire from the "Inject" node in step 5 to the new "BLE In" node.

7. Double click the "BLE In" node and select the pencil symbol to edit the properties. Select the "BLE Scanning" check box and select "Apply".

8. Run the BLE firmware on the Argon and select the Argon in the Properties box of the Edit Generic BLE node. Be sure that the Emit Notify Events check box is selected. Note that the "Edit Generic BLE node" pane populates the UUID field.

9. Drag a debug node onto the palette and connect the Generic BLE
In node to it.

10. Deploy the flow. To start listening for notifications, click on the far left slice of the Inject node icon. With the Argon running, you should see the publications (notifications) from the Argon in the debug window.

11. Add a new Inject node and set the msg.topic to the value disconnect.

12. Add a new Inject node and set the msg.top to connect.

13. See the following image and wire the two new Inject nodes to the input of the Generic BLE In node. The BLE In node should now have three inject nodes wired to it. Experiment. Start and stop the connection using these new nodes. After the connection is started, inject a listening request from the middle inject node. Watch the output from the debug node.

<img src="https://github.com/mm6/InternetOfThingsCourse/blob/master/images/ArgonToNodeREDBLEFlow.png" alt="BLE Greeting" width="400" height="400"/>

<b>BLE connecting, listening, and disconnecting with Node-RED </b>


### Part 4. A BLE Exercise

0. Recall the LightMonitor firmware from Part 2 of Project 2. In that code, we transmitted light values via HTTP to Node-RED. Here, we will do the same but with BLE. Note that we want to transmit far fewer bytes than we did when using HTTP over TCP sockets.

1. We need to define our own UUID's for our proprietary light value service and light value characteristic. You should get your UUID's from [this web site.](https://www.uuidgenerator.net/)

2. In the C++ code, you can establish proprietary UUID's with a constructor like this (but use your own UUID's from the generator mentioned in step 1.)

```
BleUuid lightValueService("beb5483e-36e1-4688-b7f5-ea07361b26a8");

```

3. Write the necessary firmware to communicate light values to the LightBlue application running on your phone. The hardware will be configured as in Project 2, Part 2.

:checkered_flag: Submit a screenshot showing the LightBlue application receiving the light values from the Argon.

4. Develop a Node-RED flow that communicates with the Argon over BLE and that displays the light values in the debug widow of the Node-RED palette. Again, the hardware will be configured as in Project 2, Part 2. You will use the same firmware as that developed in the previous question. The output displayed on the Node-RED debug window should look like the following (for a light value of 17). You will use a function node to extract the data from msg.payload and create a new value for msg.payload.

```
msg.payload : string[17]

"{"lightValue":11}"
```

:checkered_flag: Submit a screenshot showing Node-RED receiving the light values from the Argon. Be sure to capture the entire screen - showing the Node-RED flow and the debug window.

:checkered_flag: Submit a screenshot showing the Javascript code that you wrote for the function node of Node-RED. This is the function that extracts the light value and passes the JSON string onto the debug node.

### Part 5. Node RED publishes the data to MQTT for publish/subscribe

0. See Project 2 and run an instance of MQTT.

1. Using the Node-RED flow from Part 4, publish the light values read from the sensor on the Argon to MQTT. Use an "mqtt out" node and publish to the topic "lightValues" with a quality of service equal to 0.

:checkered_flag: Submit a screenshot showing Node-RED receiving the light values from the Argon and publishing JSON strings to MQTT. The MQTT window should be visible in your screenshot.

### Part 6. Subscribe to MQTT with Node-RED and write to InfluxDB

0. [Visit this page and install InfluxDB on your machine.](https://docs.influxdata.com/influxdb/v2.0/install/)

1. Check if InfluxDB is running by visiting:

```
http://localhost:8086

```

2. [Install the InfluxDB nodes for Node-RED.](https://flows.nodered.org/node/node-red-contrib-influxdb)

3. Check if "influx" appears under the node_modules directory and check if Node-RED has new nodes: "influxdb in", "influxdb out" and "influx batch". You may ignore errors if these nodes are in place on Node-RED.

4. In Node-Red, drag an "mqtt in" node onto the palette. This begins a new flow but may appear on the same palette as your previous work. This node will subscribe to the topic "lightValues" that are being published by the "mqtt out" node from Part 5. Connect the "mqtt in" node to a debug node.

:checkered_flag: Submit a screenshot showing a Node-RED flow receiving the light values from the Argon and publishing JSON strings to MQTT. A second flow will be on the Node-RED palette and this flow will subscribe to the values being published by the first flow. The output of the second flow will appear on the debug pane.

5. In Node-RED, drag a InfluxDB out node onto the palette.

<img src="https://github.com/mm6/InternetOfThingsCourse/blob/master/images/ArgonOverBLE_to_NR_to_MQTT_to_NR_to_Influxdb.png" alt="InfluxDB Graph of Light Values" width="800" height="400"/>

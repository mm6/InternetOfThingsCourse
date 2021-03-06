
# 95-733 Internet of Things Fall 2021

# Project 3 Due: 11:59 PM Fall 2021

### Topics: BLE, Node RED, MQTT, the Web, and InfluxDB

:checkered_flag: Submit to Canvas a single .pdf file named Your_Last_Name_First_Name_Project3.pdf. This single pdf will contain your responses to the questions marked with a checkered flag. It is important that you ***clearly label*** each answer with the project part number and that you provide your name and email address at the top of the .pdf.

### Objectives

### Overview and setup

### Part 1. Programming the Argon to behave as a BLE peripheral device and installing LightBlue to behave as a BLE central device

1. Move the Argon's antennae from its Wi-fi connection to the BLE connection.

2. Study the following BLE peripheral code and flash it to your Argon.

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
3. Test your Argon's BLE code by [installing the LightBlue PunchThrough app](https://punchthrough.com/lightblue/) on your phone and making a BLE connection to your Argon.

This may require a bit of browsing with LightBlue to find the right BLE source. The dBm value (decibels relative to one milliwatt) is used to define the strength of the BLE signal. Anything greater than -75 dBm is considered a reliable signal. The closer that you get to 0, the more reliable the signal. On the PunchThrough application, you can use signal strength to help locate which signal is coming from your Argon.

:checkered_flag: Take two screenshots showing LightBlue connected to your Argon. The first screenshot (named LightBlue1.jpg) will contain the Advertisement Data screen showing the raw advertisement packet. The second screenshot (named LightBlue2.jpg) will show the Temperature Measurement Properties screen and the Read/Indicated values of the Health Thermometer's Temperature Measurement - it is important that you display the values arriving on your phone from the Argon. You will need to subscribe to receive these values.

### Part 2. Transmit text over BLE

1. Experiment by making minimal modifications to the firmware in Part 1 so that the LightBlue BLE screen displays "Hello" followed by your name. In other words, we will not be receiving temperature updates but, instead, we will be receiving a personalized greeting over BLE.

2. My solution looks like the following:

<img src="https://github.com/mm6/InternetOfThingsCourse/blob/master/images/Hello_Michael_LightBlue.jpg" alt="BLE Greeting" width="400" height="800"/>


:checkered_flag: Submit two files. The first will be a copy of the modified firmware and the second will be a screen capture of your phone running LightBlue and showing your name in the hello greeting.

### Part 3. Using Node RED to act as a gateway device

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

Within Node-RED, expand the "Network" icon on the left and verify the presence of two BLE nodes: Generic BLE In and Generic BLE out. 

3.

### Part 4. Node RED publishes the data to MQTT for publish/subscribe

1. Run mosquitto

### Part 5. Two subscribers - InfluxDB and the World Wide Web

1. Install InfluxDB

# 95-733 Internet of Things           Project 3

## IoT and Webhooks                   Due: Tuesday, March 16, 11:59:00 PM


There are three parts to Project 3.

+ In Part 1, you will configure hardware and write firmware to detect light levels and publish these levels to the cloud (at Particle).
+ In Part 2, you will configure a webhook so that the light levels are published from Particle to ThingSpeak. You will also build a small dashboard of widgets at ThingSpeak to graphically display your light level data.
+ In Part 3, you will use constrained low level networking and configure your Argon to behave as a BLE peripheral.  

### Part 1. Hardware setup

1. Hold the breadboard so that row 1 is away and row 30 is near (on the bottom).
2. Place the Argon bottom on row 26 of the breadboard.
3. The Argon top will now just cover row 7.
4. Be sure that you can read the writing on the Argon. It will be right side up.
5. Place the long leg of the photo transistor in row 13, (A0) (on the left of the Argon).
6. Place the short leg of the photo transistor in row 10 (3V3).
7. Place a 220 ohm resistor in row 12 (GND) and in row 10 (A0).
8. After flashing the code from Figure 1 to your Argon, run the command "particle serial monitor" on the command line interface.
10. Test your system by changing your lighting and monitoring the numeric light levels on the command line interface.

```
// File name: LightMonitor
// View output from the command line with
// particle serial monitor

int photoResistor = A0;
int analogValue;

unsigned long loop_timer;

void setup() {
  Serial.println("Simple setup complete");
  loop_timer = millis();
}


void loop() {
    if(millis() - loop_timer >= 1000UL) {
        loop_timer = millis();
        analogValue = analogRead(photoResistor);
        Serial.printlnf("AnalogValue == %u", analogValue);
    }
}
```
Figure 1

   Write a firmware program named ReadAndPublishLightValues that reads the light level (A0) value every 30
   seconds. And every 30 seconds it publishes a message to the Particle console. To generate a string from an int in C++, you may use the following code:

```
    char value[80];
    sprintf(value, "%d", lightValue);
    Particle.publish("LightValue", value ,PRIVATE);          
```

   We do not want to publish faster than once every 30 seconds. In this way, the
   Particle Cloud and the Webhook call to Thingspeak can keep up. The Particle console should display
   your name value pairs.

### Part 2. Using a Webhook

   In this part, you will configure the Particle cloud service to forward your light value messages to Thingspeak. This will result in a new HTTP request to Thingspeak after each call on the Particle cloud from your Argon. This is called a Webhook. It is a simple way to implement publish/subscribe messaging. Your Argon will publish a message to the event name "LightValue" and Thingspeak will be called for each
   publication. Thingspeak acts as webhook subscriber.

   You must configure a channel in Thingspeak so that it is ready to receive the LightValue message. You will also need a "write API-Key" from Thingspeak and this key needs to be provided to the Webhook on Particle.

   [Create a ThingSpeak account and work through this tutorial](https://docs.particle.io/guide/tools-and-features/webhooks/)

   This tutorial has everything that you need to create the account at Thingspeak
   and prepare the Webhook at Particle. Be sure to use the "form" approach when
   constructing the Webhook.

   Your channel will be named "LightValueChannel" and Field1 will contain "LightValue".

   Edit your chart on ThingSpeak so that the title is "Argon Light Levels". The y-axis is labelled with "Light Values". Choose a dynamic line graph. The y-axis Min will be 0. The y-axis Max will be 200. The x-axis will be labelled with "Time". Take the defaults for color and background and be sure to leave the remaining fields blank.

   Add a numeric display widget to your ThingSpeak dashboard. Its name is "Light Level" and its update is every 15 seconds. Its Data Type is integer.

   Add a lamp widget to your ThingSpeak dashboard. The lamp will turn on if the Argon reports of values greater than 100.


### Part 3. Capturing a BLE signal from an Argon.

   Particle provides a tutorial on the Argon and BLE [here](https://docs.particle.io/tutorials/device-os/bluetooth-le/). Using the firmware from the tutorial (copied below), configure your Argon to act as a BLE peripheral. Submit a screen shot showing a BLE central device connected to your Argon. I have had success using [LightBlue described here](https://punchthrough.com/lightblue/). The article on the Argon and BLE at  Particle recommends using the [nRF Toolbox mobile app](https://www.nordicsemi.com/Software-and-tools/Development-Tools/nRF-Toolbox) from Nordic Semiconductor. Either of these (or perhaps another) will do fine.

```
#include "Particle.h"

// This code is found here: https://docs.particle.io/tutorials/device-os/bluetooth-le/
// This example does not require the cloud so you can run it in manual mode or
// normal cloud-connected mode
// SYSTEM_MODE(MANUAL);

SerialLogHandler logHandler(LOG_LEVEL_TRACE);

const unsigned long UPDATE_INTERVAL_MS = 2000;
unsigned long lastUpdate = 0;

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


### Submission guide

Place the following in a single directory and zip that directory into a file named yourAndrewIDProject3.zip.
Submit the single zip file to Canvas.

a) Turn in a copy of your well-documented firmware ReadAndPublishLightValues.ino. Include your name
as the author and include a description of the code. On each line of code, explain briefly
what that line is doing.

b) Turn in a screen shot showing the graphical display at Thingspeak. The display will
show a line graph showing the different values over time. It will also display a lamp widget and a numeric value widget. Name the screenshot yourAndrewIDProject3PartBScreenShot. The line graph must
show at least 5 distinct light level values connected by lines.

c) Turn in a screenshot showing a central device connected to an Argon running as a BLE peripheral. Name the screen shot yourAndrewIDProject3PartCScreenShot.

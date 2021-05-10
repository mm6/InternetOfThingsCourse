# 95-733 Internet of Things Wednesday, May 26, 2021

## Project 1  Due: 11:59 PM, Wednesday, June 9, 2021

### Part 1: Programming a Particle Argon to report heartbeats to the Particle cloud and Node- RED

#### Necessary Installations

0) [Follow these directions and install Node.js.](https://nodejs.org/en/download/)

1) The node package manager (npm) was installed as part of node.js in step 0. Next, use npm to install Node-RED locally on your laptop or desktop machine.

On Mac OSX, run the shell command:
```
sudo npm install -g --unsafe-perm node-red
```

On Windows, run the shell command:
```
npm install -g --unsafe-perm node-red

```

2) Execute Node-RED with the shell command:
```
   node-red
```
3) Test a running instance of Node-Red.
   Use a browser and visit the localhost URL:
```
   http://127.0.0.1:1880/
```

#### Argon Setup, Particle IDE, and Particle CLI
We will be programming the Argon by developing our firmware code in C++, compiling to machine code, and downloading via Particle's over the air (OTA) update.

4) [Establish credentials at Particle.io](https://www.particle.io/)

5) [Install the Particle Command Line Interface (CLI) by following these directions.](https://docs.particle.io/tutorials/developer-tools/cli/)

6) [Set up your Particle Argon.](https://docs.particle.io/quickstart/argon/)
It is recommended that you complete this tutorial. That is, blink an LED on your microcontroller as per the instructions in the Argon Quickstart. In addition, carefully read over the documentation in the firmware.

#### Publish heartbeat messages to the Particle console

7) Use the code below to publish heartbeat messages to the Particle console.

```
/*
Author: mm6 with various code snippets taken from Particle.io.
This firmware generates periodic heartbeats from an Argon to
the Particle cloud using Particle.publish(string name,string value).
The first argument will be the name of the event. The event name is
'heartbeat'. The second argument will be a JSON string holding
the device ID.
*/

// Setting DEBUG == true generates debugging output to a shell.
// To view these messages, install the Particle CLI and enter the command:
// particle serial monitor

boolean DEBUG = true;

// Establish the number of seconds to wait until calling the server.
int NUMSECONDS = 10;

// The variable timeCtr will be used to hold the current time in milliseconds.
int timeCtr = 0;

// The device ID will be stored here after being retrieved from the device.
// This is a unique, 96 bit identifier.
// This looks like the following: 0x3d002d000cf7353536383631.

String deviceID = "";

// buf will hold the JSON string
char buf[80];

// This class makes handling JSON data easy.
// Associate it with the buf array of char.
JSONBufferWriter writer(buf, sizeof(buf));


void setup() {
    // to allow for debug using the CLI
    Serial.begin(9600);

    // get the unique id of this device as 24 hex characters
    deviceID = System.deviceID().c_str();

    // display the id to the command line interface
    if (DEBUG) Serial.println(deviceID);

    // establish the JSON string that we will send

    writer.beginObject();
    writer.name("deviceID").value(deviceID);
    writer.endObject();

    // The JSON is now available through the buf array.

}

// Event name to send to the server at Particle
String name = "heartbeat";

void loop() {
     // If timeCtr is above the current time wait until the current time catches up.
     // Initially, timeCtr is 0. It is then set to the current time plus 10 seconds.
     // millis() returns an updated time.
     if (timeCtr <= millis()) {

            // publish to Particle the event name and the JSON string
            Particle.publish(name,String(buf));

            // display a status report if DEBUG is true
            if (DEBUG) Serial.println("Heartbeat sent to Particle");

            // set timeCtr to current time plus NUMSECONDS seconds
            timeCtr = millis() + (NUMSECONDS * 1000);
     }
}
```
8) Study the code above. After compiling and deploying the code to your Argon, use a shell and check if the CLI is working properly.

9) The following are useful CLI commands.

```
$particle login          Login to Particle
$particle serial monitor Listen for Serial.print
$particle update-cli     Update the CLI
```
[There are plenty of additional CLI commands.](https://docs.particle.io/tutorials/developer-tools/cli/)

10) Using the Particle console, verify that it is receiving the heartbearts from the Argon.

#### Using Node-RED, subscribe to the heartbeat messages that are being published to particle.

11) Our goal is for Node-RED to subscribe to and receive messages from the Particle console. These messages are being published to the Particle console by our Argon. We want Node-RED to hear about the heart beats. [Read about Node-RED and integrate Node-RED
with the Particle console.](https://docs.particle.io/community/node-red/)

12) After completing the work in step 11, you should have a Node-RED platform receiving messages from your Argon every 10 seconds or so. Each message should contain a JSON string with the device ID. Each message should appear in the right pane of the Node-RED UI.
The Node-RED palette should have a subscribe node wired to a debug node.

### Part 2: Node-RED Programming

#### Data processing

0) The objective of this part is to gain skills in the creation and execution of Node-RED flows. We will work with the heartbeat data that we are receiving from our Particle Argon. [Follow this link for a good place to learn about Node-RED and its capabilities.](https://nodered.org/docs/)

1) Wire a function node in between your subscribe node and your debug node. The function node will add a timestamp to the incoming message. Use this code in your function node:

```
// create a javascript object using the JSON message payload
var newMessage = JSON.parse(msg.payload);
// add a time field to the new object
newMessage.time = new Date();
// represent the new object as JSON
msg.payload = JSON.stringify(newMessage);
// pass it on to the next node
return msg;
```

2) Add another function node that checks the timestamp. If the current message arrived late (after 15 seconds) then set an ontime variable to false, otherwise set it to true. Include the new field in the json message.  

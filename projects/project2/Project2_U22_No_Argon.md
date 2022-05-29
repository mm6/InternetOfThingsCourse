
# 95-733 Internet of Things Adelaide 2022

# Project 2 Due: Wednesday, June 15, 2022

### Topics: MQTT, Particle Argon, Node-RED, Node.js, Google Charts

:checkered_flag: Submit to Canvas a single .pdf file named Your_Last_Name_First_Name_Project2.pdf. This single pdf will contain your responses to the questions marked with a checkered flag. It is important that you ***clearly label*** each answer with the project part number and that you provide your name and email address at the top of the .pdf.

Many IoT applications monitor real time data from sensors and
transmit these data to a service offering publish and subscribe capabilities. The service behaves as a broker. The sensor plays the role of publisher and it may have no idea who its subscribers are. It simply transmits data to the broker. A subscriber subscribes to messages from the broker and, in this way, is able to consume data from the sensors.

The publish subscribe approach exemplifies loose coupling, provides for scalability, and is well established in the internet of things.

### Objectives

One objective of this project is to learn to use an MQTT publish and subscribe broker. There are many implementations of MQTT. We will use a broker named Mosquitto.

MQTT is targeted at low power, constrained, unreliable devices. It will likely play an important role in a mature IoT. In class, we will describe how MQTT works and how many of the big players are leveraging MQTT.

The second objective is to learn the Google Charts Javascript library. The internet of things will rely heavily on real time visualizations in the browser. Google Charts will likely play a significant role in this area.

The third objective is to continue our work with the Particle Argon microcontroller.

There will be some challenges along the way.


### Overview and setup


This is a five part assignment. The first part involves setting up an MQTT broker and interacting with the broker using browsers and WebSockets. The web applications will be built with Node.js.

The second part requires that you simulate an Argon with a Node-RED node.

The third part involves the construction of a Node-RED flow that receives messages from the Argon simulation node, makes modifications to the messages, and publishes the messages to MQTT. Note that we are not routing our messages through the Particle Cloud - as we did in Project 1.

The fourth part involves writing a web application that provides browser code that subscribes to MQTT for the Argon's simulated messages. The browser will produce a textual display.

Part 5 is the same as Part 4 but includes Google Charts for visualization.

In Project 1, we monitored an Argon heartbeat simulation in real time with a browser. We leveraged WebSockets and wrote our own server side code that sent a message to all listeners.

In much of this project, we will not write our own server side code (as we did in Project 1). Instead, we will use Mosquitto - an open source MQTT broker. [Download Mosquitto from here.](http://mosquitto.org/download/)

Note: In the past, some students had trouble with Mosquitto but had an easy time downloading Hive MQ's MQTT broker. The instructions below refer to my success in using Mosquito. Feel free to use whatever MQTT broker that you are comfortable with. You may use the local Hive MQ (downloaded to your machine
or the remote version on the cloud.) Hive MQ's MQTT (both local and cloud) are available [from here](https://www.hivemq.com/try-out/).

You can install Mosquitto on your MAC using brew. See [here](http://brew.sh) and then use "brew install mosquitto". There are directions on the mosquitto web site for Windows users.

When you run mosquitto, it will run with its default configuration. We need to change its configuration to allow for WebSocket connections (we want to visit the server from
Javascript). To change the configuration of Mosquitto, we can modify the configuration file found at /usr/local/etc/mosquitto/mosquitto.conf (on a MAC). My configuration file contains these lines at the very end of the file:

```
listener 1883
protocol mqtt
listener 9002
protocol websockets
allow_anonymous true

```

My copy of mosquitto is located at /usr/local/sbin. If I change to that directory, I can run mosquitto (picking up the configuration file) with the command

```
mosquitto -c /usr/local/etc/mosquitto/mosquitto.conf -v

```
Other students have their configuration file stored in /opt/homebrew/etc/mosquitto/mosquitto.conf. You may have to look around a bit.

Once you have Mosquitto running, we want to connect to the server from Javascript running in a browser. We do not want to write the client side MQTT code ourselves. Even though we have
easy access to WebSockets, it would be far better to use an existing Javascript library
that provides a convenient API to MQTT. The implementation of the API will, of course,
use WebSockets below the scenes.

In our web pages, we will include the Javascript library from Paho. It is called mqttws31.js.

To make use of the mqttws31.js library, there is a very nice "getting started" section at the following link. It has a simple example of a Javascript client that will run in a web page. It is very useful for us and is found [here.](https://eclipse.org/paho/clients/js/)

I am including a copy of this client side Javascript here. Be sure to study how this works. You will need to incorporate it into your browser when you want your browser to communicate with an MQTT broker. Notice that it makes use of the Paho.MQTT library.

```
// Create a client instance
client = new Paho.MQTT.Client(location.hostname, Number(location.port), "clientId");

// set callback handlers
client.onConnectionLost = onConnectionLost;
client.onMessageArrived = onMessageArrived;

// connect the client
client.connect({onSuccess:onConnect});


// called when the client connects
function onConnect() {
  // Once a connection has been made, make a subscription and send a message.
  console.log("onConnect");
  client.subscribe("World");
  message = new Paho.MQTT.Message("Hello");
  message.destinationName = "World";
  client.send(message);
}

// called when the client loses its connection
function onConnectionLost(responseObject) {
  if (responseObject.errorCode !== 0) {
    console.log("onConnectionLost:"+responseObject.errorMessage);
  }
}

// called when a message arrives
function onMessageArrived(message) {
  console.log("onMessageArrived:"+message.payloadString);
}
```

The mqttws31.js file holds the Paho library and can be included in your HTML file with this script tag:

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.js" type="text/javascript"></script>
```

There is a nice discussion [here on MQTT.](https://developer.ibm.com/articles/iot-mqtt-why-good-for-iot/) It explains the publish subscribe patterns and shows an example using sensors.

### Part 1.

Your Part 1 we will experiment using real time data on the web.

1. 5 Points. Create an empty directory named Project2_Part1_Question_1.
Create a subdirectory named "public" and populate it with an index.html file. [This file contains the code found here.](http://www.w3schools.com/jsref/tryit.asp?filename=tryjsref_onmousemove)
In the Project2_Part1_Question_1 directory, just above "public", create a file named index.js with the following code:

```
// index.js
// A directory named "public" is available in the current directory.
// The public directory holds index.html containing HTML and Javascript.
// A browser will visit with http://localhost:3000/index.html

const express = require('express')
const port = 3000
app = express();
// allow access to the files in public
app.use(express.static('public'));

app.listen(port, () => {
  console.log(`Mouse moving app listening at http://localhost:${port}/index.html`)
})
```
In the Project2_Part1_Question_1 directory, run the following three commands:

```
npm init               note: accept the defaults
npm install express
node index.js
```
You should be able to use a browser to visit http://localhost:3000/index.html.

It is important that you take some time to study the html and Javascript in the index.html file.  

Note that every time the mouse moves (within the box) an event is generated
and the x and y coordinates are displayed. It is required that you add your
own detailed comments to this code - explaining clearly how it works. The grader
will grade based on the quality of these comments.

:checkered_flag: Take a screenshot showing the browser screen.  Name your screenshot Project2Part1Question1.png.

:checkered_flag: Include your HTML and Javascript with comments.  Name your file Project2Part1Question1.html.

2. 15 Points. Here we want to sense the mouse movements and publish them to Mosquitto running MQTT. Create a new directory named Project2_Part1_Question_2. Include the same index.js file that you used in Part 1. Create a subdirectory named "public" containing an index.html file that publishes each new mouse coordinate pair to your MQTT broker. Your solution must make good use of the Javascript library - mqttws31.js.

3. 15 Points. Here we want to subscribe to the events being published in Part 1 Question 2. Create a new directory named Project2_Part1_Question_3.
Create subdirectories and files as you did in Question 2. This time, however, the web application will listen on port 3002 . In the index.js file, change the listening port from 3000 to 3002. This is so that we do not interfere with the server in Question 2 (listening on port 3000). The index.html file will make good use of mqttws31.js. It will subscribe to events published to Mosquitto and it will display the coordinates being received (there will be no box, just text being displayed). Test your system by running two browsers at the same time. One browser will display the box (from Question 2) and the other will display rapidly changing coordinates as the user of the other browser moves the mouse within the box.

In my solution, I included the following code before any Javascript code that makes use of it. This code is used to establish a connection to MQTT:

```
client = new Paho.MQTT.Client('localhost', Number(9002), "MouseTrackerSubscriber");
```


Note: MQTT is very fussy about client names. Each client that visits must present a unique name. In this part of Project 2, we need two names - one for the publisher and another for the subscriber. If you add several subscribers, you will need several names. The client shown just above has the name "MouseTrackerSubscriber".

Note too that the communication between the broker and the browsers is done with WebSockets but we have abstracted those details away. The details are all hidden within the mqttws31.js library. By using "abstraction" we are hiding details and separating concerns - these are very important principles in computer science and in many areas of engineering.

:checkered_flag: Take a screenshot showing two browser screens.  Name your screenshot Project2Part1Questions2And3.png.

:checkered_flag: Include your HTML and Javascript with comments.  Name your files Project2Part1Question2.html and Project2Part1Question3.html

### Part 2 Monitoring light levels by simulating a microcontroller


1. Our goal is to build a Node-RED flow with a final debug node generating text such as this:

```
"{"deviceID":"e00fce68dfb40ca61243495e","lightReading":"89","time":"2021-06-05T21:00:09.832Z"}"
```

2. We need random light values between 0 and 200. In Node-RED, use the hamburger icon and select manage palette. Choose to install
a node. The node that we want is called node-red-node-random. This needs to be fetched from the web. It should be installed and visible
as a function node named "random". Configure the node with the name "LightLevelSimulator". Configure so that it generates random
values between 0 and 200.

3. Drag an inject node onto your flow. It should be configured to repeat every 5 seconds. Its name will be "Interval" and its message.payload
will be of type timestamp. Drag an arrow from the output of this node to the input of the "LightLevelSimulator" node.

4. Drag a function node onto the palette. Name this node "AddTimeAndDeviceID". Its input will come from the output of the "LightSimulator" node. This function node will add a timestamp, a device ID, and will generate JSON text in the message.payload. The code for this node is provided here:
```
// Get the light level
var lightLevel = msg.payload;
// Create an empty object
var myMessageObject = {};
// add a timestamp
myMessageObject.time = new Date();
// Add a device ID
myMessageObject.deviceID = "e00fce68dfb40ca61243495e";
// add the light level
myMessageObject.lightReading = lightLevel.toString();
// stringify the object to JSON and place in the payload
msg.payload = JSON.stringify(myMessageObject);
// pass it on to the next node
return msg;
```

5. Drag a debug node onto the palette. Connect the output of "AddTimeAndDeviceID" to the input of the debug node. Deploy the
flow to the Node-RED server. You should now see a new output every 5 seconds. The order of the fields in the JSON does not matter.


:checkered_flag: Take a screenshot showing the Node-RED flow and the debug window showing JSON strings. Name your screenshot Project2Part2.png.


### Part 3. Publishing Light Levels to MQTT

1. Add an "MQTT OUT" node to the Node-RED palette. Configure the new node to publish to the topic "argonLightLevel". The quality of service (QoS) is 0 and Mosquitto's port is 1883 running on localhost. The name of this Node is "To MQTT".

2. Connect the "AddTimeAndDeviceID" node to the "To MQTT" node. Run mosquitto and deploy the flow. You should see events being published to mosquitto every 5 seconds or so.

:checkered_flag: Take a screenshot showing the Node-RED flow and the MQTT window as it receives visits. Name your screenshot Project2Part3.png.

### Part 4. Several browsers subscribe to microcontroller light levels

1. Write a web application using Node.js that includes Javascript that subscribes to the topic "argonLightLevel". The Javascript will be included in an HTML file and the browser display will show the changing values as they are reported from the Argon.

2. When the HTML runs in the browser, ask the user to provide a client side name. This name will be used to provide the client with a unique identifier as it connects to MQTT. If we do not change the name for each client, MQTT will assume that separate visits are all from the same client and will only respond to the last visitor. In other words, we would only be able to subscribe from one browser instance. We want to have several simultaneous visitors. How you design this input request is in your hands. You will need to work a bit with the HTML and Javascript.

:checkered_flag: Take a screenshot showing more than one browser receiving the same data from the broker. Name your screenshot Project2Part4.png.

:checkered_flag: Create a text file with the HTML and Javascript  code that is inside your index.html file. Name this file Project2Part4.html.

### Part 5. Subscribe to MQTT and visualize with Google Charts

1) [Spend some time reviewing Google Charts.](https://developers.google.com/chart) In particular, we are interested in the Gauge Chart.

2) Write a web application using Node.js. The web application will deliver HTML and Javascript to the browser and the the browser will subscribe to the MQTT "argonLightLevel" topic. Use Google Charts to visualize the data with a gauge chart. Several browser should be able to visit at the same time and view the same gauge.

![Two browsers subscribe with different ID's.](https://github.com/mm6/InternetOfThingsCourse/blob/master/images/TwoBrowsersSubscribe.png?raw=true)

These two browsers are each subscribing with a different client ID.

3) Add an additional Google Chart to the same web page. This would be any chart that you select (other than Gauge).

:checkered_flag: Take a screenshot showing the visualization. Name your screenshot Project2Part5.png.

:checkered_flag: Create a text file with the HTML and Javascript  code that is inside your index.html file. Name this file Project2Part5.html.

### Optional notes on Using a remote (rather than local) MQTT broker

If you are unable to install Mosquito on your local machine, I have had success with the following broker on the cloud:

broker.hivemq.com  for MQTT use port  1883
and for Websockets use port  8000

In my Javascript, I am using a line like this:
```

 var loc = {'hostname' : 'broker.hivemq.com', 'port' : '8000' };
```
When programming in Java, I would use a line like this:

```
String broker  = "tcp://broker.hivemq.com:1883";
```

Mosquito is fussy about the client ID. In the following line of code, I have used my Andrew ID.

```
client = new Paho.MQTT.Client(loc.hostname, Number(loc.port), 'mm6');
```

### Optional notes on Installing Mosquito on a Windows machine


Download mosquitto setup file from [here.](http://www.eclipse.org/downloads/download.php?file=/mosquitto/binary/win32/mosquitto-1.4.10-install-win32.exe)

Run the setup file and follow the steps to complete the installation.
note: Default destination folder is C:\Program Files (x86)\mosquitto

Download and Install Win32 OpenSSL v1.0.2h [from here.](http://slproweb.com/products/Win32OpenSSL.html)

Install it to C:\temp\OpenSSL-Win32.

Copy libeay32.dll and ssleay32.dll from C:\temp\OpenSSL-Win32\bin to C:\Program Files x86\mosquitto  


Download and Install pthreadVC2.dll [from here.](ftp://sources.redhat.com/pub/pthreads-win32/dll-latest/dll/x86/)


Copy pthreadVC2.dll to C:\Program Files (x86)\mosquitto)

### Optional notes on testing on a Windows machine

Open a command prompt and run the following commands:

```
cd C:\Program Files (x86)\mosquitto

mosquitto.exe -v -c mosquitto.conf
```

You should get messages similar to this:  

1474406476: mosquitto version 1.4.10 (build date 24/08/2016 21:03:24.73) starting  

1474406476: Config loaded from mosquitto.conf.  

1474406476: Opening ipv6 listen socket on port 1883.  

1474406476: Opening ipv4 listen socket on port 1883.  

1474406479: mosquitto version 1.4.10 terminating

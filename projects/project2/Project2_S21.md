# DRAFT   DRAFT   DRAFT  DRAFT   DRAFT   

# 95-733 Internet of Things   Wednesday, June 9, 2021

## Project 2 Due Date: 11:59 PM Wednesday,June 21, 2021

### Topics: MQTT, Particle Argon, Node-RED, Node.js, Google Charts

# DRAFT   DRAFT   DRAFT  DRAFT   DRAFT
Plan:
     Part 1. Preparation: Work with MQTT from the browser (Node.js)
     Part 2. Argon light values to Node-RED using HTTP
             Hardware setup and firmware
     Part 3. Node-RED adds JSON-LD and publishes to an MQTT broker
     Part 4. Several Browsers present a real time graphical
             display using Googl Charts - Digital readout, gauge and line graph (all served up from Node.js).
     Part 5. (Optional but way cool) Transmit the light values to
             Node-RED using BLE

Many IoT applications collect real time data from sensors and
transmits these data to a service offering publish and subscribe capabilities. The service behaves as a broker. The sensor plays the role of publisher and it may have no idea who its subscribers are. It simply transmits data to the broker. A subscriber subscribes to messages from the broker and, in this way, is able to consume data from the sensors.

The publish subscribe approach exemplifies loose coupling and provides for scalability.

The publish subscribe design pattern is well established in the internet of things.

Note: In the section of this project where you work with the Particle Argon and MQTT, you may need to turn
your fire wall off and then reboot your machine. Firewalls can be sticky and may only
appear to be off. Turn your fire wall back on after completing this assignment.

### Objectives

One objective of this project is to learn to use an MQTT publish and subscribe broker. There are many implementations of MQTT. We will use a broker named Mosquitto.

MQTT is targeted at low power, constrained, unreliable devices. It will likely play an important role in a mature IoT. In class, we will describe how many of the big players are using MQTT.

The second objective is to learn the Google Charts Javascript library. The internet of things will rely heavily on real time visualizations in the browser. Google Charts will likely play a significant role in this area.

The third objective is to continue our work with the Particle Argon microcontroller.

### Overview and setup


This is a three part assignment. The first part asks that you develop a small
system using Javascript and an MQTT broker. The second part asks that you
build a small system using a Java program as a publisher to MQTT and a
Javascript program as a subscriber to messages coming from MQTT. The third
part asks that you create a simple application that will be deployed (flashed)
to a Particle Argon. This application will make good use of MQTT.

In Project 1, we monitored Argon heartbeats in real time with a browser. We leveraged WebSockets and wrote our own server side code that sent a message to all listeners.

In Parts 1 and 2 of this project, we will not write our own server side code (as we did in Project 1). Instead, we will use Mosquitto - an open source MQTT broker. Download Mosquitto
[from here](http://mosquitto.org/download/).

Note: In the past, some students had trouble with Mosquitto but had an easy time
downloading Hive MQ's MQTT broker. The instructions below
refer to my success in using Mosquito. Feel free to use whatever MQTT broker that
you are comfortable with. You may use the local Hive MQ (downloaded to your machine
or the remote version on the cloud.) Hive MQ's MQTT (both local and cloud) are
available [from here](https://www.hivemq.com/try-out/).

You can install Mosquitto on your MAC using brew. See [here](http://brew.sh) and then
use "brew install mosquitto". There are directions on the mosquitto web site for
Windows users.

When you run mosquitto, it will run with its default configuration. We need to change
its configuration to allow for WebSocket connections (we want to visit the server from
Javascript). To change the configuration of Mosquitto, we can modify the configuration
file found at /usr/local/etc/mosquitto/mosquitto.conf (on a MAC). My configuration file
contains these lines:

```
listener 1883
protocol mqtt
listener 9002
protocol websockets
allow_anonymous true

```

My copy of mosquitto is located at /usr/local/sbin. If I change to that
directory, I can run mosquitto (picking up the configuration file) with the command

```
mosquitto -c /usr/local/etc/mosquitto/mosquitto.conf -v

```

Once you have Mosquitto running, we want to connect to the server from Javascript running
in a browser. We do not want to write the client side MQTT code ourselves. Even though we have
easy access to WebSockets, it would be far better to use an existing Javascript library
that provides a convenient API to MQTT. The implementation of the API will, of course,
use websockets below the scenes.

In our web pages, we will include the Javascript library from Paho. It is called mqttws31.js.

To make use of the mqttws31.js library, there is a very nice "getting started" section at the following link. It has a simple example of a
Javascript client that will run in a web page. It is very useful for us and is found [here.](https://eclipse.org/paho/clients/js/)

I am including a copy of this client side Javascript here. Be sure to study how this works. You will need to incorporate it into your browser when you want your browser to communicate with an MQTT broker. Notice that it makes use of the Paho.MQTT library which we can include with a script tag shown below.

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

The mqttws31.js Javascript library can be included in your HTML file with this script tag:

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.js" type="text/javascript"></script>
```

There is a nice discussion [here on MQTT.](https://developer.ibm.com/articles/iot-mqtt-why-good-for-iot/)

### Part 1.

Your Part 1 we will experiment using real time data on the web.

1. 5 Points. Create an empty directory named Project2_Part1.
Create a subdirectory named "public" and populate it with an index.html file. This fill contain the code found [here.](http://www.w3schools.com/jsref/tryit.asp?filename=tryjsref_onmousemove).
In the Project2_Part1 directory, just above "public", create a file named index.js with the following code:

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
In the Project2_Part1 directory, run the following two commands:

```
npm init
node index.js
```
You should be able to use a browser to visit http://localhost:3000/index.html.

Take some time to study the html and Javascript in the index.html file.  

Note that every time the mouse moves (within the box) an event is generated
and the x and y coordinates are displayed. It is required that you add your
own detailed comments to this code - explaining clearly how it works. The grader
will grade based on the quality of these comments.

2. 15 Points. Here we want to sense the mouse movements and publish them to Mosquitto running MQTT. Create a new directory named Project2_Part2. Include the same index.js file that you used in Part 1. Create a subdirectory named "public" containing an index.html file that publishes each new mouse coordinate pair to your MQTT broker. Your solution must make good use of the Javascript library - mqttws31.js.

3. 15 Points. Here we want to subscribe to the events being published in Part 2. Create a new directory named Project2_Part3.
Create subdirectories and files as you did in part 2. This time, however, the web application will listen on port 3002. The index.html file will make good use of mqttws31.js. It will subscribe to events published to Mosquitto and it will display the coordinates being received (there will be no box, just text being displayed). Test your system by running two browsers at the same time. One browser will display the box (from Part 2) and the other will display rapidly changing coordinates as the user of the other browser moves the mouse within the box.

Note: MQTT is very fussy about client names. Each client that visits must present a unique name. In this part of Project 2, we need two names - one for the publisher and another for the subscriber. If you added several subscribers, you would need several names.

Note too that the communication between the broker and the browsers is done with websockets but we have abstracted those details away. The details are all hidden within the mqttws31.js library. By using "abstraction" we are hiding details and separating concerns - these are very important principles in computer science and in many areas of engineering.

### Part 2 Monitoring light levels with a microcontroller

#### Hardware Set Up

1. Hold the Argon breadboard so that row 1 is away and row 30 is near (on the bottom).
2. Place the Argon bottom on row 26 of the breadboard.
3. The Argon top will now just cover row 7.
4. Be sure that you can read the writing on the Argon. It will be right side up.
5. Place the long leg of the phototransistor in row 13, (A0) (on the left of the Argon).
6. Place the short leg of the phototransistor in row 10 (3V3).
7. Place a 220 ohm resistor in row 12 (GND) and in row 13 (A0).
8. Your hardware setup should look like the following:
![Argon Light Monitor](https://github.com/mm6/InternetOfThingsCourse/blob/master/images/ArgonLightMonitor.png?raw=true)

9. We need to flash code that monitors light levels to the Argon. We will name this firmware "LightMonitor". Use the Particle cloud to compile and deploy this firmware to your microcontroller.

```
// File name: LightMonitor
// View output from the command line with
// particle serial monitor

int photoTransistor = A0;
int analogValue;

unsigned long loop_timer;

void setup() {
  Serial.println("Simple setup complete");
  loop_timer = millis();
}

void loop() {
    if(millis() - loop_timer >= 5000UL) {
        loop_timer = millis();
        analogValue = analogRead(photoTransistor);
        Serial.printlnf("AnalogValue == %u", analogValue);
    }
}
```

10. To monitor the Argon, run the command "particle serial monitor" from the command line interface.

11. Test your system by changing your lighting and monitoring the numeric light levels on the command line interface. A value near 0 would signal no light and a value of 1000 or so would mean the phototransistor is near a light bulb.

12. We are reading the light values every 5 seconds and the values are available to the serial monitor. Next, we would like to transmit these values to Node-RED every 5 seconds. We will use standard HTTP and JSON messages to do so.

13. Using the "Particle Libraries" icon (on the far left just above the question mark), add the httpclient library and include it in the LightMonitor project. Your code should now include the C++ statement:
```
#include <HttpClient.h>

```

14. Read over the following code. And add this code just above the setup() function:

```
// Define an http variable to be of type HttpClient.
HttpClient http;

// We always pass Http headers on each request to the Http Server
// Here, we only define a single header. The NULL, NULL pair is
// used
// to terminate the list of headers.

 // The Content-Type header is used to inform the server
 // of the type of message that it will be receiving. Here,
 // we tell the server to expect to receive data marked up in
 // JSON.

 http_header_t headers[] = {  
    { "Content-Type", "application/json" },  
    { NULL, NULL }   
 };  

 // Here we define structures to hold the request and the response data.
 // These are declared with types defined in the header file included above.

 http_request_t request;  
 http_response_t response;

 //A variable to hold the device unique ID
 String deviceID = "";

```
15. Read over this code and then add it inside the setup() function:

```
// The IP address of the server running on our machine.
// Do not use localhost. The microcontroller would attempt
// to visit itself with localhost
// Check your system to learn its IP address.
request.ip = IPAddress(192,168,86,164);  

// Specify the port that our server is listening on.
// This will be in the browser where Node-RED is running.
request.port = 1880;

// get the unique id of this device as 24 hex characters
deviceID = System.deviceID().c_str();

// display the id to the command line interface
Serial.println(deviceID);  

```
16. Below the setup() function, add a new function designed to display the response data from an HTTP request.

```
// Provided with a response, display it to the command line interface.
 void printResponse(http_response_t &response) {  
   Serial.println("HTTP Response: ");  
   Serial.println(response.status);  
   Serial.println(response.body);  
 }  
```
17. To make the HTTP POST request, we can use the following function:
```
void doPostRequest() {

  // Provide the path to the service
  // The HTTP IN node in Node-RED needs
  // is configured with a URL of microcontrollerLightValue
  request.path = "/microcontrollerLightValue";

  // Build the JSON request
  char json[1000] = "{\"deviceID\":\"";
  // Add the deviceID data
  strcat(json,deviceID);
  // add the closing quotes
  strcat(json,"\"");

  // add the JSON ending
  strcat(json,"}");

  Serial.println(json);
  // assign the JSON string to the HTTP request body
  request.body = json;
  // post the request
  http.post(request, response, headers);
  // show response
  printResponse(response);
}
```
18) The code above should work, producing legal JSON post requests.
Your assignment is to add the analog light value to the JSON string. The string will look like the following:

```
{"deviceID":"e00fce68dfb40ca61243495e","lightReading":67}
```
In order to convert the light level reading to C++ string data, you can use the following code:

```
std::string analogValueString = std::to_string(analogValue);
char const *pchar = analogValueString.c_str();

```
And you can add it to the JSON string like this:

```
strcat(json,pchar);
```
So, your C++ task is to add a JSON label, "lightReading" for this value.

Note that the Node-RED port needs to be specified in the firmware. On my my machine, my Node-RED port is 1880.

Note too that prior to making the changes described in 18, you would be wise to get Node-RED running and receiving the limited post requests. That is, do step 19 first.

19) Run Node-RED and create three nodes: an HTTP IN node, an HTTP out node, and a debug node to view the messages that arrive from the Argon.

### Part 3.

In Part 3 we will experiment with MQTT and the Particle Argon.

Imagine an IoT instructor who wants to call roll quickly. During class, each student has a Wifi
connected Argon. In a few moments, the instructor has a roster of all students present. This roster
is available on a web site accessible by the instructor.

1. 15 Points. Using a Particle Argon, develop firmware that will publish your name and a URL to
an MQTT service. Name this firmware file 'transmitid.ino'. Your name, URL pair will be published to the topic
'student/id'. Use the IDE provided at build.particle.io. To get started making calls to MQTT from an
Argon, use the code provided at Particle. In my solution, I worked from the example called
'mqtttest.ino'. Write your firmware so that your name and URL is published every 5 seconds to the MQTT
service. Note the call to 'client.loop()'. This is an important call to make in order for your client (Argon)
and your server (MQTT) to stay connected. Whenever your Argon sends your name and URL to the
server, it should blink a light. The light will remain on for one second - notifying the student that
her transmission is taking place.

It is important that you place your own comments in transmitid.ino. These comments should convince the
reader that you understand what is going on in the code.

The MQTT library is available by searching the libraries on Particle. You need to include MQTT.h and
MQTT.cpp in your Particle Application. After selecting the library, you have to hit the
'Include In Project' button. Your code will automatically receive the #include<MQTT.h> statement.

The name, URL pair will be transmitted in JSON format. For example, in my solution I am sending
the JSON object { "name":"Michael McCarthy", "URL":"http://www.andrew.cmu.edu/user/mm6"}. You are required
to do the same. For your URL, choose http://www.andrew.cmu.edu/user/your_andrew_ID.

2. 15 Points. Using IntelliJ, build a web project called ArgonMQTTSubscriber. Within this project,
write an index.html file and javascript code that subscribes to your MQTT service. It will subscribe to any
messages published to 'student/id'. When the Argon publishes a name and URL to the topic 'student/id', your
web site will display the name and URL on the browser. You have to plan on many names being published (even
though you only have one Argon to test with). If many names arrive, they should all be listed, as they
arrive, on the browser. In order to do this, your Javascript will create a Set object. When a name
arrives that is not in the set, display it on the browser. If a name arrives that is already in the set, simply ignore it. Test this by changing the name and url coming from your Argon. You will need to flash your firmware with a new name and URL.


### Submission guide

Documentation is required. Spend some time cleaning up your code and adding good comments to
your code.

Create a directory with your Andrew ID as the name. For example, my directory would
be called mm6.

Part 1:
    Place the three projects into the directory.
Part 2:
    Place the four projects into the directory.
Part 3:
    Place the web project and a copy of transmitName.ino into the directory.

Zip the directory. Submit a single zipped file to Canvas.

We will require that you schedule some time during the instructor or TA office hours to demonstrate
your solution. If you have difficulty running Mosquitto on your machine, see us soon. If we are unable
to get Mosquitto running there is a free online version available (Hive MQ). Get started soon and use
Hive if you are not able to run a local broker.

### Optional notes on Using a Remote Broker


If you are unable to install Mosquito on your local machine, I have had success with the following
broker on the cloud:

broker.hivemq.com  for MQTT use port  1883
and for Websockets use port  8000

In my Javascript, I am using a line like this:
```

 var loc = {'hostname' : 'broker.hivemq.com', 'port' : '8000' };
```
In my Java code, I am using a line like this:

```
String broker  = "tcp://broker.hivemq.com:1883";
```

Mosquito is fussy about the client ID. In the following line of code,
I have used my Andrew ID. You should do that as well. In this way, we will
not step on each other.

```
client = new Paho.MQTT.Client(loc.hostname, Number(loc.port), 'mm6');
```

### Optional notes on Installing Mosquito on a  PC


Download mosquitto setup file from [here.](http://www.eclipse.org/downloads/download.php?file=/mosquitto/binary/win32/mosquitto-1.4.10-install-win32.exe)

Run the setup file and follow the steps to complete the installation.
note: Default destination folder is C:\Program Files (x86)\mosquitto

Download and Install Win32 OpenSSL v1.0.2h from [here](http://slproweb.com/products/Win32OpenSSL.html
Install it to C:\temp\OpenSSL-Win32)

Copy libeay32.dll and ssleay32.dll from C:\temp\OpenSSL-Win32\bin to C:\Program Files (x86)\mosquitto

Download pthreadVC2.dll from [here.](ftp://sources.redhat.com/pub/pthreads-win32/dll-latest/dll/x86/ and Copy it to C:\Program Files (x86)\mosquitto)

### Optional notes on PC Testing

open command prompt and run the following commands

```
cd C:\Program Files (x86)\mosquitto

mosquitto.exe -v -c mosquitto.conf
```

You should get messages similar to this
1474406476: mosquitto version 1.4.10 (build date 24/08/2016 21:03:24.73) starting
1474406476: Config loaded from mosquitto.conf.
1474406476: Opening ipv6 listen socket on port 1883.
1474406476: Opening ipv4 listen socket on port 1883.
1474406479: mosquitto version 1.4.10 terminating

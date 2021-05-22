# 95-733 Internet of Things Wednesday, May 26, 2021

## Project 1  Due: 11:59 PM, Wednesday, June 9, 2021

### Part 1: Programming a Particle Argon to report heartbeats to the Particle cloud and Node-RED

#### Start up items

In order to quickly build web sites, we will use Node.js and Express.

In order to route and process IoT messages, we will use Node-RED.

The selected microcontroller that we will use is the Particle Argon. It has the ability to communicate via Bluetooth LE and Wi-Fi. We need to initialize the Argon and establish credentials at Particle. We will use the Particle cloud's over the air (OTA) update capability to flash firmware to the Argon.

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
We will be programming the Argon microcontroller by developing our firmware code in C++, compiling to machine code, and downloading to the device via Particle's over the air (OTA) update.

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
8) ***Study the code above.*** After compiling and deploying the code to your Argon, use a shell and check if the CLI is working properly.

9) The following are useful CLI commands.

```
$particle login          Login to Particle
$particle serial monitor Listen for Serial.print
$particle update-cli     Update the CLI
```
[There are plenty of additional CLI commands.](https://docs.particle.io/tutorials/developer-tools/cli/)

10) Using the Particle console, verify that it is receiving the heartbearts from the Argon.

#### Using Node-RED, subscribe to the heartbeat messages that are being published to particle.

11) Our goal is for Node-RED to subscribe to and receive messages from the Particle console. These messages are being published to the Particle console by our Argon. We want Node-RED to hear about the heart beats. [This article introduces you to Node-RED and the "The Particle Nodes" section specifically is where the reader learns how to connect the Argon to Node-RED. Read about Node-RED and integrate Node-RED with the Particle console.](https://docs.particle.io/community/node-red/)


12) After completing the work in step 11, you should have a Node-RED platform receiving messages from your Argon every 10 seconds or so. Each message should contain a JSON string with the device ID. Each message should appear in the right pane of the Node-RED UI.
The Node-RED palette should have a subscribe node wired to a debug node.

:checkered_flag:**Take a screenshot showing the Node-RED palette. The debug panel on the right will show several JSON strings that have arrived from the Argon. Name your screenshot Project1Part1.png.**

### Part 2: Node-RED Programming

#### Working with flows

0) The objective of this part is to gain skills in the creation and execution of Node-RED flows. We will work with the heartbeat data that we are receiving from our Particle Argon. This is a continuation from our work in Part 1.

[Follow this link for a good place to learn about Node-RED and its capabilities.](https://nodered.org/docs/)

1) Wire a function node in between your subscribe node and your debug node. The function node will add a timestamp to the incoming message. Study this code and use it in your function node:

```
// The message has arrived in the msg object.
// Create a javascript object using the JSON message payload.
var newMessage = JSON.parse(msg.payload);
// Add a time field to the new object.
// Use the current date and time.
newMessage.time = new Date();
// Represent the new object as JSON.
msg.payload = JSON.stringify(newMessage);
// pass it on to the next node
return msg;
```

2) Add another function node that checks the timestamp. If the current message arrived late (after 12 seconds) then set a variable named "onTime" to false, otherwise set the "onTime" variable to true. Include the new "onTime" field and its value in the json message that leaves this node. Note: the very first message to arrive is never late. You can test this flow by unplugging your USB for 5 seconds or so and rebooting your Argon. [In solving this problem, you may find this resource to be helpful.](https://nodered.org/docs/user-guide/writing-functions#storing-data)

:checkered_flag:**Take a screenshot showing the Node-RED palette. The debug panel on the right will show several JSON strings that have arrived from the Argon. Name your screenshot Project1Part2.png.**

### Part 3: Build a web site using Node.js

In this Part you will build as simple web server using Node.js.

0) We need to store our web artifacts in an empty directory. Create an empty directory named Project1_Part3.

1) Within the directory, create a file named ViewSimpleMessage.js with the following content:

```
// ViewSimpleMessage.js
// Display a simple message on a browser
const http = require("http");
const host = 'localhost';
const port = 8000;
// The req variable will hold request information from the browser.
// The res variable is used to send results back to the browser.

const simpleListener = function (req, res) {
    res.writeHead(200);
    res.end("A simple text message on a browser");
};

// Associate the server with the listener
const server = http.createServer(simpleListener);

// Begin handling browser visits
server.listen(port, host, () => {
    // runs when listening begins
    console.log(`Server is running on http://${host}:${port}`);
});

```

2) Run the server from the shell with the command:

```
node ViewSimpleMessage.js

```
3) Test by using a browser to visit http://localhost:8000.

4) Create a file named index.html with the following code:

```

<!DOCTYPE html>

<head>
    <title>Simple message in HTML from your name</title>
</head>

<body>    
        <h1>Simple message in HTML from your name</h1>
</body>

</html>


```
5) Near the top of the ViewSimpleMessage.js file, add another constant to provide access to the file system:

```
const fs = require('fs').promises;

```


6) We need to get the file and send it to the browser on each visit. Replace the code in simpleListener with the following code:

```

fs.readFile(__dirname + "/index.html")
        .then(contents => {
            res.setHeader("Content-Type", "text/html");
            res.writeHead(200);
            res.end(contents);
        })

```

7) Test by using a browser to visit http://localhost:8000.

:checkered_flag:**Take a screenshot showing the browser screen.  This screenshot will show the browser displaying an HTML document. It will include your name. Name your screenshot Project1Part3.png.**


### Part 4: Build a web site using Node.js and Express

In this Part, you will build a simple web site using Node.js and the Express framework.

0) Create an empty directory called Project1_Part4.
1) Within the new directory, run the command:

```
npm init

```
and accept all of the defaults except the entry point. Set the entry point to HelloWorld.js.

2) Within the directory holding the package.json file (created by the previous command), install Express with the command:

```
npm install express --save

```
This command will create a sub-directory named "node_modules". This sub-directory will contain modules that we can use.

3) Create a file named HelloWorld.js and populate it with the code below:

```
// HelloWorld.js
// Simple demonstration using Node.js and Express
// run at the command prompt with
// node HelloWorld.js
// Visit at http://localhost:3000/HelloWorld

const express = require('express');
app = express();
const port = 3000;

app.get('/HelloWorld', (req, res) => {
  console.log("We have a visitor");
  res.send('Hello World From Node.js and Express');
})

app.listen(port, () => {
  console.log(`Example app listening for GET at http://localhost:${port}/HelloWorld`);
})
```

4) Run node and Express with the command:

```
node HelloWorld.js

```
5) Visit the server with a browser and make a standard HTTP GET request to the following URL:

```
http://localhost:3000/HelloWorld

```
:checkered_flag:**Take a screenshot showing the browser screen.  This screenshot will show the browser displaying an HTML document. Name your screenshot Project1Part4.png.**


### Part 5: Configure Node-RED to make calls to a web site

In this Part, you will configure a node in Node-RED to make HTTP calls to a web server developed using Node.js and Express.This is a continuation from our work in Part 2.

0) Create an empty directory called Project1_Part5.
1) Within the new directory, run the command:

```
npm init

```
and accept all of the defaults except the entry point. Set the entry point to viewLastHeartBeat.js.

2) Within the directory holding the package.json file (created by the previous command), install Express with the command:

```
npm install express --save

```
This command will create a sub-directory named "node_modules". This sub-directory will contain modules that we can use.

3) Create a file named viewLastHeartBeat.js and populate it with the code below:

```
// Web site created with viewLastHeartBeat.js.
// Allow Node-RED to visit as well.
// Use Express to abstract away details.
// Get the last heartbeat from Node-RED in the app.post
// method. Record the lastHeartBeat time stamp in
// the lastVisit variable.
// When a browser visits with a GET request, display the
// time of the last visit.
const express = require('express')
const port = 3000
app = express();

// initialize lastVisit
var lastVisit = 0;

// We need to parse the body of the post request
// from Node-RED
var bodyParser = require('body-parser')
// and we need to parse JSON data
app.use(bodyParser.json() );

// Handle a visit from a browser calling with GET.
// return the last visit of Node-RED.
app.get('/ViewLastHeartBeat', (req, res) => {
  console.log('Browser visit for last heartbeat');
  // respond to browser
  res.send('Last time Argon visited via Node-RED ' + lastVisit);
})

// This function is called with an HTTP POST by Node-RED.
// The HTTP request has a content-type header set to
// application/json.
// The JSON data has deviceID, time, and onTime values.
app.post('/SetNewHeartBeat', function (req, res) {
  console.log('Visit from Argon ');
  console.log(req.body);
  console.log(req.body.deviceID)
  lastVisit = req.body.time;
  // respond to Node-RED
  res.send('Argon update received');
})

app.listen(port, () => {
  console.log(`Browser views last heartbeat at http://localhost:${port}/ViewLastHeartBeat`)
})

```
4) At this point, you can test this web service with a browser. The browser will send an HTTP GET request and will execute the app.get() function. Our next problem is to visit this service from Node-RED and have Node-RED send an HTTP POST message with JSON data.

5) Drag an HTTP Request node on to the Node-RED palette. Ensure that the following three properties are set in that node. The "method" property should be set to "-set by msg.method -". The
"URL" property should contain "http://". The "Return" property should contain "a UTF-8 string".

6) In another function node that will run ***prior to*** the HTTP Request node, use the following Javascript.

```
var newMessageObj = JSON.parse(msg.payload);
msg.payload = JSON.stringify(newMessageObj);
msg.method = "POST";
msg.headers = {};
msg.headers['content-type'] = 'application/json';
msg.url= 'http://localhost:3000/SetNewHeartBeat'
return msg;

```
7) Deploy the new flow. Vist the web service with a browser and monitor heartbeats.


:checkered_flag:**Take two screenshots. One will show the browser screen. Name this screenshot Project1Part5Browser.png. The second will show the the Node-RED palette and the debug window with responses coming from the web service. Name this screenshot Project1Part5Node-RED.png. These screenshots should make it clear to the grader that you have a working system.**

### Part 6: Build an AJAX web site using Express and Node-RED

In this Part, you will add AJAX (Asynchronous Javascript and XML) capabilities to the web site developed in Part 5.

0) Create an empty directory named Project1_Part6

### Part 7: Build a websocket web site using Express and Node-RED

In this Part, you will add websockets to the web site developed in Part 5. This Part will not use AJAX.

0) Create an empty directory named Project1_Part7.

1) npm init
2)

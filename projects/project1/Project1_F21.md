# 95-733 Internet of Things Fall 2021

## Project 1  Due: Tuesday, September 14, Fall 2021

:checkered_flag: Submit to Canvas a single .pdf file named Your_Last_Name_First_Name_Project1.pdf. This single pdf will contain your responses to the questions marked with a checkered flag. It is important that you ***clearly label*** each answer with the project part number and that you provide your name and email address at the top of the .pdf.

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

11) Our goal is for Node-RED to subscribe to and receive messages from the Particle console. These messages are being published to the Particle console by our Argon. We want Node-RED to hear about the heart beats. [This article introduces you to Node-RED and the "The Particle Nodes" section specifically is where the reader learns how to connect the Particle Cloud to Node-RED. Read about Node-RED and integrate Node-RED with the Particle Cloud.](https://docs.particle.io/community/node-red/)


12) After completing the work in step 11, you should have a Node-RED platform receiving messages from your Argon every 10 seconds or so. Each message should contain a JSON string with the device ID. Each message should appear in the right pane of the Node-RED UI.
The Node-RED palette should have a subscribe node wired to a debug node.

:checkered_flag:**Take a screenshot showing the Node-RED palette. The debug panel on the right will show several JSON strings that have arrived from the Argon. Name your screenshot Project1Part1.png.**

### Part 2: Node-RED Programming

#### Working with flows

0) The objective of this part is to gain skills in the creation and execution of Node-RED flows. We will work with the heartbeat data that we are receiving from our Particle Argon. This is a continuation from our work in Part 1. [Follow this link for a good place to learn about Node-RED and its capabilities.](https://nodered.org/docs/)

1) Wire a function node in between your subscribe node and your debug node. The function node will add a timestamp to the incoming message. Study this code and use it in your function node:

```
// The message has arrived in the msg object.
// Create a javascript object using the JSON message payload.
// The payload is a simple JSON string and will be parsed
// to create a Javascript object.
var newMessage = JSON.parse(msg.payload);
// Add a time field to the new object.
// Use the current date and time.
newMessage.time = new Date();
// Represent the new object as JSON.
msg.payload = JSON.stringify(newMessage);
// pass it on to the next node
return msg;
```

2) Add another function node that checks the timestamp. If the current message arrived late (after 12 seconds) then set a variable named "onTime" to false, otherwise set the "onTime" variable to true. Include the new "onTime" field and its value in the json message that leaves this node. Note: the very first message to arrive is never late. You can test this flow by unplugging your USB for 10 seconds or so and rebooting your Argon. You will need to think a bit about how to do this programming. Javascript is good at handling dates and it is easy to subtract one date from another. But you will need to research this a bit. [Here is another resource that you might find helpful.](https://nodered.org/docs/user-guide/writing-functions#storing-data)

:checkered_flag:**Take a screenshot showing the Node-RED palette. The debug panel on the right will show several JSON strings that have arrived from the Argon. The debug panel should show messages that were on time and some that were not on time. Name your screenshot Project1Part2.png.**

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


### Part 5: Configure Node-RED to make calls to a web server

We would like for the last heartbeat time to be visible on a browser. Each time the user visits the server (with a browser), the user will be notified of the last heartbeat. If the microcontroller has not visited in the past 12 seconds, the user will see a 'suspected failure' message.

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
4) Run the command "node viewLastHeartBeat". At this point, you can test this web service with a browser. The browser will send an HTTP GET request and will execute the app.get() function. But, we are not yet receiving heartbeats from the microcontroller. Our next problem is to visit this service from Node-RED and have Node-RED send an HTTP POST message with JSON data.

5) Drag an HTTP Request node on to the Node-RED palette - the palette from Part 2. Ensure that the following three properties are set in that node. The "method" property should be set to "-set by msg.method -". The
"URL" property should contain "http://". The "Return" property should contain "a UTF-8 string".

6) Add another function node that will run ***prior to*** the HTTP Request node described in step 5. Use the following Javascript:

```
// prepare for an HTTP call
msg.method = "POST";
msg.headers = {};
msg.headers['content-type'] = 'application/json';
msg.url= 'http://localhost:3000/SetNewHeartBeat'
return msg;
```

7) Deploy the new flow. Visit the web service with a browser and monitor heartbeats.

8) If everything above is working well, it is time to make the appropriate changes to the app.get method so that it responds with a "suspected failure" message if Node-RED has not called in the last 12 seconds. Node-RED may not have called because the microcontroller has not called Node-RED. Or, perhaps, Node-RED is down. Plug and unplug your microcontroller to test this case.

:checkered_flag:**Take three screenshots. The first will show the browser screen with a timestamp of a recent visit. The second will show a "suspected failure" message. Name these screenshots Project1Part5BrowserA.png and Project1Part5BrowserB.png. The third will show the Node-RED palette and the debug window with responses coming from the web service. Name this screenshot Project1Part5Node-RED.png. These screenshots should make it clear to the grader that you have a working system. In addition, provide a copy of viewLastHeartBeat.js - with the modifications made in step 8.**

### Part 6: Build an AJAX web site using Express and Node-RED

Note that in Part 5, in order to see the heartbeat status, we have to perform a "full page refresh". In this part, we want to build a single page application that allows us to visit the service without performing a full page refresh. We want to send a GET request whenever a button is pressed.

In this Part, AJAX (Asynchronous Javascript and XML) capabilities are added to the heartbeat application. An HTML page is sent to the browser and the web page contains Javascript code that will call the Javascript code on the server.

0) Create an empty directory named Project1_Part6.

1) Add a subdirectory named "public". Inside the public directory, create an index.html file. This file will contain HTML as well as references to two Javascript files: Ajax.js and ArgonStatus.js. The index.html file appears next.

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  <script type="text/javascript" language="javascript" src="Ajax.js"></script>
  <script type="text/javascript" language="javascript" src="ArgonStatus.js"></script>
  <style>
  .button {
    border: none;
    color: white;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
  }
.button1 {background-color: #4CAF50;} /* Green */
</style>

</head>
<body>
<button onClick="getStatus()" class="button button1">Get Device Update</button>
<div>
  <h2>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Argon Status using AJAX</h2>
  <table>
    <thead>
      <th>Device ID</th>
      <th>Last Visit</th>
      <th>Device working</th>
    </thead>
    <tbody>
      <tr>
      <td>
        <div id = "deviceID"></div>
      </td>
      <td>
        <div id = "lastVisit"></div>
      </td>
      <td>
        <div id = "onTime"></div>
      </td>
      </tr>
    </tbody>
  </table>
  <script>getStatus()</script>
</body>
</html>

```
2) ArgonStatus.js appears next. Be sure to study this code.

```
// ArgonStatus.js

// A call on getStatus causes an HTTP GET request back to the server.
// The response data is available to the updateStatus function.
function getStatus() {

    var req = newXMLHttpRequest();

    req.onreadystatechange = getReadyStateHandler(req, updateStatus);

    req.open("GET", "getStatusInJSON", true);
    req.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    req.send();
}

// Call back handler to update the HTML
// when a response arrives
function updateStatus(statusJSON) {
    // create an object from the JSON string
    var statusObj =JSON.parse(statusJSON);
    var time = statusObj.lastVisit;
    var deviceID = statusObj.deviceID
    var onTime = statusObj.onTime;
    // place the response data in the HTML
    document.getElementById("lastVisit").innerHTML = time;
    document.getElementById("deviceID").innerHTML = deviceID;
    document.getElementById("onTime").innerHTML = onTime;
}

```
3) Ajax.js provides access to the XHR object in various browsers. Be sure to study this code.

```
// Ajax.js
// Returns an new XMLHttpRequest object, or false if the browser
// doesn't support it

function newXMLHttpRequest() {

    var xmlreq = false;

    // Create XMLHttpRequest object in non-Microsoft browsers
    if (window.XMLHttpRequest) {
        xmlreq = new XMLHttpRequest();

    } else if (window.ActiveXObject) {

        try {
            // Try to create XMLHttpRequest in later versions
            // of Internet Explorer

            xmlreq = new ActiveXObject("Msxml2.XMLHTTP");

        } catch (e1) {

            // Failed to create required ActiveXObject

            try {
                // Try version supported by older versions
                // of Internet Explorer

                xmlreq = new ActiveXObject("Microsoft.XMLHTTP");

            } catch (e2) {

                // Unable to create an XMLHttpRequest by any means
                xmlreq = false;
            }
        }
    }

    return xmlreq;
}

/*
   * Returns a function that waits for the specified XMLHttpRequest
   * to complete, then passes it XML response to the given handler function.
 * req - The XMLHttpRequest whose state is changing
 * responseXmlHandler - Function to pass the XML response to
 */
function getReadyStateHandler(req, responseXmlHandler) {

    // Return an anonymous function that listens to the XMLHttpRequest instance
    return function () {

        // If the request's status is "complete"
        if (req.readyState == 4) {

            // Check that we received a successful response from the server
            if (req.status == 200) {

                // Pass the payload of the response to the handler function.
                responseXmlHandler(req.response);

            } else {

                // An HTTP problem has occurred
                alert("HTTP error "+req.status+": "+req.statusText);
            }
        }
    }
}
```
4) In the directory just above public, in Project1_Part6, store the file named index.js. This file, when run with "node index.js" makes the files in the public directory available to browsers. Before running the index.js server, you will need to run "npm install express" in the Project1_Part6 directory.

```
// index.js
// A directory named "public" is available in the current directory.
// The public directory holds html and javascript files.
// The public directory holds the main page: index.html.
// A browser will visit with http://localhost:3000/index.html

const express = require('express')
const port = 3000
app = express();
// allow access to the files in public
app.use(express.static('public'));

// initialize lastVisit
var lastVisit = "";
var lastVisitDate = new Date();

// initialize deviceID
var deviceID = "";

// initialize onTime
var onTime = true;

var storedDate = new Date();

// We need to parse the body of the post request
// from Node-RED
var bodyParser = require('body-parser')
// and we need to parse JSON data
app.use(bodyParser.json() );

// handle an AJAX visit from a browser
// return the last visit of the Argon
app.get('/getStatusInJSON', (req, res) => {
  console.log('Browser AJAX for last visit time');
  storedDate = new Date(); // current time
  // Logic needed here
  var returnObj = {"lastVisit" : lastVisit, "deviceID" : deviceID, "onTime" : onTime};
  // respond to browser
  res.send(JSON.stringify(returnObj));
})

// Called with an HTTP POST by Node-RED
// The HTTP request has a content-type header set to
// application/json.
// The JSON data has deviceID, time, and onTime values.
app.post('/SetNewHeartBeat', function (req, res) {
  console.log('Visit from Argon ');
  console.log(req.body);
  console.log(req.body.deviceID)
  lastVisit = req.body.time;
  deviceID = req.body.deviceID;
  lastVisitDate = new Date(lastVisit);
  onTime = req.body.onTime;
  console.log("On time == " + onTime);

  // respond to Node-RED
  res.send('Argon visit')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}/index.html`)
})
```
5) The above code should work but we are only reporting the failure status as viewed by Node-RED and Node-RED only updates the service when a message arrives from the microcontroller. Make the necessary modifications to index.js so that it displays "Suspected Failure" if the microcontroller (or Node-RED) has not been seen in 12 seconds or more.

:checkered_flag:**Take three screenshots. The first will show the browser screen with a button and a timestamp of a recent visit. The second will show a "suspected failure" message. Name these screenshots Project1Part6BrowserA.png and Project1Part6BrowserB.png. The third will show the Node-RED palette and the debug window with responses coming from the web service. Name this screenshot Project1Part5Node-RED.png. These screenshots should make it clear to the grader that you have a working system. In addition, provide a copy of the modified index.js - with the suspected failure logic described in step 5.**

### Part 7: Build a websocket web site using Express and Node-RED

In this Part, you will add WebSockets to our heartbeat application. This Part will not use AJAX. Instead, we are looking for real time updates from the service. The user of the browser will be able to sit back and view the updates.

0) Create an empty directory named Project1_Part7.

1) Create two subdirectories. One will be named "UsedByHTML" and the other will be named "HTMLClient".

2) In the HTMLClient directory, create a file named index.html with the following content:

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  <script type="text/javascript" language="javascript" src="websocket.js"></script>
  <script type="text/javascript" language="javascript" src="microcontrollerstatus.js"></script>
</head>
<body>
<div style="float: left; width: 500px">
  <h2>Microcontroller Status Page</h2>
  <ul id="output">
  </ul>
</body>
</html>
```
3) In the HTMLClient directory, run the following commands:

```
npm init -y
npm install express

```
4) In the HTMLClient directory, create a file named server.js with the content shown next. This server simply provides a browser with access to index.html, websocket.js, and microcontrollerstatus.js.

```
// server.js

const express = require('express');
const path = require('path');

const app = express();
const port = 8080;

app.get('/', function(req, res) {
  res.sendFile(path.join(__dirname, '/index.html'));
});

app.get('/websocket.js', function(req, res) {
  res.sendFile(path.join(__dirname, '/websocket.js'));
});

app.get('/microcontrollerstatus.js', function(req, res) {
  res.sendFile(path.join(__dirname, '/microcontrollerstatus.js'));
});
app.listen(port);
console.log('Web server started at http://localhost:' + port);
```
5) In the HTMLClient directory, create the two Javascript files that will be loaded into the browser. These files are websocket.js and microcontrollerstatus.js.

websocket.js

```
// Open a new WebSocket connection (ws), use wss for secure connection
var wsUri = 'ws://localhost:6969';
var websocket = new WebSocket(wsUri);

//Optional callback, invoked if a connection error has occurred
websocket.onerror = function(evt) { onError(evt) };
function onError(evt) {
    //writeToScreen('<span style="color: red;">ERROR:</span> ' + evt.data);
}
// Optional callback, invoked when the connection is terminated
websocket.onclose = function() { alert('Connection closed'); }

// For testing purposes
// var output = document.getElementById("output");

// Optional callback, invoked when a WebSocket connection is established
websocket.onopen = function(evt) {
    onOpen(evt)
};
function onOpen() {
    // writeToScreen("Connected to " + wsUri);
}

/* optional
function writeToScreen(message) {
    output.innerHTML += message + "<br>";
}
*/
// A callback function invoked for each new message from the server
websocket.onmessage = function(evt) { onMessage(evt) };
function onMessage(evt) {
    console.log("received: " + evt.data);
    updateStatus(evt.data);
}

// Client-initiated send text to the websocket
function sendText(msg) {
    console.log("sending text: " + msg);
    websocket.send(msg);
}

```
microcontrollerstatus.js

```
// Called by websocket.js to update the id and time
function updateStatus(msg) {
    // parse json message to create an object
    var msgObj = JSON.parse(msg);
    // get the deviceID
    var id = msgObj.deviceID;
    // get the time
    var time = msgObj.time;
    // find location in document to write
    var contents = document.getElementById("output");
    // clear the existing output
    contents.innerHTML = "";
    // create a new list item tag
    var listItem = document.createElement("li");
    // add the tag with status data
    listItem.appendChild(document.createTextNode(id+" arrived at "+time));
    // place it in the document
    contents.appendChild(listItem);
    }
```

6) You should now be able to run the server provided in the HTMLClient directory. However, it will not be able to connect to a websocket and will generate an error in a pop up window when the browser visits http://localhost:8080.

```
node server.js

```
7) We need to have a WebSocket service for our browser to connect to. Go into the subdirectory named "UsedByHTML" created back in step 1.

8) Create the file server.js with the following content:

```
// server.js
const express = require('express');
const http = require('http');
const WebSocket = require('ws');

const port = 6969;
const server = http.createServer(express);
const wss = new WebSocket.Server({ server })

wss.on('connection', function connection(ws) {
  console.log("Connection established");
  ws.on('message', function incoming(data) {
    wss.clients.forEach(function each(client) {
      if (client !== ws && client.readyState === WebSocket.OPEN) {
        client.send(data);
      }
    })
  })
})

server.listen(port, function() {
  console.log(`Server is listening on ${port}!`)
})

```
9) Inside the subdirectory named UsedByHTML, run the following commands:

```
npm init -y
npm install websocket
npm install ws
npm install express
npm install http
node server.js

```

10) We should now be able to run our browser and visit  http://localhost:8080 without an error. The web page should connect to the bidirectional web socket on port 6969. But, we are missing the updates from Node-RED. We need to add a WebSocket node to the Node-RED palette so that the microcontroller's updates arrive.

11) In Node-RED, disable the HTTP Request node and the function node that prepares the HTTP request. Add a new WebSocket out node named "Node-RED to WebSocket". The type property should be "Connect to". The url field should be ws://localhost:6969 (note, we are using the "ws" scheme and not the "http" scheme).

12) Vist the web site at http://localhost:8080 and heartbeats should appear on the Microcontroller Status Page as they arrive.

13) Make the necessary modifications so that we we receive the message "Suspected Failure" if we hear nothing from Node-RED.

:checkered_flag:**Take three screenshots. The first will show the browser screen with the Microcontroller Status page and a timestamp of a recent visit. The second will show a "suspected failure" message. Name these screenshots Project1Part7BrowserA.png and Project1Part7BrowserB.png. The third will show the Node-RED palette and the debug window. Name this screenshot Project1Part7Node-RED.png. These screenshots should make it clear to the grader that you have a working system. In addition, provide a copy of the modified server.js - with the suspected failure logic described in step 13.**

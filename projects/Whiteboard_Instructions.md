# 95-733 Internet of Things Demonstration Code

## This is a Whiteboard websocket example from Oracle.

The objective of this exercise is to demonstrate real time browser server
interaction via web sockets. Web sockets are often used in IoT and we will
use web sockets in an IoT setting later in the course. Here we step through the
process of building a web application that uses web sockets and learn how
to properly install the client and server side code in IntelliJ.


Each browser that visits acts like a publisher - publishing events to the server.
Each browser that visits also acts as a subscriber - subscribing to messages published
by other browsers.

Notes:

The browser will first visit http://localhost:8080/WhiteboardAppProject_war_exploded/

The websocket uses ws://localhost:8080/WhiteboardAppProject_war_exploded/whiteboardendpoint

## Steps:

1) Create a new IntelliJ project.

2) Choose Java Enterprise

3) Under Additional Libraries and Frameworks select these two options:

   Web Application

   WebSocket
      We select "Use Library". The error in "Use Library:" is removed by creating
and browsing to your TomEE Installation/lib/WebSocketAPI.jar

4) Name the project WhiteboardAppProject

5) Create a new package under src and give it the name

6) In the package, create the following classes:
   Coordinates.java
   Figure.java
   FigureDecoder.java
   FigureEncoder.java
   MyWhiteboard.java

7) Copy in this code to each of these files. But be sure to include the package name at
   the very top.

```
   // Coordinates.java
   public class Coordinates {
        private float x;
    private float y;

    public Coordinates() {
    }

    public Coordinates(float x, float y) {
        this.x = x;
        this.y = y;
    }

    public float getX() {
        return x;
    }

    public void setX(float x) {
        this.x = x;
    }

    public float getY() {
        return y;
    }

    public void setY(float y) {
        this.y = y;
    }
}

```
```

// Figure.java
import java.io.StringWriter;
import javax.json.Json;
import javax.json.JsonObject;

public class Figure {
    private JsonObject json;

    public Figure() {
    }

    @Override
    public String toString() {
        StringWriter writer = new StringWriter();
        Json.createWriter(writer).write(json);
        return writer.toString();
    }

    public Figure(JsonObject json) {
        this.json = json;
    }

    public JsonObject getJson() {
        return json;
    }

    public void setJson(JsonObject json) {
        this.json = json;
    }
}

```
```
// FigureDecoder.java

import java.io.StringReader;
import javax.json.Json;
import javax.json.JsonException;
import javax.json.JsonObject;
import javax.websocket.DecodeException;
import javax.websocket.Decoder;
import javax.websocket.EndpointConfig;


public class FigureDecoder implements Decoder.Text<Figure> {

    @Override
    public Figure decode(String string) throws DecodeException {
        System.out.println("decoding: " + string);
        JsonObject jsonObject = Json.createReader(new StringReader(string)).readObject();
        return new Figure(jsonObject);
    }

    @Override
    public boolean willDecode(String string) {
        try {
            Json.createReader(new StringReader(string)).readObject();
            return true;
        } catch (JsonException ex) {
            ex.printStackTrace();
            return false;
        }
    }

    @Override
    public void init(EndpointConfig config) {
        System.out.println("init");
    }

    @Override
    public void destroy() {
        System.out.println("destroy");
    }

}
```
```
// FigureEncoder.java

import javax.websocket.EncodeException;
import javax.websocket.Encoder;
import javax.websocket.EndpointConfig;


public class FigureEncoder implements Encoder.Text<Figure> {

    @Override
    public String encode(Figure figure) throws EncodeException {
        return figure.getJson().toString();
    }

    @Override
    public void init(EndpointConfig config) {
        System.out.println("init");
    }

    @Override
    public void destroy() {
        System.out.println("destroy");
    }

}
```
```
// MyWhiteboard.java

import java.io.IOException;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import javax.websocket.EncodeException;
import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;


@ServerEndpoint(value="/whiteboardendpoint", encoders = {FigureEncoder.class}, decoders = {FigureDecoder.class})
public class MyWhiteboard {

    // create a synchronized (thread safe) set of sessions in a hashset
    // names this collection peers
    private static Set<Session> peers = Collections.synchronizedSet(new HashSet<Session>());


    // when the web socket is created, save the associated session
    @OnOpen
    public void onOpen (Session peer) {
        peers.add(peer);
        System.out.println("Added a peer");
    }

    // when a message arrives over the socket, call the peers
    @OnMessage
    public void broadcastFigure(Figure figure, Session session) throws IOException, EncodeException {
        System.out.println("broadcastFigure: " + figure);

        // for each peer in the peers set of sessions
        for (Session peer : peers) {
            System.out.println("In broadcast loop");
            if (!peer.equals(session)) {
                // this is not being sent back to the caller
                System.out.println("Sending object to peer");
                // getBasicRemote returns a reference to a RemoteEndpoint object representing the peer of
                // this conversation that is able to send messages synchronously to the peer.
                // Here, the encoder is called on figure
                peer.getBasicRemote().sendObject(figure);
            }
        }
    }
    @OnClose
    public void onClose (Session peer) {
        System.out.println("Closing with @OnClose()");
        peers.remove(peer);
    }
}
```

8) Delete the index.jsp under the web folder. Right click the web folder and include the HTML file
   index.html
```
  <!DOCTYPE html>
<html>
<head>
    <title>Whiteboard using WebSockets</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
</head>
<body>
<h1>Collaborative Whiteboard App</h1>
<span id="output"></span>
<table>
    <tr>
        <td>
            <canvas id="myCanvas" width="150" height="150" style="border:1px solid #000000;"></canvas>
        </td>
        <td>
            <form name="inputForm">
                <table>
                    <tr>
                        <th>Color</th>
                        <td><input type="radio" name="color" value="#FF0000" checked="true">Red</td>
                        <td><input type="radio" name="color" value="#0000FF">Blue</td>
                        <td><input type="radio" name="color" value="#FF9900">Orange</td>
                        <td><input type="radio" name="color" value="#33CC33">Green</td>
                    </tr>

                    <tr>
                        <th>Shape</th>
                        <td><input type="radio" name="shape" value="square" checked="true">Square</td>
                        <td><input type="radio" name="shape" value="circle">Circle</td>
                        <td> </td>
                        <td> </td>
                    </tr>

                </table>
            </form>
        </td>
    </tr>
</table>
<script type="text/javascript" src="websocket.js"></script>
<script type="text/javascript" src="whiteboard.js"></script>
</body>
</html>
```

9) Right click the web folder and include the Javascript file websocket.js

```
// initiate the handshake
// Open a new WebSocket connection (ws), use wss for secure connection
var wsUri = "ws://" + document.location.host + document.location.pathname + "whiteboardendpoint";
var websocket = new WebSocket(wsUri);

//Optional callback, invoked if a connection error has occurred
websocket.onerror = function(evt) { onError(evt) };
function onError(evt) {
    writeToScreen('<span style="color: red;">ERROR:</span> ' + evt.data);
}
// Optional callback, invoked when the connection is terminated
websocket.onclose = function() { alert('Connection closed'); }

// For testing purposes
var output = document.getElementById("output");

// Optional callback, invoked when a WebSocket connection is established
websocket.onopen = function(evt) {
    onOpen(evt)
};
function onOpen() {
    writeToScreen("Connected to " + wsUri);
}

function writeToScreen(message) {
    output.innerHTML += message + "<br>";
}


// A callback function invoked for each new message from the server
websocket.onmessage = function(evt) { onMessage(evt) };
function onMessage(evt) {
    console.log("received: " + evt.data);
    drawImageText(evt.data);
}

function sendText(json) {
    console.log("sending text: " + json);
    //Client-initiated message to the server
    websocket.send(json);
}

```

10) Right click the web folder and include the Javascript file whiteboard.js

```
// find myCanvas in HTML document
var canvas = document.getElementById("myCanvas");

// get a context object providing methods to draw
var context = canvas.getContext("2d");

// if a click event occurs to the canvas, call defineImage function
canvas.addEventListener("click", defineImage, false);


// On event, get the position in the canvas of the mouse
function getCurrentPos(evt) {
    var rect = canvas.getBoundingClientRect();
    return {
        x: evt.clientX - rect.left,
        y: evt.clientY - rect.top
    };
}

// Determines the selected color, shape and location of the mouse click.
// Creates a JSON representation. Draws that and sends it to the server.

function defineImage(evt) {

    // where did the click occur
    var currentPos = getCurrentPos(evt);

    // find the color checked
    for (i = 0; i < document.inputForm.color.length; i++) {
        if (document.inputForm.color[i].checked) {
            var color = document.inputForm.color[i];
            break;
        }
    }

    // find the shape checked
    for (i = 0; i < document.inputForm.shape.length; i++) {
        if (document.inputForm.shape[i].checked) {
            var shape = document.inputForm.shape[i];
            break;
        }
    }

    // create a string from a json object from shape and color and position
    var json = JSON.stringify({
        "shape": shape.value,
        "color": color.value,
        "coords": {
            "x": currentPos.x,
            "y": currentPos.y
        }
    });
    // draw it here locally
    drawImageText(json);
    // draw it everywhere
    // send a json string to the server
    sendText(json);
}


// Takes a json string and draws it
function drawImageText(image) {
    console.log("drawImageText");
    var json = JSON.parse(image);
    context.fillStyle = json.color;
    switch (json.shape) {
        case "circle":
            context.beginPath();
            context.arc(json.coords.x, json.coords.y, 5, 0, 2 * Math.PI, false);
            context.fill();
            break;
        case "square":
        default:
            context.fillRect(json.coords.x, json.coords.y, 10, 10);
            break;
    }
}
```

11) Click on the green triangle and experiment by visiting with different browsers.

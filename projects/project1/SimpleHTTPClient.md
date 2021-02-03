# 95-733 Internet of Things Demonstration Code

## This is code for a Particle Argon to visit a web application.

This is C++ code. It needs to be compiled and the compiled code
will execute on a Particle Argon micro controller. Use the Particle
web IDE to compile this code and perform an over the air deployment
to your Argon.

This is a simple HTTP client that makes periodic visits to an HTTP
server. The periodic visits act like heart beats to inform the server
that the device is still alive.

To view debug messages, enter this line in your terminal:
```
particle serial monitor
```
```
/*
Author: mm6 with various code snippets taken from Particle.io.
This firmware generates periodic heartbeats from an Argon.
*/

// Setting DEBUG == true generates debugging output to a shell.
// To view these messages, install the Particle CLI and enter the command:
// particle serial monitor

boolean DEBUG = true;

// Establish the number of seconds to wait until calling the server.
int NUMSECONDS = 10;

// Define a char array to hold response data.
char reply[512];

// Use the TCPClient to write HTTP request
TCPClient client;

// We need the location of waiting server.
// Using localhost would be a mistake.
// The Argon treats localhost as itself.

// Examine your own machine to find its IP address.

byte serverIP[] = { 192, 168, 86, 164 };


// Establish the server port for the web application.
int port = 8080;

// We only want to visit every 10 seconds or so.
int timeCtr = 0;



// Device ID will be stored here after being retrieved from the device.
// This is a unique, 96 bit identifier.
// This looks like the following: 0x3d002d000cf7353536383631.

String deviceID = "";

void setup() {
    // to allow for debug using the CLI
    Serial.begin(9600);

    // get the unique id of this device as 24 hex characters
   deviceID = System.deviceID().c_str();

   // display the id to the command line interface
   Serial.println(deviceID);

}

// Message to send to server
String msg = "";

void loop() {
     // If timeCtr is above the current time wait until the current time catches up
     if (timeCtr <= millis()) {

           if(DEBUG) Serial.println("Connecting to web server: ");
           if (client.connect(serverIP, port)) {
               if(DEBUG) Serial.println("Connected...");
               out("POST /ArgonTracker/ArgonServlet HTTP/1.1\r\n");
               out("Host: mylocalserver:8080\r\n");
               out("User-Agent: Argon/1.0\r\n");
               out("Content-Type: application/x-www-form-urlencoded\r\n");
               out("Connection: close\r\n");
               // send the device ID to the server
               msg = "Argon_ID=" + deviceID;
               String length = String( msg.length() );
               out("Content-length: " + length + "\r\n\r\n"); // Blank line after POST headers
               out(msg);
               if(DEBUG) Serial.println("Data sent to server\n" + msg + "Size == " + msg.length());
               if(DEBUG) Serial.println("Closing Connection...\n");
               in(reply, 3000);
            }
            else {  // server not found to be available
               Serial.println("Connection failed");
               Serial.println("Disconnecting.");
               client.stop();
            }
            // set timeCtr to current time plus NUMSECONDS seconds
            timeCtr = millis() + (NUMSECONDS * 1000);
     }
}


/* Write a string using tcpclient */
void out(const char *s) {

    client.write( (const uint8_t*)s, strlen(s) );

    //if (DEBUG)Serial.write( (const uint8_t*)s, strlen(s) );

}

/* Read a reply using timeouts. */
/* The reply data is left in a char array being pointed to by ptr. */

void in(char *ptr, uint8_t timeout) {
        int pos = 0;
        unsigned long lastTime = millis();

        while( client.available()==0 && millis()-lastTime<timeout) { //timeout

            //if(DEBUG) Serial.print("Waiting for response data");
        }  //do nothing

        // data has arrived
        lastTime = millis();
        unsigned long lastdata = millis();

        while ( client.available() || (millis()-lastdata < 500)) {  //500 millisecond timeout
            if (client.available()) {
                char c = client.read();
                // show resonse if debug is true
                if(DEBUG)Serial.print(c);
                lastdata = millis();
                ptr[pos] = c;
                pos++;
            }
            if (pos >= 512 - 1)
            break;
        }
        ptr[pos] = '\0'; //null terminate the char array
        while (client.available()) client.read(); // read and drop any extra data
        client.flush();
        delay(400);
        client.stop();
        if(DEBUG){
            Serial.print("Done, Total bytes: ");
            Serial.println(pos);
        }

}

```

# 95-733 Internet of Things    Tuesday, September 15, 2020

## Project 2 Due Date: 11:59 PM Tuesday, September 29, 2020

### Topics: MQTT, Google Charts, Particle Photon  


The internet of things (IoT) often collects real time data from sensors and
transmits these data to a service offering publish and subscribe capabilities. The service
is usually called a broker. The sensor acts as a publisher. The sensor has no idea who
its subscribers are. It simply transmits data to the broker. The subscriber subscribes
to messages from the broker and, in this way, is able to read data from the sensors.
The publish subscribe approach exemplifies loose coupling and provides for scalability.

The publish subscribe design pattern is well established in the internet of things.

In the section where you work with the Particle Photon and MQTT, you may need to turn
your fire wall off and then reboot your machine. Firewalls can be sticky and may only
appear to be off. Turn your fire wall back on after completing this assignment.

### Objectives

One objective of this project is to learn to use an MQTT publish subscribe broker
named Mosquitto. MQTT is targeted at low power constrained devices. It will likely play an
important role in a mature IoT. In class, we will discuss how some big players are
using MQTT.

The second objective is to learn Google Charts. The internet of things will rely
heavily on real time visualization in the browser. Google Charts may play a significant
role in this area.

The third objective is to continue our work with the Particle Photon microcontroller.

### Overview and setup


This is a three part assignment. The first part asks that you develop a small
system using Javascript and an MQTT broker. The second part asks that you
build a small system using a Java program as a publisher to MQTT and a
Javascript program as a subscriber to messages coming from MQTT. The third
part asks that you create a simple application that will be deployed (flashed)
to a Particle Photon. This application will make good use of MQTT.

In Project 1, we used a whiteboard application from Oracle to build an
application that allowed us to collaborate in real time without polling. We leveraged
HTML WebSockets and wrote our own server side code that sent a message to all listeners.

In Parts 1 and 2 of this project, we will not write our own server side code. Instead,
we will use an open source MQTT broker called Mosquitto. Mosquitto can be downloaded
[from here](http://mosquitto.org/download/).

Note: In the past, some students had trouble with Mosquito but had an easy time
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
```

My copy of mosquitto is located at /usr/local/sbin. If I change to that
directory, I can run mosquitto (picking up the configuration file) with the command

```
mosquitto -c /usr/local/etc/mosquitto/mosquitto.conf -v

```

Once you have Mosquitto running, we want to connect to the server from Javascript running
in a browser. We do not want to write the client side MQTT code ourselves. Even though we have
easy access to WebSockets, it would be far better to use an existing Javascript  library
that provides a convenient API to MQTT. The implementation of the API will, of course,
use Websockets below the scenes.

In our web pages, we will include the Javascript library from Paho. It is called mqttws31.js.
There is a very nice getting started section at the following link. It has a simple example of a
Javascript client that will run in a web page. It is very useful for us and is found [here.](https://eclipse.org/paho/clients/js/)

The mqttws31.js Javascript library can be included in your HTML file with this script tag:

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.js" type="text/javascript"></script>
```

There is a nice discussion [here on MQTT.](https://developer.ibm.com/articles/iot-mqtt-why-good-for-iot/)

### Part 1.


Your Part 1 we will experiment using real time data on the web.

1. 5 Points. Build a IntelliJ project named Project2Basic with the code found [here.](http://www.w3schools.com/jsref/tryit.asp?filename=tryjsref_onmousemove)
Note that every time the mouse moves (within the box) an event is generated
and the x and y coordinates are displayed. It is required that you add your
own detailed comments to this code - explaining clearly how it works. The grader
will grade based on the quality of these comments.

2. 15 Points. Here we want to sense the mouse movements and pass them along to Mosquitto running
   MQTT. Copy Project2Basic (along with your comments) to a new IntelliJ project called
   MouseTrackerPublishSubscribe. Your first task is to write a jsp file (named publish.jsp)
   that publishes each new mouse coordinate pair to your MQTT broker.
   Your solution must make good use of the Javascript library - mqttws31.js.

   Note: Copying projects can be done in IntelliJ by creating the new project
   and copying files one at a time.

3. 15 Points. Add a new JSP file named subscriber.jsp to your existing
   project. It too will make use of mqttws31.js. Your subscriber.jsp will
   subscribe to events published to Mosquitto. It will simply display the coordinates
   being received (there will be no box as is shown in publish.jsp).

   Note: In my solution, I created two .jsp files in the same project. One named
   publisher.jsp and another named subscriber.jsp. I added a welcome-file-list element
   to my web.xml and specified one of the .jsp files (publish.jsp) as the initial file
   to load when the browser visits.

   Note: MQTT is very fussy about client names. Each client that visits must present a
   unique name. In this part of Project 2, we need two names - one for the publisher
   and another for the subscriber.

### Part 2.

In Part 2, we will be communicating with Mosquitto in two ways. We will use Websockets from within
our Javascript code - running within a browser. And we will also use standard TCP sockets (and a client
library) from within a stand-alone Java program. In order to make this happen, we need to
ensure that the Mosquitto configuration includes the following four lines. You will need to
carefully use the appropriate ports within your code.

```
listener 1883
protocol mqtt
listener 9002
protocol websockets
```

For the Java client, we will be using the Eclipse Paho Java Client libraries.
This is very easy to do using Maven and IntelliJ. In short, Maven is an opinionated build tool.
That is, it is based around the idea of "convention over configuration". There is a nice 5 minute
video here on using Maven on the IntelliJ IDE [here.](https://www.youtube.com/watch?v=pt3uB0sd5kY)

The following URL has the repository definition and the dependency definition needed for your pom.xml.
It also has a very nice sample Java client that you will want to work from. You should get this code to
compile using IntelliJ and Maven. The code is found [here.](https://eclipse.org/paho/clients/java/)


1. 5 Points. Build a IntelliJ project named TemperatureSensorSimulatorProject. This project will hold a stand alone
Java program named TemperatureSensor.java. Every 5 seconds, TemperatureSensor.java will generate a
random temperature (uniformly distributed) between the values of 0 and 100 degrees fahrenheit. If the
random temperature is between 0 and 45 degrees inclusive, TemperatureSensor.java sends the temperature
and a timestamp to the topic called "pittsburgh/temperature/coldTemps" on Mosquitto. If the temperature is
greater than 45 degrees but less than or equal to 80 degrees, TemperatureSensor.java sends the temperature
and a timestamp to the topic called "pittsburgh/temperature/niceTemps" on Mosquitto.  If the temperature is greater
than 80 degrees then TemperatureSensor.java sends the temperature and a timestamp to
the topic called "pittsburgh/temperature/hotTemps" on Mosquitto.

    Notes: There is a Maven pom file below that works with IntelliJ.
           Enter the groupId as a normal package name.
           The artifactID is a directory name.
           After copying the pom.xml file below, be sure to "Import Changes" in
           the "Maven projects need to be imported dialog box".
           Click "Add Configuration/+/Application"
           Enter a name and a Java class name/Apply/OK

    Note: A typical execution appears as follows:

```
Connected

Sending a cold temp message to topic pittsburgh/temperature/coldTemps
Publishing message: It is cold out 17 degrees at time: Sun Sep 15 18:34:18 EDT 2019
Message published
Sending a nice temp message to topic pittsburgh/temperature/niceTemps
Publishing message: It is nice out 47 degrees at time: Sun Sep 15 18:34:23 EDT 2019
Message published
Sending a hot temp message to topic pittsburgh/temperature/hotTemps
Publishing message: It is hot out 100 degrees at time: Sun Sep 15 18:34:28 EDT 2019
Message published
Sending a nice temp message to topic pittsburgh/temperature/niceTemps
Publishing message: It is nice out 74 degrees at time: Sun Sep 15 18:34:33 EDT 2019
Message published
Sending a cold temp message to topic pittsburgh/temperature/coldTemps
```


2. 5 Points. Build a IntelliJ web application that allows the user to subscribe to a particular topic
from Mosquitto. The user might be interested in "pittsburgh/temperature/coldTemps", "pittsburgh/temperature/niceTemps",
"pittsburgh/temperature/hotTemps", or all temperatures in Pittsburgh. Name the web application project TemperatureSubscriberProject.

This project will require the Javascript library mqttws31.js. To get data on all temperatures, use
topic wildcards. The detailed design of the site is in your hands. It will allow the user to select
from the four possible options and will then show each temperature and timestamp pair published to
that topic.

3. 10 Points. Build a IntelliJ project named DiceRollingClientProject. This project will hold a stand alone
Java program named DiceRollingClient.java. Every one second, DiceRollingClient.java will generate two random
integers between 1 and 6 inclusive (each is uniformly distributed). It will then publish a JSON string to
Mosquito in the following format {"Die1" : Number, "Die2" : Number }. For example, in the first three seconds,
the program might transmit {"Die1" : 3, "Die2" : 1 } {"Die1" : 3, "Die2" : 6 } and
{"Die1" : 2, "Die2" : 4 }.

4. 15 Points. Build a IntelliJ web application project named DiceRollingMonitorProject. The web
application will act as a subscriber to MQTT messages. This project will require the Javascript
library mqttws31.js. It will also make good use of Google Charts to display the following on a
browser:

    a) Two Google Chart gauges. One showing the result of the first die roll and the other showing the
       result of the second die roll. These gauges will display the integers 1 through 6 inclusive. The
       pointer on each gauge will jump from one integer to the next upon each roll of the dice.

    b) A third Google Chart gauge showing the sum of the values on the two die. It will be numbered
       from 2 to 12 and its pointer will jump among these values on each roll.

    c) A fourth Google Chart gauge showing the average sum rolled so far. If the first three rolls are
       (1,1), (3,1), and (1,2) then this gauge would display a 3. That is, (2 + 4 + 3)/3 = 3. This gauge
       will be numbered from 2 to 12 but will display the average as a real number. If the first two rolls
       are (4,5) and (1,1) then this gauge will display the value 5.5 and the pointer on the gauge will
       point midway between 5 and 6 on the gauge.

    d) The last monitoring device that you will display is a Google Chart Line Chart. The Y-axis will
       run from 0 to 1.0 and the X-axis will run from 2 to 12. It will display a line representing the
       relative frequency of each possible sum. Suppose the first 11 rolls result in (1,1), (2,1), (3,1)
       (3,2), (2,4), (4,3), (6,2), (3,6), (5,5), (6,5), and (6,6). The line chart would show a straight
       line running above 2 through 12 at a constant height of 1/11 on the y-axis. This line graph will
       change as the dice rolls are pushed to the MQTT subscriber.

### Part 3.

In Part 3 we will experiment with MQTT and the Particle Photon.

Imagine an IoT instructor who wants to call roll quickly. During class, each student has a Wifi
connected Photon. In a few moments, the instructor has a roster of all students present. This roster
is available on a web site accessible by the instructor.

1. 15 Points. Using a Particle Photon, develop firmware that will publish your name and a URL to
an MQTT service. Name this firmware file 'transmitid.ino'. Your name, URL pair will be published to the topic
'student/id'. Use the IDE provided at build.particle.io. To get started making calls to MQTT from
Photon, use the code provided at Particle. In my solution, I worked from the example called
'mqtttest.ino'. Write your firmware so that your name and URL is published every 5 seconds to the MQTT
service. Note the call to 'client.loop()'. This is an important call to make in order for your client (Photon)
and your server (MQTT) to stay connected. Whenever your Photon sends your name and URL to the
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

2. 15 Points. Using IntelliJ, build a web project called PhotonMQTTSubscriber. Within this project,
write an index.html file and javascript code that subscribes to your MQTT service. It will subscribe to any
messages published to 'student/id'. When the Photon publishes a name and URL to the topic 'student/id', your
web site will display the name and URL on the browser. You have to plan on many names being published (even
though you only have one Photon to test with). If many names arrive, they should all be listed, as they
arrive, on the browser. In order to do this, your Javascript will create a Set object. When a name
arrives that is not in the set, display it on the browser. If a name arrives that is already in the set, simply ignore it. Test this by changing the name and url coming from your Photon. You will need to flash your firmware with a new name and URL.


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



An MQTT POM File for Maven
==========================

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>edu.cmu.andrew.mm6</groupId>
<artifactId>TemperatureSensorSimulatorProject</artifactId>
<version>1.0-SNAPSHOT</version>
<packaging>jar</packaging>
<repositories>
    <repository>
        <id>Eclipse Paho Repo</id>
        <url>https://repo.eclipse.org/content/repositories/paho-releases/</url>
    </repository>
</repositories>
<dependencies>
    <dependency>
        <groupId>org.eclipse.paho</groupId>
        <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
        <version>1.0.2</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.hamcrest</groupId>
        <artifactId>hamcrest-core</artifactId>
        <version>1.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
<name>TemperatureSensorProject</name>
</project>
```

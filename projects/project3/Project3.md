# 95-733 Internet of Things           Project 3

## IoT and Webhooks                   Due: Tuesday, June 23, 11:59:00 PM


This project is short and should not take you too long. It represents a very nice
conclusion to our class work and readings. All along the way, it makes use of
open standards (HTTP, Webhooks, SSL, JSON). It leaves the final data at ThingSpeak -
a MatLab enabled environment where data analytics can take place. While we are not
doing data analytics in this project, you should be aware that the sensor data is
available to MatLab related tools.


Goal: Publish data from your Photon to the service provided by Particle.
      Subscribe to that data through another service provided by ThingSpeak.
      View a graphical display of the data as it arrives from your Photon.
      Work with Webhooks, a lightweight implementation of the publish/subscribe
      pattern.

Part 1. Publishing messages from an IoT device to a proxy service.

   Set up your photon so that there is a single resistor connecting D0 to 3V3 as shown
   in "Step 5: digitalRead" of the [Tinker tutorial](https://docs.particle.io/guide/getting-started/tinker/photon/)

   When the wire connects D0 to 3V3 (pins 10 and 21), the value of D0 is high (Note:
   there is no light on the Photon that indicates this). When the wire is disconnected,
   there is no power to D0 and it is low. These values can be detected with the
   Tinker application running on your phone. It is suggested that you play with Tinker and
   see how this works. There is no need to study the Tinker code.

   In this project, we will use the connecting and disconnecting of the wire between
   D0 and 3V3 as a switch. When the wire is connected, the Photon will send out a "1".
   When the wire is disconnected, the Photon will send out a "0".

   Write a firmware program named ReadAndPublishD0.ino that reads the D0 value every 30
   seconds. In your setup function, you will need the statement:
```
         pinMode(D0,INPUT_PULLDOWN);
```
   And in your loop function you will need to read the D0 pin with a call on digitalRead.

   Every 30 seconds, call one of the following functions (depending on the value
   of D0):
```
         Particle.publish("OnOrOffValue", "0")
         Particle.publish("OnOrOffValue", "1")
```
   If D0 is high, pass a "1" to the Particle Cloud. If D0 is low, pass a "0" to Particle
   Cloud. We do not want to go any faster than once every 30 seconds. In this way, the
   Particle Cloud and the Webhook call to Thingspeak can keep up.

   Just before you publish (with a "1" or "0" ) turn on the D7 LED. After publishing,
   wait one second and set D7 back to low. In this way, the user will be able to see
   when the publishing event is occurring.

   Note that you can view these published events in the Particle Console. It is
   highly recommended that you get Part 1 working before going on to Part 2.

Part 2. Using a Webhook.

   In this part, you will configure the Particle cloud service to forward your "0" and
   "1" messages to Thingspeak. This will result in a new HTTP request to Thingspeak
   after each call on the Particle cloud from your Photon. This is called a Webhook. It
   is a simple way to implement publish/subscribe messaging. Your Photon will publish
   a message to the event "OnOrOffValue" and Thingspeak will be called for each
   publication. Thingspeak is a subscriber.

   You must configure a channel in Thingspeak so that it is ready to receive the message
   "OnOrOffValue". You will also need a "write API-Key" from Thingspeak and this key
   needs to be provided to the Webhook on Particle.

   [Create a ThingSpeak account and work through this tutorial](https://docs.particle.io/guide/tools-and-features/webhooks/)

   This tutorial has everything that you need to create the account at Thingspeak
   and prepare the Webhook at Particle. Be sure to use the "form" approach when
   constructing the Webhook.

   Your channel will be named "OnOrOffChannel" and Field1 will be contain "OnOrOffValue".

Submission
==========
a) Turn in a copy of your well-documented firmware ReadAndPublishD0.ino. Include your name
as the author and include a description of the code. On each line of code, explain briefly
what that line is doing.

b) Turn in a screen shot showing the graphical display at Thingspeak. The display will
show a line graph showing the different values (either 0 or 1) that have arrived
from the Particle Cloud. Name the screenshot {your andrew ID}Project3Screen. It must
show at least 5 distinct values (0's and 1's) connected by lines. In addition, your
name will appear in the title of the graph. You can configure this with the tooling at
ThingSpeak.

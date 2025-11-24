# Carnegie Mellon University

# 95-733 Internet of Things Week 5

## Topics

In week 5, two use cases are introduced, Sensor Andrew and OpenChirp. Both of these
are IoT initiatives built at Carnegie Mellon. Sensor Andrew's use of XMPP is highlighted and so is OpenChirp's use of LoRa and LoRaWan.

<!--
The publish/subscribe pattern is revisited using HTTP Webhooks. Thingspeak Webhooks are used in Project 3.

Project 3 also demonstrates how a microcontroller can act as a Bluetooth Low Energy peripheral.
-->

We do three demonstrations in class using a constrained network. First,
we use an Android phone running LightBlue to connect to a Photon 2
running BLE firmware. Second, we connect the Photon 2 to Node-RED using BLE. Third, we add a node to Node-RED that converts the binary data received into a textual representation.

We also discuss service discovery and the REST architectural style. CoAp is introduced as RESTful protocol and three Web of Things integration patterns are highlighted.

The Web of Things Architecture from W3C and Mozilla are introduced.

HTTP and CoAp are compared. It is noted that the microcontroller used in class interacts with the Particle cloud using CoAp.

+ BLE demonstrations
+ Introduction to XMPP
+ Sensor Andrew Architecture
+ Programming XMPP and IoT Discovery
+ Web of Things Integration patterns
+ Constrained Application Protocol(CoAp)

## Slides

+ [Photon 2 BLE Temperature Firmware](https://github.com/sn-code-inside/guide-to-iot/blob/main/Ch09_code/ble-health-temperature.ino)

+ [Node-RED BLE Temperature Function Node From Binary to Text for Temperature](https://github.com/sn-code-inside/guide-to-iot/blob/main/Ch09_code/FromBinaryToText.js)

+ [Webhooks](https://www.andrew.cmu.edu/user/mm6/95-733/PowerPoint/05_WebhooksPublishSubscribe.pdf)


+ [XMPP Brief](https://www.andrew.cmu.edu/user/mm6/95-733/PowerPoint/05_XMPP_Overview.pdf)

<!--
+ [XMPP Programming and Sensor Andrew ](https://www.andrew.cmu.edu/user/mm6/95-733/PowerPoint/05_XMPP.pdf)
-->

+ [REST, Integration Patterns, and CoAP brief](https://www.andrew.cmu.edu/user/mm6/95-733/PowerPoint/05_RESTandCoAP.pdf)


## Required readings

+ [Jonathan Zittrain US Senate Testimony](https://www.andrew.cmu.edu/user/mm6/95-733/iot/Jonathan_Zittrain_Testimony.pdf)

+ [Web of Things Architecture at W3C](https://www.w3.org/TR/wot-architecture/)
+  From WoT Architecture, read Use Cases Section 4, WoT Architecture Section 6, WoT Architecture Building blocks Section 7.
+ [Web of Things Architecture Matthias Kovatsch(W3C) Video Lecture](https://www.youtube.com/watch?v=xgkglOZiF9M)

## Projects

+ [Project 3 BLE, Node-RED, MQTT, and the Web](../projects/project3/Project3.md)

<!--
+ [Project 3 Webhooks and ThingSpeak and BLE ](../projects/project3/Project3.md)
+ [Project 3 Webhooks and ThingSpeak Photon Only ](../projects/project3/Project3_Photon.md)
+ [Project 4 Student defined project - due in one week](../projects/project4/Project4.md)
-->
## Quizzes

+ Quiz 4 on OpenChirp and Sensor Andrew

## Video Lectures
<!--
+ [16_Lecture5](https://heinzcollege.mediasite.com/Mediasite/Play/302ca40f3f3b4a9880d09901989a13721d)
+ [17_Lecture5](https://heinzcollege.mediasite.com/Mediasite/Play/251d1c7c5320408f9ded1c75fc7a53d81d)
-->
## Optional Readings

+ [Node-Red and InfluxDB](https://www.influxdata.com/blog/iot-easy-node-red-influxdb/)

+ [LoRa and LoRaWAN Video](https://youtu.be/6WMzRrmMjQU)

+ [Google Knowledge Graph](https://www.youtube.com/watch?v=mmQl6VGvX-c)

+ [IBMs IOT Knowledge Graph](https://www.youtube.com/watch?v=ebBTdH62yLg)

+ [Google uses JSON-LD embedded in HTML](https://developers.google.com/schemas/formats/json-ld)

+ [Google uses JSON-LD in Knowledge Graph API](https://developers.google.com/knowledge-graph/)

+ [JSON-LD Video Basics(1)](https://www.youtube.com/watch?v=vioCbTo3C-4)

+ [JSON-LD Video Core Markup(2)](https://www.youtube.com/watch?v=UmvWk_TQ30A)

+ [JSON-LD Specification](https://www.w3.org/TR/json-ld/)

+ [From the Internet of Things to the Web of Things: Resource Oriented Architecture and Best Practices](https://www.vs.inf.ethz.ch/publ/papers/dguinard-fromth-2010.pdf)

+ [Web of Things](https://www.w3.org/2017/04/w3c-web-of-things-intro.pdf)

+ [Web of Things Video](https://www.postscapes.com/pulse/web-of-things-the-pursuit-of-interoperability-in-iot/)

+ [Rasberry Pi Demo with Betsy](https://www.youtube.com/watch?v=DPHzm3f2lps)

+ [Google, Yahoo, and Bing support Schema.org](http://schema.org)

+ [TBL and the Semantic Web](http://www.youtube.com/watch?v=HeUrEh-nqtU)

+ [TBL and Linked Data](http://5stardata.info)

+ [Linked Data at Nature](http://data.nature.com)

+ [A List Apart on RDFa](http://www.alistapart.com/articles/introduction-to-rdfa/)

+ [Google's Use of RDFa](http://support.google.com/webmasters/bin/answer.py?hl=en&amp;answer=99170&amp;topic=1088472&amp;ctx=topic)

+ [What is RDF?](http://www.xml.com/pub/a/2001/01/24/rdf.html)

+ [RDFa Primer](http://www.w3.org/TR/xhtml-rdfa-primer/)

# Carnegie Mellon University

# 95-733 Internet of Things

### Review Notes for Final Exam

### Primary article: Enabling the Internet of Things

+ localized scalability
+ physical web
    URI vs. URL vs. URN
    gateway devices
    proxy service
    smartphone interaction - direct and proxy
    closed systems and private APIs
    NFC, QR, BLE
    Schema.org
    AJAX
    Alternative approaches to IoT
        peer-to-peer with gateway
        centralized IoT service
    Infrared beacons and URLs from Cooltown
    Requirements for connecting directly to the internet and
    how that impacts cost and power consumption
    low performance communication standards: ZigBee, 6LoWPAN, Bluetooth Classic
    proximity sharing
    discovery and trust
    edge computing and cloudlets
    the problem with specialized native smart phone apps
    discovery
    ubiquitous information gathering, context sensing, and control
    privacy and security
    associating a Web URI with every person, place and thing
    BLE Beacons and URIs

Four styles of web interaction:
===============================
    Traditional request response
    AJAX  (study example shopping cart)
    JSONP
    WebSockets

Web sockets
===========
Web sockets in Javascript
Websockets in JEE  (encoders and decoders)

Publish subscribe and MQTT
==========================
    Decoupling
    MQTT
    MQTT message format
    MQTT and web sockets
    Durable and non-durable subscriptions
    MQTT last will and testament
    MQTT qualities of service
    MQTT hierarchical namespace

Seminal article: The Computer for the 21st Century, 1991
========================================================
    Tabs, pads, and boards
    ubiquitous computing
    The active badge
    Scrap computers

Primary article: People, Places, Things: Web Presence for the Real World
========================================================================
    HP Cooltown
    Web presence for people, places and things
    URLs for addressing
    URL Beaconing and URL sensing for discovery
    Building on the web
       - low barrier to entry
       - device, language, and service independent protocol
       - content evolution and not interface heterogeneity
       - depth and higher-level content-provision services
       - Ubiquity
    eSquirt
    ID resolution

XMPP
====
    Traditional IM applications
    SMTP architecture
    XMPP architecture
    Publish/Subscribe
    Jabber JID
    XML document transfer in XMPP
    <stream>, <Presence>, <IQ>, <Message>
    XML Schema

Be sure to understand:
======================
    OpenChirp
    LoRaWAN
    LPWAN
    LoRaBug


Microcontrollers and PAN Protocols
=================================
    Smart connected products changing established industries
    What is a micro controller?
    Capabilities of smart connected products
    Monitoring, Control, Optimization, Autonomy
    EnOcean, ZigBee, Thread, Bluetooth, Wi-Fi
    802.15.4 and 6LoWPAN


Google Charts
=============
     Google Charts basics

Photon
======
     What is firmware?
     How did our Photon communicate with our servlets?
     Be able to read and interpret a simple firmware application.

Review Project 1
================
     AJAX
     JSON
     Websockets

Review Project 2
================
     MouseTracker Publish/Subscribe using MQTT
     Java TemperatureSensor using MQTT and MQTT topics
     Java DiceRolling using MQTT and GoogleCharts
     Photon publisher sending a name to MQTT

Industrial Internet of Things (IIOT)
====================================

     SCADA
     PLC

CoAP and REST
=============
     Direct Integration Pattern
     Gateway Integration Pattern
     Cloud Integration Pattern
     REST design principles
     Constrained Application Protocol (CoAP)
     Compare CoAP with HTTP interaction
     How does CoAP provide reliability over UDP?
     CoAP Request/Acknowledge/Callback
     CoAP publish subscribe
     CoAP resource discovery

Review Project 3
================
Webhooks
Particle Cloud
Thing Speak

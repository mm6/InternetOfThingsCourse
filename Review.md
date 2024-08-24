# Carnegie Mellon University

# 95-733 Internet of Things

### Review Notes for Final Exam (subject to change)

### Primary article: Enabling the Internet of Things

+ localized scalability
+ physical web
+ URI vs. URL vs. URN
+ gateway devices
+ proxy service
+ smartphone interaction - direct and proxy
+ closed systems and private APIs
+ NFC, QR Codes, and BLE
+ Schema.org
+ AJAX
+ Alternative approaches to IoT
    + peer-to-peer with gateway
    + centralized IoT service
+ Infrared beacons and URLs from Cooltown
+ Requirements for connecting directly to the internet and how that impacts cost and power consumption
+ low performance communication standards: ZigBee, 6LoWPAN, Bluetooth Classic
+ proximity sharing
+ discovery and trust
+ edge computing and cloudlets
+ the problem with specialized native smart phone apps
+ discovery
+ ubiquitous information gathering, context sensing, and control
+ privacy and security
+ associating a Web URI with every person, place and thing
+ BLE Beacons and URIs

### Four styles of web interaction using HTTP lecture notes

+ Traditional request response
+ AJAX interactions
+ WebSockets

### Web sockets lecture notes

+ Web sockets in Javascript
+ Websockets in JEE  (encoders and decoders)

### The publish subscribe pattern and MQTT lecture notes

+ Decoupling
+ MQTT
+ MQTT message format
+ MQTT and web sockets
+ Durable and non-durable subscriptions
+ MQTT last will and testament
+ MQTT qualities of service
+ MQTT hierarchical namespace

### Seminal article: The Computer for the 21st Century, 1991

+ Tabs, pads, and boards
+ Ubiquitous computing
+ The active badge
+ Scrap computers
+ Example use case

### Primary article: People, Places, Things: Web Presence for the Real World

+ HP Cooltown
+ Web presence for people, places and things
+ URLs for addressing
+ URL Beaconing and URL sensing for discovery
+ Building on the web
    + low barrier to entry
    + device, language, and service independent protocol
    + content evolution and not interface heterogeneity
    + depth and higher-level content-provision services
    + Ubiquity
    + eSquirt
    + ID resolution

### Primary article: Sensor Andrew at Carnegie Mellon

+ Traditional IM applications
+ SMTP architecture
+ XMPP architecture
+ Publish/Subscribe messaging
+ Jabber JID
+ XML document transfer in XMPP
+ XMPP elements: stream element, presence element, IQ element, Message element
+ XML Schema for XMPP

### Primary article: OpenChirp at Carnegie Mellon

+ OpenChirp
+ LoRaWAN
+ LPWAN
+ LoRaBug
+ Use Case

### Microcontrollers and PAN Protocols lecture notes

+ Smart connected products changing established industries
+ What is a micro controller?
+ Capabilities of smart connected products
+ Monitoring, Control, Optimization, Autonomy
+ EnOcean, ZigBee, Thread, Bluetooth, Wi-Fi, 802.15.4 and 6LoWPAN

### Bluetooth lecture notes

+ Bluetooth and Bluetooth Low Energy
+ Discovery, clients and servers
+ GAP
+ GATT
+ Profiles, Services, and Characteristics

### Web of Things Architecture at W3C

+ [Jeff Jaffe (W3C) at Industry of Things World video](https://www.youtube.com/watch?v=VKKxnk-aRhY&feature=youtu.be&list=PLEyJjfPQS2aCnJ7fktjgmnwvl_RmsvQbd)
+ Servient
+ Thing Description (TD)
+ WoT Scrpting API

### Self-sovereign Identity and Blockchains lecture notes

+ SSI Offline
+ SSI Online
+ Existing systems and standards
+ blockchains

### Smart cities lecture notes

+ Transportation
+ Human Services
+ Safety
+ Conservation

### Google Charts

+ Google Charts basics
+ Google charts in course project

### Photon 2 microcontroller lecture notes

+ What is firmware?
+ How did our Photon 2 communicate with our node.js server?
+ Be able to read and interpret a simple firmware application.
+ The Particle Device Cloud

### Industrial Internet of Things (IIOT) lecture notes

+ SCADA
+ PLC

### CoAP and REST lecture notes

+ Direct Integration Pattern
+ Gateway Integration Pattern
+ Cloud Integration Pattern
+ REST design principles
+ Constrained Application Protocol (CoAP)
+ Compare CoAP with HTTP interaction
+ How does CoAP provide reliability over UDP?
+ CoAP Request/Acknowledge/Callback
+ CoAP publish subscribe
+ CoAP resource discovery

### Flow Programming and Edge Analytics

+ Node-Red

### Review Project 1

+ Heartbeat firmware for the microcontroller
+ Node-RED flows and JavaScript
+ Build a web site using Node.js
+ Design a node-red flow that makes HTTP requests to Node.js
+ AJAX and Node.js
+ Websocket programming using Node.js and Node-RED


### Review Project 2

+ MouseTracker Publish/Subscribe using MQTT
+ Monitoring light levels with a microcontroller
+ Making HTTP requests from a microcontroller
+ Publishing light levels with Node-RED to MQTT
+ Multiple subscribers to MQTT
+ Visualization with Google Charts

### Review Project 3

+ BLE and the LightBlue phone application
+ BLE to a Node-RED gateway
+ Collection and visualization of light values with a time series database on the cloud

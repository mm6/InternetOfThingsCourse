Adding the Particle Node to Node-RED

In Particle.io
    console.particle.io/devices/authentication/new client
    New OAuth Client window/ Two-Legged Auth (Server)/provide name
    Get client ID and secret
    Copy the Client ID to your machine mysimpledevicesimulation-9392
    Copy the Client Secret to your machine 1cbc41dbabc2999423dc5bff6cb2c3c8377bcbc9
    Select I've copied my Secret

In Node-RED:

    Hamburger icon/Manage Palette/User Settings/Install
    In the search box enter Particle
    Install @particle/node-red-contrib-particle-official
    Close

To publish an IoT event to Particle

    From the Node-Red Particle nodes, copy the publish node onto the palette
    Double click the node
    The name is Argon
    The event is heartbeat
    Click the Auth pencil
    Paste your Client ID and secret and do an Add
    Place an inject node onto the Palette
    Wire the inject node to the publish node
    Deploy the flow
    Click the left tab on the inject node
    Check the Events at Particle.io to see if you are publishing timestamps
    Double click the publish node and change from a timestamp to a string "Hello"
    Double click and publish the string {"deviceID":100921}
    Edit the inject node so that it generates the message to Particle every 10 seconds

    

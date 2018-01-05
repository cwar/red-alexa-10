# red-alexa-10
Tutorial and source code to set up a raspberry pi to control x10 devices using node-red. Specifically, this to turn on/off a lamp and open a door... but these concepts are an easy way to jump start sending x10 signals with Amazon Alexa and a Raspberry Pi.

## Prepartion

### Devices Used

* Amazon Echo Dot
* Raspberry Pi 3 Model B
* CM19A X10 RF Transceiver
* X10 TM751 Wireless Transceiver Module
* X10 UM506 Universal Module
* Open Sesame Door Opener w/ Remote

### Technologies Used

* X10
* Amazon Alexa
* [Node-RED](https://nodered.org/)
* [node-red-contrib-alexa-local](https://www.npmjs.com/package/node-red-contrib-alexa-local)
* Python

## Raspberry Pi Setup

To set up Raspberry Pi with the neccessary software (Python and Node-Red), the easiest way is to install the NOOBS OS for Raspberry Pi. This requires a Micro SD Card and a Micro SD Card reader, so make sure you have both of those and follow the guide on setting up NOOBS on your raspberry pi found [here](https://www.raspberrypi.org/documentation/installation/noobs.md).

### Python Driver

The driver we will be using to control the CM19A X10 RF Transceiver is found in this repository (pycm19a.py). This is the version I used at the time I completed this project. If you are looking for the most recent version of this file, it can be found (along with additional documentation on the driver) in [this github repository](https://github.com/burnsfisher/x10pantilt).

### Node-RED setup

#### Starting Node-RED

Within NOOBS, you should see Node-RED listed in the main menu under Programming. Navigating to this menu item and clicking on it will launch a terminal that launches Node-RED. This is nice, but you can just launch a terminal anywhere and issue the command ```node-red``` and node-red will launch. You can, by default, navigate to the UI by going to http://127.0.0.1:1880 in a web browser.

#### Adding the Alexa node

Out of the box, Node-RED can't be invoked by Alexa. Fortunately, the non-core node [alexa-local](https://www.npmjs.com/package/node-red-contrib-alexa-local) allows us to easily have Alexa interface with Node-RED and as a bonus this interfacing is done completely on the local network. 

Before we use the node we must add it to Node-RED. This is done using [npm](https://www.npmjs.com/) (node package manager). To make this node available for Node-RED, just run ```npm install node-red-contrib-alexa-local``` in the Node-RED directory (usually ~/.node-red). After this you must restart Node-RED for the node to show up, if it was running during install.

#### Preparing Node-RED

Node-RED works on flows. A flow is a group of nodes connected by links. Links are lines that connect nodes, and they connect each node by ports exposed by each node. Ports can be input nodes (on the left side of the node, graphically) or output nodes (on the right side of nodes).

Here is an example of what we are working towards in this tutorial:

![alt-text](https://s3.amazonaws.com/uploads.hipchat.com/33959/1621012/f1At6Jnd94wIxWZ/upload.png)

#### Beginning the Node-RED flow

Now that we have the required nodes, it's time to link it all together.

First, drag an alexa local node onto the pallet and give it a name. This design as is (after deployment, more on that in just a moment) will allow Alexa on the network to discover the device. All you need to do at this point is ask Alexa to 'Discover Devices' and it should report back that it has found the device! When you tell Alexa to turn the device on or off now, the alexa local node that represents the device will send forward a message with a payload that is either 'on' or 'off' depending on the voice command to Alexa. A world of opportunities has now been opened up! But for this tutorial we'll be keeping it focused on x10 interfacing with the x10 devices listed above.

To deploy the Node-RED flow, just click the deploy button in the top right of the Node-RED screen.

#### Running the Python X10 Driver

The Python X10 driver referenced can be though of a fairly simple black box. It can be very useful to glance at the code to understand what is going on behind the scenes, but it's not neccesary (especially with the binary on/off model we will be using for this project).


.... to be continued
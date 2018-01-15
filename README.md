# red-alexa-10

Tutorial and source code to set up a raspberry pi to control x10 devices using node-red. Specifically, this to turn on/off a lamp and open a door... but these concepts are an easy way to jump start sending x10 signals with Amazon Alexa and a Raspberry Pi. This will give you a high level over view of Raspberry Pi, X10 and Node-Red.

## Table Of Contents
  - [Prepartion](#prepartion)
    - [Devices Used](#devices-used)
    - [Technologies Used](#technologies-used)
  - [Raspberry Pi Setup](#raspberry-pi-setup)
    - [Python Driver](#python-driver)
    - [Node-RED setup](#node-red-setup)
      - [Starting Node-RED](#starting-node-red)
      - [Adding the Alexa node](#adding-the-alexa-node)
      - [Preparing Node-RED](#preparing-node-red)
      - [Beginning the Node-RED flow](#beginning-the-node-red-flow)
    - [Running the Python X10 Driver](#running-the-python-x10-driver)

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

### Setting USB Permissions

By default, NOOBS provides a default user 'pi' in the group 'users'. The default USB permissions are insufficent for the python driver to access for this user.

To view the USB devices available, execute ```sudo lsusb```. Assuming you've connected your X10 CM19A transceiver, you should see it listed as "X10 Wireless Technology, Inc. Firecracker Interface" with the USB ID of ```0bc7:0002``` which are the Vendor and Product IDs separated by a ```:```. There should also be Bus and Device IDs listed.

We could just ```chmod 600 /dev/bus/usb/<bus ID>/<device ID>``` to grant read/write access to the owner of this device and then ```chown pi /dev/bus/usb/<bus ID>/<device ID>``` to set Pi as the user of the device. However, if the USB device is unplugged, the raspberry pi restarts or for any other reason your USB device is ejected the permissions will change and the IDs may change.

What we can do is add a rule to the directory ```/etc/udev/rules.d```. Take the ```40-persistent-ftdi.rules``` file from this repository and put it in this directory. This file identifies the USB device based on the Vendor and Product IDs and gives the owner read/write permissions as well as sets the owner to Pi.

Once this file is in place, a reboot or rules reload (by executing the command ```udevadm control --reload-rules```) is required for the rules to take effect. 

The permissions are now set, we can move on to operating the X10 device.

### Running the Python X10 Driver

The Python X10 driver referenced can be thought of a fairly simple black box. It can be very useful to glance at the code to understand what is going on behind the scenes, but it's not neccesary (especially with the binary on/off model we will be using for this project).

The script can be invoked via command line in this way by using a combination of an on/off toggle (```+``` or ```-```), house code (between ```a``` and ```p```) and devic number (between ```1``` and ```16```) as an argument to the command. For example, to turn on house code a device number 1 the following command should be issued.

```bash
python pycm19a.py +a1
```


### Python Driver

The driver we will be using to control the CM19A X10 RF Transceiver is found in this repository (pycm19a.py). This is the version I used at the time I completed this project. If you are looking for the most recent version of this file, it can be found (along with additional documentation on the driver) in [this github repository](https://github.com/burnsfisher/x10pantilt).

### Node-RED setup

#### Starting Node-RED

Within NOOBS, you should see Node-RED listed in the main menu under Programming. Navigating to this menu item and clicking on it will launch a terminal that launches Node-RED. This is nice, but you can just launch a terminal anywhere and issue the command ```node-red``` and node-red will launch. You can, by default, navigate to the UI by going to http://127.0.0.1:1880 in a web browser.

#### Adding the Alexa node

Out of the box, Node-RED can't be invoked by Alexa. Fortunately, the non-core node [alexa-local](https://www.npmjs.com/package/node-red-contrib-alexa-local) allows us to easily have Alexa interface with Node-RED and as a bonus this interfacing is done completely on the local network. 

Before we use the node we must add it to Node-RED. This is done using [npm](https://www.npmjs.com/) (node package manager). To make this node available for Node-RED, just run ```npm install``` in the directory that you've cloned this repo. After this you must restart Node-RED for the node to show up, if it was running during install. However, now that we have the Alexa node installed locally, we have to specify the user directory with ```node-red -u . ```. ```-u``` for user directory, which tells node-red where to look for user data like additional nodes and ```.``` which means the directory we're currently in.

#### Preparing Node-RED

Node-RED works on flows. A flow is a group of nodes connected by links. Links are lines that connect nodes, and they connect each node by ports exposed by each node. Ports can be input nodes (on the left side of the node, graphically) or output nodes (on the right side of nodes).

Here is an example of what we are working towards in this tutorial:

![alt-text](https://s3.amazonaws.com/uploads.hipchat.com/33959/1621012/f1At6Jnd94wIxWZ/upload.png)

#### Beginning Node-RED flow

Now that we have the required nodes, it's time to link it all together.

First, drag an alexa local node onto the pallet and give it a name. This design as is (after deployment, more on that in just a moment) will allow Alexa on the network to discover the device. All you need to do at this point is ask Alexa to 'Discover Devices' and it should report back that it has found the device! When you tell Alexa to turn the device on or off now, the alexa local node that represents the device will send forward a message with a payload that is either 'on' or 'off' depending on the voice command to Alexa. A world of opportunities has now been opened up! But for this tutorial we'll be keeping it focused on x10 interfacing with the x10 devices listed above.

To deploy the Node-RED flow, just click the deploy button in the top right of the Node-RED screen.

#### Opening the door

Now that we have a flow containing nodes that represent Alexa recognized devices, time to make them do stuff.

The simplest of our two devices we are setting up is the door. We just need to send a signal to the door to send it into action. To do this, we just need for our Alexa node to generate a message that execs the python driver like we tested before.

To execute this with node red, we will be using the Exec node. Drag an Exec node onto your flow and double click to edit the node.

Let's say you have your door remote set to house code ```a``` and device number ```2```. The driver command to relay to the door would be ```python pycm19a.py +a2```. Using a relative path, if you cloned this repo in a folder called ```projects``` would be ```~/projects/x10/red-alexa-10```. So in the Command field of the Exec node edit screen, putting it all togther, should be ```python ~/projects/x10/red-alexa-10/pycm19a.py +a2```. Remember, + turns on a device and since we really only care about relaying any sort of signal this will work for us (even if it is kind of a cheat, we'll see how to use data from the message from the Alexa node to perform specific actions in the next section).

Link the two nodes together (output from the alexa-local door node to the input of the Exec node you just created) and once you've deployed you will have all the functionality required to open the door with Alexa.

#### Switching the light on and off

Understanding the basics of a Node-Red flow and having one completed, we can create another flow (either a second device on the same flow tab or you can create a new flow in a new tab) for the lamp.

When you ask alexa to turn an alexa-local device on or off, that is included in the msg that many node-red nodes use to pass along information. The information is usually included in the payload (```msg.payload```) but it can be other places as well.

In this simple example, the payload will be a string that is either ```on``` or ```off```. Using that data, we will have node-red make a decision as to which argument to send to the house code/device number for the lamp. We will assume that the lamp is on ```a1```.

To make the decision for us, we will use a Switch node. This node allows you to route messages based on their property values. Edit the node, leave the property set to ```msg.payload``` and define two rules. For both rules we are going to see if ```msg.payload``` is equal (```==```) to the string ```on``` or ```off```. In the switch node you can compare to many different data types, but since we are comparing strings you can leave the data type dropdown as it defaults to string.

To the right of each rule you will see an arrow pointing to a number. This represents which port that the message will be routed to. The ports are also represented on the node-red flow on the right sides of the node. You can link the port to any input node of any other port.

So create two new Exec nodes as before. This time, create one Exec node that turns the light on (```python ~/projects/x10/red-alexa-10/pycm19a.py +a1```) and one that turns the light off (```python ~/projects/x10/red-alexa-10/pycm19a.py -a1```). Simply connect the output ports on the switch route with the appropriate exec node, deploy, and you can control the lights!
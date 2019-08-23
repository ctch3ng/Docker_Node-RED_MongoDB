# Docker_Node-RED_MongoDB
Instructions for preparing docker containers installed with Node-RED and MongoDB [MacOS]

Create the following folder

```
/User/<your MacOS username>/MongoDB/db
```

#### Note: In the following instructions, we are using the following folder under the user **ABC**

```
/User/ABC/MongoDB/db
```

## Prepare a container with MongoDB

```
docker run -d -p 27017-27019:27017-27019 --name mymongodb_mapped -v /Users/ABC/MongoDB/db:/data/db mongo:4.2.0
```
#### Note: The above commands pull the MongoDB docker image (version 4.2.0) from the web, create a comtainer out of it and name it as **mymongodb_mapped**, open the corresponding ports 27017-27019, and map the folder on the host **/User/ABC/MongoDB/db** to the folder in the container **/data/db**

## Prepare a container with Node-RED

```
docker run -it -p 1880:1880 -p 1883:1883 -p 8883:8883 -p 8080:8080 --name mynodered_mapped --link mymongodb_mapped:mongodb nodered/node-red-docker
```
#### Note: The above commands pull the Node-RED docker image (latest version) from the web, create a container out of it and name it as **mynodered_mapped**, open the corresponding ports (1880, 1883, and 8883), and link it to the **mymongodb_mapped** container and give it a hostname **mymongodb**

## Start the containers

```
docker start mymongodb_mapped
docker start mynodered_mapped
```

You can now visit **http://127.0.0.1:1880** to access Node-RED from your browser. 

## Install the extra nodes in Node-RED

* Click the **Three Strips** icon on the top-right corner
* Select **Manage palette** 
* Select the **Install** Tab, search with the keyword **mqtt** 
* Install **node-red-contrib-mqtt-broker** (v. 0.2.4)

* Next, search wtih the keyword **mongo** 
* Install **node-red-node-mongodb** (v. 0.0.14)

#### Alternatively, you can install these nodes using the following commands in a terminal

```
docker exec -it mynodered_mapped bash
npm install node-red-contrib-mqtt-broker
npm install node-red-node-mongodb
exit
docker restart mynodered_mapped
```

## Remarks

In Node-RED, you should use the link **mongodb://mongodb:27017** instead of **mongodb://127.0.0.1:27017** for accessing the linked MongoDB container.

You may use the following Node-RED flow for testing

```
[
    {
        "id": "eddfbd23.4b507",
        "type": "tab",
        "label": "Flow 1",
        "disabled": false,
        "info": ""
    },
    {
        "id": "1d02a0ba.be093f",
        "type": "inject",
        "z": "eddfbd23.4b507",
        "name": "Virtual_DHT11",
        "topic": "",
        "payload": "23C",
        "payloadType": "str",
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "x": 245.5,
        "y": 72,
        "wires": [
            [
                "56e3f4d9.6ac26c",
                "9c3d61e1.547a2"
            ]
        ]
    },
    {
        "id": "27819a1e.4bbbc6",
        "type": "debug",
        "z": "eddfbd23.4b507",
        "name": "TempDB_MongoDB",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 518.5,
        "y": 406,
        "wires": []
    },
    {
        "id": "56e3f4d9.6ac26c",
        "type": "debug",
        "z": "eddfbd23.4b507",
        "name": "Temp1_MQTT",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 462.5,
        "y": 124,
        "wires": []
    },
    {
        "id": "eab13d37.bc256",
        "type": "debug",
        "z": "eddfbd23.4b507",
        "name": "MQTT_Broker",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 461.5,
        "y": 283,
        "wires": []
    },
    {
        "id": "303e7719.a98348",
        "type": "mongodb out",
        "z": "eddfbd23.4b507",
        "mongodb": "75d5b53e.7a75dc",
        "name": "",
        "collection": "Temp",
        "payonly": false,
        "upsert": false,
        "multi": false,
        "operation": "insert",
        "x": 499.5,
        "y": 345,
        "wires": []
    },
    {
        "id": "5c7b7b0e.8c3d54",
        "type": "mosca in",
        "z": "eddfbd23.4b507",
        "mqtt_port": 1883,
        "mqtt_ws_port": 8080,
        "name": "",
        "username": "",
        "password": "",
        "dburl": "mongodb://mongodb:27017/mqtt",
        "x": 241,
        "y": 282,
        "wires": [
            [
                "eab13d37.bc256"
            ]
        ]
    },
    {
        "id": "9c3d61e1.547a2",
        "type": "mqtt out",
        "z": "eddfbd23.4b507",
        "name": "",
        "topic": "home/bedroom1/temp1",
        "qos": "2",
        "retain": "true",
        "broker": "343ab08f.068e4",
        "x": 499.5,
        "y": 72,
        "wires": []
    },
    {
        "id": "5b3ceb37.99e6c4",
        "type": "mqtt in",
        "z": "eddfbd23.4b507",
        "name": "",
        "topic": "home/+/temp1",
        "qos": "2",
        "datatype": "auto",
        "broker": "343ab08f.068e4",
        "x": 233.5,
        "y": 407,
        "wires": [
            [
                "27819a1e.4bbbc6",
                "303e7719.a98348"
            ]
        ]
    },
    {
        "id": "3751ef83.9d5fd",
        "type": "inject",
        "z": "eddfbd23.4b507",
        "name": "Virtual_DHT11",
        "topic": "",
        "payload": "25C",
        "payloadType": "str",
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "x": 246,
        "y": 170,
        "wires": [
            [
                "ba231c44.5b6fb",
                "68a5318.eb422d"
            ]
        ]
    },
    {
        "id": "ba231c44.5b6fb",
        "type": "debug",
        "z": "eddfbd23.4b507",
        "name": "Temp2_MQTT",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 463,
        "y": 222,
        "wires": []
    },
    {
        "id": "68a5318.eb422d",
        "type": "mqtt out",
        "z": "eddfbd23.4b507",
        "name": "",
        "topic": "home/kitchen/temp1",
        "qos": "2",
        "retain": "true",
        "broker": "a5778332.3d44a",
        "x": 490,
        "y": 170,
        "wires": []
    },
    {
        "id": "75d5b53e.7a75dc",
        "type": "mongodb",
        "z": "",
        "hostname": "mongodb",
        "port": "27017",
        "db": "TempDB",
        "name": ""
    },
    {
        "id": "343ab08f.068e4",
        "type": "mqtt-broker",
        "z": "",
        "name": "",
        "broker": "127.0.0.1",
        "port": "1883",
        "clientid": "Node1",
        "usetls": false,
        "compatmode": true,
        "keepalive": "60",
        "cleansession": true,
        "birthTopic": "",
        "birthQos": "0",
        "birthPayload": "",
        "closeTopic": "",
        "closeQos": "0",
        "closePayload": "",
        "willTopic": "",
        "willQos": "0",
        "willPayload": ""
    },
    {
        "id": "a5778332.3d44a",
        "type": "mqtt-broker",
        "z": "",
        "name": "",
        "broker": "127.0.0.1",
        "port": "1883",
        "clientid": "Node2",
        "usetls": false,
        "compatmode": true,
        "keepalive": "60",
        "cleansession": true,
        "birthTopic": "",
        "birthQos": "0",
        "birthPayload": "",
        "closeTopic": "",
        "closeQos": "0",
        "closePayload": "",
        "willTopic": "",
        "willQos": "0",
        "willPayload": ""
    }
]
```

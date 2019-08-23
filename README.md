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
        "id": "757ace53.ae32c",
        "type": "tab",
        "label": "Flow 1",
        "disabled": false,
        "info": ""
    },
    {
        "id": "d9c6c986.c99848",
        "type": "inject",
        "z": "757ace53.ae32c",
        "name": "Virtual_DHT11",
        "topic": "",
        "payload": "23C",
        "payloadType": "str",
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "x": 95.5,
        "y": 49,
        "wires": [
            [
                "b9c46395.0fa6b",
                "34498bb6.7ea504"
            ]
        ]
    },
    {
        "id": "e3f05f05.2a1df",
        "type": "debug",
        "z": "757ace53.ae32c",
        "name": "TempDB_MongoDB",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 370.5,
        "y": 299,
        "wires": []
    },
    {
        "id": "b9c46395.0fa6b",
        "type": "debug",
        "z": "757ace53.ae32c",
        "name": "Temp1_MQTT",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 312.5,
        "y": 101,
        "wires": []
    },
    {
        "id": "aa318688.bbf698",
        "type": "debug",
        "z": "757ace53.ae32c",
        "name": "MQTT_Broker",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 315.5,
        "y": 179,
        "wires": []
    },
    {
        "id": "f8f9bec1.7b6dd",
        "type": "mongodb out",
        "z": "757ace53.ae32c",
        "mongodb": "787bed46.0a8eb4",
        "name": "",
        "collection": "Temp",
        "payonly": false,
        "upsert": false,
        "multi": false,
        "operation": "insert",
        "x": 351.5,
        "y": 238,
        "wires": []
    },
    {
        "id": "5156f987.5de488",
        "type": "mosca in",
        "z": "757ace53.ae32c",
        "mqtt_port": 1883,
        "mqtt_ws_port": 8080,
        "name": "",
        "username": "",
        "password": "",
        "dburl": "mongodb://mongodb:27017/mqtt",
        "x": 95,
        "y": 178,
        "wires": [
            [
                "aa318688.bbf698"
            ]
        ]
    },
    {
        "id": "34498bb6.7ea504",
        "type": "mqtt out",
        "z": "757ace53.ae32c",
        "name": "",
        "topic": "home/bedroom1/temp1",
        "qos": "",
        "retain": "",
        "broker": "defcb849.eb5488",
        "x": 349.5,
        "y": 49,
        "wires": []
    },
    {
        "id": "89aa7147.11151",
        "type": "mqtt in",
        "z": "757ace53.ae32c",
        "name": "",
        "topic": "home/bedroom1/temp1",
        "qos": "2",
        "datatype": "auto",
        "broker": "defcb849.eb5488",
        "x": 105.5,
        "y": 300,
        "wires": [
            [
                "e3f05f05.2a1df",
                "f8f9bec1.7b6dd"
            ]
        ]
    },
    {
        "id": "787bed46.0a8eb4",
        "type": "mongodb",
        "z": "",
        "hostname": "mongodb",
        "port": "27017",
        "db": "TempDB",
        "name": ""
    },
    {
        "id": "defcb849.eb5488",
        "type": "mqtt-broker",
        "z": "",
        "name": "",
        "broker": "127.0.0.1",
        "port": "1883",
        "clientid": "",
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

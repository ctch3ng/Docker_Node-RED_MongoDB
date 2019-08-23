# Docker_Node-RED_MongoDB
Instructions for preparing docker containers installed with Node-RED and MongoDB

Create the following folders

```
/Users/<your username>/MongoDB/db
/Users/<your username>/mosquitto/config
/Users/<your username>/mosquitto/data
/Users/<your username>/mosquitto/log
```

#### Note: In the following instructions, we are using the following folders under the user **ABC**

```
/Users/ABC/MongoDB/db
/Users/ABC/mosquitto/config
/Users/ABC/mosquitto/data
/Users/ABC/mosquitto/log
```

## Prepare a container with MongoDB

```
docker run -d -p 27017-27019:27017-27019 --name mymongodb -v /Users/ABC/MongoDB/db/:/data/db/ mongo:4.2.0
```
#### Note: The above commands pull the MongoDB docker image (version 4.2.0) from the web, create a comtainer out of it and name it as **mymongodb**, open the corresponding ports 27017-27019, and map the folder on the host to the folder in the container

## Prepare a container with Mosquitto

```
docker run -it -p 1883:1883 -p 9001:9001 -v /Users/ABC/mosquitto/config/:/mosquitto/config/ -v /Users/ABC/mosquitto/data/:/mosquitto/data/ -v /Users/ABC/mosquitto/log/:/mosquitto/log/ --name mymosquitto eclipse-mosquitto
```
#### Note: The above commands pull the Mosquitto docker image (latest version) from the web, create a comtainer out of it and name it as **mymosquitto**, open the corresponding ports (1883 and 9001), and map the folders on the host to the folders in the container

## Prepare a container with Node-RED

```
docker run -it -p 1880:1880 --name mynodered --link mymongodb:mongodb --link mymosquitto:mosquitto nodered/node-red-docker
```
#### Note: The above commands pull the Node-RED docker image (latest version) from the web, create a container out of it and name it as **mynodered**, open the corresponding ports (1880), link it to the **mymongodb** container and give it a hostname **mongodb**, and further link it to the **mymosquitto** container and give it a hostname **mosquitto**

## Start the containers

```
docker start mymongodb
docker start mymosquitto
docker start mynodered
```

You can now visit **http://127.0.0.1:1880** to access Node-RED from your browser. 

## Install the extra node in Node-RED

* Click the **Three Strips** icon on the top-right corner
* Select **Manage palette** 
* Search wtih the keyword **mongo** 
* Install **node-red-node-mongodb** (v. 0.0.14)

#### Alternatively, you can install node using the following commands in a terminal

```
docker exec -it mynodered bash
npm install node-red-node-mongodb
exit
docker restart mynodered
```

## Remarks

In Node-RED, you should use the link **mongodb://mongodb:27017** instead of **mongodb://127.0.0.1:27017** for accessing the linked MongoDB container. Also, use the link **mosquitto:1883** instead of **127.0.0.1:1883** for accessing the linked Mosquitto container

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

# Docker_Node-RED_MongoDB
Instructions for preparing docker containers installed with Node-RED and MongoDB

Create the following folders

```
/Users/<your username>/mosquitto/config
/Users/<your username>/mosquitto/data
/Users/<your username>/mosquitto/log
```

#### Note: In the following instructions, we are using the following folders under the user **ABC**

```
/Users/ABC/mosquitto/config
/Users/ABC/mosquitto/data
/Users/ABC/mosquitto/log
```

## Prepare a container with MongoDB

Run the following command in a command prompt
#### Remember to repalce **ABC** with your username  
```
docker volume create --name=mongodata
docker run -d -p 27017-27019:27017-27019 --name mymongodb -v mongodata:/data/db/ mongo:4.2.0
```
#### Note: The above commands pull the MongoDB docker image (version 4.2.0) from the web, create a comtainer out of it and name it as **mymongodb**, open the corresponding ports 27017-27019, and map the docker volume **mongodata** to the folder in the container

## Prepare a container with Mosquitto

Under **/Users/ABC/mosquitto/config**, create a text file **mosquitto.conf** with the following content
```
port 1883
listener 9001
protocol websockets
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
```
Run the following command in a command prompt
#### Remember to repalce **ABC** with your username
```
docker run -d -p 1883:1883 -p 9001:9001 -v c:/Users/ABC/mosquitto/config/:/mosquitto/config/ -v c:/Users/ABC/mosquitto/data/:/mosquitto/data/ -v c:/Users/ABC/mosquitto/log/:/mosquitto/log/ --name mymosquitto eclipse-mosquitto
```
#### Note: The above commands pull the Mosquitto docker image (latest version) from the web, create a comtainer out of it and name it as **mymosquitto**, open the corresponding ports (1883 and 9001), and map the folders on the host to the folders in the container

## Prepare a container with Node-RED

Run the following command in a command prompt
```
docker run -d -p 1880:1880 --name mynodered --link mymongodb:mongodb --link mymosquitto:mosquitto nodered/node-red-docker
```
#### Note: The above commands pull the Node-RED docker image (latest version) from the web, create a container out of it and name it as **mynodered**, open the corresponding ports (1880), link it to the **mymongodb** container and give it a hostname **mongodb**, and further link it to the **mymosquitto** container and give it a hostname **mosquitto**

## Start the containers

Run the following command in a command prompt
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
        "id": "54fc2c53.c340c4",
        "type": "tab",
        "label": "Flow 1",
        "disabled": false,
        "info": ""
    },
    {
        "id": "bcb25fbf.8dbd",
        "type": "inject",
        "z": "54fc2c53.c340c4",
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
                "a512e1c4.2372f",
                "df25f182.6e943"
            ]
        ]
    },
    {
        "id": "f844ded6.87865",
        "type": "debug",
        "z": "54fc2c53.c340c4",
        "name": "TempDB_MongoDB",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "true",
        "targetType": "full",
        "x": 518.5,
        "y": 406,
        "wires": []
    },
    {
        "id": "a512e1c4.2372f",
        "type": "debug",
        "z": "54fc2c53.c340c4",
        "name": "Temp1_MQTT",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "true",
        "targetType": "full",
        "x": 462.5,
        "y": 124,
        "wires": []
    },
    {
        "id": "7ea7822e.dc08ec",
        "type": "mongodb out",
        "z": "54fc2c53.c340c4",
        "mongodb": "d0631cdd.2f1fd",
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
        "id": "df25f182.6e943",
        "type": "mqtt out",
        "z": "54fc2c53.c340c4",
        "name": "",
        "topic": "home/bedroom1/temp1",
        "qos": "2",
        "retain": "true",
        "broker": "4cde74ed.6ed88c",
        "x": 499.5,
        "y": 72,
        "wires": []
    },
    {
        "id": "15112d56.0b33d3",
        "type": "mqtt in",
        "z": "54fc2c53.c340c4",
        "name": "",
        "topic": "home/+/temp1",
        "qos": "2",
        "datatype": "auto",
        "broker": "9ed4875f.42b258",
        "x": 233.5,
        "y": 407,
        "wires": [
            [
                "f844ded6.87865",
                "7ea7822e.dc08ec"
            ]
        ]
    },
    {
        "id": "3f3391ea.cf1cae",
        "type": "inject",
        "z": "54fc2c53.c340c4",
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
                "812a3956.8ff7a8",
                "9c072c44.a1b07"
            ]
        ]
    },
    {
        "id": "812a3956.8ff7a8",
        "type": "debug",
        "z": "54fc2c53.c340c4",
        "name": "Temp2_MQTT",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "true",
        "targetType": "full",
        "x": 463,
        "y": 222,
        "wires": []
    },
    {
        "id": "9c072c44.a1b07",
        "type": "mqtt out",
        "z": "54fc2c53.c340c4",
        "name": "",
        "topic": "home/kitchen/temp1",
        "qos": "2",
        "retain": "true",
        "broker": "2fc92d67.b84d02",
        "x": 490,
        "y": 170,
        "wires": []
    },
    {
        "id": "d0631cdd.2f1fd",
        "type": "mongodb",
        "z": "",
        "hostname": "mongodb",
        "port": "27017",
        "db": "TempDB",
        "name": ""
    },
    {
        "id": "4cde74ed.6ed88c",
        "type": "mqtt-broker",
        "z": "",
        "name": "",
        "broker": "mosquitto",
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
        "id": "9ed4875f.42b258",
        "type": "mqtt-broker",
        "z": "",
        "name": "",
        "broker": "mosquitto",
        "port": "1883",
        "clientid": "Node3",
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
        "id": "2fc92d67.b84d02",
        "type": "mqtt-broker",
        "z": "",
        "name": "",
        "broker": "mosquitto",
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

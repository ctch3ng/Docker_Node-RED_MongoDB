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
docker run -it -p 1880:1880 -p 1883:1883 -p 8883:8883 --name mynodered_MQTT_mapped --link mymongodb_mapped:mongodb nodered/node-red-docker
```
#### Note: The above commands pull the Node-RED docker image (latest version) from the web, create a container out of it and name it as **mynodered_MQTT_mapped**, open the corresponding ports (1880, 1883, and 8883), and link it to the **mymongodb_mapped** container and give it a hostname **mymongodb**

## Start the containers

```
docker start mymongodb_mapped
docker start mynodered_MQTT_mapped
```

You can now visit **http://127.0.0.1:1880** to access Node-RED from your browser. 

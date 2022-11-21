---
layout: default
title: Install procedure
description: How to install the different components of the system.
permalink: /install
---

{:.title}
{{ page.title }}

* auto-gen TOC:
{:toc}


## Prerequisites

### Install Docker

```shell
sudo apt install docker.io
```


## Install DSE environment


### Opscenter

DataStax OpsCenter is a visual management and monitoring solution for DataStax 
Enterprise (DSE)

It is installed with its docker image

```shell
docker run -e DS_LICENSE=accept -p 8888:8888 --name opscenter -d datastax/dse-opscenter:6.8.19
```

With the port forwarding that has been defined at the creation ot the image it
can be accessed on [http://radon-1.apps.l:8888](http://radon-1.apps.l:8888) (
where radon-1.apps.l is the hostname of the host, the IP address can also be used).


### DSE Server (Cassandra)


DataStax Enterprise provides a distribution of Apache Cassandra with several
additional tools (Graph database, Search, Analytics, ...)

- We need to create the persistent folders to store the database and the logs 
for Cassandra

```shell
docker volume create dse_lib
docker volume create dse_log
```

* We can install DSE server with the docker image, using the Docker volumes to 
enable persistence

```shell
docker run -e DS_LICENSE=accept \
           -e JVM_EXTRA_OPTS="-Xms512m -Xmx2560m" \
           --link opscenter:opscenter \
           --name dse -d \
           -v dse_lib:/var/lib/cassandra \
           -v dse_log:/var/log/cassandra \
           datastax/dse-server:6.8.25 -k -s -g
```

**_Note 1: On certain configuration (LXC on ProxMox), the size of the heap for the
JVM has to be set manually. Otherwise we can experience memory issues and a 
DSE server shutting down without any error message in the logs._**

**_Note 2: We may consider using the `--network host` option in a production 
environment to improve performance._**


### Studio (Optional)

DataStax Studio is an interactive tool for CQL (Cassandra Query Language) and 
DSE Graph. It is not mandatory but it can be useful to experiment with graphs.

It is installed with its docker image

```shell
docker run -e DS_LICENSE=accept -p 9091:9091 --link dse --name studio -d datastax/dse-studio
```


With the port forwarding that has been defined at the creation ot the image it
can be accessed on [http://radon-1.apps.l:9091](http://radon-1.apps.l:9091) (
where radon-1.apps.l is the hostname of the host).

### Configure cluster

The cluster have to be configured with the opscenter web application. It can be 
accessed on [http://radon-1.apps.l:8888](http://radon-1.apps.l:8888).

- Click on Manage existing cluster

- Enter the dse container IP address in the host name field. The IP can be found
with the command `docker exec dse hostname -i`. It should probably be 172.17.0.3.

- Click on Install agents manually. The agent is already installed on the DSE 
image, so no installation is required.


## Install MQTT

Eclipse Mosquitto is an open source message broker that implements the MQTT 
protocol. It can be installed with a Docker image.

### Create Config file

Mosquitto 2.0 restrict connections to the loopback, we need to allow 
connections via a configuration file.

```shell
$ cat mosquitto-no-auth.conf
listener 1883
allow_anonymous true
```

### Create the MQTT docker

```shell
docker run -it -p 1883:1883 -p 9001:9001 --name mqtt -d eclipse-mosquitto -c /mosquitto-no-auth.conf
```

Once started its IP address can be found with the command 
```shell
docker exec mqtt hostname -i
````

It should probably be 172.17.0.5.


## Install radon-web

### Download sources

- Create a specific folder for the sources

```shell
mkdir src
cd src
```

- Get radon-lib package

```shell
git clone https://github.com/radon-provenance/radon-lib.git
```

- Get radon-web package

```shell
git clone https://github.com/radon-provenance/radon-web.git
```

- Modify or create environment variables

```shell
export DSE_HOST="`docker exec dse hostname -i`"
export MQTT_HOST="`docker exec mqtt hostname -i`"
```

The environment variables are set in the radon-web/Dockerfile or in radon-lib/.env. 
If they are defined in the Dockerfile they have the priority


### Create radon-admin docker image

```
docker build -t radon-admin-image --build-arg DSE_HOST --build-arg MQTT_HOST  -f radon-lib/Dockerfile .
```

On some machine it may be necessary to use this syntax:

```shell
docker build -t radon-admin-image --build-arg DSE_HOST=${DSE_HOST} --build-arg MQTT_HOST=${MQTT_HOST} -f radon-lib/Dockerfile .
```

**_Note: The image has to be created from a higher level (./src) than radon-lib to be 
consistent with the radon-web install which requires access to both radon-lib 
and radon-web packages._**


### Initialise Cassandra model

- Launch a radon-admin container, it can be used locally to setup Radon.

```
docker run -it --rm radon-admin-image:latest /bin/bash
```

- Create the database

```
-> radmin init
```

**_Note 1: Other things can be created at this time like other users, groups, 
... But these are the mandatory options that has to be created before the 
radon-web server can be created._**

- Create other users (optional)

```
-> radmin mkuser
```

**_Note 2: The radon-admin image is deleted when we exit its shell but it can be
recreated at any moment if needed. The radmin commands are also installed in the
radon-web image_**


### Install radon-web server

- Create docker image

```
docker build -t radon-web-image --build-arg DSE_HOST --build-arg MQTT_HOST  -f radon-web/Dockerfile .
```

**_Note: The image has to be created from a higher level (./src) than radon-web
as it requires access to both radon-lib and radon-web packages._**


- Create and run radon-web container

```
docker run --name radon-web \
           -p 8000:8000 \
           -d radon-web-image:latest gunicorn project.wsgi:application --bind 0.0.0.0:8000 
```

- Radon web server can be accessed on [http://radon-1.apps.l:8000](http://radon-1.apps.l:8000).

- If needed, we can login to the docker container with the command 
`docker exec -it radon-web bash`


## Install radon-listener

### Get Code

```shell
git clone https://github.com/radon-provenance/radon-listener.git
```

### Configure hostnames

- Configure variables for the different hosts

```shell
export DSE_HOST="`docker exec dse hostname -i`"
export MQTT_HOST="`docker exec mqtt hostname -i`"
```

- Configure ./pom.xml

Set the hostname for mqtt host in <mqttHost> (This is temporary)
  

### Create Docker image for the listener

```shell
docker build -t radon-listener-image --build-arg DSE_HOST --build-arg MQTT_HOST  -f radon-listener/Dockerfile .
```

  
### Run the rule engine

```shell
docker run -it --rm radon-listener-image:latest mvn exec:java -Dexec.mainClass="org.radon.listener.RadonApp"
```

If installed correctly it should display notifications when an action fires a rule in Radon.





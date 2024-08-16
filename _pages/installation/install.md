---
title: Radon Data Management System
permalink: /install/install.html
layout: page
sidebar_menu: Installation
sidebar_sub: Install manually
summary: Detailed installation for the installation of the different docker 
         images
---


{% include toc.html %}

## Prerequisites

### Install Docker

```shell
sudo apt install docker.io
```

Eventually you may need to add your user to the docker group:

```shell
sudo adduser $USER docker
```

## Install DSE environment



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
           -p 9042:9042 \
           --name dse -d \
           -v dse_lib:/var/lib/cassandra \
           -v dse_log:/var/log/cassandra \
           datastax/dse-server:6.8.47 -k -s -g
```


{% include note.html content="On certain configuration (LXC on ProxMox), the 
size of the heap for the JVM has to be set manually. Otherwise we can experience 
memory issues and a DSE server shutting down without any error message in the 
logs." %}

{% include note.html content="We may consider using the `--network host` option 
in a production environment to improve performance." %}


### Studio (Optional)

DataStax Studio is an interactive tool for CQL (Cassandra Query Language) and 
DSE Graph. It is not mandatory but it can be useful to experiment with graphs.

It is installed with its docker image

```shell
docker run -e DS_LICENSE=accept \
           -p 9091:9091 \
           --link dse \
           --name studio \
           -d datastax/dse-studio
```


With the port forwarding that has been defined at the creation ot the image it
can be accessed on [http://radon-1.apps.l:9091](http://radon-1.apps.l:9091){:target="_blank"}
(where radon-1.apps.l is the hostname of the host).


## Install MQTT

Eclipse Mosquitto is an open source message broker that implements the MQTT 
protocol. It can be installed with a Docker image.

### Create Volume for persistence

```shell
docker volume create mosquitto_data
docker volume create mosquitto_log
```

### Create Config file

Mosquitto 2.0 restrict connections to the loopback, we need to allow 
connections via a configuration file.

```shell
$ nano mosquitto.conf
listener 1883
allow_anonymous true
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
```

### Create the MQTT docker

```shell
docker run -it -p 1883:1883 \
           -p 9001:9001 \
           --name mqtt \
           -v mosquitto_data:/mosquitto/data \
           -v mosquitto_log:/mosquitto/log \
           -v $PWD/mosquitto.conf:/mosquitto/config/mosquitto.conf \
           -d eclipse-mosquitto 
```


Once started its IP address can be found with the command 
```shell
docker exec mqtt hostname -i
````

It should probably be 172.17.0.3.


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

```shell
docker build -t radon-admin-image \
             --build-arg DSE_HOST=${DSE_HOST} \
             --build-arg MQTT_HOST=${MQTT_HOST} \
             -f radon-lib/Dockerfile .
```

{% include note.html content="The image has to be created from a higher level 
(./src) than radon-lib to be consistent with the radon-web install which 
requires access to both radon-lib and radon-web packages." %}


### Initialise Cassandra model

- Launch a radon-admin container, it can be used locally to setup Radon.

```
docker run -it --rm radon-admin-image:latest /bin/bash
```

- Create the database

```
-> radmin init
```

{% include note.html content="Other things can be created later like other users, groups, 
... But this creates the mandatory things that has to exist before the 
radon-web server can be launched." %}


### Install radon-web server

- Create docker image

```
docker build -t radon-web-image \
             --build-arg DSE_HOST=${DSE_HOST} \
             --build-arg MQTT_HOST=${MQTT_HOST} \
             -f radon-web/Dockerfile .
```

{% include note.html content="The image has to be created from a higher level (./src) than radon-web
as it requires access to both radon-lib and radon-web packages." %}


- Create and run radon-web container

```
docker run --name radon-web \
           -p 8000:8000 \
           -d radon-web-image:latest \
           gunicorn project.wsgi:application --bind 0.0.0.0:8000 \
           --workers 10 --threads 10 --worker-tmp-dir /dev/shm
```

- Radon web server can be accessed on [http://radon-1.apps.l:8000](http://radon-1.apps.l:8000){:target="_blank"}.

- If needed, we can login to the docker container with the command 
`docker exec -it radon-web bash`

- Radon web server serves the microservices needed for the rules. It has to be started
before we can create anything on the server (users/collections/...).




## Install radon-listener

### Get Code

```shell
git clone https://github.com/radon-provenance/radon-listener.git
cd radon-listener
```

### Configure hostnames

- Configure variables for the different hosts

```shell
export RADON_HOST="`docker exec radon-web hostname -i`"
export MQTT_HOST="`docker exec mqtt hostname -i`"
```


### Create Docker image for the listener

```shell
docker build -t radon-listener-image .
```

  
### Run the rule engine

```shell
docker run --name radon-listener \
           -d radon-listener-image:latest \
           -m $MQTT_HOST -r $RADON_HOST
```


## Finalise the installation


- Launch a radon-admin container, it can be used locally to setup Radon.


```
docker run -it --rm radon-admin-image:latest /bin/bash
```

- Create default users (optional)

```
radmin populate
```


- Create other users (optional)

```
-> radmin mkuser
```

{% include note.html content="The radon-admin image is deleted when we exit its shell but it can be
recreated at any moment if needed. The radmin commands are also installed in the
radon-web image" %}

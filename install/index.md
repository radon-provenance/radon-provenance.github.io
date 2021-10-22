
# Prerequisites

## Install Docker

```sudo apt install docker.io```


# Install DSE environment


## Opscenter

DataStax OpsCenter is a visual management and monitoring solution for DataStax 
Enterprise (DSE)

It is installed with its docker image

```
docker run -e DS_LICENSE=accept -p 8888:8888 --name opscenter -d datastax/dse-opscenter:6.8.16
```

With the port forwarding that has been defined at the creation ot the image it
can be accessed on [http://radon-1.apps.l:8888](http://radon-1.apps.l:8888) (
where radon-1.apps.l is the hostname of the host, the IP address can also be used).


## DSE Server (Cassandra)


DataStax Enterprise provides a distribution of Apache Cassandra with several
additional tools (Graph database, Search, Analytics, ...)

- We need to create the persistent folders to store the database and the logs 
for Cassandra

```
sudo mkdir -p /dse/cassandra/lib
sudo mkdir -p /dse/cassandra/log

sudo chown 999:docker /dse/cassandra/lib
sudo chown 999:docker /dse/cassandra/log

sudo chmod g+w /dse/cassandra/lib
sudo chmod g+w /dse/cassandra/log
```

**_Note: DSE runs with a specific user with a uid of 999 in the docker image. We 
need to use the same uid on the host system to allow the DSE server to write files on
the mounted volumes._**

* We can install DSE server with the docker image, using the absolute path for 
Docker volumes to enable persistence

```
docker run -e DS_LICENSE=accept \
           -e JVM_EXTRA_OPTS="-Xms512m -Xmx2560m" \
           --link opscenter:opscenter \
           --name dse -d \
           -v /dse/cassandra/lib:/var/lib/cassandra \
           -v /dse/cassandra/log:/var/log/cassandra \
           datastax/dse-server:6.8.16 -k -s -g
```

**_Note 1: On certain configuration (LXC on ProxMox), the size of the heap for the
JVM has to be set manually. Otherwise we can experience memory issues and a 
DSE server shutting down without any error message in the logs._**

**_Note 2: We may consider using the `--network host` option in a production 
environment to improve performance._**


## Studio (Optional)

DataStax Studio is an interactive tool for CQL (Cassandra Query Language) and 
DSE Graph. It is not mandatory but it can be useful to experiment with graphs.

It is installed with its docker image

```
docker run -e DS_LICENSE=accept -p 9091:9091 --link dse --name studio -d datastax/dse-studio
```


With the port forwarding that has been defined at the creation ot the image it
can be accessed on [http://radon-1.apps.l:9091](http://radon-1.apps.l:9091) (
where radon-1.apps.l is the hostname of the host).

## Configure cluster

The cluster have to be configured with the opscenter web application. It can be 
accessed on [http://radon-1.apps.l:8888](http://radon-1.apps.l:8888).

- Click on Manage existing cluster

- Enter the dse container IP address in the host name field. The IP can be found
with the command `docker exec dse hostname -i`. It should probably be 172.17.0.3.

- Click on Install agents manually. The agent is already installed on the DSE 
image, so no installation is required.


# Install MQTT

Eclipse Mosquitto is an open source message broker that implements the MQTT 
protocol. It can be installed with a Docker image.

```
docker run -it -p 1883:1883 -p 9001:9001 --name mqtt -d eclipse-mosquitto
```

Once started its IP address can be found with the command `docker exec mqtt hostname -i`.
It should probably be 172.17.0.5.


# Install radon-web

## Download sources

- Create a specific folder for the sources

```
mkdir src
cd src
```

- Get radon-lib package

```
git clone https://github.com/radon-provenance/radon-lib.git
```

- Get radon-web package

```
git clone https://github.com/radon-provenance/radon-web.git
```

- Modify or create environment variables

{% raw %}
```
export DSE_HOST="`docker exec dse hostname -i`"
export MQTT_HOST="`docker exec mqtt hostname -i`"
```
{% endraw %}

The environment variables are set in the radon-web/Dockerfile or in radon-lib/.env. 
If they are defined in the Dockerfile they have the priority


## Create radon-admin docker image

```
docker build -t radon-admin-image --build-arg DSE_HOST --build-arg MQTT_HOST  -f radon-lib/Dockerfile .
```

**_Note: The image has to be created from a higher level (./src) than radon-lib to be 
consistent with the radon-web install which requires access to both radon-lib 
and radon-web packages._**


## Initialise Cassandra model

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


## Install radon-web server

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


# Restart services

When the different servers are configured correctly we can use the start/stop
commands from docker to manage the different servers which all have names.

```
docker start opscenter
docker start dse
docker start studio

docker start mqtt

docker start radon-web
```




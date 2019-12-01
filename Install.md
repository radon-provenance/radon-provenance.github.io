

# Install DSE


## Opscenter

docker run -e DS_LICENSE=accept -p 8888:8888 --name opscenter -d datastax/dse-opscenter 


## Cassandra

Create the persistent folder to store the database and the logs for Cassandra

mkdir -p ./cassandra/lib
mkdir -p ./cassandra/log

Use the absolute path to Docker


docker run -e DS_LICENSE=accept \
           --link opscenter:opscenter \
           --name dse -d \
           -v /home/jerome/Work/Radon/dse_install/cassandra/lib:/var/lib/cassandra \
           -v /home/jerome/Work/Radon/dse_install/cassandra/log:/var/log/cassandra \
           datastax/dse-server -g


## Studio

docker run -e DS_LICENSE=accept -p 9091:9091 --link dse --name studio -d datastax/dse-studio


## Configure cluster

Get Opscenter IP address

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' opscenter
-> 172.17.0.2

go to http://172.17.0.2:8888

Click on Manage existing cluster

Enter the dse container IP address in the host name field

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dse
-> 172.17.0.3

Click on Install agents manually


# Install MQTT

docker run -it -p 1883:1883 -p 9001:9001 --name mqtt -d eclipse-mosquitto


docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mqtt
-> 172.17.0.5

# Install radon-web


## Get radon-lib package

git https://github.com/radon-provenance/radon-lib.git


## Get radon-web package

git https://github.com/radon-provenance/radon-web.git


## Modify or create environment variables

DSE_HOST="172.17.0.3"
MQTT_HOST="172.17.0.5"

The IP adress can be found with 

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dse
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mqtt

The environment variables are set in the radon-web/Dockerfile or in radon-lib/.env. 
If they are defined in the Dockerfile they have the priority


## Create radon-admin docker image

in ./src where docker can find radon-lib/

docker build -t radon-admin-image -f radon-lib/Dockerfile .


## Initialise Cassandra model

Launch a radon-admin container

docker run -it --rm radon-admin-image:latest /bin/bash

Create the database

-> radmin create

Create a user

-> radmin mkuser


## Create radon-web docker image

in ./src where docker can find radon-web/ and radon-lib/

docker build -t radon-web-image -f radon-web/Dockerfile .


## Create and run radon-web container

docker run --rm --name radon-web \
           -p 8000:8000 \
           -v $(pwd)/radon-lib:/code/radon-lib\
           -v $(pwd)/radon-web:/code/radon-web\
           -d radon-web-image:latest gunicorn project.wsgi:application --bind 0.0.0.0:8000 

-> Get ip address
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' radon-web

-> http://172.17.0.6:8000


docker exec -it radon-web bash
  

# Run 

docker start opscenter
docker start dse
docker start studio

docker start mqtt

docker start radon-web





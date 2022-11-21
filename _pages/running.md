---
layout: default
title: Install procedure
description: How to install the different components of the system.
permalink: /running
---

{:.title}
{{ page.title }}

When the different servers are configured correctly we can use the start/stop
commands from docker to manage the different servers which all have names.


# DSE


```shell
docker start opscenter
docker start dse
docker start studio
```

- Opscenter can be accessed at [http://radon-1.apps.l:8888](http://radon-1.apps.l:8888)

- Studio can be accessed at [http://radon-1.apps.l:9091](http://radon-1.apps.l:9091)

# MQTT


```shell
docker start mqtt
```


# Radon

```shell
docker start radon-web
```

- Radon Web can be accessed at [http://radon-1.apps.l:8000](http://radon-1.apps.l:8000)


# Listener

```shell
docker run -it --rm radon-listener-image:latest mvn exec:java -Dexec.mainClass="org.radon.listener.RadonApp"
```



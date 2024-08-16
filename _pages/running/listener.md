---
title: Radon Data Management System
permalink: /running/listener.html
layout: default
sidebar_menu: Running
sidebar_sub: Radon Listener
---

# Radon listener

The listener is subscribed to every notification coming from the core library, it
has to run before any action can occur in the system as it is responsible of
propagating the requests from the core to the rule engine which will execute
the workflows accordingly to the expected behaviour.

## Start/Stop


```shell
docker run -it --rm radon-listener-image:latest \
           mvn exec:java \
           -Dexec.mainClass="org.radon.listener.RadonApp"
```

The image will be disposed of when the process is killed. When started, the 
listener will output the results of the message coming through and the different
rules which are effectively fire.
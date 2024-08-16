---
title: Radon Data Management System
permalink: /running/web.html
layout: default
sidebar_menu: Running
sidebar_sub: Radon Web
---

# Radon Web interface

Radon web serves the web interface and a set of REST API with a Django server.
Everything is configured automatically with a docker image which should be created
at install.


## Start/Stop

```shell
docker start radon-web
```

```shell
docker stop radon-web
```


## Usage

- Radon Web interface can be accessed at 
[http://radon.apps.l:8000](http://radon.apps.l:8000){:target="_blank"}

- The CDMI REST API is located at
[http://radon.apps.l:8000/api/cdmi](http://radon.apps.l:8000/api/cdmi){:target="_blank"}

- The microservices are served on
[http://radon.apps.l:8000/msi](http://radon.apps.l:8000/msi){:target="_blank"}


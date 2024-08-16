---
title: Radon Data Management System
permalink: /running/dse.html
layout: default
sidebar_menu: Running
sidebar_sub: DSE
---

# Running DSE tools

The Cassandra database is obtained via the Datastax DSE tool. It offers a
Cassandra database, configured to also manage graphs and search capabilities.
Additionally we are using Datastax Studio to visualise and interact with graphs.
Both tools come as Docker images, respectively named *dse* and *studio* in the
recommended installation guide.

## Start/Stop

```shell
docker start dse
docker start studio
```

```shell
docker stop studio
docker stop dse
```

## Usage

- For development, DSE can be accessed in a terminal with

```shell
docker exec -it dse cqlsh
```

- Studio can be accessed in a web browser at 
[http://radon-1.apps.l:9091](http://radon-1.apps.l:9091){:target="_blank"}

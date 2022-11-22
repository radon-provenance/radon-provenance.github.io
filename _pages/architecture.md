---
layout: default
title: System Architecture
description: Description of Radon architecture.
permalink: /architecture
---

{:.title}
{{ page.title }}

# Presentation

Radon is an open source Data Management System designed for large-scale storage.
It provides distributed, robust and extensible storage capabilities, coupling
an object store to an easy to use trigger/rules system.
Radon is essentially a classic separation of the catalogue from the data. That
is, you keep all the essential information about the object in one place,
optimized for directory type operations, and you store the object itself
elsewhere.  This is how traditional file systems work and classically how any
cataloguing system works, because there are many occasions where you want to
process the data about the object without actually accessing the data itself.
The catalogue provides so called CRUD operations for both metadata and objects
themselves, where CRUD is an acronym for Create, Update, Read and Delete.

This schema shows the different components of the system

![Architecture](/assets/images/architecture.png)

The next sections will present each components more precisely.


# Presentation Layer

## [Radon Web Application](https://github.com/radon-provenance/radon-web){:target="_blank"}

The Web Interface is run as a Docker image on several nodes of the Radon cluster. 
It is powered by Django/Gunicorn and provides a simple interface to the Data 
Management System via a Web browser.

All the CRUD functions are available through the interface and it displays the
hierarchy of the data objects/collections created in the system.

## [Radon Command Line Interface](https://github.com/radon-provenance/radon-cli){:target="_blank"}

Radon CLI is an independent Python client and command line tool that can be
used to dialog with a Radon cluster. It uses Radon RESTful APIs to communicate
with the system.


# Application Layer

## RESTful API

Radon provides several RESTful APIs that can be used to dialog with the archive.

#### [CDMI API](https://github.com/radon-provenance/radon-web){:target="_blank"}

The [Cloud Data Management Interface](https://www.snia.org/cdmi){:target="_blank"}
defines the functional interface that applications will use to create, retrieve, 
update and delete data elements from the cluster. As part of this interface the 
client is able to discover the capabilities that the cloud storage offers and 
use this interface to manage containers and the data that is placed in them. In 
addition, metadata can be set on containers and their contained data elements 
through this interface.

Radon supports a large part of the 1.1.1 specification.

#### [Admin API](https://github.com/radon-provenance/radon-web){:target="_blank"}

The Radon Admin RESTful API allows the remote management of the system. It
provides functions to manage users and groups.

## [Radon Library](https://github.com/radon-provenance/radon-lib){:target="_blank"}

The Radon library is the shared library for the Radon Data Management System. 
It is used by other Radon components to interact with the system, hiding the 
Cassandra model which shouldn't be queried directly.
It also provides a CLI that can be used on the node to manage the archive
directly.

## Graph

Radon can be configured to enable the graph store of the Cassandra database we are
using (DataStax Enterprise). This can be used to create alternate views on the
data or the metadata.

## Metadata Catalog

Wether data objects are stored in the radon archive or kept elsewhere, metadata 
are stored on Cassandra tables. This allows the standardization of the metadata
management, with a scalable approach.
It also describes how the data and the associated metadata are replicated on 
the cluster machines.

## Notification Queue

When *create*, *update*, or *delete* operations are performed on either a
*collection*,  a *resource* (*container* and *data object* respectively in CDMI
nomenclature), a *user* or a *group*, a message is sent to a MQTT broker. This
message contains information about the operation and resource.

An additional row is also written in a dedicated Cassandra table for audit 
purpose.

Radon is using the Eclipse Mosquitto message broker, run as a docker container.

## Workflow Engine

The workflow engine is a daemon that listens to the MQTT. It executes user 
scripts when events occur on the Radon Management System.

Current workflow engine is implemented with the Drools Rule Engine. Each event
appearing in the MQTT queue is transformed as a fact in Drools database,
then a set of rules can be triggered accordingly.


# Data Layer

## Radon Data Store

Radon offers the possibility to store resources binary data as blobs chunks in 
a Cassandra table. These binary datas are linked to the Radon hierarchy via 
their UUID.


## External Data Store

We are using the CDMI references to link the binary data of a resource to an
external repository. The reference object exists in the Radon hierarchy, with
the possibility to create metadata, ACL, ... but the actual data is stored
somewhere else.

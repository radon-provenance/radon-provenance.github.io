---
layout: default
title: Radon Web
description: Django Web Server for Radon.
permalink: /components/radon_web
---

{:.title}
{{ page.title }}


This component provides a web server which serves a web interface to access
the data management system and a set of RESTful APIs to access the data from
other applications. It is run as a Docker image on several nodes of the Radon 
cluster and is powered by Django/Gunicorn.

All the CRUD functions are available through the interface and it displays the 
hierarchy of the data objects/collections created in the system.

It requires the radon-lib library to work to access the data stored in the 
Cassandra database.


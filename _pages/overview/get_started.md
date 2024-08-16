---
title: Get Started
permalink: /get_started.html
layout: page
sidebar_menu: Overview
sidebar_sub: Get started
summary: These brief instructions will help you get started with the different
 interactions with the system when it is running.
---


{% include toc.html %}

# Default users

At the initialisation of the system, Radon creates some users and groups which can
be used to access the system. These groups and users have to be deleted or 
deactivated in a production environment.
Groups are 'users' and admins', they are used to define ACLs for collections and
resources.
Users are 'admin' and 'test', both with the password 'radon'. The 'admin' user has
full access to the system.


# Web interface

## Login

Once installed, Radon provides a web interface run in a radon-web docker image. It 
is available in a browser at [Radon Web Application](http://127.0.0.1:8000){:target="_blank"}.
The initial launch display a connection screen where the user can log to the 
system.

![RadonWeb](/assets/images/radon-web-login.png){:width="100%"}

## Hierarchy

Once logged we can see the main collections registered in the system. It's a
classical hierarchical data store, with available collections and resources in
the center of the interface.

![RadonWeb](/assets/images/radon-web-hierarchy.png){:width="100%"}

For each collections and resources we can edit the object to define additional 
user metadata, stored as Key/Value pairs.

![RadonWeb](/assets/images/radon-web-metadata.png){:width="100%"}


For admin users the left menu gives access to the management of the system with
the possible actions:

## Users

This interface can be used to add, modify a delete users. Users are stored in 
Radon but the system can be configured to use LDAP users. In that case we need 
to create a LDAP user in Radon with the same username and the authentication
in the system will be delegated to the LDAP server.

![RadonWeb](/assets/images/radon-web-users.png){:width="100%"}

## Groups

This interface can be used to add or delete groups and to add or delete users in
the groups. Groups are used to define {% include tooltip.html name="acl" %}
for object in the system, when an ACL is defined for an object, all users 
belonging to the groups will be affected.

![RadonWeb](/assets/images/radon-web-groups.png){:width="100%"}

## Activity

This is a list of all the latest activities in the system. An activity is an
operation occurring in the system, mainly a {% include tooltip.html name="crud" %}
operation on an object of the system. These activities are sent to a message
queue and the Radon listener will capture the message to generate an event in
for the rule engine.

![RadonWeb](/assets/images/radon-web-activity.png){:width="100%"}

## Settings

This is a page to configure some parameters of the system. For the moment it is
used to define the metadata keys that are indexed in the system. If a metadata
name match with one of the name in the list, Radon will create an index for this
term and it will be available in a Solr query.

![RadonWeb](/assets/images/radon-web-settings.png){:width="100%"}


# Command Line interface

Radon provides a {% include tooltip.html name="cli" %} application which can be 
used to interact with a running system from the terminal. It can be downloaded
from its own [Github Repository](https://github.com/radon-provenance/radon-cli){:target="_blank"}.
Once installed user can start a session with a system by providing the URL of a 
Radon Web server and a username/password with the command :

```
radon init --url=http://radon.example.com --username=USER --password=PASS
```

Once connected, users can use a set of commands to manage the system. For 
instance:

- `radon ls <path>` to display the content of a collection
- `radon put <src> <dst>` to upload a file to the system
- `radon admin lu` to list all existing users
- ...

A more detailed description of all available commands can be found 
[here](/usage/cli.html).


# CDMI REST API

Radon plaform runs a web server powered by Django. It provides an accessible
and well structured set of access methods to the data management system. Access 
methods comes in the form of RESTful services, accessible via the base url of a 
the node running radon-web. The object store provides a {% include tooltip.html name="cdmi" %}
implementation for HTTP access to digital objects in the registry. The CDMI 
standard is fairly well designed, and the richest of the available “standards”.

The object store can be used to organise collections and digital objects in the 
store. Radon implements a CDMI interface that defines the functional interface 
that applications may use to create, retrieve, update and delete data elements 
from the Object Store ({% include tooltip.html name="crud" %} operations). In 
addition, metadata can be set on collections and their contained data elements 
through this interface. The CDMI web service is accessible at /api/cdmi from the 
root URI.

These are some examples of the queries which can be sent:
  - `PUT <root URI>/api/cdmi/<CollectionName>/<NewCollectionName>/` tp create a
   new collection
  - `GET <root URI>/api/cdmi/<CollectionName>/<TheCollectionName>/` to list a 
   collection
  - `DELETE <root URI>/api/cdmi/<CollectionName>/<DataObjectName>` to delete a 
   data object
  - ...
 
A more detailed description of all available commands can be found 
[here](/usage/cdmi.html).
 
 

---
title: Radon Data Management System
permalink: /technical.html
layout: default
sidebar_menu: Overview
sidebar_sub: Technical Description
summary: This is a technical description of the different components of the Radon
 architecture.
---


{% include toc.html %}


# Core library

## Code

The core library, written in Python, serves as a powerful tool for managing the 
Cassandra data model. It abstracts the complexities of interacting with 
Cassandra by hiding the underlying CQL (Cassandra Query Language) queries that 
are used to create and update rows in the database. This abstraction layer 
simplifies the development process, allowing developers to focus on business 
logic rather than database operations.

The code can be downloaded on its own github repository : 
[Radon Library](https://github.com/radon-provenance/radon-lib){:target="_blank"}

It is used by the radon web package but it can be installed locally to allow 
access to a Radon instance in a Python application (This would be for testing 
only, the REST api being the main entry point to the system)

```shell
python3 -m venv ~/ve/radon-lib
source ~/ve/radon-lib/bin/activate
pip install -r requirements.txt
python setup.py develop
```

## Cassandra Data Model


![Cassandra Data Model](/assets/images/cassandra_data_model.png){:width="100%"}

  _**Tree Node**_: The tree node table stores the virtual hierarchy of the data
management system in rows. A row can be a collection or a resource, according
to the *is_object* column. An element of the hierarchy is uniquely identified
by its full path from the root and a *version* number, this full path is the
concatenation of the *container* (aka its parent) and its *name*. Following
the CDMI standard, a collection name ends with a \"/\". For a resource, the actual
data bits are referenced with its *object_url*, for an object stored in Radon
it will be in the form of "radon://object_uuid".
*sys_meta* and *user_meta* are a dictionary of key/values pair where values are
stored in JSON format. *sys_meta* are automatically managed by Radon while 
*user_meta* can be modified with the different client interfaces. Finally ACL is
a dictionary of ACE defined for the different groups.

  _**Data Object**_: Data objects are binary bits stored in a Cassandra table as
  *blobs*. They can be split as sequence of blobs if we don't want to store too 
  large chunks of data. It also stores some properties like *checksum*, *size*
  and *compressed*.

  _**Ace**_: Following CDMI standard, ACE are Access Control Entries, they
  define if objects are authorised and permitted or denied. They apply to an
  *identifier* which is a group in Radon. They contain an *acetype* which 
  defines if the entry is Allow or Deny. The *aceflag* describe how the 
  permissions are inherited from the parent collections and the *acemask*
  defines the permissions.

  _**Group*_: Radon has its own way to define groups. It is used to set ACL for
  a set of users.
  
  _**User**_: Radon has a list of users who can log to the system and access
  the data objects, the *login* is a unique identifier. *administrators* have 
  full access to the system management. If needed, authentication can be
  delegated to an LDAP server (in that case the *ldap field should be set to
  "true"  and a Radon user with a name matching the login in LDAP has to be 
  defined). *Groups* is a list of group names the user belongs to.
  
  _**Config**_: These are a list of dynamic options stored in Radon.
  
  _**Notification*_: Radon stores a copy of the notifications sent to the MQTT
  server when an action takes place in the system. This table can be parsed to
  audit the system and get a log of what happened in the system.



## MQTT messages

Because the script that interacts with the Radon System is running as a
different process (or possibly on a different machine), normal intra-process
communication will not work, so some form of inter-process or network
communication is needed. Radon uses [MQTT](http://mqtt.org/){:target="_blank"} 
for this communication, which is an example of a publish/subscribe model. MQTT 
has a central "broker" that receives all messages and forwards them on to 
clients who have subscribed to a topic or topics. In the case of MQTT, topics 
are a text string in the form of a UNIX-style path or URI, which defines a 
hierarchy. For instance `create/resource/path`.

The format of topics used in the listener is:

`<Operation>/>OperationType>/<ObjectType>/<ObjectIdentifier>`

Where:
  * `<Operation>` can be "create", "delete" or "update"
  * `<OperationType>` can be "request", "success" or "fail"
  * `<ObjectType>` can be "resource", "collection", "user" or "group"
  * `<ObjectIdentifier>` can be
    * `/collection/collection/...` for a collection
    * `/collection/collection/.../resource` for a resource
    * `username` or `groupname` for user and group

Subscribers can match all or part of a topic using the wildcards `#` and `+`,
where `+` matches a single level of the hierarchy and `#` matches a complete
sub-tree of the hierarchy and must be the last character in the topic if it is
used. With the current implementation, the radon listener subscribes to all 
messages and transforms the MQTT messages to facts that it can use to represent
 the state of the system at a given time.


## Radon Search


### Presentation

Radon Search is built on top of DSE Search. It uses Apache Solr to store indexes
in specific tables in the Cassandra database. Radon provides a set of default 
indexes to search on object paths in the repository and on metadata on a 
user-defined set of metadata names.


### Configure fields

#### Path

"**path**" is the default field, it is created when the system is initialized and
can be used to search for objects by their names or collection names.


#### Metadata

A set of DC field are created when the system is setup. If a metadata with the
same name is used, it can be indexed and searched with a solr query.


![Metadata fields](/assets/images/metadata_field.png){:width="100%"}


### Queries

Radon search text field in the web interface can be used to pass solr queries, using
the declared fields.

For instance:

```
path:test
path:te*
path:(test or user)

dc_creator:jerome
...
```


## Radon Microservices


Most of Radon low-level functionalities are exposed through a RESTful API. This
allows the rule engine, written in Java, to execute code of the core API, written
in Python when its runs workflows associated to triggered rules.

Here is a list of the existing microservices:

- create_collection
- create_resource
- create_group
- create_user
- delete_collection
- delete_group
- delete_resource
- delete_user
- update_collection
- update_resource
- update_user
- update_group

Each microservice is accessible at the URL "http://127.0.0.1:8000/msi/msi_name",
with parameters passed as JSON input.


## Rules

Drools rules are defined in the file **src/main/resources/META-INF/rules.drl**, it 
follows the Drools Rule Language ([DRL](https://docs.jboss.org/drools/release/5.2.0.Final/drools-expert-docs/html/ch05.html){:target="_blank"}) The default file shows some example of the syntax for the 
different rules. The current configuration needs the Docker image to be 
recompiled to modify the rules as they are not loaded dynamically.

The right part of the rules are Java code, they can use code define in the file
**workflows/Microservices.java**. The **Microservices.execScript(cmd)** can
be used to execute command line scripts but the main currently used microservices
are the Radon REST API encapsulated as Java calls.


### Existing rules :

To define the behaviour of a standard data management system, we have designed
a minimal set of rules to describe how the system should react when a user
interacts with the system:

```java
rule "CreateRequest User"
    when
        /createRequestUser[user : user, meta : meta]
    then
        System.out.println("Request user creation: " + user.getLogin());
        Microservices.createUser(user, meta);
end


rule "CreateRequest Group"
    when
        /createRequestGroup[group : group, meta : meta]
    then
        System.out.println("Request group creation: " + group.getName());
        Microservices.createGroup(group, meta);
end


rule "CreateRequest Collection"
    when
        /createRequestCollection[coll : coll, meta : meta]
    then
        System.out.println("Request coll creation: " + coll.getContainer() + "/" + coll.getName());
        Microservices.createCollection(coll, meta);
end


rule "CreateRequest Resource"
    when
        /createRequestResource[resc : resc, meta : meta]
    then
        System.out.println("Request resc creation: " + resc.getContainer() + "/" + resc.getName());
        Microservices.createResource(resc, meta);
end



/***************************** REQUEST UPDATE ****************************************/


rule "UpdateRequest User"
    when
        /updateRequestUser[user : user, meta : meta]
    then
        System.out.println("Request user update: " + user.getLogin());
        Microservices.updateUser(user, meta);
end


rule "UpdateRequest Group"
    when
        /updateRequestGroup[group : group, meta : meta]
    then
        System.out.println("Request group update: " + group.getName());
        Microservices.updateGroup(group, meta);
end


rule "UpdateRequest Collection"
    when
        /updateRequestCollection[coll : coll, meta : meta]
    then
        System.out.println("Request coll update: " + coll.getContainer() + "/" + coll.getName());
        Microservices.updateCollection(coll, meta);
end


rule "UpdateRequest Resource"
    when
        /updateRequestResource[resc : resc, meta : meta]
    then
        System.out.println("Request resc update: " + resc.getContainer() + "/" + resc.getName());
        Microservices.updateResource(resc, meta);
end



/***************************** REQUEST DELETE ****************************************/



rule "DeleteRequest User"
    when
        /deleteRequestUser[user : user, meta : meta]
    then
        System.out.println("Request user delete: " + user.getLogin());
        Microservices.deleteUser(user, meta);
end


rule "DeleteRequest Group"
    when
        /deleteRequestGroup[group : group, meta : meta]
    then
        System.out.println("Request group delete: " + group.getName());
        Microservices.deleteGroup(group, meta);
end


rule "DeleteRequest Collection"
    when
        /deleteRequestCollection[coll : coll, meta : meta]
    then
        System.out.println("Request collection delete: " + coll.getContainer() + "/" + coll.getName());
        Microservices.deleteCollection(coll, meta);
end


rule "DeleteRequest Resource"
    when
        /deleteRequestResource[resc : resc, meta : meta]
    then
        System.out.println("Request resource delete: " + resc.getContainer() + "/" + resc.getName());
        Microservices.deleteResource(resc, meta);
end
```



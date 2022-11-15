---
layout: default
---

# Radon Listener


## Presentation

The listener is a daemon that executes user scripts when events occur in the 
Radon Management System. Current workflow engine is implemented with the Drools 
Rule Engine. Each event appearing in the MQTT queue is transformed as a fact in
Drools database, then a set of rules can be triggered accordingly.


## Overview of operation

When *create*, *update*, or *delete* operations are performed on either a
*collection*,  a *resource* (*container* and *data object* respectively in CDMI
nomenclature), a *user* or a *group*, a message is sent to a MQTT broker running
on the local machine containing information about the operation and resource.
On the other side the listener is subscribed to the queue and creates facts in
the Drools database. The rule engine can then compare the fact database with its
rule database to see if the conditions of one if the rule is valid. If that's the
case it can execute the right part of the according rule.

### Messages

Because the script that interacts with the Radon System is running as a
different process (or possibly on a different machine), normal intra-process
communication will not work, so some form of inter-process or network
communication is needed. Radon uses [MQTT](http://mqtt.org/) for this
communication, which is an example of a publish/subscribe model. MQTT has a
central "broker" that receives all messages and forwards them on to clients who
have subscribed to a topic or topics. In the case of MQTT, topics are a text
string in the form of a UNIX-style path or URI, which defines a hierarchy.
For instance `create/resource/path`.

The format of topics used in the listener is:

`<Operation>/<ObjectType>/<ObjectIdentifier>`

Where:
  * `<Operation>` can be "create", "delete" or "update"
  * `<ObjectType>` can be "resource", "collection", "user" or "group"
  * `<ObjectIdentifier>` can be
    * `/collection/collection/...` for a collection
    * `/collection/collection/.../resource` for a resource
    * `username` or `groupname` for user and group

Subscribers can match all or part of a topic using the wildcards `#` and `+`,
where `+` matches a single level of the hierarchy and `#` matches a complete
sub-tree of the hierarchy and must be the last character in the topic if it is
used. With the current implementation Drools subscribes to all message and 
transforms the MQTT messages to facts that it can use to represent the state of 
the system at a given time.


### Rules

Drools rules are defined in the file src/main/resources/META-INF/rules.drl, it 
follows the Drools Rule Language ([DRL](https://docs.jboss.org/drools/release/5.2.0.Final/drools-expert-docs/html/ch05.html))
The default file shows some example of the syntax for the different rules. It
mainly displays output in the command line to validate that the communication
is working between Radon and the rule engine.

The right part of the rules are Java code, there's a method that can be used
to execute command line scripts but this is still a work in progress.
See SystemCommands.execScript(cmd) in the file SystemCommands.java. The idea
would be to have a full set of microservices to manipulate the objects of the
repository from the right part of the rules.


### Run the rule engine

The rule set will probably need to be mounted from a volume in a future
version to simplify the use.

```shell
docker run -it --rm radon-listener-image:latest mvn exec:java -Dexec.mainClass="org.radon.listener.RadonApp"
```




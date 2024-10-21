---
title: Radon Data Management System
permalink: /tutorial/rules.html
layout: default
sidebar_menu: Tutorial
sidebar_sub: Working with rules
---

# Working with rules

{% include note.html content="The current status of the listener which executes
the Drools rule engine still requires a lot of code to be written in the listener
package directly." %}

{% include note.html content="Rules are managed by Drools in its own Docker
image. For the moment it is necessary to update the Docker image when we need
to modify the ruleset." %}

{% include note.html content="Rules actions are following DRL Syntax which uses 
Java code to define the actions to be performed. Some helper functions are
already available to call RESTful API or commandline applications but they will
need to be adapted." %}


## Modifying a rule

In the github project, rules are located in the file 
```radon-listener/src/main/resources/org/radon/listener/rules.drl```. A 
modification of the rule requires a restart of the listener process.

The right part of the rule is using Java code of the radon-listener, organised
in the package org.radon.listener.workflows. Classes that contains code to be
used in the rules have to be declared at the top of the rule file like a Java file. 


## Conditions of the rules

Each CRUD action occuring in the system generates a fact in the Drools fact store
Create, Update or Delete). They can be used as a condition of a rule. This is 
the list of the existing facts users can use in the rules:

- createRequestCollection, createSuccessCollection, createFailCollection
- createRequestUser, createSuccessUser, createFailUser
- createRequestResource, createSuccessResource, createFailResource
- createRequestGroup, createSuccessGroup, createFailGroup

- deleteRequestCollection, deleteSuccessCollection, deleteFailCollection
- deleteRequestUser, deleteSuccessUser, deleteFailUser
- deleteRequestResource, deleteSuccessResource, deleteFailResource
- deleteRequestGroup, deleteSuccessGroup, deleteFailGroup

- updateRequestCollection, updateSuccessCollection, updateFailCollection
- updateRequestUser, updateSuccessUser, updateFailUser
- updateRequestResource, updateSuccessResource, updateFailResource
- updateRequestGroup, updateSuccessGroup, updateFailGroup

Requests are sent before the action, they are used to initiate the action in the
system. Success are sent when the action has been successful while Fail are sent
when an error occured.

Each facts have two parameters, the objects implied in the action (Resource,
Collection, User or Group) and a Metadata object which stores additional 
informations on the actions. For the moment only the user who initiated the action.

### Examples of conditions:

 - Check that a user asked for the deletion of a group
 
  ```
  when /deleteRequestGroup[group : group, meta : meta]
  ```
  
 - Check that a user asked for the creation of a resource
 
  ```
  when /createRequestResource[resc : resc, meta : meta]
  ```

 - Check that a user asked for the creation of a collection
 
 ```
 when /createSuccessCollection[coll : coll, meta : meta]
 ```
 
 - Check that a user asked for the creation of a resource in a specific collection
   named ```/test```
    
 ```
 when
    c: /createSuccessResource[resc : resc, meta : meta]
    eval( c.getResc().getContainer() == "/test/" )
``` 


### Examples of actions

Drools rule language allows the import of Java code to be used in the right part
of the rules to define actions associated to a condition. Code is located in the 
radon-listener project, more precisely in the package ```org.radon.listener.workflows```.

- SystemCommands is an example of a method which can be used to execute a command 
line application. This application has to be included in the Docker image somehow
and is not the most straightforward way to integrate other workflows.

- Microservices is the module Radon is using to call microservices served by
the radon-web component. It contains helper functions to manage the low-level 
operations needed to operate the archive like creating or deleting objects. This
class can be used as an example to connect to an external web service REST api.

Examples of rules action can be : 

- Printing a log message :

```
System.out.println("Request user creation: " + user.getLogin());
```

- Creating a new collection : 

```
Microservices.createCollection(coll, meta);
```

where coll is a Collection object which contains the correct payload expected
by Radon API. So a JSON object with parent container and the name of the new
collection.

```
{ 
  "obj": {
    "name" : "NewCollection",
    "container": "/test/
}
```


## Examples of new rules


### Creation of a new microservice which calls a REST api


- In file ```radon-listener/src/main/java/org/radon/listener/workflows/Microservices.java```:

```java
public static String callMyMicroservice(String parameters) {
    try {
        targetURI = new URI("http://127.0.0.1:8000/myMethod/");
        System.out.println("Calling " + targetURI);
        HttpRequest httpRequest = HttpRequest.newBuilder()
                .uri(targetURI)
                .POST(HttpRequest.BodyPublishers.ofString(parameters))
                .build();
        HttpClient httpClient = HttpClient.newHttpClient();
        HttpResponse<String> response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString());
        if (response.statusCode() == 200) {
            return response.body();
        }
        
    } catch (URISyntaxException | IOException e) {
        logger.error("Problem with the microservice call");
    } catch (InterruptedException  e) {
        logger.error("Problem with the microservice call");
        Thread.currentThread().interrupt();
    } 
    return "";
}
```

### Creation of a new rule which calls the microservice for each creation of ".txt" resource

```java
rule "Create Txt Resource"
    when
        c: /createSuccessResource[resc : resc, meta : meta]
        eval( c.getResc().getName().endsWith("csv") )
    then
        System.out.println("Calling my microservice for resource : " c.getResc().getName());
        Microservices.callMyMicroservice("test");
end
```

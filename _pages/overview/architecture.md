---
title: Radon Architecture
permalink: /architecture.html
layout: page
sidebar_menu: Overview
sidebar_sub: Architecture
summary: This is a description of the different components of the Radon
 architecture and how they are linked together.
---


{% include toc.html %}

# Introduction


## Purpose

Radon is an open-source Data Management System designed to address the 
challenges of large-scale storage environments. It provides a distributed, robust, 
and extensible storage solution that seamlessly integrates an object store with a 
user-friendly trigger/rules system. This combination ensures that Radon can 
handle vast amounts of data efficiently while offering flexibility and scalability.

One of Radon's key feature is its separation of the catalogue from the data. 
This means that all essential metadata about an object is stored in a catalogue, 
optimised for directory-type operations, while the actual object data can be 
stored in a different location. This design mirrors traditional file systems and 
cataloguing systems, which is advantageous because it allows users to process 
metadata without needing to access the underlying data. This separation is 
particularly useful in scenarios where quick access to metadata is required for 
operations such as indexing, searching, and managing data lifecycles.

The catalogue in Radon supports comprehensive CRUD (Create, Read, Update, Delete) 
operations for both metadata and the objects themselves. This functionality 
provides a flexible and efficient way to manage and interact with stored data. 
Users can easily create new entries, read and retrieve existing data, update 
information as needed, and delete obsolete or unnecessary data. This robust CRUD 
capability ensures that Radon can meet the diverse needs of various applications 
and use cases, from simple data storage to complex data management workflows.

Additionally, Radon's trigger/rules system enhances its functionality by 
allowing users to define automated actions based on specific events or conditions. 
This feature can be used to automate routine tasks, enforce data policies, and 
ensure data integrity and consistency across the system. For example, users can 
set up rules to automatically archive data after a certain period, trigger 
alerts when data thresholds are reached, or enforce access controls based on 
metadata attributes.

Overall, Radon is designed to provide a powerful, scalable, and user-friendly 
data management solution that can adapt to the evolving needs of large-scale 
storage environments. Its innovative architecture, combined with its robust 
feature set, makes it an ideal choice for organisations looking to efficiently 
manage and leverage their data assets.

## Radon Key functionalities


- _**Distributed and Robust Storage Solution**_: Radon offers a scalable and 
extensible storage system that integrates an object store with a user-friendly 
trigger/rules system, ensuring efficient handling of large-scale data.

- _**Separation of Catalogue and Data**_: Essential metadata about objects is 
stored in a catalogue optimised for directory-type operations, while the actual 
object data is stored separately. This design allows for quick metadata 
processing without accessing the underlying data, facilitating operations like 
indexing, searching, and data lifecycle management.

- _**Comprehensive CRUD Operations**_: Radon's catalogue supports full CRUD 
(Create, Read, Update, Delete) operations for both metadata and objects. Users 
can easily manage data by creating new entries, retrieving existing data, 
updating information, and deleting obsolete data, catering to diverse 
application needs.

- _**Automated Trigger/Rules System**_: Users can define automated actions based 
on specific events or conditions. This feature helps automate routine tasks, 
enforce data policies, and maintain data integrity and consistency. Examples 
include automatic data archiving, alert triggers for data thresholds, and access 
control enforcement based on metadata attributes.

- _**Scalable and User-Friendly Architecture**_: Radon's architecture is 
designed to adapt to the evolving needs of large-scale storage environments. Its 
innovative design and robust feature set make it an ideal choice for 
organisations aiming to efficiently manage and leverage their data assets


## Target Audience

This documentation is intended for developers, system architects, and IT 
administrators who are involved in the design, development, deployment, and 
maintenance of the Radon data management system. It provides a detailed 
understanding of the system's architecture, components, and operational aspects.


## Scope

This document covers the architectural aspects of the middleware application, 
including its high-level design, component descriptions, data flow, deployment 
strategies, monitoring, and security considerations. It aims to provide a 
comprehensive overview that aids in the effective implementation and management 
of the middleware.


## Technologies Used

- _**Programming Languages**_: The core library is written in Python 3. The rule 
engine code is using Java.

- _**Frameworks and Libraries**_: Django, Bootstrap, Drools, Docker.

- _**Databases**_: Cassandra

- _**Communication Protocols**_: MQTT, REST Api



# System Architecture


## Overall architecture

Radon benefits from the scalability of a Cassandra ring of nodes to store the
different data it manages. We are using {% include tooltip.html name="dse" %} which provides
a graph database and search capabilities using {% include tooltip.html name="solr" %}.
Following the {% include tooltip.html name="cdmi" %} standard data objects are 
called resources. They represent a registered entity in the catalogue, this can
be any kind of file or document and can be stored directly in Radon or referenced
via an URL to an external data store. On top of these data objects Radon maintains
a virtual hierarchy to organise them in collections. Every registered entities 
(data objects or collections) have an associated set of metadata, stored as 
key/values pairs. Additionally a graph structure can be included to offer a 
different view on the data, according to the use cases. 

The core library is written in Python, it manages the Cassandra data model and
hides the underlying {% include tooltip.html name="cql" %} queries which create
and update rows in the database. It provides a Python implementation of a
Radon object model, with a set of Python classes which can be used in applications
to access data stored in the database. Additionally the main functions are 
exposed through a RESTful api, allowing access to Radon objects from a number of
external applications.

Every operation in the system generates a notification which is sent to an MQTT
server. This notification contains a payload with all the information on the
objects impacted by the operation. On the other end of the queue Radon provides
a listener which subscribes to the notification queue. When a message is received
it creates a fact in the rule engine database which describes the event. The rule 
engine will evaluate the fact and if it matches one of its registered rule it 
will trigger the associated rule action. By refining the rules definitions it is
possible to adapt the behaviour of the system to specific use cases.

On top of that Radon provides a set of interfaces to access the system. The web 
interface can display the virtual hierarchy of the collections and data objects
of the system as well as metadata and search queries. It is the simplest way to
manage the system, defining ACL or search indices. For a wider integration with
terminal applications, Radon provides a command line interface based on the REST
api. It allows end users to navigate in the collections and upload or download
data objects from the system. Radon also serves a Datastax Studio graph editor,
accessible from a web browser. It's still a work in progress but this can be used
to send Gremlin queries to a Radon instance to visualise and manipulate graph data.

## High-Level Architecture Diagram


![Architecture](/assets/images/architecture.png){:width="100%"}


## Components Description

### Client Interfaces

- _**Web Interface**_

The web interface for the data management system incorporates user and group 
authentication mechanisms to ensure secure access. Users log in with unique 
credentials, authentication can be delegated to an LDAP server. Group management
allows users to be part of specific groups with predefined access levels, ensuring 
appropriate permissions. Upon login, users are directed to the hierarchy of 
collections and data objects, while admins have access to additional web pages
for managing users, groups, and system settings. The data management section 
allows users to upload and download data files in various formats. Resources
are organised into collections and subcollections, with permissions to create or
delete collections. Metadata management features include adding, editing, and 
deleting metadata for each resources and collections, with open fields which can
be adapted to use cases. Advanced search functionalities can be customised to
create indices on specific metadata. User and group management tools allow 
admin users to view and edit profiles, create and manage groups, assign users 
and define permissions. Fine-grained access control ensures appropriate 
permissions for groups for collections or resources. The interface is designed 
to be responsive and intuitive, using Bootstrap5 and a Django server.

Some screenshots:

<div id="radonWebCarousel" class="carousel carousel-dark slide">

  <div class="carousel-indicators">
    <button type="button" data-bs-target="#radonWebCarousel" data-bs-slide-to="0" class="active" aria-current="true" aria-label="Slide 1"></button>
    <button type="button" data-bs-target="#radonWebCarousel" data-bs-slide-to="1" aria-label="Slide 2"></button>
    <button type="button" data-bs-target="#radonWebCarousel" data-bs-slide-to="2" aria-label="Slide 3"></button>
    <button type="button" data-bs-target="#radonWebCarousel" data-bs-slide-to="3" aria-label="Slide 4"></button>
  </div>

  <div class="carousel-inner">
    <div class="carousel-item active">
      <img src="/assets/images/radon-web-login.png" class="d-block w-100" alt="...">
    </div>
    <div class="carousel-item">
      <img src="/assets/images/radon-web-hierarchy.png" class="d-block w-100" alt="...">
    </div>
    <div class="carousel-item">
      <img src="/assets/images/radon-web-metadata.png" class="d-block w-100" alt="...">
    </div>
    <div class="carousel-item">
      <img src="/assets/images/radon-web-users.png" class="d-block w-100" alt="...">
    </div>
  </div>
  <button class="carousel-control-prev" type="button" data-bs-target="#radonWebCarousel" data-bs-slide="prev">
    <span class="carousel-control-prev-icon" aria-hidden="true"></span>
    <span class="visually-hidden">Previous</span>
  </button>
  <button class="carousel-control-next" type="button" data-bs-target="#radonWebCarousel" data-bs-slide="next">
    <span class="carousel-control-next-icon" aria-hidden="true"></span>
    <span class="visually-hidden">Next</span>
  </button>
</div>

- _**Graph Editor**_

The graph editor component is a versatile tool integrated into web browsers, 
designed to facilitate the creation, visualisation, and manipulation of graphs. 
This component offers an interface that allows users to easily add, remove, and 
edit nodes and edges, making it ideal for a wide range of applications, from 
academic research to business analytics.

One of the standout features of this graph editor is its seamless integration 
with the data management system. This linkage ensures that any changes made 
within the graph editor are automatically synchronised with the underlying 
data repository. Users can import data directly from the management system, 
enabling the visualisation of complex datasets in a graphical format. 
Conversely, modifications made within the graph editor can be exported back to 
the data management system, ensuring data consistency and integrity.

The web-based nature of the graph editor component means it is accessible from 
any device with an Internet connection, providing flexibility and convenience. 
It stores its data in the Cassandra database, allowing scalability of the model. 
Additionally, the component makes use of Gremlin queries that allow users to 
perform operations like shortest path calculations, clustering, and centrality
 measures, providing deeper insights into the data.

Overall, the graph editor component serves as a powerful bridge between data 
visualisation and data management, enhancing the user's ability to understand 
and manipulate complex datasets effectively.


![Graph Editor Screenshot](/assets/images/radon-web-graph.png){:width="100%"}

- _**Command Line Interface**_

The Radon CLI is a robust command line interface designed to facilitate seamless 
interaction with a comprehensive data management system. Accessible directly 
from the terminal, this CLI component empowers administrators with a wide array 
of commands to manage, query, and manipulate data efficiently. Built on a REST 
API, the CLI ensures high performance and reliability, leveraging the RESTful 
architecture for backend communication. Administrators can perform various tasks, 
including user authentication and management with commands like login, logout, 
create user or delete group. Data operations are streamlined through commands 
such as list collection, put data object, add metadata or delete entity, enabling
effective CRUD operations. The CLI's integration with the REST API ensures 
real-time execution of commands, delivering immediate feedback and results. 
Overall, the Radon CLI is an indispensable tool for administrators, 
offering a streamlined and efficient way to manage data within a terminal
environment, with scalability and flexibility suitable for a wide range of 
data management tasks.

- _**RESTful API**_

The RESTful {% include tooltip.html name="api" %} serves as a bridge between 
clients and the data management system, enabling seamless communication and data 
exchange. This API adheres to the principles of Representational State Transfer 
(REST), which ensures that it is stateless, scalable, and capable of handling 
a variety of data formats such as JSON and XML. By leveraging standard HTTP 
methods like GET, POST, PUT and DELETE, the API allows clients to perform CRUD 
(Create, Read, Update, Delete) operations on the data stored within the 
management system.

One of the key features of this RESTful API is its client-agnostic nature. This 
means that it can be accessed from any client, whether it's a web application, 
mobile app, or even a command-line tool. The API endpoints are designed to be 
intuitive and consistent, making it easy for developers to integrate and 
interact with the data management system, it adheres to the CDMI standard. 
Authentication and authorisation mechanisms ensure that only authorised clients 
can access or modify the data, maintaining the security and integrity of the 
system. Through the RESTful API, clients can query, filter, and sort data, as 
well as perform complex operations like batch processing and data aggregation. 

- _**Microservices**_

In a modern software architecture, a set of microservices is designed to provide 
robust and scalable functionalities. These microservices are accessible through 
RESTful APIs, ensuring that they can be easily consumed by any client, 
regardless of the platform or technology stack. Each microservice is responsible 
for a specific capability, promoting a modular and maintainable codebase.

The microservices are seamlessly integrated with the Radon data management 
system, they offer full CRUD (Create, Read, Update, Delete) functionalities. 
This system ensures that data operations are efficient and consistent across 
all services. By leveraging RESTful APIs, clients can perform CRUD operations 
on the underlying data, enabling dynamic and real-time interactions with the 
system.

The architecture is designed to be highly flexible, allowing for easy scaling 
and deployment of individual microservices. This flexibility ensures that the 
system can adapt to varying loads and evolving requirements. Additionally, the 
use of RESTful APIs facilitates integration with external systems and services, 
enhancing the overall interoperability and extensibility of the platform.

One of the main usage for this microservices is for the development of the 
rulebase for the rule engine. Each microservices are accessible from the right
part or the rule, the worklflow associated to a condition, allowing the 
creation of complex workflows to define the behaviour of the system upon
notifications.


### Middleware Core:

- _**Core Library**_

The core library, written in Python, serves as a powerful tool for managing the 
Cassandra data model. It abstracts the complexities of interacting with 
Cassandra by hiding the underlying CQL (Cassandra Query Language) queries that 
are used to create and update rows in the database. This abstraction layer 
simplifies the development process, allowing developers to focus on business 
logic rather than database operations.

At the heart of this library is a Python implementation of a Radon object model. 
This model provides a structured and intuitive way to interact with the data 
stored in Cassandra. The library includes a comprehensive set of Python classes 
that represent various entities and relationships within the database. These 
classes can be seamlessly integrated into applications, enabling developers 
to perform CRUD (Create, Read, Update, Delete) operations on the data with ease.

By leveraging this core library, applications can efficiently manage data 
stored in Cassandra without needing to write complex CQL queries. The library 
ensures that all data operations are performed in a consistent and optimised 
manner, enhancing both performance and reliability. Additionally, the use of 
Python classes to represent the data model promotes code reusability and 
maintainability, making it easier to build and scale applications over time.


- _**Notification server and Rule Engine**_

In the system, every operation performed generates a notification that is sent 
to a dedicated MQTT server. This notification is comprehensive, containing a 
payload with detailed information about all the objects impacted by the 
operation. This ensures that every relevant piece of data is captured and 
transmitted efficiently, maintaining the integrity and completeness of the 
information flow. It also provide audit logs to analyse what happened in the 
system.

On the receiving end of this setup, Radon provides a highly efficient listener 
that subscribes to the notification queue. This listener is constantly on the 
lookout for incoming messages. Upon receiving a message, it immediately creates 
a fact in the rule engine database. This fact is a precise description of the 
event that has occurred, encapsulating all necessary details. The fact database 
models the current state of the data management system.

The rule engine then takes over, evaluating the newly created fact. It checks 
if the fact matches any of the registered rules within its system. If a match 
is found, the rule engine triggers the corresponding rule action. This automated 
response mechanism ensures that the system can react swiftly and appropriately 
to various events, enhancing its overall efficiency and responsiveness.

One of the key strengths of this system is its flexibility. By refining the 
definitions of the rules, it is possible to tailor the system's behaviour to meet 
specific use cases. This means that the system can be adapted to handle a wide 
range of scenarios, making it highly versatile. Whether the requirements change 
or new use cases emerge, the system can be adjusted accordingly, ensuring it 
remains relevant and effective.

Furthermore, this adaptability allows for continuous improvement. As new rules 
are defined and existing ones refined, the system evolves, becoming more adept 
at handling complex situations. This dynamic nature ensures that the system not 
only meets current needs but is also prepared for future challenges.



### Backend services

_**Radon Data Store**_

Radon is configured to use a ring of Datastax {% include tooltip.html name="dse" %} 
nodes for its database. It's built on the Apache Cassandra NoSQL database, with
interesting components integrating graph, search and analytics capabilities.

Radon uses several tables to manage different kind of data

  - A *data store* to keep resources binary data as blobs chunks in a Cassandra 
table. These binary datas are linked to the Radon hierarchy via their UUID.
 
  - A *hierarchical store* to create a logical structure which organise data in
collections/sub-collections. Following the CDMI standard, data objects are 
referenced with a URL, internal for a data object stored in Radon or external if
we want to link to a binary object stored in an external data store.

  - A *metadata store* which can keep metadata to any data object or collection,
internal or referenced. Metadata are stored as lists of Key/Values pairs, with 
values stored as JSON objects.

  - A *graph store* which can be used to create alternative views of the data.
This is still a work in progress, graphs can be created through Gremlin queries
sent via the Radon lib to the DSE server.


_**Rule Engine Store**_

The rule engine is implemented with Drools. It has its own fact store to keep the
state of the system. Rules for the engine are stored in text files. Ultimately
it would be interesting to store everything in the Cassandra database too.



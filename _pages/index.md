---
title: Radon Data Management System Overview
permalink: /
layout: page
sidebar_menu: Overview
sidebar_sub: Presentation
---


Radon is an open source Data Management System designed for large-scale storage.
It provides distributed, robust and extensible storage capabilities, coupling
an object store with metadata capabilities to a rule engine that can react to
events in the system and execute ad hoc workflows. This combination allows
for scalable systems that can self-manage, are not reliant on complex workflow
systems or external management frameworks. Relatively simple configurations
allow for distributed ingest, transformation and curation, without the need to
have complex scheduling frameworks.


<div class="d-flex justify-content-center">
<map name="mainmap">
  <area 
    alt="Storage"
    title="Storage"
    href="#storage"
    shape="poly"
    coords="0,0,345,0,345,95,250,95,220,115,210,145,0,145" />
  <area 
    alt="Access"
    title="Access"
    href="#access"
    shape="poly"
    coords="0,157,210,157,220,190,250,210,345,210,345,300,0,300" />
  <area 
    alt="Discovery"
    title="Discovery"
    href="#discovery"
    shape="poly"
    coords="352,0,705,0,705,145,490,145,485,145,445,95,352,95" />
  <area 
    alt="Workflows"
    title="Workflows"
    href="#workflows"
    shape="poly"
    coords="352,210,445,210,485,190,495,157,705,157,705,300,352,300" />
</map>
<img src="/assets/images/main.png" usemap="#mainmap"/>
</div>


## Storage

Radon supports data sharing, or interoperability between external data stores.
This “virtualisation” capability is important because of the need to operate
across radically different types of storage and processing technologies; and
for the ability of users to access and share data. The actual data (e.g. the
blob of a digital object) can be held in Radon or referenced via its URL.

Radon supports the generation and maintenance of audit trails for specific
operations or events. This is important in order to trace the history of the
operations, and to prove that all operations on the digital entities were
performed by authorised users, including the update of the authenticity
metadata itself.

Radon supports authentication as part of its ability to track the identity of
users working in a high-security environment. This is important given
confidentiality and data protection requirements. Radon is controlled using
standardised ACL (access control lists), which can be applied at any level and
will apply to the sub-tree in the same way as storage policy.


## Access

Radon data model creates a logical hierarchy which can be used to organise the
data objects in collections and sub-collections. The path of an object in the
data store becomes the identifier of an element that can be used to access it.
Additionally, Radon supports a graph database which can be used to provide 
different views on the data, accessing it through Gremlin queries.

Radon supports a query mechanism to store and retrieve the data. It implements
a substantial subset of the Cloud Data Management Interface (CDMI) RESTful API
to access the data. To retrieve or store an object, the full path in the
container hierarchy must be specified. Alternatively, retrieval may be via the
repository assigned unique identifier, as per the CDMI specification. It is also
possible to locate objects by specifying metadata names or values in the URI
query field.


## Discovery

Radon data model allows the creation of user metadata to any objects registered
in the system, for internal data objects or references stored in different data
providers. It uses a dictionary of Key/Values pair to store this information.
Metadata and data objects names are indexed and can be queried using Solr queries.

Additionaly, Radon data model can be extended to store data in a graph database. 
Using Gremlin language it is then possible to express semantic queries to 
explore deep relations between entities.



## Workflows

Radon uses the MQTT protocol to publish notifications when events take place in
the system. The rule engine (“listener”) subscribes to these notifications and
triggers rule actions in any scripting language (e.g. Python). The rule engine
can be used to automate the enforcement of policies as required for the use
cases. Radon supports data management policies at a micro level (e.g.
replication) and at a macro level (e.g. policies used to manage the versions of
a data object).




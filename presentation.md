# Presentation

Radon is an open source Data Management System designed for large-scale storage.
It provides distributed, robust and extensible storage capabilities, coupling
an object store to an easy to use trigger/rules system. This combination allows
for scalable systems that can self-manage, are not reliant on complex workflow
systems or external management frameworks. Relatively simple configurations
allow for distributed ingest, transformation and curation, without the need to
have complex scheduling frameworks.

Radon supports a query mechanism to store and retrieve the data. It implements
a substantial subset of the Cloud Data Management Interface (CDMI) RESTful API
to access the data. To retrieve or store an object, the full path in the
container hierarchy must be specified. Alternatively, retrieval may be via the
repository assigned unique identifier, as per the CDMI specification. It is also
possible to locate objects by specifying metadata names or values in the URI
query field.

Radon supports data sharing, or interoperability between external data stores.
This “virtualisation” capability is important because of the need to operate
across radically different types of storage and processing technologies; and
for the ability of users to access and share data. The actual data (e.g. the
blob of a digital object) is not held in Radon but referenced via its URL.

Radon supports the generation and maintenance of audit trails for specific
operations or events. This is important in order to trace the history of the
operations, and to prove that all operations on the digital entities were
performed by authorised users, including the update of the authenticity
metadata itself.

Radon uses the MQTT protocol to publish notifications when events take place in
the system. The rule engine (“listener”) subscribes to these notifications and
triggers rule actions in any scripting language (e.g. Python). The rule engine
can be used to automate the enforcement of policies as required for the use
cases. Radon supports data management policies at a micro level (e.g.
replication) and at a macro level (e.g. policies used to manage the versions of
a data object).

Radon supports authentication as part of its ability to track the identity of
users working in a high-security environment. This is important given
confidentiality and data protection requirements. Radon is controlled using
standardised ACL (access control lists), which can be applied at any level and
will apply to the sub-tree in the same way as storage policy.

Radon provides an additional graph store to model the relationships between the
entities managed by the system. Using Gremlin language it is possible to express
semantic queries to explore deep relations between entities.

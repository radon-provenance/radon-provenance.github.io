---
layout: default
---


# Radon Search


## Presentation

Radon Search is built on top of DSE Search. It uses Apache Solr to store indexes
in specific tables in the Cassandra database. Radon provides a set of default 
indexes to search on object paths in the repository and on metadata on a 
user-defined set of metadata names.


## Configure fields

### Path

"path" is the default field, it is created when the system is initialized and
can be used to search for objects by their names or collection names.


### Metadata

A set of DC field are created when the system is setup. If a metadata with the
same name is used, it can be indexed and searched with a solr query.


![Metadata fields](/assets/images/metadata_field.png)


## Queries

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


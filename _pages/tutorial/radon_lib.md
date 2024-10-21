---
title: Radon Data Management System
permalink: /tutorial/radon_lib.html
layout: default
sidebar_menu: Tutorial
sidebar_sub: Radon Library
---

# Using Radon Library


{% include warning.html content="Accessing Python Radon library directly can
be dangerous as it permits to access the database directly without using the
security layer and the rule engine. It will probably be modified or removed in 
future versions." %}


## Prepare the working environment

- Create a working directory

```shell
mkdir ~/tutorial/
cd tutorial
```

- Create a virtual environment

```shell
python3 -m venv ~/ve/tutorial
source ~/ve/tutorial/bin/activate
```

- Modify or create environment variables

```shell
export DSE_HOST="`docker exec dse hostname -i`"
export MQTT_HOST="`docker exec mqtt hostname -i`"
```

If dse and mqtt are not running locally, DSE_HOST and MQTT_HOST can be set to the
correct IP address directly.


- Get radon-lib package

```shell
git clone https://github.com/radon-provenance/radon-lib.git
```


- Install the dependencies

```shell
cd radon-lib
pip install -r requirements.txt
python setup.py develop
cd ..
```

## Write the Python code

```python
import sys
import pprint

from radon.database import initialise
from radon.model.collection import Collection
from radon.model.resource import Resource
from radon.model.notification import (
    create_collection_request,
    create_resource_request,
    update_resource_request,
    wait_response
)
from radon.model.payload import (
    PayloadCreateCollectionRequest,    
    PayloadCreateResourceRequest,
    PayloadUpdateResourceRequest
)

from radon.util import (
    merge,
    split
)

def create_collection(path):
    # Get the name of the parent collection and the name of the new collection
    parent, name = split(path)
    # Payload for the message that will be sent to the notification queue
    payload_json = {
        "obj": {
            "name" : name,
            "container": parent,
            "path": path,
        },
        "meta": {"sender": "user_tutorial"}
    }
    # Send a notification for the creation of new collection
    notif = create_collection_request(PayloadCreateCollectionRequest(payload_json))
    # We wait for the completion of the request as it can take time to
    # be effective and we need it to be created before continuing
    resp = wait_response(notif.req_id)
    # resp == 0 means correct creation (ie we a message create_collection_success)
    # has been sent to the queue
    if resp == 0:
       return Collection.find(path)
    
    return None

def create_resource(path):
    # Get the name of the parent collection and the name of the new resource
    parent, name = split(path)
    # Payload for the message that will be sent to the notification queue
    payload_json = {
        "obj": {
            "name" : name,
            "container": parent,
            "path": path, 
        },
        "meta": {"sender": "user_tutorial"}
    }
    # Send a notification for the creation of new resource
    notif = create_resource_request(PayloadCreateResourceRequest(payload_json))
    # We wait for the completion of the request as it can take time to
    # be effective and we need it to be created before continuing
    resp = wait_response(notif.req_id)
    # resp == 0 means correct creation (ie we a message create_resource_success)
    # has been sent to the queue
    if resp == 0:
       resc = Resource.find(path)
       resc.put(b"This is a text file for the tutorial")
       return resc
    
    return None

def update_resource(path, metadata):
    # Payload for the message that will be sent to the notification queue
    payload_json = {
        "obj": {
            "path": path,
            "metadata": metadata,
        },
        "meta": {"sender": "user_tutorial"}
    }
    # Send a notification for the modification of new resource
    notif = update_resource_request(PayloadUpdateResourceRequest(payload_json))
    # We wait for the completion of the request as it can take time to
    # be effective and we need it to be created before continuing
    resp = wait_response(notif.req_id)
    # resp == 0 means correct creation (ie we a message update_resource_success)
    # has been sent to the queue
    return

if __name__ == "__main__":
    # Initialise connection with Radon/DSE
    initialise()
    
    # Get information on the root of the archive
    c = Collection.find("/")
    print("Metadata for the root collection")
    pprint.pp(c.to_dict())

    # Create a new collection, check first that it doesn't exist
    # Collection paths has to ends with a '/' (from CDMI spec)
    path_coll = "/test_api/"
    # Search for the collection
    c = Collection.find(path_coll)
    if not c:
        c = create_collection(path_coll)
        if c:
            print("Collection created")
    else:
        print("collection already exists")
    if c:
        print("Metadata for the collection " + path_coll)
        pprint.pp(c.to_dict())
    else:
        print("Collection not created")
        sys.exit()

    # Add a new resource in the new collection
    path_resc = path_coll + "test_resc.txt"
    # Search for the resource
    r = Resource.find(path_resc)
    if not r:
        r = create_resource(path_resc)
        if r:
            print("Resource created")
    else:
        print("Resource already exists")
    if r:
        print("Metadata for the resource " + path_resc)
        pprint.pp(r.to_dict())
        print("Content of the resource " + path_resc)
        for chk in r.chunk_content():
            print(chk)
    else:
        print("Resource not created")
        sys.exit()
    
    # Update a resource with user metadata
    path_resc = path_coll + "test_resc.txt"
    r = Resource.find(path_resc)
    if r:
        metadata = {"author": "me", "date": "today"}
        update_resource(path_resc, metadata)
    r = Resource.find(path_resc)
    if r:
        print("Metadata for the resource " + path_resc)
        pprint.pp(r.get_cdmi_user_meta())
```


{% include note.html content="In the hierarchy a path ending with a '/' refers
to a collection, otherwise it refers to a resource." %}

{% include note.html content="To send notifications to the rule engine, it is
necessary to use the methods from _radon.model.notification_ instead of the 
methods in the Collection/Resource objects. This will initiate a request that 
will trigger a rule execution. " %}

{% include note.html content="Notifications methods are following the message
model: action_object_messagetype where action is create/update/delete, object
is collection/resource/group/user and messagetype is request/success/fail." %}

{% include note.html content="When requesting an operation it may be useful to
use the method *wait_response* to check the result of the operation. It will
also wait for the completion of the task as it may takes some time due to the
NOSQL database." %}




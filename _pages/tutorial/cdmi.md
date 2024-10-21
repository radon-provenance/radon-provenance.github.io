---
title: Radon Data Management System
permalink: /tutorial/cdmi.html
layout: default
sidebar_menu: Tutorial
sidebar_sub: CDMI Rest API
---

# Using CDMI Rest API


{% include note.html content="Using the Rest API is the recommended way to 
access the data store. It is available from any client application, supports
user authentication and ensures that notifications are correctly triggered for
the rule engine." %}

The Rest API can be called from a large number of applications. The current
tutorial shows how a set of calls can be made from a Python script. Once the
usage is understood, you can find the full list of access point in the 
[usage/CDMI REST api](/usage/cdmi.html) page

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

- Install the dependencies

```shell
pip install requests
```




## Python code

```python
import sys
import pprint
import requests


URL_REST_API = "http://127.0.0.1:8000/api/cdmi"
USERNAME = "CHANGEME"
PASSWORD = "CHANGEME"
AUTH = (USERNAME, PASSWORD)


def get_cdmi(path):
    headers = {"X-CDMI-Specification-Version": "1.1"}
    res = requests.get(URL_REST_API + path, 
                       headers=headers, auth=AUTH, verify=False)
    if res.status_code == 200:
        return res.json()
    else:
        return None

def create_collection(path):
    headers = {"X-CDMI-Specification-Version": "1.1",
               "Content-Type": "application/cdmi-container"}
    res = requests.put(URL_REST_API + path, 
                       headers=headers, auth=AUTH, verify=False)
    if res.status_code == 201:
        return res.json()
    else:
        return None

def create_resource(path):
    # We create a resource in http mode, there's a cdmi mode for more
    # complex request (for instance adding metadata or ACL)
    headers = {"Content-Type": "text/plain"}
    data = "This is a text file for the tutorial"
    res = requests.put(URL_REST_API + path, data=data,
                       headers=headers, auth=AUTH, verify=False)
    if res.status_code == 201:
        return True
    else:
        return False

def get_resource_content(path):
    headers = {"Accept": "application/octet-stream"}
    cfh = requests.get(URL_REST_API + path, headers=headers, 
                       auth=AUTH, stream=True, verify=False)
    if cfh.status_code == 404:
        return ""
    data = ""
    for chunk in cfh.iter_content(8192):
        # We read bytes so we need to decode them
        data += chunk.decode()
    return data

def update_resource(path, metadata):
    headers = {"X-CDMI-Specification-Version": "1.1",
               "Content-type": "application/cdmi-object"}
    data = {"metadata": metadata}
    res = requests.put(URL_REST_API + path, headers=headers, 
                       auth=AUTH, json=data, verify=False)
    print(res)
    

if __name__ == "__main__":
    # Get information on the root of the archive
    c = get_cdmi("/")
    print("Metadata for the root collection")
    pprint.pp(c)

    # Create a new collection, check first that it doesn't exist
    path_coll = "/test_cdmi/"
    # Search for the collection
    c = get_cdmi(path_coll)
    if not c:
        c = create_collection(path_coll)
        if c:
            print("Collection created")
    else:
        print("collection already exists")
    if c:
        print("Metadata for the collection " + path_coll)
        pprint.pp(c)
    else:
        print("Collection not created")
        sys.exit()

    # Add a new resource in the new collection
    path_resc = path_coll + "test_resc.txt"
    # Search for the resource
    r = get_cdmi(path_resc)
    if not r:
        r = create_resource(path_resc)
        if r:
            r = get_cdmi(path_resc)
            print("Resource created")
    else:
        print("Resource already exists")
    if r:
        print("Metadata for the resource " + path_resc)
        pprint.pp(r)
        print("Content of the resource " + path_resc)
        content = get_resource_content(path_resc)
        print(content)
    else:
        print("Resource not created")
        sys.exit()
    
    # Update a resource with user metadata
    path_resc = path_coll + "test_resc.txt"
    # Search for the resource
    r = get_cdmi(path_resc)
    if r:
        metadata = {"author": "me", "date": "today"}
        update_resource(path_resc, metadata)
    r = get_cdmi(path_resc)
    if r:
        print("Metadata for the resource " + path_resc)
        pprint.pp(r)
```


{% include note.html content="In the hierarchy and in the URL, a path ending 
with a '/' refers to a collection, otherwise it refers to a resource." %}

{% include note.html content="Some requests can be made using HTTP or CDMI, the
header \"X-CDMI-Specification-Version\" is there to define the request mode." %}

{% include note.html content="\"Content-Type\" header has to be defined properly
in order to create valid requests. Values can be \"application/cdmi-container\"
for collections or \"application/cdmi-object\" for resources." %}

{% include note.html content="User are authenticated using HTTP Basic 
Authentication, with a couple username/password. " %}


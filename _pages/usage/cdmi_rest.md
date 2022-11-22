---
layout: default
title: CDMI RESTful API
description: RESTful API for CDMI.
permalink: /usage/cdmi_rest
---

{:.title}
{{ page.title }}

* auto-gen TOC:
{:toc}

# Presentation


The [Radon Web project](https://github.com/radon-provenance/radon-web){:target="_blank"} 
is a translation layer that provides an accessible and well structured set of 
access methods to the data management system. Access methods comes in the form 
of RESTful services, accessible via the base url of a radon node running 
radon-web. The object store is used to store and register digital objects in 
the cluster. The data model is that of a hierarchical object store, _i.e._ 
objects are chunks of data that are stored in selected containers and may have 
associated metadata in the form of name/value pairs. The repository is agnostic 
to the data and metadata it stores, and the interpretation of both is the 
responsibility of the various applications, interpreters and engines that use 
the store. The object store provides a CDMI implementation for HTTP access to 
digital objects in the registry. The CDMI standard is fairly well designed, 
and the richest of the available “standards”. We layer a CDMI interface to 
provide remote (https) based access to the objects and metadata.


# Cloud Data Management Interface

The object store can be used to organize collections and digital objects in the
store. It implements the Cloud Data Management Interface (CDMI) that defines the
functional interface that applications may use to create, retrieve, update and
delete data elements from the Object Store (CRUD operations). In addition,
metadata can be set on collections and their contained data elements through
this interface.
The CDMI web service is accessible at _**/api/cdmi**_ from the root URI.


# Collections


## **Create a collection using HTTP**

_Synopsys_:

  To create a new collection object, the following request shall be performed:

  ```PUT <root URI>/api/cdmi/<CollectionName>/<NewCollectionName>/```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection that already exist, with one ‘/’ between each pair of collection names.
  * `<NewCollectionName>` is the name for the collection to be created.

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>201 Created</td>      <td>The new collection was created</td>                        </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>


_Example 1_:

  PUT to the collection URI to create a collection:

```
PUT /api/cdmi/MyCollection/ HTTP/1.1
Host: 192.168.12.12
```

  Response:

```HTTP/1.1 201 Created```


_Example 2 (using httpie)_:

```
http -a radon put http://radon-2.apps.l:8000/api/cdmi/testcdmi/ 
```



## **Create a collection using CDMI**


_Synopsys_:

  To create a new collection object, the following request shall be performed:

  ```PUT <root URI>/api/cdmi/<CollectionName>/<NewCollectionName>/```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection that already exist, with one ‘/’ between each pair of collection names.
  * `<NewCollectionName>` is the name for the collection to be created.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>                  <th>Requirement</th> </tr>
  <tr> <td>Accept</td>                       <td>Header String</td> <td>"application/cdmi-container"</td> <td>Optional</td>    </tr>
  <tr> <td>Content-Type</td>                 <td>Header String</td> <td>"application/cdmi-container"</td> <td>Mandatory</td>   </tr>        
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                        <td>Mandatory</td>   </tr>
</table>

_Request Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>        <th>Description</th>                        <th>Requirement</th> </tr>
  <tr> <td>metadata</td>   <td>JSON Object</td> <td>Metadata for the collection object</td> <td>Optional</td>    </tr>
</table>

_Response Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>                  <th>Requirement</th> </tr>
  <tr> <td>Content-Type</td>                 <td>Header String</td> <td>"application/cdmi-container"</td> <td>Mandatory</td>   </tr>        
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                        <td>Mandatory</td>   </tr>
</table>

_Response Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>        <th>Description</th>                    <th>Requirement</th> </tr>
  <tr> <td>objectType</td> <td>JSON String</td> <td>"application/cdmi-container"</td>   <td>Mandatory</td>   </tr>
  <tr> <td>objectID</td>   <td>JSON String</td> <td>ObjectID of the object</td>         <td>Mandatory</td>   </tr>  
  <tr> <td>objectName</td> <td>JSON String</td> <td>Name of the object</td>             <td>Mandatory</td>   </tr>     
  <tr> <td>parentURI</td>  <td>JSON String</td> <td>URI for the parent object</td>      <td>Mandatory</td>   </tr>    
  <tr> <td>parentID</td>   <td>JSON String</td> <td>Object ID of the parent object</td> <td>Mandatory</td>   </tr>    
  <tr> <td>metadata</td>   <td>JSON Object</td> <td>Metadata for the object</td>        <td>Optional</td>    </tr>     
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>201 Created</td>      <td>The new collection was created</td>                        </tr>
  <tr> <td>400 Bad Request</td>  <td>The request contains invalid parameters</td>               </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  PUT to the URI the collection object name and metadata:

```
PUT /api/cdmi/MyCollection/ HTTP/1.1
Host: 192.168.12.12
Accept: application/cdmi-container
Content-Type: application/cdmi-container
X-CDMI-Specification-Version: 1.1

{
  “metadata”: {}
}
```

  Response:

```
HTTP/1.1 201 Created
Content-Type: application/cdmi-container
X-CDMI-Specification-Version: 1.1

{
  "objectType" : "application/cdmi-container",
  "objectID" : "00007ED900104E1D14771DC67C27BF8B",
  "objectName" : "MyCollection/",  
  "parentURI" : "/",
  "parentID" : "00007E7F0010128E42D87EE34F5A6560",
  "metadata" : {
                ...
               },
}
```

_Example 2 (using httpie)_:

```
http -a radon put http://radon-2.apps.l:8000/api/cdmi/testcdmi/ Content-Type:application/cdmi-container X-CDMI-Specification-Version:1.1
```


## **Delete a collection using HTTP**

_Synopsys_:

  To delete an existing container object, including all contained children,
  the following request shall be performed:

  ```DELETE <root URI>/api/cdmi/<CollectionName>/<TheCollectionName>/```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection objects.
  * `<TheCollectionName>` is the name for the collection to be deleted.

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>204 No Content</td>   <td>The collection was deleted</td>                            </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  DELETE to the collection URI:

```
DELETE /api/cdmi/MyCollection/ HTTP/1.1
Host: 192.168.12.12
  ```

  Response:

```HTTP/1.1 204 No Content```

_Example 2 (using httpie)_:

```
http -a radon delete http://radon-2.apps.l:8000/api/cdmi/testcdmi/
```


## **Delete a collection using CDMI**

_Synopsys_:

  To delete an existing container object, including all contained children,
  the following request shall be performed:

  ```DELETE <root URI>/api/cdmi/<CollectionName>/<NewCollectionName>/```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection names.
  * `<TheCollectionName>` is the name for the collection to be deleted.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>                  <th>Requirement</th> </tr>   
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                        <td>Mandatory</td>   </tr>
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>204 No Content</td>   <td>The collection was deleted</td>                            </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  DELETE the collection at URI:

```
DELETE /api/cdmi/MyCollection/ HTTP/1.1
Host: 192.168.12.12
X-CDMI-Specification-Version: 1.1
```

  Response:

```
HTTP/1.1 204 No Content
```

_Example 2 (using httpie)_:

```
http -a radon delete http://radon-2.apps.l:8000/api/cdmi/testcdmi/ X-CDMI-Specification-Version:1.1
```


## **Read a collection using CDMI**

_Synopsys_:

  To read all fields from an existing collection object, the following request
  shall be performed:

  ```GET <root URI>/api/cdmi/<CollectionName>/<TheCollectionName>/```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection objects.
  * `<TheCollectionName>` is the name specified for the collection object to be read from.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>                  <th>Requirement</th> </tr>
  <tr> <td>Accept</td>                       <td>Header String</td> <td>"application/cdmi-container"</td> <td>Optional</td>    </tr>     
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                        <td>Mandatory</td>   </tr>
</table>

_Response Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>                 <th>Description</th>                                                  <th>Requirement</th> </tr>
  <tr> <td>objectType</td> <td>JSON String</td>          <td>"application/cdmi-container"</td>                                 <td>Mandatory</td>   </tr>
  <tr> <td>objectID</td>   <td>JSON String</td>          <td>ObjectID of the object</td>                                       <td>Mandatory</td>   </tr>  
  <tr> <td>objectName</td> <td>JSON String</td>          <td>Name of the object</td>                                           <td>Mandatory</td>   </tr>     
  <tr> <td>parentURI</td>  <td>JSON String</td>          <td>URI for the parent object</td>                                    <td>Mandatory</td>   </tr>    
  <tr> <td>parentID</td>   <td>JSON String</td>          <td>Object ID of the parent object</td>                               <td>Mandatory</td>   </tr>    
  <tr> <td>metadata</td>   <td>JSON Object</td>          <td>Metadata for the object</td>                                      <td>Optional</td>    </tr>  
  <tr> <td>children</td>   <td>JSON Array of JSON Strings</td> <td>Name of the children objects in the collection object.</td> <td>Mandatory</td>   </tr>      
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                                     </tr>
  <tr> <td>200 OK</td>           <td>The metadata for the collection is provided in the Body</td> </tr>
  <tr> <td>400 Bad Request</td>  <td>The request contains invalid parameters</td>                         </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td>           </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>                       </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>                 </tr>
</table>

_Example 1_:

  GET to the collection object URI to read all the fields of the collection object:

```
GET /api/cdmi/MyCollection/ HTTP/1.1
Host: 192.168.12.12
Accept: application/cdmi-container
X-CDMI-Specification-Version: 1.1
```

  Response:

```
HTTP/1.1 200 OK
Content-Type: application/cdmi-container
X-CDMI-Specification-Version: 1.1

{
  "objectType" : "application/cdmi-container",
  "objectID" : "00007ED900104E1D14771DC67C27BF8B",
  "objectName" : "MyCollection/",  
  "parentURI" : "/",
  "parentID" : "00007E7F0010128E42D87EE34F5A6560",
  "metadata" : {
                ...
               },
  "children" : [
               "child1",
               “child2”,
               …
               ]
}
```

_Example 2 (using httpie)_:

```
http -a radon get http://radon-2.apps.l:8000/api/cdmi/testcdmi/ X-CDMI-Specification-Version:1.1
```




## **Update a collection using CDMI**

_Synopsys_:

  To update some or all fields in an existing collection object, the following
  request shall be performed:

  ```PUT <root URI>/api/cdmi/<CollectionName>/<TheCollectionName>/```

  To add, update, and remove specific metadata items of an existing collection
  object, the following request shall be performed:

  ```PUT <root URI>/api/cdmi/<CollectionName>/<TheCollectionName>/?metadata:<metadataname>```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collectionobjects.
  * `<TheCollectionName>` is the name for the collection to be updated.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>                  <th>Requirement</th> </tr>
  <tr> <td>Accept</td>                       <td>Header String</td> <td>"application/cdmi-container"</td> <td>Optional</td>    </tr>       
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                        <td>Mandatory</td>   </tr>
</table>

_Request Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>        <th>Description</th>                        <th>Requirement</th> </tr>
  <tr> <td>metadata</td>   <td>JSON Object</td> <td>Metadata for the collection object</td> <td>Optional</td>    </tr>
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>204 No Content</td>   <td>The collection content was updated</td>                    </tr>
  <tr> <td>400 Bad Request</td>  <td>The request contains invalid parameters</td>               </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  PUT to the URI the collection object to set new metadata:

```
PUT /api/cdmi/MyCollection/ HTTP/1.1
Host: 192.168.12.12
Content-Type: application/cdmi-container
X-CDMI-Specification-Version: 1.1

{
  “metadata”: {}
}
```

  Response:

```
HTTP/1.1 204 No Content
```

_Example 2 (using httpie)_:

```
echo '{"metadata": {"test_api": "Value"}}' | http -a radon put http://radon-2.apps.l:8000/api/cdmi/testcdmi/ X-CDMI-Specification-Version:1.1
```


# Data Objects


## **Create a data object using HTTP**

_Synopsys_:

  The following HTTP PUT creates a new data object at the specified URI:

  ```PUT <root URI>/api/cdmi/<CollectionName>/<DataObjectName>```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection that already exist.
  * `<DataObjectName>` is the name for the data object to be created.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>        <th>Type</th>          <th>Description</th>                                                <th>Requirement</th> </tr>
  <tr> <td>Content-Type</td>  <td>Header String</td> <td>The content type of the data to be stored as a data object</td> <td>Optional</td>    </tr>     
  <tr> <td>Content-Range</td> <td>Header String</td> <td>A valid range-specifier</td>                                    <td>Optional</td>   </tr>
</table>

_Request Body_:

The request Body contains the data to be stored.

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>201 Created</td>      <td>The new data object was created</td>                       </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  PUT to the collection URI the data object name and contents:

```
PUT /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
Content-Type: text/plain;charset=utf-8
Content-Length: 37

This is the Value of this Data Object
```

  Response:


```HTTP/1.1 201 Created```

_Example 2 (using httpie)_:

```
echo 'Test File creation HTTP Mode' | http -a radon put http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_http.txt
```


## **Create a data object using CDMI**

_Synopsys_:

  To create a new data object, the following request shall be performed:

  ```PUT <root URI>/api/cdmi/<CollectionName>/<DataObjectName>/```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection that already exist, with one ‘/’ between each pair of collection names.
  * `<DataObjectName>` is the name for the data object to be created.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>               <th>Requirement</th> </tr>
  <tr> <td>Accept</td>                       <td>Header String</td> <td>"application/cdmi-object"</td> <td>Optional</td>    </tr>
  <tr> <td>Content-Type</td>                 <td>Header String</td> <td>"application/cdmi-object"</td> <td>Mandatory</td>   </tr>        
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                     <td>Mandatory</td>   </tr>
</table>

_Request Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>        <th>Description</th>                                            <th>Requirement</th> </tr>
  <tr> <td>mimetype</td>   <td>JSON String</td> <td>Mime type of the data contained within the value field</td> <td>Optional</td>    </tr>
  <tr> <td>metadata</td>   <td>JSON Object</td> <td>Metadata for the collection object</td>                     <td>Optional</td>    </tr>
  <tr> <td>value</td>      <td>JSON String</td> <td>The data object value</td>                                  <td>Optional</td>    </tr>
</table>

_Response Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>               <th>Requirement</th> </tr>
  <tr> <td>Content-Type</td>                 <td>Header String</td> <td>"application/cdmi-object"</td> <td>Mandatory</td>   </tr>        
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                     <td>Mandatory</td>   </tr>
</table>

_Response Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>        <th>Description</th>                               <th>Requirement</th> </tr>
  <tr> <td>objectType</td> <td>JSON String</td> <td>"application/cdmi-container"</td>              <td>Mandatory</td>   </tr>
  <tr> <td>objectID</td>   <td>JSON String</td> <td>ObjectID of the object</td>                    <td>Mandatory</td>   </tr>  
  <tr> <td>objectName</td> <td>JSON String</td> <td>Name of the object</td>                        <td>Mandatory</td>   </tr>     
  <tr> <td>parentURI</td>  <td>JSON String</td> <td>URI for the parent object</td>                 <td>Mandatory</td>   </tr>    
  <tr> <td>parentID</td>   <td>JSON String</td> <td>Object ID of the parent object</td>            <td>Mandatory</td>   </tr>   
  <tr> <td>mimetype</td>   <td>JSON String</td> <td>Mime type of the value of the data object</td> <td>Optional</td>    </tr>
  <tr> <td>metadata</td>   <td>JSON Object</td> <td>Metadata for the object</td>                   <td>Optional</td>    </tr>     
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>201 Created</td>      <td>The new data object was created</td>                       </tr>
  <tr> <td>400 Bad Request</td>  <td>The request contains invalid parameters</td>               </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  PUT to the collection URI the data object name and contents:

```
PUT /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
Accept: application/cdmi-object
Content-Type: application/cdmi-object
X-CDMI-Specification-Version: 1.1

{
  "mimetype" : "text/plain",
  "metadata": {...},
  "value" : "This is the Value of this Data Object"
}
```

  Response:

```
HTTP/1.1 201 Created
Content-Type: application/cdmi-object
X-CDMI-Specification-Version: 1.1

{
  "objectType" : "application/cdmi-container",
  "objectID" : "00007ED90010D891022876A8DE0BC0FD",
  "objectName" : "MyDataObject.txt",  
  "parentURI" : "/MyContainer/",
  "parentID" : "00007E7F00102E230ED82694DAA975D2",
  "mimetype" : "text/plain",
  "metadata" : {
                "cdmi_size" : "37"
               },
}
```

_Example 2 (using httpie)_:

```
echo '{"value": "Test File creation CDMI Mode"}' | http -a radon put http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_cdmi.txt X-CDMI-Specification-Version:1.1
```


## **Delete a data object using HTTP**

_Synopsys_:

  The following HTTP DELETE deletes an existing data object at the specified URI:

  ```DELETE <root URI>/api/cdmi/<CollectionName>/<DataObjectName>```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection objects.
  * `<DataObjectName>` is the name of the data object to be deleted.

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>204 No Content</td>   <td>The data object was deleted</td>                           </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  DELETE to the data object URI:

```
DELETE /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
```

  Response:

```HTTP/1.1 204 No Content```

_Example 2 (using httpie)_:

```
http -a radon delete http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_http.txt
```


## **Delete a data object using CDMI**

_Synopsys_:

  The following HTTP DELETE deletes an existing data object at the specified URI:

  ```DELETE <root URI>/api/cdmi/<CollectionName>/<DataObjectName>```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection names.
  * `<DataObjectName>` is the name of the data object to be deleted.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>                  <th>Requirement</th> </tr>   
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                        <td>Mandatory</td>   </tr>
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>204 No Content</td>   <td>The collection was deleted</td>                            </tr>
  <tr> <td>400 Bad Request</td>  <td>The request contains invalid parameters</td>               </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  DELETE the data object at URI:

```
DELETE /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
X-CDMI-Specification-Version: 1.1
```

  Response:

```
HTTP/1.1 204 No Content
```

_Example 2 (using httpie)_:

```
http -a radon delete http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_cdmi.txt X-CDMI-Specification-Version:1.1
```


## **Read a data object using HTTP**

_Synopsys_:

  The following HTTP GET reads from an existing data object at the specified URI:

  ```GET <root URI>/api/cdmi/<CollectionName>/<DataObjectName>```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection objects.
  * `<DataObjectName>` is the name specified of the data object to be read from.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th> <th>Type</th>          <th>Description</th>             <th>Requirement</th> </tr>
  <tr> <td>Range</td>  <td>Header String</td> <td>A valid range specifier</td> <td>Optional</td>    </tr>   
</table>

_Response Headers_:

<table class="table_api">
  <tr> <th>Header</th>       <th>Type</th>          <th>Description</th>                     <th>Requirement</th> </tr>
  <tr> <td>Content-Type</td> <td>Header String</td> <td>The mimetype of the data object</td> <td>Mandatory</td>   </tr>  
</table>

_Response Body_:

The response Body is the content of the data object.

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>         <th>Description</th>                                           </tr>
  <tr> <td>200 OK</td>              <td>The data object content was returned in the response</td>  </tr>
  <tr> <td>206 Partial Content</td> <td>The data object content was returned in the response</td>  </tr>
  <tr> <td>401 Unauthorized</td>    <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>       <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>       <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  GET to the data object URI to read the value of the data object:

```
GET /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
```

  Response:

```
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 37

This is the Value of this Data Object
```

_Example 2_:

  GET to the data object URI to read the first 11 bytes of the value of the data object:

```
GET /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
Range: bytes=0-10
```

  Response:

```
HTTP/1.1 206 Partial Content
Content-Type: text/plain
Content-Range: bytes 0-10/37
Content-Length: 11

This is the
```

_Example 3 (using httpie)_:

```
http -a radon get http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_http.txt
```

_Example 4 (using httpie)_:

```
http -a radon get http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_http.txt Range:bytes=0-10
```



## **Read a data object using CDMI**

_Synopsys_:

  The following HTTP GET reads from an existing data object at the specified URI:

  ```GET <root URI>/api/cdmi/<CollectionName>/<DataObjectName>```

  ```GET <root URI>/api/cdmi/<CollectionName>/<DataObjectName>?value:<range>;...```

  ```GET <root URI>/api/cdmi/<CollectionName>/<DataObjectName>?metadata:<prefix>;...```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection objects.
  * `<DataObjectName>` is the name specified for the data object to be read from.
  * `<range>` is a byte range of the data object value to be returned in the value field.
  * `<prefix>` is a matching prefix that returns all metadata items that start with the prefix value.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>               <th>Requirement</th> </tr>
  <tr> <td>Accept</td>                       <td>Header String</td> <td>"application/cdmi-object"</td> <td>Optional</td>    </tr>     
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                     <td>Mandatory</td>   </tr>
</table>

_Response Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>               <th>Requirement</th> </tr>
  <tr> <td>Content-Type</td>                 <td>Header String</td> <td>"application/cdmi-object"</td> <td>Mandatory</td>   </tr>      
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                     <td>Mandatory</td>   </tr>
</table>

_Response Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>        <th>Description</th>                               <th>Requirement</th> </tr>
  <tr> <td>objectType</td> <td>JSON String</td> <td>"application/cdmi-object"</td>                 <td>Mandatory</td>   </tr>
  <tr> <td>objectID</td>   <td>JSON String</td> <td>ObjectID of the object</td>                    <td>Mandatory</td>   </tr>  
  <tr> <td>objectName</td> <td>JSON String</td> <td>Name of the object</td>                        <td>Mandatory</td>   </tr>     
  <tr> <td>parentURI</td>  <td>JSON String</td> <td>URI for the parent object</td>                 <td>Mandatory</td>   </tr>    
  <tr> <td>parentID</td>   <td>JSON String</td> <td>Object ID of the parent object</td>            <td>Mandatory</td>   </tr>    
  <tr> <td>mimetype</td>   <td>JSON String</td> <td>MIME type of the value of the data object</td> <td>Mandatory</td>   </tr>     
  <tr> <td>metadata</td>   <td>JSON Object</td> <td>Metadata for the object</td>                   <td>Mandatory</td>   </tr>  
  <tr> <td>value</td>      <td>JSON String</td> <td>data object value</td>                         <td>Conditional</td> </tr>      
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                                     </tr>
  <tr> <td>200 OK</td>           <td>The metadata for the collection is provided in the Body</td> </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td>           </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>                       </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>                 </tr>
</table>

_Example 1_:

  GET to the data object URI to read all fields of the data object:

```
GET /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
Accept: application/cdmi-object
X-CDMI-Specification-Version: 1.1
```

  Response:

```
HTTP/1.1 200 OK
Content-Type: application/cdmi-object
X-CDMI-Specification-Version: 1.1

{
  "objectType" : "application/cdmi-object",
  "objectID" : "00007ED90010D891022876A8DE0BC0FD",
  "objectName" : "MyDataObject.txt",  
  "parentURI" : "MyCollection/",
  "parentID" : "00007E7F00102E230ED82694DAA975D2",
  "mimetype" : "text/plain",
  "metadata" : {
                "cdmi_size" : "37"
               },
  "valuerange" : "0-36",
  "value" : "This is the Value of this Data Object"

}
```

_Example 2 (using httpie)_:

```
http -a radon get http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_http.txt X-CDMI-Specification-Version:1.1
```


## **Update a data object using HTTP**

_Synopsys_:

  The following HTTP PUT updates an existing data object at the specified URI:

  ```PUT <root URI>/api/cdmi/<CollectionName>/<DataObjectName>```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection that already exist.
  * `<DataObjectName>` is the name for the data object to be updated.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>        <th>Type</th>          <th>Description</th>                                                <th>Requirement</th> </tr>
  <tr> <td>Content-Type</td>  <td>Header String</td> <td>The content type of the data to be stored as a data object</td> <td>Optional</td>    </tr>     
  <tr> <td>Content-Range</td> <td>Header String</td> <td>A valid range-specifier</td>                                    <td>Optional</td>   </tr>
</table>

_Request Body_:

  The request Body contains the data to be stored.

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>201 No Content</td>   <td>The new data object was updated</td>                       </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  PUT to the data object URI to update the value of the data object:

```
PUT /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
Content-Type: text/plain
Content-Length: 37

This is the value of this Data Object
```

  Response:

```HTTP/1.1 204 No Content```

_Example 2 (using httpie)_:

```
echo 'File update HTTP Mode' | http -a radon put http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_http.txt
```




## **Update a data object using CDMI**

_Synopsys_:

  The following HTTP PUT updates an existing data object at the specified URI:

  ```PUT <root URI>/api/cdmi/<CollectionName>/<DataObjectName>/```

  ```PUT <root URI>/api/cdmi/<CollectionName>/<DataObjectName>/?value:<range>```

  ```PUT <root URI>/api/cdmi/<CollectionName>/<DataObjectName>/?metadata:<metadataname>```

  where:
  * `<root URI>` is the URL of the web server.
  * `<CollectionName>` is zero or more intermediate collection objects.
  * `<DataObjectName>` is the name for the data object to be updated.
  * `<range>` is a byte range for the data object value to be updated.
  * `<metadataname>` is the name for the metadata to be updated.

_Request Headers_:

<table class="table_api">
  <tr> <th>Header</th>                       <th>Type</th>          <th>Description</th>               <th>Requirement</th> </tr>
  <tr> <td>Accept</td>                       <td>Header String</td> <td>"application/cdmi-object"</td> <td>Mandatory</td>   </tr>       
  <tr> <td>X-CDMI-Specification-Version</td> <td>Header String</td> <td>"1.1"</td>                     <td>Mandatory</td>   </tr>
</table>

_Request Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>        <th>Description</th>                                            <th>Requirement</th> </tr>
  <tr> <td>mimetype</td>   <td>JSON String</td> <td>Mime type of the data contained within the value field</td> <td>Optional</td>    </tr>
  <tr> <td>metadata</td>   <td>JSON Object</td> <td>Metadata for the collection object</td>                     <td>Optional</td>    </tr>
  <tr> <td>value</td>      <td>JSON String</td> <td>The data object value</td>                                  <td>Optional</td>    </tr>
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>204 No Content</td>   <td>The data object content was updated</td>                   </tr>
  <tr> <td>400 Bad Request</td>  <td>The request contains invalid parameters</td>               </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
  <tr> <td>404 Not Found</td>    <td>The resource was not found at the specified URI</td>       </tr>
</table>

_Example 1_:

  PUT to the data object URI to set new field values:

```
PUT /api/cdmi/MyCollection/MyDataObject.txt HTTP/1.1
Host: 192.168.12.12
Content-Type: application/cdmi-object
X-CDMI-Specification-Version: 1.1

{
  "mimetype" : "text/plain",
  "metadata": { "colour": "red", },
  "value" : "This is the Value of this Data Object"
}
```

  Response:

```
HTTP/1.1 204 No Content
```

_Example 2_:

```
echo '{ "value": "Test File update CDMI Mode", "metadata": { "colour": "red" } }' | http -a radon put http://radon-2.apps.l:8000/api/cdmi/testcdmi/test_cdmi.txt X-CDMI-Specification-Version:1.1
```

# Find RESTful API


## Presentation

Radon provides a minimal search API. It indexes the names of collections or
data objects and the name and values of metadata.

## Find objects matching a term

_Synopsys_:

  We have defined a minimal find API accessible under /api/find. It can search
  in names or metadata of collections and data objects and returns matching
  objects.

  The following HTTP GET query the registry:

  ```GET <root URI>/api/find?findTerms=<term>```

  ```GET <root URI>/api/find?findTerms=<term>&where=<where>```

  where:
  * `<root URI>` is the URL of the web server.
  * `<term>` is the keyword we are looking for.
  * `<where>` is the place we are looking, it can be “name”,
  “metadata” or “both”. The default value is “both”.


_Response Body_:

<table class="table_api">
  <tr> <th>Field Name</th> <th>Type</th>                       <th>Description</th>                          <th>Requirement</th> </tr>
  <tr> <td>result</td>     <td>JSON Array of JSON Strings</td> <td>List of URI for the matching object.</td> <td>Mandatory</td>   </tr>
</table>

_Response Status_:

<table class="table_api">
  <tr> <th>HTTP Status</th>      <th>Description</th>                                           </tr>
  <tr> <td>200 OK</td>           <td>The result are returned in the response body</td>          </tr>
  <tr> <td>400 Bad Request</td>  <td>The request contains invalid parameters</td>               </tr>
  <tr> <td>401 Unauthorized</td> <td>The authentication credentials are missing or invalid</td> </tr>
  <tr> <td>403 Forbidden</td>    <td>The client lacks the proper authorization</td>             </tr>
</table>

_Example 1_:

  GET to the find URI to get matching objects:

```
GET /api/find?findTerms=test HTTP/1.1
Host: 192.168.12.12
```

  Response:

```
HTTP/1.1 200 OK

{
  “result” : [“/MyContainer/”,
              “/MyCollection/MyDataObject.txt”,
		          …,
             ]
}
```

_Example 2_:

```
http -a radon get http://radon-2.apps.l:8000/api/find?findTerms=test
```
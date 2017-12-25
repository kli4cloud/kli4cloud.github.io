---
layout: post
title:  "Designing a REST API (Part I)"
author: "jose"
date:   2017-12-24 13:00:00 +0000
category: REST
images:
  - url: /images/rest-api.png
    alt: REST logo
    title: REST logo
---
REST is acronym for REpresentational State Transfer. It is architectural style for distributed hypermedia systems, providing interoperability between computer systems. REST-compliant services allow requesting systems to interact with textual representations of resources. It was introduced and defined in 2000 by Roy Fielding in his [doctoral dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm).

Like any other architectural style, REST also has it’s own constraints which must be satisfied if an interface needs to be referred as RESTful. These constraints are listed below.

## REST constraints

* **Client–server:** Separation between interface and data storage. With this separation we improve the portability of the user interface across multiple platforms and improve scalability by simplifying the server components.

* **Stateless:** Each request from client to server must contain all of the information necessary to understand the request, and cannot take advantage of any stored context on the server. Session state is therefore kept entirely on the client.

* **Cacheable:** Cache constraints require that the data within a response to a request be implicitly or explicitly labeled as cacheable or non-cacheable. If a response is cacheable, then a client cache is given the right to reuse that response data for later, equivalent requests.

* **Uniform interface:** By applying the software engineering principle of generality to the component interface, the overall system architecture is simplified and the visibility of interactions is improved. In order to obtain a uniform interface, multiple architectural constraints are needed to guide the behavior of components. REST is defined by four interface constraints: identification of resources; manipulation of resources through representations; self-descriptive messages; and, hypermedia as the engine of application state.

* **Layered system:** The layered system style allows an architecture to be composed of hierarchical layers by constraining component behavior such that each component cannot “see” beyond the immediate layer with which they are interacting.

* **Code on demand (optional):** REST allows client functionality to be extended by downloading and executing code in the form of applets or scripts. This simplifies clients by reducing the number of features required to be pre-implemented.

##  Resource

The key abstraction of information in REST is a resource. Any information that can be named can be a resource: a document or image, a temporal service, a collection of other resources, a non-virtual object (e.g. a person), and so on. REST uses a resource identifier to identify the particular resource involved in an interaction between components.

##  Mapping HTTP methods to REST

HTTP defines a set of request methods to indicate the desired action to be performed for a given resource. Although they can also be nouns, these request methods are sometimes referred to as HTTP verbs. Each of them implements a different semantic, but some common features are shared by a group of them: i.e. a request method can be safe, idempotent, or cacheable.

HTTP methods compliance with their definitions under the HTTP/1.1 standard:

* **GET:** Requests a representation of one specific or a list of resources
* **HEAD:** Asks for a response identical to that of a GET request, but without the response body, returning just headers
* **OPTIONS:** Return available HTTP methods and other options
* **DELETE:** Delete a resource
* **POST:** Create a new resource
* **PUT:** Update/override an existing resource. It use to have the id of the resource at the end of the URL
* **PATCH:** Partial update of a resource. Update only the fields passed to the service

| Method  | Safe | Idempotent | Cacheable |
| ------- | ---- | ---------- | --------- |
| GET     | Yes  | Yes        | Yes       |
| HEAD    | Yes  | Yes        | Yes       |
| OPTIONS | Yes  | Yes        | No        |
| DELETE  | No   | Yes        | No        |
| POST    | No   | No         | Yes       |
| PUT     | No   | Yes        | No        |
| PATCH   | No   | No         | No        |

### GET

GET method requests a representation of one specific resource or a list of them. GET should only retrieve data and should have no other effect on the data so the method should be idempotent, safe and cacheable.

GET method doesn't use to have body in the request but according to the specification it's optional so you can send a GET request with body from the client.

#### Frequent status codes returned

* **200:** Successful response with no error and the body response contains the resource
* **404:** The server can't find the requested resource

#### List Example
##### Request
```
  GET /users/ HTTP/1.1
```
##### Response
```
  HTTP/1.1 200 OK
  Content-Type: application/json;charset=UTF-8
  [{ id: '1', name: 'John', surname: 'Snow'}, {id: '2', name: 'Ronald', surname: 'McDonal'}]
```
#### Unique Resource example:
##### Request
```
  GET /users/1 HTTP/1.1
```
##### Response
```
  Status: 200 OK
  Content-Type: application/json;charset=UTF-8
  { id: '1', name: 'John', surname: 'Snow'}
```

### HEAD

HEAD method requests the headers that are returned if the specified resource would be requested with an HTTP GET method. The response only contains headers. Such a GET method this should be safe, idempotent and cacheable. The meta information contained in the HTTP headers in response to a HEAD request should be identical to the information sent in response to a GET request

HEAD method requests a representation of one specific or a list of resources. GET should only retrieve data and should have no other effect on the data so the method should be idempotent and safe and cacheable.

#### Use cases

* HEAD method can be used for testing hypertext links for validity, accessibility, and recent modification.
* It can be used get the Content-Length header deciding to download a large resource to save bandwidth
* It's used frequently to check if cached entity differs from the current entity and then treat the cache entry as stale

#### Frequent status codes returned

* **200:** Successful response with headers and no body
* **404:** The server can't find the requested resource

#### Example

##### Request
```
  HEAD /users/12345 HTTP/1.1
```
##### Response
```
  Status: 200 OK
  Content-Type: application/json;charset=UTF-8
```

### POST

POST method sends data to the server in order to create a new resource. The type of the request body is indicated by the Content-Type header. The URL specifies the type of the resource created and the body contains the data required for the new resource. We can also use POST to change the status of one resource (i.e. Flip a coin). The POST method is not idempotent and cacheable.

POST method should return the entity created in the response body and a Location header with the reference to the resource created.

#### Frequent status codes returned

* **201:** Resource created successfully
* **415:** Unsupported media type
* **304:** Resource not modified
* **400:** Bad request
* **422:** Unprocessable Entity

#### Example

##### Request
```
  POST /users
  Accept: application/json
  Content-Type: application/json
  { name: 'John', surname: 'Snow'}
```
##### Response with body
```
  Status: 201 Created
  Content-Type: application/json;charset=UTF-8
  Location: /users/1
  { id: '1', name: 'John', surname: 'Snow'}
```

### PUT

PUT method sends data to the server in order to create a new resource or replace it if the resource already exists. The type of the request body is indicated by the Content-Type header. The URL specifies the type of the resource and the identifier, body contains the new data of the resource. The PUT method is idempotent and not cacheable.

PUT method should return the entity updated in the response body. It can be implemented allowing creation of new resources with the ID indicated in the URL, in that case we should return a 201.

**Note:** When PUT method processes the data should replace all the fields in the existing resource, fields not present should be treated as null values and replaced them with null values.

#### Frequent status codes returned

* **200:** Resource updated successfully
* **201:** Resource created successfully
* **415:** Unsupported media type
* **400:** Bad request
* **422:** Unprocessable Entity

#### Example

##### Request
```
  PUT /users/1234
  Content-Type: application/json
  { name: 'John', surname: 'Snow'}
```
##### Response
```
  Status: 200
  Content-Type: application/json
  { id: '1', name: 'John', surname: 'Snow'}
```

### PATCH

PATCH method sends data to the server in order to do a _partial update_ of an existing resource atomically. The type of the request body is indicated by the Content-Type header. The URL specifies the type of the resource and the identifier, body contains the data to update the resource. The PATCH method is not idempotent and not cacheable.

PATCH method should make the changes atomically i.e. if one operation fails none of them should be applied and in the success case the operation should return the entity updated in the response body.

#### Content-Type and Format

In the PATCH RFC there are a defined Content-Type for PATH operations (application/json-patch+json or application/xml-patch+xml)

* **op:** Indicates the operation to perform.  Its value MUST be one of the followings:
  * **add:** Inserts a new value in an array, add a new member in an object or replace the member in a object if it already exists.
  * **remove:** Removes the value at the target location.
  * **replace:** Replaces the value at the target location with a new value. You can implement a replace as result of a remove operation followed by an add.
  * **move:** Removes the value at a specified location and adds it to the target location.
  * **copy:** Copies the value at a specified location to the target location.
* **test:** Verify that a value at the target location is equal to a specified value.
* **path:** JSON-Pointer value that references a location where the operation is performed
* **value:** Value for test, add, replace operations. It conI can be an strings, numbers, arrays, objects or literals
* **from:** Location for the origin value in move and copy operations.

#### JSON Example
 ```
 Content-Type: application/json-patch+json
 [
     { "op": "test", "path": "/a/b/c", "value": "foo" },
     { "op": "add", "path": "/a/b/c", "value": [ "foo", "bar" ] },
     { "op": "replace", "path": "/a/b/c", "value": 42 },
     { "op": "move", "from": "/a/b/c", "path": "/a/b/d" },
     { "op": "copy", "from": "/a/b/d", "path": "/a/b/e" }
 ]
```
#### XML Example
```
Content-Type: application/xml-patch+xml
<p:patch xmlns="http://example.com/ns1"
          xmlns:y="http://example.com/ns2"
          xmlns:p="urn:ietf:rfc:7351">
     <p:add sel="doc/elem[@a='foo']">
         <child id="ert4773">
             <y:node/>
         </child>
     </p:add>
     <p:replace sel="doc/note/text()">Patched doc</p:replace>
     <p:remove sel="*/elem[@a='bar']/y:child" ws="both"/>
     <p:add sel="*/elem[@a='bar']" type="@b">new attr</p:add>
 </p:patch>
```  

#### Frequent status codes returned

* **200:** Resource updated successfully
* **404:** The server can't find the requested resource
* **415:** Unsupported media type
* **400:** Bad request
* **422:** Unprocessable Entity
* **409:** Conflicting state. When one of the path or from assumed exist and they don't exist at the moment of the execution

### DELETE

DELETE method removes a specific resource. The URL specifies the type of the resource to delete and the id of the resource to delete. The DELETE method is idempotent so if the resource is not found the service shouldn't return a not found exception, it's not cacheable and not safe.

#### Frequent status codes returned

* **204:** Resource is deleted successfully
* **202:** This action will likely succeed but has not yet been enacted
* **410:** The resource is no longer available and will not be available again

#### Example
##### Request
```
  DELETE /users/123456
```
##### Response
```
  Status: 204
```  

##  POST vs PUT methods

POST method is used to create resources but PUT can be used to create resources or update resources, the main difference is that if we use POST the server will decide the URI for the created resource while using PUT method the client will decide the new resource URI and if the resource already exists the server should update it with the new representation sent by the client.

POST method is cacheable and not idempotent whereas that the PUT method is not cacheable and idempotent.

#### POST example
##### Request
```
  POST /users
  Content-Type: application/json
  { name: 'John', surname: 'Snow'}
```
##### Response
```
  Status: 201
  Content-Type: application/json
  Location: /users/1
  { id: '1', name: 'John', surname: 'Snow'}
```

#### PUT example for a new resource

##### Request
```
  PUT /users/1
  Content-Type: application/json
  { name: 'John', surname: 'Snow'}
```
##### Response
```
  Status: 201
  Content-Type: application/json
  { id: '1', name: 'John', surname: 'Snow'}
```

##  PUT vs PATCH methods

PUT and PATCH methods allow updates of existing resources. There are a few differences between those methods:

* PUT sends a new representation of the resource and PATCH sends a list of operations to be applied to the resource
* PUT replaces the resource with the new representation, PATCH makes a partial update of the resource, updating only a few resource's fields
* PUT is idempotent and PATCH not

## Idempotent

A HTTP method is considered "idempotent" if the intended effect on the server of multiple identical requests with that method is the same as the effect for a single such request. GET, HEAD, OPTIONS, PUT and DELETE methods are idempotent.

Idempotent methods are important because that means the request can be repeated automatically if a communication failure occurs before the client is able to read the server's response.  For example, if a client sends a PUT request and the underlying connection is closed before any response is received, then the client can establish a new connection and retry the idempotent request.

To be idempotent, only the actual back-end state of the server is considered, the HTTP status code returned by each request may differ:

**i.e.** First call of a DELETE will likely return a 200, while successive ones will likely return a 410.

##  References

* <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP" target="_blank">Mozilla doc</a>
* <a href="https://tools.ietf.org/html/rfc2068" target="_blank">HTTP 1.1 RFC</a>

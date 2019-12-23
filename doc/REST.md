# Leitstand REST API Guidelines

## Summary
This document describes fundamental design guidelines for all Leitstand REST APIs.
A formal description for each EMS REST API is provided as [Open API 3.0 specification](https://swagger.io/specification/).

## Statelessness and Idempotency

All REST API functions are _stateless_ and ideally also _idempotent._

A REST API is stateless if no session needs to be established to store temporary state information.
As a result, the system is in a consistent state after each REST API call.

A REST API is idempotent if a repeatedly execution of the same operation always leads to the same result. 
[RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231) states that `PUT`, `DELETE` and _safe_ requests have to be idempotent. 
A _safe_ request defined as request that does not modify the state of the requested resource.

A key advantage of idempotent operations is that can safely be retried if a communication error occured.

## Push, not pull
Polling state information does not scale well in a distributed environment and come with latency. 
Hence all relevant state changes have to be advertised actively.

## Dependency Management
Dependencies, including the direction of a dependency, must be managed carefully:

1. __An API must not introduce circular dependencies.__ 
   A circular dependency exists, if system A invokes an API of system B and in turn system B invokes an API of system A to complete the same use case. 
   If a circular dependency exists, the involved systems cannot be updated independently from each other (with respect to 	the use case relying on the circular dependency).
1. __Asynchronous API call subscription shall be part of the API.__ 
   For example, it is discouraged that an asynchronous API call returns some sort of ID and the client has to query a message bus using that ID to get notified about the outcome.
   Rather the API provides an option to specify the URL to send the result of an asynchronous operation to and defines the format of the result message.
   This pattern can be used to resolve circular dependencies.

## JavaScript Object Notation (JSON)
The preferred data-exchange format is [JSON](https://www.json.org).

### Property Naming Conventions
JSON lacks type definitions and therefore the type information is not part of the JSON message. 
Consequently, JSON object property names need to be defined carefully and must obey the following rules:

1. The name of a scalar property identifies the scalar type unambiguously.
2. A property with the same name has the same semantics across all Leistand API objects.

For example, each network element has a unique, descriptive name, which is stored in the `element_name` property for all Leitstand API objects. 
In order to satisfy both rules, the element name must not be stored in another attribute than `element_name` and the `element_name` property always contains an element name.

This leads to consistent JSON objects and simplifies object validation.

## Hyper Text Transfer Protocol (HTTP)
HTTP is an application protocol devised for resource management. 
While being introduced for the transfer of Hype Text resources, HTTP is not limited to certain kinds of resources. 
The `Content-Type` header defines the type of resource being transfered and of course the semantics of HTTP can be used to transfer JSON representation of resources or collections of resources respectively. 

### URI Path Conventions

The Uniform Resource Identifier (URI) identifies each resource _unambiguously_. 

Resources and collections of resources are represented by _nouns_. 
A plural form of a noun represents a collection of resources, while the singular form of a noun represents a single resource. 
Leitstand follows the recommendation to form the plural by appending a _s_ to the noun, even if this is grammatically incorrect (e.g. _proxys_ instead of _proxies_).

Hierarchies of resources are represented by path segments accordingly. 
For example, `/elements` refers to the collection of elements while `/elements/l1.nbg/settings` refers to the settings of the element `l1.nbg`.

Resource operations are identified by a _verb_ prefixed with an underscore (`_`) by convention.
For example, `/elements/l1.nbg/_clone` clones element `l1.nbg` and assigns a new serial number to the clone.
The clone operation is part of the switch replacement process and simplifies to apply the existing element configuration to a new hardware.


### API Version Management
The API version is part of the request URI and identifies the requested version of an API. 
The API version gets incremented whenever a non-backwardcompatible modification has been made.

### HTTP Verbs

HTTP defines different request methods which are also known as HTTP verbs. The REST API leverages these verbs as follows:

- `GET` is used to load a resource.
- `PUT` is used to store a resource. 
   A `PUT` to a resource replaces the existing resource or creates a new resource if the specified resource does not exist. 
   A `PUT` to resource collection replaces the entire collection.
    `PUT` operations are _idempotent_.
- `DELETE` is used to remove a resource from the server. 
   Resource collections are not removable.
   A resource collection is considered _empty_ if it does not contain any items,
   whereas the collection as such always exists. 
   Consequently, any attempt to `DELETE` a resource collection raises an error. 
   Instead `PUT` an empty resource collection to empty a collection or `DELETE` all members of the collection in dedicated calls separately.
- `POST` is used to invoke an operation on a resource or to add a resource to a resource collection.
- `PATCH` is used to apply a patch to a collection of resources. 
   A patch can add new resources to a collection and/or remove existing resources from a collection.

### HTTP Reason Codes

The Leitstand REST APIs leverages the HTTP reason codes as follows:

- `200 OK` reports the successful execution of the operation. For `GET` requests, the response  contains the requested resource or collection of resources respectively. For any other operation, the response describes the outcome of the operation as an array of status messages.
- `201 Created` reports the creation of a new resource. As already mentioned, collections cannot be created, because they're considered as being empty instead of non-existent. The `Location` HTTP header contains the URI of the created resource.
- `202 Accepted` reports that an _asynchronous_ operation has been triggered successfully. The server has accepted to execute the operation, but the completion of operation itself is pending. Asynchronous operation issue an event to notify the caller about the outcome of the operation. The definition of the event is part of the API. The subscription of the event is part of the API invocation to avoid circular dependencies between service consumer and service provider.
- `204 No content` reports the successful execution of a `void` operation, i.e. an operation that does not provide a response message. Typically used for `DELETE` operations.
- `303 Temporarily moved` is used by the service registry to redirect the caller to the actual service.
- `304 Not modified` is used if a client wants to load a resource but the resource has not been updated since the specified last modification date or the specified entity tag equals the resource's entity tag.
- `400 Bad request` indicates that the request cannot be processed due to syntactical errors, such as malformed JSON or incorrect data types for example.
- `404 Not found` indicates that the requested resource does not exist.
- `405 Method not allowed` indicates that the intended HTTP verb is not supported for the specified resource. For instance, an attempt to `DELETE` a resource collection will raise this error.
- `409 Conflict` indicates that a concurrent modification of the same resource was detected.
- `412 Precondition failed` is reserved for reporting conditional errors in combination with `E-Tag`, `If-Match` and `If-None-Match` headers.
- `422 Unprocessable entity` indicates that a request was syntactically correct but cannot be processed for logical reasons. For instance,  an attempt to modify an immutable value will raise this error.
- `500 Internal server error` indicates a server-side error.

### HTTP Caching Directives

Each Leitstand API sets HTTP caching directives such that a client always gets the latest version of a resource when executing a `GET` request. 

Caching is either completely disabled or relies on the `If-Not-Modified-Since` or `If-None-Match`HTTP  headers in order to validate the up-to-dateness of a cached resource.
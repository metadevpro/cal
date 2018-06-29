# CAL (Common API Level)

**Version 0.0.1**

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14), [RFC2119](https://tools.ietf.org/html/rfc2119), and [RFC8174](https://tools.ietf.org/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

## Summary

**CAL (Common API Level)** is a set of small guidelines for implementing APIs in a standard way to promote interoperability between implementations.
When producers expose APIs conforming CAL, consumers can benefit from the standardization and conventions used.

Samples are provided in REST style as the main form for communication but not enforced only to this transport.

CAL recommendations are labeled. Different implementations can decide to comply with selected CAL recommendations.

Most recommendations are independent of each others. When this is not the case, when dependencies or mutually exclusive choices occurs, it will be noted explicitly.

This document contains the Specification of CAL.


## Table of Contents

- [Goals](#goals)
- [Non-Goals](#non-goals)
- [Declaration](#declaration)
- [Naming conventions](#naming-conventions)
  - [Verbs usage](#verbs-usage)
    - [CONNECT](#connect)
    - [DELETE](#delete)
    - [GET](#get)
    - [HEAD](#head)
    - [OPTIONS](#options)
    - [PATCH](#patch)
    - [POST](#post)
    - [PUT](#put)
    - [TRACE](#trace)
- [Messages Definition](#messages-definition)
  - [Criteria Message](#criteria-message)
  - [Response Message](#response-message)
  - [Error Message](#error-message)
    - [CAL-E0](#cal-e0)
    - [CAL-E1](#cal-e1)
- [CAL Recommendations](#cal-recommendations)
  - [CAL-0. Basic Resource Query](#cal-0-basic-resource-query)
  - [CAL-1. Basic Resource Management](#cal-1-basic-resource-management)
  - [CAL-1B. Delta Changes](#cal-1b-delta-changes)
  - [CAL-2. Pagination](#cal-2-pagination)
    - [CAL-2A. Client-side pagination](#cal-2a-client-side-pagination)
    - [CAL-2B. Server-side pagination](#cal-2b-server-side-pagination)
    - [CAL-2C. Time-based pagination](#cal-2c-time-based-pagination)
  - [CAL-3. Order](#cal-3-order)
  - [CAL-4. Filtering](#cal-4-filtering)
  - [CAL-5. Projection](#cal-5-projection)
  - [CAL-6. Batch Support](#cal-6-batch-support)
  - [CAL-7. Complex Query Support](#cal-6-complex-query-support)
  - [CAL-HAL. Hypermedia](#cal-hal-hypermedia)
  - [CAL-META. Metadata Services](#cal-meta-metadata-services)
  - [CAL-H. Health Info](#cal-h-health-info)
    - [CAL-H1. Ping](#cal-h1-ping)
    - [CAL-H2. Metrics](#cal-h2-metrics)
    - [CAL-H3. Auto-diagnosis](#cal-h3-auto-diagnosis)
- [Reference Implementations](#reference-implementations)
- [References](#references)

## Goals

As an API profile, CAL goals are:

- Full compatibility with [OpenAPI Spec](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md) (CAL builds on top of OpenAPI).
- API homogeneity: *"predecible syntax for same semantics"*
- API discoverability
- Principle of less surprise
- Convention-based
- Easy learning curve for API consumers
- Enable the use of server-side, client-side libraries, and/or middleware to automate repetitive task.
- Easy consumption by machines.

## Non-Goals

CAL do not expect to be:

- A new GraphQL or OData standards with single end-point

## Declaration

An API implementation must expose its CAL compliance with the following mime type:
Specific CAL recommendations can be listed as CAL1, ... CALN in the `X-Cal-Support` header.

```
Accept: application/json, application/vdr.cal.v1+json
X-Cal-Support: CAL1, CAL2, CAL3, CAL10
```

## Naming & Conventions

- Resources names are build using kebab-case conventions.
- Resources names SHOULD be used in plural **forms** to preserve homogeneity.

Sample resource **Cat**

- plural name:  `/cats`  (recommended)
- singular name:  `/cat` (to avoid)

Sample composed resource **BlackCat**

- plural name:  `/black-cats` (recommended)
- singular name:  `/black-cat`  (to avoid)

End-points described in this document are OPTIONAL: they can be implemented or excluded depending on the requirements. But if the functionality is present, the end-point is expected to be compatible as described in this document for easy discoverability.

### Verbs usage

HTTP Verbs in REST APIs are not neutral and provide semantics. CAL encourages preserving the following ones when possible:

#### CONNECT

Used in HTTP/S to setup a channel. Not directly used in CAL to provide services.

#### DELETE

Used to delete a resource.
Some stacks do not support DELETE verb with a request body. For such cases, delete by query is alternatively provided as POST with request body see [CAL-7](#cal-7-complex-query-support).

#### GET

Used only for queries with no side-effects. Semantic is ALWAYS preserved to data retrieval.
Audit and log requirements COULD generate and store additional information but the queried resource is never altered.

#### HEAD

Used to retrieve headers. Not used in CAL to provide services.

#### OPTIONS

Used to expose communication options for the target resource.
Not directly used in CAL to provide services. CORS requests are implemented using OPTIONS.
Server CAN expose supported verbs for a resource using `Allow` header.  

#### PATCH

MUST be used only to allow partial updates to resources when the API support such type of semantics. See [CAL-1B](#cal-1b-delta-changes).

#### POST

Used to send messages to server. Generic wrapper for services not matching any other more specific verb. POST operations are not safe (contains side-effects by default) and not idempotent.
Request message is enclosed in body. Its format is declared with the `Content-Type` header.
Type of response can be negotiated via `Accept` header.

#### PUT

Used to update an existing resource.
PUT has idempotent semantics. Replaying a PUT request successively has the same effect.

**NOTE**: Although, [some authors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT) allow PUT verb for creating resources, in CAL, PUT is **not used** for creating a resource (POST verb preferred). PUT is reserved only for modification of the resource.

#### TRACE

Used to debug and diagnose communication problems. Not used by CAL to provide services.

## Messages Definition

CAL provides a set of standard message types to standardize common operations.

### Criteria Message

Criteria message supports encoding query parameters for rich queries.

Sample document:

```json
{
  "count": true,
  "order": "-age, name",
  "offset": 40,
  "limit": 20,
  "groupBy": "city",
  "where": [
    { "op": "and", "clauses": [
      { "op": "eq", "field": "name", "value": "n1" },
      { "op": "gt", "field": "age", "value": 21 }
    ]}
  ],
  "distinct": true,
  "projection": "id, name, lastname, age"
}
```

### Response Message

Response message includes an envelop to support pagination information if needed/supported.

```json
{
  "meta": {
    "totalCount": 2335,
    "offset": 60,
    "limit": 30,
  },
  "data": [
    { "id": "object 1 ..." },
    { "id": "object 2 ..." }
  ],
  "_links": {
    "first": { "href": "/orders?offset=0&limit=30" },
    "previous": { "href": "/orders?offset=30&limit=30" },
    "self": { "href": "/orders?offset=60&limit=30" },
    "next": { "href": "/orders?offset=90&limit=30" },
    "last": { "href": "/orders?offset=2310&limit=30" }
  }
}
```

### Error Message

#### CAL-E0

When HTTP Error codes are enough, the recommendation is to use them as defined in the [HTTP Spec/RFC7231](https://tools.ietf.org/html/rfc7231#section-6) and avoid as much as possible returning application-dependent error codes.

#### CAL-E1

When errors need to return specific application error codes, an Error object for response is needed.
Also, if HTTP is not available as transport and we need to use other transports mechanisms, using an error structure will help to wrap the error and deliver it in a consistent way.

In this recommendation, [HTTP Spec/RFC7231](https://tools.ietf.org/html/rfc7231#section-6) error code responses still apply (i.e., complement each other).

Error messages are encoded in the following form:

```json
{
  "error": "CODE003",
  "description": "Name is a compulsory field.",
  "template": "{field} is a compulsory field.",
  "args": {
    "field": "name"
  },
  "context": null
}
```

## CAL Recommendations

### CAL-0 Basic Resource Query

1. `GET /resourceName`

Returns a list of resources. Pagination is not supported in CAL0.
Encoding could be any mime type. Typically `application/json`, or `application/xml` or any other encoding negotiated and supported by server and customers.

Server can return:

- `200 OK` + direct array of resources
- `204 No content` + no response
- `401 Unauthorized`

### CAL-1 Basic Resource Management

Includes CAL-0 plus basic operations for working with resources:

2. `GET /resourceName/{id}`

Returns a specific resource given its identifier (`id`).

Server can return:

- `200 OK` + resource
- `404 Not found`
- `401 Unauthorized`

3. `POST /resourceName`

Creates a specific resource. Resource-compliant message is sent encoded on body.

Server can return:

- `201 Created`
- `202 Accepted`
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`

4. `PUT /resourceName/{id}`

Updates a specific resource by id. Resource-compliant message is sent encoded on body.

Server can return:

- `200 OK`
- `202 Accepted`
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`
- `404 Not Found`

5. `DELETE /resourceName/{id}`

Deletes a specific resource given its identifier (`id`).

Server can return:

- `202 Accepted`
- `204 No content` Delete was successful.
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`
- `404 Not Found`

### CAL-1B Delta Changes

Delta changes to resources is a controvert issue:

  1. In pure Resource-Oriented REST APIs, delta changes are allowed and encouraged as an efficient way to send over the wire only the properties to be updated. This can be done only, when no *side-effects* must be triggered.
  
  2. On the contrary, in Domain-Driven APIs, there is a business logic layer encapsulating all the changes on services or methods. Therefore, **PATCH** operations **are forbidden** because it would break the encapsulation of the business logic.

If using API semantics like (2) **DO NOT** provide support CAL-1B. Recommendation is to create new operations using **POST** semantics to pass a message to the operation with the expected arguments for execution.

On the contrary, **PUT** operations are reserved to edit all properties of the resource.

Only when using API semantics like (1) and ensuring no additional side-effect on the properties to be changed, you COULD implement this recommendation exposing the following operation:

Use the verb PATCH, including in the body the subset of properties of the resource to be changed.

Example:

```
PATCH /resourceName/{id}

{
  "city": "Seville",
  "phone": "900 123 456"
}
```

Server can return:

- `200 OK` + no body
- `202 Accepted`
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`
- `404 Not Found`

### CAL-2 Pagination

Pagination is a though topic involving several cases of use. Al least, the most common uses cases identified are as follows:

  1. Client side defines the page size (based in UX; size of window or user preference, for example)
  2. Server side defines the page size, caching and optimizes for such pages.
  3. Time-based access: for real time data like tweets line-feed or stock-market quotes.

**NOTE:** This strategy must be selected per resource basis. Pick only one per resource.

#### CAL-2A Client-side pagination

Client defines the page size and the block requested. Access to a random block is supported.
In this case, it can model it in the following form:

All query operations support count and pagination.
Pagination parameters are:

- `limit`: (integer) blocksize (provided by client, it can have a sensible default in server, and SHOULD be limited by server to avoid performance degradation)
- `offset`: (integer) number of elements to skip from the beginning

If server responses include `meta.totalCount`, the client can derive links to access to all pages if needed.

#### CAL-2B Server-side pagination

When the server defines the page size, the client can only move the pointer to jump between pages.

Server must respond with `meta.limit` to inform the client about the page size being used.
If server responses include `meta.totalCount`, the client can provide links to access to all pages if needed.

For cursors-based query APIs, or forward-only queries, the client can use the HAL link `next` to access the next page.

#### CAL-2C Time-based pagination
Real-time data is streamed in chucks to clients. Clients ask for more data passing the last id or last timestamp of data they received. This last-known-object is enough for server to realize the next block to send.

In the same way, the client can use the HAL links `next` & `previous` to access the next and previous page respectively. URLs used in next and previous links encode the id or timestamp token.

**Note:** In all cases, the response of data uses the `Response` message form to envelop pagination information into the `meta` object instead of raw array of response objects.

### CAL-3 Order

Sorting support on queries:

- `order`: (string) order expression of the form:
  - `order=name`: order by name ascending
  - `order=-age name`: order by age descending, then, by name ascending

### CAL-4 Filtering

CAL-3 plus filtering support on queries.
 
- `criteria`: (string) filtering criteria expression (TBD)


### CAL-5 Projection
CAL-4 plus projection support on queries.

- `fields`: (string) projection criteria expression (TBD)

Example: `/resources?fields=id,name,child.id,child.name`

### CAL-6 Batch Support
CAL-5 plus batch support on queries.

1. `POST /resources` support a list of objects to create.
2. `PUT /resources` support a list of objects to modify.
3. `DELETE /resources` support a query criteria of objects to delete.

Can return:

- 200 OK with a list of resources created/modified/deleted or errors if some of them failed.
- or 202 Accepted with a list of tokens for tracking the operation status.

### CAL-7 Complex Queries Support
CAL-6 + extra API for complex query support.

Complex queries are better encoded in body than query strings. When query string is not a choice, new end-points can be provided to resolve queries and delete criteria.

The end-points are the following ones:

1. `POST /resources/query` for complex queries.
2. `POST /resources/delete-by-query` for deletion by query.

### CAL-HAL Hypermedia
[HAL Specification](http://stateless.co/hal_specification.html) is the recommendation in CAL for adding hypermedia support.
Every returned resource SHOULD include a `_links` object providing related actions.

### CAL-META Metadata Services
CAL-META is a metadata-discovery end-point to allow applications to discover resources, operations, roles, permissions.

1. `/cal/meta/resources`  Return the list of resources
2. `/cal/meta/resource/{name}`  Return the details of an specific resource
3. `/cal/meta/roles`      Return a list of roles
4. `/cal/meta/roles/{name}`      Return a role by name
5. `/cal/meta/roles/{name}/permissions`  Return the permissions for a given role

Metadata end-points can be secured and exposed ony to authorized parties.

Metadata types (TBD).

### CAL-H Health Info
For supporting best practices, some extra end-points can help to standardize heart-beat, metrics and auto-diagnosis when operating the services.

#### CAL-H1. Ping

Implements a heart-beat command for implementing service availability checking and load balancer communication:

`GET /ping`

Response:

```json
200 OK

{ "msg": "pong" }
```

This end-point COULD BE secured if needed.
The typical consumer for this end-point is a load balancer, Consul, Naggios or any other service availability tool.

#### CAL-H2. Metrics
`GET /metrics`

Returns a set of metrics for checking service health and performance.

This end-point SHOULD BE secured in most of cases.
The consumer for this end-point is the Operation team and metric tools like [Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/).
If different formats or tools are used, `Accept` header SHOULD be used to negotiate the format to use.

#### CAL-H3. Auto-diagnosis
`GET /autodiagnosis`

1. Returns the version of the product, name, environment and configuration settings to diagnose environment or deployment problems.

2. Returns a list of checks performed to ensure the system dependencies are OK (e.g., an admin status page info).

Example checks:

```json
{
  "name": "app0",
  "version": "1.2.3",
  "checks": [
    { "name":"db", "desc": "DB is accesible.", "result": true },
    { "name":"certs", "desc": "Certs present at /etc/certs.", "result": true },
    { "name":"mailserver", "error": "Mail server not responding at mail.acme.com", "result": false },
    // ...
  ]
}
```

This end-point MUST BE secured as a general rule. The main consumer for this end-point is the Operation team. Exposing product version and internal configuration to third-parties could be used as an attack-vector.

## Reference Implementations

CAL recommendations have been designed to promote interoperability and it is language and framework neutral.

A reference implementation will be available here for reference. (TBD)
Other reference implementation will also be listed here at they become available.

## References

1. HTTP/1.1 Specification [RFC7231](https://tools.ietf.org/html/rfc7231#section-6).
2. [HAL Specification](http://stateless.co/hal_specification.html).
3. [OpenAPI Specification v. 3.0.1](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md).

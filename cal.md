# CAL (Common API Level)

## Summary

**CAL (Common API Level)** is a set of small guidelines for implementing APIs in a standard way to promote interoperability between implementations.
When producers expose APIs conforming CAL, consumers can benefit from the standardization and conventions used.

Samples are provided in REST style as the main form for communication but not enforced only to this transport.

CAL recommendations are labeled. Different implementations can decide to comply with selected CAL recommendations.

Most recommendations are independent of each others. When this is not the case, when dependencies or mutually exclusive choices occurs, it will be noted explicitly.

This document contains the Specification of CAL.

## Version

Current version is **0.0.1**, initial version on 2018.05.17 Pedro J. Molina, Metadev

## Goals

CAL goals:

- API homogeneity: *"predecible syntax for same semantics"*
- API discoverability
- Principle of less surprise
- Convention-based
- Easy learning curve for API consumers
- Enable the use of server-side, client-side libraries, and/or middleware to automate repetitive task.
- Easy consumption by machines.

## Non-Goals

CAL do not expect to be:

- A new GraphQL or OData standards with single endpoint

## Declaration

An API implementation must expose its CAL compliance with the following mime type:
Specific CAL recommentations can be listed as CAL1, ... CALN in the `X-Cal-Support` header.

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

Endpoint described in this document are OPTIONAL: the can be implemented or excluded depending on the requirements. But if the functionality is present, the end-point is expected to be compatible as described in this document for easy discoverability.

## Criteria Message

Criteria message supports encoding query parameters for rich queries.

Sample document:

```json
{
  "count": true,
  "order": "-age, name",
  "skip": 40,
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

## Response Message

Response message includes an envelop to support pagination information if needed/supported.

```json
{
  "meta": {
    "totalCount": 2335,
    "skip": 60,
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

## Error Message

### CAL-E0

When HTTP Error codes are enough, the recomendation is to use them as defined in the [HTTP Spec/RFC7231](https://tools.ietf.org/html/rfc7231#section-6) and avoid as much as possible returning application dependent error codes.

### CAL-E1

When errors needs to return specific application error codes, an Error object for response is needed.
Also, if HTTP is not available as transport and we need to use other transports mechanisms, using an error structure will help to wrap the error and deliver it in a consistent way.

In this recomendation, [HTTP Spec/RFC7231](https://tools.ietf.org/html/rfc7231#section-6) error codes responses still apply (complement each other).

Error messages are encode in the following form:

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

## CAL0. Basic resource query

1. `GET /resourceName`

Returns a list of instances in the resource. Pagination is not supported in CAL0.
Encoding could be any mime type. Typically `application/json`, or `application/xml` or any other encoding negotiated and supported by server and customers.

Server can return:

- `200 OK` + direct array of instances
- `204 No content` + no response
- `401 Unauthorized`

## CAL1. Basic resource management

Includes CAL0 plus basic operations for working with resource instances:

2. `GET /resourceName/{id}`

Returns an specific instance of a resource given its identifier (id).

Server can return:

- `200 OK` + resource
- `404 Not found`
- `401 Unauthorized`

3. `POST /resourceName`

Creates an specific resource. Resource compliant message is sent encoded on body.

Server can return:

- `201 Created`
- `202 Accepted`
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`

4. `PUT /resourceName/{id}`

Updates an specific resource by id. Resource compliant message is sent encoded on body.

Server can return:

- `200 OK`
- `202 Accepted`
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`
- `404 Not Found`

5. `DELETE /resourceName/{id}`

Deletes an specific resource by id.

Server can return:

- `202 Accepted`
- `204 No content` Delete was successful.
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`
- `404 Not Found`

## CAL1B. Delta changes to resources

Delta changes to resources is a controvert issue:

  1. In a pure resource oriented REST APIs, delta changes are allowed and encouraged as an efficient way to send over the wire only the properties to be updated. This can be done only, when no *side-effects* must be triggered.
  
  2. In Domain Driven APIs, there is a business logic layer encapsulating all the changes on services or methods. Therefore, **PATCH** operations are forbidden because it would break the encapsulation of the business logic.

If using API semantics like (2) **DO NOT** provide support CAL1B. Recommendation is to create new operations using **POST** semantics to pass a message to the operation with the expected arguments for execution.

If using API semantics like (1) and has no side-effect on the properties to be changed, you can expose the following operation:

Using the verb PATCH, including in the body the subset of properties of the resource instance to be changed.

Example:

```
PATCH /resourceName/{id}

{
  "city": "Seville",
  "phone": "900 123 456"
}
```

Server can return:

- `200 OK`
- `202 Accepted`
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`
- `404 Not Found`

## CAL2. Pagination

Pagination is a though topic involving several cases of use. Al least the most common uses cases identified are as follows:

  a. Client side defines the page size (based in UX; size of window or user preference, for example)b. Server side defines the page size, caching and optimizes for such pages.
.
  c. Time-based access: for real time data like tweets line-feed or stock-market quotes.

**NOTE:** This strategy must be selected per resource basis. Pick only one per resource.

### CAL2A. Client side

Client defines the page size and the block requested. Access to a random block is supported.
In this case, we can model it in the following form:

All query operations support count and pagination.
Pagination parameters are:

- `limit`: (integer) blocksize (provided by client, can have a sensible default in server, limited by server to avoid performance degradation)
- `offset`: (integer) number of elements to skip from the beginning

If server responses includes `meta.totalCount`, the client can provide links to access to all pages if needed.

### CAL2B. Server side

When the server side defines the page size. The customer can only move the pointer to jump between pages.

Server must respond `meta.limit` to inform client about the page size used.
If server responses includes `meta.totalCount`, the client can provide links to access to all pages if needed.

For cursors-based query APIs, or forward only queries, the client can use the HAL link `next` to access the next page.

### CAL2C. Time based
Real-time data is streamed in chucks to clients. CLients ask for more data passing the last id or last timestamp of data they received. This last-known-object is enough for server to realize the next block to send.

In the same way, the client can use the HAL links `next` & `previous` to access the next and previous page respectively.

**Note:** In all cases, the response of data uses the `Response` message form to envelop pagination info into the `meta` object instead of raw array of response objects.

## CAL3. Order

Order support on queries:

- `order`: (string) order expression of the form:
  - `order=name`: order by name ascending
  - `order=-age name`: order by age descending, then, by name ascending

## CAL4. Filtering

CAL3 plus filtering support on queries.
 
- `criteria`: (string) filtering criteria expression (TDB)


## CAL5. Projection
CAL4 plus projection support on queries.

- `fields`: (string) projection criteria expression (TDB)

Example: `/resources?fields=id,name,child.id,child.name`

## CAL6. Batch support
CAL5 plus batch support on queries.

1. `POST /resources` support a list of objects to create.
2. `PUT /resources` support a list of objects to modify.
3. `DELETE /resources` support a query criteria of objects to delete.

Can return:

- 200 OK with a list of resources created/modified/deleted or errors if some of them failed.
- or 202 Accepted with a list of tokens for tracking the operation status.

## CAL7. Complex queries support
CAL6 + extra API for complex query support.

Complex queries are better encoded in body than query strings. When query string is not a choice, new endpoints can be provided to resolve queries and delete criteria.

The endpoints are the following ones:

1. `POST /resources/query` for complex queries.
2. `POST /resources/delete-by-query` for deletion by query.

## CAL-HAL
[HAL Specification](http://stateless.co/hal_specification.html) is recommended to add hypermedia support.
Every object SHOULD include a `_links` object providing related actions.

## CAL-META
CAL-META is a metadata-discovery endpoint to allow applications to discover resources, operations, roles, permissions.

1. `/cal/meta/resources`  Return the list of resources
2. `/cal/meta/resource/{name}`  Return the details of an specific resource
3. `/cal/meta/roles`      Return a list of roles
4. `/cal/meta/roles/{name}`      Return a role by name
5. `/cal/meta/roles/{name}/permissions`  Return the permissions for a given role

Metadata endpoints can be secured and exposed ony to authorized parties.

## Health Info
For best practices on production some extra endpoints can help to standardize heart-beat, metrics and auto-diagnosis.

### Ping

Implements a heart-beat command for implementing service availability checking and load balancer communication,

`GET /ping`

Response:

```json
200 OK

{ "msg": "pong" }
```

This endpoint COULD BE secured if needed.
The typical consumer for this endpoint is a load balancer, Consul, Naggios or any other service availability tool.

### Metrics
`GET /metrics`

Returns a set of metrics for checking service health and performance.

This endpoint SHOULD BE secured in most of cases.
The consumer for this endpoint is the Operation team and metric tools like f.e. Prometheus.

### Auto-diagnosis
`GET /autodiagnosis`

1. Returns the version of the product, name, environment and configuration settings to diagnose environment or deployment problems.

2. Returns a list of checks performed to ensure the system dependencies are OK. (Status page info).

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

This endpoint MUST BE secured in 99,99% of cases. The main consumer for this endpoint is the Operation team.

## Reference Implementation

CALx spec has been designed to promote interoperability and it is language and framework neutral.

A reference implementation is available here for reference. (TBD)
Other reference implementation will be listed here at they become available.


## References

1. HTTP/1.1 Specification [RFC7231](https://tools.ietf.org/html/rfc7231#section-6).

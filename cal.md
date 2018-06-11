# CAL (Common API Level)

## Summary

CAL (Common API Level) is a set of incremental guidelines for implementing APIs in an standard way to promote interoperability between implementations.
When producers expose APIs conforming CAL, consumers can benefit from the standardization and conventions used.

Samples are provided in REST style as the main form for communication but not enforced only to this transport.

CAL compliance can be achieved incrementally increasing the level depending on the degree of compliance.

This document contains the Specification of CAL.

## Version

Current version is **0.0.1**, initial version on 2018.05.17 Pedro J. Molina, Metadev

## Goals

CAL goals:

- API homogeneity
- API discoverability
- Principle of less surprise
- Convention-based
- Easy learning curve for API consumers
- Use of server-side and client-side libraries to automate repetitive task.
- Easy consumption by machines.

## Declaration

An API implementation must expose its CAL compliance with the following mime type:

```
Accept: application/json, application/vdr.cal.v1+json
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
  "order": "-age name"
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
  "totalCount": 12345,
  "skip": 100,
  "limit": 30
  "data": [
    ...
  ]
}
```

## Error Message

Error messages are encode in the following form:

```json
{
  "error": "CODE003",
  "description": "Name is a compulsory field."
  "template": "{field} is a compulsory field.",
  "args": [
    "field": "name"
  ],
  "context": null
}
```


## CAL0. Basic resource query

1. `GET /resourceName`

Returns a list of instances in the resource. Pagination is not supported in CAL0.
Encoding could be any mime type. Typically `application/json`, or `application/xml` or any other encoding negotiated and supported by server and customers.

Server can return:

- `200 OK` + direct array of instances
- `204 No content` + empty array
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
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`

4. `PUT /resourceName/{id}`

Updates an specific resource by id. Resource compliant message is sent encoded on body.

Server can return:

- `200 OK`
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`

5. `DELETE /resourceName/{id}`

Deletes an specific resource by id.

Server can return:

- `200 OK`
- `422 Error precondition failed`
- `400 Malformed message`
- `401 Unauthorized`

## CAL2. Pagination

CAL1 plus pagination support with the following semantics:
All query operations support count and pagination.
Pagination parameters are:

- `limit`: (integer) blocksize
- `skip`: (integer) number of elements to skip from initial one

**Note:** Response of data uses the `Response` message form to envelop pagination info instead of raw array of response objects.

## CAL3. Order

CAL2 plus order support on queries:

- `order`: (string) order expression of the form: 
- `name`: order by name ascending
- `-age name`: order by age descending, the by name ascending 

## CAL4. Filtering

CAL3 plus filtering support on queries.
 
- `criteria`: (string) filtering criteria expression (TDB)


## CAL5. Projection
CAL4 plus projection support on queries.

- `projection`: (string) projection criteria expression (TDB)

## CAL6. Batch support
CAL5 plus batch support on queries.

1. POST /resources support a list of objects to create.
2. PUT /resources support a list of objects to modify.
3. DELETE /resources support a query criteria of objects to delete.

## CAL7. Complex queries support
CAL6 + extra API for complex query support.

Complex queries are better encoded in body than query strings. When query string is not a choice, new endpoints can be provided to resolve queries and delete criteria.

The endpoints are the following ones:

1. `POST /resources/query` for complex queries.
2. `POST /resources/delete-by-query` for deletion by query.

## HAL
[HAL Specification](http://stateless.co/hal_specification.html) is recommended to add hypermedia support.
Every object SHOULD include a `_links` object providing related actions.

## CAL-META
CAL-META is a metadata discovery endpoint to allow applications to discover resources, operations, roles, permissions.

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
`GET /auto-diagnosis`

1. Returns the version of the product, name, environment and configuration settings to diagnose environment or deployment problems.

2. Returns a list of checks performed to ensure the system dependencies are OK. (Status page info).

Example checks:

```json
{
  "name": "app0",
  "version": "1.2.3"
  "checks": [
    { "name":"db", "desc": "DB is accesible.", "result": true },
    { "name":"certs", "desc": "Certs present at /etc/certs.", "result": true },
    { "name":"mailserver", "error": "Mail server not responding at mail.acme.com", "result": false },
    ...
  ]
}
```

This endpoint MUST BE secured in 99,99% of cases. The main consumer for this endpoint is the Operation team.

## Reference Implementation

CALx spec has been designed to promote interoperability and it is language and framework neutral.

A reference implementation is available here for reference. (TBD)
Other reference implementation will be listed here at they become available.

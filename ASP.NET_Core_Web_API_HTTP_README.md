# ASP.NET Core Web API

## Lesson 2 --- Parts 2, 3, and 4

This document covers:

-   **Part 2:** HTTP Methods in Depth
-   **Part 3:** Important HTTP Concepts --- Safe and Idempotent Methods
-   **Part 4:** HTTP Status Codes

------------------------------------------------------------------------

# Part 2 --- HTTP Methods in Depth

HTTP methods describe the operation a client wants to perform on a
resource.

In a RESTful API, the URL identifies the resource, while the HTTP method
describes the intended operation.

For example:

``` http
GET    /api/products
GET    /api/products/10
POST   /api/products
PUT    /api/products/10
PATCH  /api/products/10
DELETE /api/products/10
```

## 2.1 GET --- Retrieve Resources

`GET` retrieves information from the server.

Get all products:

``` http
GET /api/products
```

Example response:

``` json
[
  {
    "id": 1,
    "name": "Laptop",
    "price": 25000
  },
  {
    "id": 2,
    "name": "Mouse",
    "price": 500
  }
]
```

Get one product:

``` http
GET /api/products/1
```

Example response:

``` json
{
  "id": 1,
  "name": "Laptop",
  "price": 25000
}
```

### Important points

-   GET should not modify server state.
-   GET responses can be cached when the appropriate HTTP caching rules
    are used.
-   Query parameters can be used for filtering, searching, and
    pagination.
-   GET requests should not rely on a request body.

Examples:

``` http
GET /api/products?categoryId=5
GET /api/products?page=1&pageSize=10
GET /api/products?search=laptop
```

## 2.2 POST --- Create Resources

`POST` is commonly used to create a new resource.

Request:

``` http
POST /api/products
Content-Type: application/json
```

Request body:

``` json
{
  "name": "Monitor",
  "price": 7000
}
```

If the resource is created successfully, the server should commonly
return:

``` http
HTTP/1.1 201 Created
```

Example response body:

``` json
{
  "id": 4,
  "name": "Monitor",
  "price": 7000
}
```

### Why does the server generate the ID?

In many applications, the database generates the ID when a new record is
inserted. The API then returns the created resource, including its
generated ID.

Example SQL:

``` sql
INSERT INTO Products (Name, Price)
VALUES ('Monitor', 7000);
```

### Is POST idempotent?

Generally, no. Sending the same POST request more than once may create
multiple resources.

For example, submitting the same order request twice could create two
orders. Systems that need protection against duplicate operations can
use idempotency keys or other deduplication techniques.

## 2.3 PUT --- Replace a Resource

`PUT` is used to create or replace the state of a resource at a known
URI, depending on the API design.

Example:

``` http
PUT /api/products/1
Content-Type: application/json
```

Request body:

``` json
{
  "name": "Gaming Laptop",
  "price": 30000,
  "description": "High-performance laptop",
  "stock": 15,
  "categoryId": 2
}
```

PUT represents replacement of the resource representation. Therefore, a
typical PUT endpoint expects the complete set of fields that its
contract requires.

If an API accepts only selected fields and leaves all other fields
unchanged, that behavior is closer to PATCH semantics.

## 2.4 PATCH --- Partially Update a Resource

`PATCH` applies partial modifications to a resource.

For example, suppose we only want to change a product's price:

``` http
PATCH /api/products/1
Content-Type: application/merge-patch+json
```

Request body:

``` json
{
  "price": 28000
}
```

Only the price is changed; the other fields remain unchanged.

**Important:** PATCH does not define one universal request-body format.
An API must specify which patch format and behavior it supports. JSON
Merge Patch is one possible format.

## 2.5 PUT vs. PATCH

  -----------------------------------------------------------------------
  PUT                                 PATCH
  ----------------------------------- -----------------------------------
  Replaces the resource               Applies partial modifications
  representation                      

  Usually sends all fields required   Usually sends only the changes
  by the endpoint contract            

  Idempotent by HTTP semantics        Not necessarily idempotent

  Useful for full replacement         Useful for partial updates
  -----------------------------------------------------------------------

## 2.6 DELETE --- Delete a Resource

`DELETE` requests removal of the association between the target resource
and its current functionality or representation.

Example:

``` http
DELETE /api/products/1
```

A successful response could be:

``` http
HTTP/1.1 204 No Content
```

`204 No Content` means the operation succeeded and the response has no
body.

Another valid response is `200 OK` with a response body describing the
result.

### Is DELETE idempotent?

Yes, DELETE is idempotent according to HTTP semantics. Repeating the
same DELETE request should have the same intended effect on server state
as performing it once.

The response status may differ between requests. For example, the first
request might return 204, while a later request might return 404 because
the resource no longer exists. This does not change the idempotency of
the intended effect.

------------------------------------------------------------------------

# Part 3 --- Safe and Idempotent Methods

These concepts are important for API design, HTTP caching, retries, and
backend interviews.

## 3.1 What Is a Safe Method?

A safe method is intended to retrieve information without requesting a
change to the server's state.

For example:

``` http
GET /api/products
```

This request should retrieve products. It should not create, update, or
delete them.

Safe does not mean that absolutely nothing can happen on the server.
Logging and metrics may still be recorded. It means the client did not
request a state-changing operation.

## 3.2 What Is an Idempotent Method?

A method is idempotent if making the same request multiple times has the
same intended effect on server state as making it once.

Example:

``` http
PUT /api/products/1
```

Suppose the request sets the product name to `Gaming Laptop` and its
price to `30000`.

Sending that same replacement request five times should leave the
product in the same final state as sending it once.

The response does not have to be identical every time. Idempotency
concerns the intended effect on server state.

## 3.3 Safe and Idempotent Comparison

  HTTP Method     Safe?       Idempotent?
  ------------- ------- -----------------
  GET               Yes               Yes
  HEAD              Yes               Yes
  POST               No      Generally no
  PUT                No               Yes
  PATCH              No   Not necessarily
  DELETE             No               Yes

### Key differences

-   **Safe:** Does the method request a change to server state?
-   **Idempotent:** Does repeating the same request have the same
    intended effect as performing it once?

A method can be idempotent without being safe. PUT and DELETE are
examples.

------------------------------------------------------------------------

# Part 4 --- HTTP Status Codes

An HTTP response includes a status code that communicates the result of
a request.

Status codes are divided into five categories.

  Category   Meaning
  ---------- ---------------
  1xx        Informational
  2xx        Success
  3xx        Redirection
  4xx        Client error
  5xx        Server error

## 4.1 1xx --- Informational

These codes indicate that the request has been received and processing
is continuing.

They are less commonly handled directly in everyday controller
development.

## 4.2 2xx --- Success

These codes indicate that the request was successfully received,
understood, and accepted or processed.

  -----------------------------------------------------------------------
  Code                    Name                    When to Use
  ----------------------- ----------------------- -----------------------
  200                     OK                      Request succeeded

  201                     Created                 A resource was
                                                  successfully created

  202                     Accepted                Request accepted for
                                                  processing, but
                                                  processing may not be
                                                  complete

  204                     No Content              Request succeeded and
                                                  there is no response
                                                  body
  -----------------------------------------------------------------------

### ASP.NET Core examples

Return `200 OK`:

``` csharp
return Ok(products);
```

Return `201 Created`:

``` csharp
return CreatedAtAction(
    nameof(GetById),
    new { id = product.Id },
    product
);
```

`CreatedAtAction` commonly returns 201 and includes a `Location` header
pointing to the created resource, provided the target action and route
are configured correctly.

Return `204 No Content`:

``` csharp
return NoContent();
```

## 4.3 4xx --- Client Errors

These codes indicate that there is a problem with the request or that
the requested operation cannot be fulfilled because of the client's
request or permissions.

  -----------------------------------------------------------------------
  Code                    Name                    Explanation
  ----------------------- ----------------------- -----------------------
  400                     Bad Request             The request is invalid
                                                  or cannot be processed
                                                  due to a client-side
                                                  request problem

  401                     Unauthorized            Authentication
                                                  credentials are missing
                                                  or invalid

  403                     Forbidden               The server understood
                                                  the request, but the
                                                  user is not allowed to
                                                  perform it

  404                     Not Found               The requested resource
                                                  was not found

  405                     Method Not Allowed      The HTTP method is not
                                                  supported for the
                                                  target resource

  409                     Conflict                The request conflicts
                                                  with the current state
                                                  of the resource

  415                     Unsupported Media Type  The request content
                                                  type is not supported

  422                     Unprocessable Content   The request syntax is
                                                  valid, but the
                                                  instructions or data
                                                  cannot be processed
  -----------------------------------------------------------------------

### Important interview question: 401 vs. 403

Imagine we have an admin dashboard endpoint:

``` http
GET /api/admin/users
```

**Scenario A: The user is not logged in.**

The server returns:

``` http
401 Unauthorized
```

The user has not provided valid authentication credentials.

**Scenario B: The user is logged in but has the Customer role instead of
Admin.**

The server returns:

``` http
403 Forbidden
```

The user is authenticated but does not have permission to access the
resource.

Remember:

-   401: Authentication problem.
-   403: Authorization or permission problem.

## 4.4 5xx --- Server Errors

These codes indicate that the server failed to fulfill an apparently
valid request.

  -----------------------------------------------------------------------
  Code                    Name                    Explanation
  ----------------------- ----------------------- -----------------------
  500                     Internal Server Error   An unexpected
                                                  server-side error
                                                  occurred

  502                     Bad Gateway             A gateway or proxy
                                                  received an invalid
                                                  response from an
                                                  upstream server

  503                     Service Unavailable     The server is
                                                  temporarily unable to
                                                  handle the request

  504                     Gateway Timeout         A gateway or proxy did
                                                  not receive a timely
                                                  response from an
                                                  upstream server
  -----------------------------------------------------------------------

### Example: Internal Server Error

Suppose your API tries to access SQL Server, but an unexpected exception
occurs.

The API may return:

``` http
500 Internal Server Error
```

In professional applications:

-   Handle exceptions centrally.
-   Log useful diagnostic information.
-   Return a safe error response to the client.
-   Never expose database credentials, stack traces, or sensitive
    internal details in production responses.

------------------------------------------------------------------------

# Quick Revision

## HTTP Methods

  Method   Main Purpose
  -------- ------------------------------------------
  GET      Retrieve resources
  POST     Create a resource or submit an operation
  PUT      Replace a resource representation
  PATCH    Partially modify a resource
  DELETE   Delete a resource

## Status Codes to Remember

  Code   Meaning
  ------ -----------------------------------------------------
  200    OK
  201    Created
  202    Accepted
  204    No Content
  400    Bad Request
  401    Unauthorized --- authentication required or invalid
  403    Forbidden --- insufficient permission
  404    Not Found
  409    Conflict
  415    Unsupported Media Type
  422    Unprocessable Content
  500    Internal Server Error
  503    Service Unavailable

------------------------------------------------------------------------

# Practice Questions

Try answering these questions without looking at the explanations.

1.  What is the difference between PUT and PATCH?
2.  What is the difference between a safe method and an idempotent
    method?
3.  Why is POST generally not idempotent?
4.  Why is DELETE considered idempotent even if a repeated request
    returns 404?
5.  What is the difference between HTTP 401 and HTTP 403?
6.  When should an API return 201 Created?
7.  What is the difference between 200 OK and 204 No Content?
8.  When might an API return 409 Conflict?
9.  What is the difference between 400 and 422?
10. Why should an API avoid exposing internal exception details in a 500
    response?

## Practical Assignment

Design REST endpoints for an e-commerce API that supports:

1.  Retrieve all categories.
2.  Retrieve a category by ID.
3.  Create a category.
4.  Update a category.
5.  Delete a category.
6.  Retrieve products belonging to a specific category.
7.  Change only a product's price.

For each operation, identify:

-   HTTP method.
-   Endpoint.
-   Expected successful HTTP status code.

**Professional development objective:** Understand the semantics behind
HTTP methods and status codes, rather than memorizing them. These
concepts form the foundation for designing reliable ASP.NET Core Web
APIs.

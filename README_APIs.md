# Why Do We Need APIs?

## 1. Introduction

Imagine that we want to build a **Medical Management System** that works
on three platforms:

-   Website
-   Mobile application
-   Desktop application

Each platform needs a user interface, but they all need to work with the
same medical information and business rules.

## 2. The Traditional Approach: Three Independent Applications

We could build each application independently:

  Layer      Website                 Mobile                Desktop
  ---------- ----------------------- --------------------- ---------
  Frontend   HTML, CSS, JavaScript   Kotlin / Android UI   Java UI
  Backend    C#                      Kotlin                Java
  Database   SQL Server              SQLite                Oracle

This approach creates several problems.

### Problem 1: Isolated databases

-   Each application has its own database.
-   Patient information may be stored in three different places.
-   If a patient updates their information on the website, the mobile
    and desktop applications may not receive that update.
-   We need extra mechanisms to synchronize the data.

### Problem 2: Code duplication

-   We implement the same business logic three times.
-   For example, calculating a patient's medical bill must be
    implemented separately in C#, Kotlin, and Java.
-   Fixing a bug requires changing and testing multiple applications.

### Problem 3: Maintenance complexity

-   We need to maintain three backends and potentially three databases.
-   We need different technical expertise.
-   Testing, deployment, security, and updates become more complicated.

## 3. The Solution: Shared Backend Architecture

Instead of creating three independent applications, we can create a
shared architecture:

``` text
       WEBSITE             MOBILE             DESKTOP
     HTML/CSS/JS           Kotlin              Java
          |                  |                  |
          +------------------+------------------+
                             |
                       HTTP Requests
                             |
                    SHARED BACKEND
                     ASP.NET Core
                         (C#)
                             |
                       SHARED DATABASE
                       SQL Server
```

In this architecture:

1.  Each platform has its own frontend.
2.  All platforms communicate with one shared backend.
3.  Business logic is implemented once in C#.
4.  The backend accesses a shared database.
5.  Each client receives the data it needs through an API.

This avoids duplicated business logic and helps keep data consistent.

**Important:** The frontend applications should not connect directly to
the database. They should communicate with the backend, which handles
validation, authorization, business rules, and database access.

## 4. Communication Challenges

Imagine that the website is hosted on Server A, the backend on Server B,
and the database on Server C.

How can these different machines and technologies communicate?

There are four main challenges.

### 1. Machine-to-machine communication

How can two different machines communicate over a network?

### 2. Method invocation

How can a Java or Kotlin application request an operation implemented in
C#?

The client cannot directly execute a method inside another application's
process just because it knows the method name.

### 3. Data exchange

What format should we use to send and receive data between applications?

For example, how should a Java application send patient information to a
C# backend?

### 4. Service description

How can the client discover:

-   Which operations or endpoints are available?
-   What parameters are required?
-   What data types should be sent?
-   What response will be returned?

Historically, two important approaches have been used to address these
challenges: SOAP-based Web Services and RESTful Web Services.

## 5. SOAP (Simple Object Access Protocol)

SOAP is a **protocol** for exchanging structured messages between
applications.

It addresses the communication challenges as follows:

  -----------------------------------------------------------------------
  Challenge                           SOAP approach
  ----------------------------------- -----------------------------------
  Machine communication               Commonly uses HTTP; other transport
                                      protocols are also possible

  Method invocation                   SOAP messages invoke operations
                                      defined by the service contract;
                                      client proxies can simplify calls

  Data exchange                       XML

  Service description                 WSDL
  -----------------------------------------------------------------------

### A. Machine communication

SOAP commonly uses HTTP to send messages between different machines.
SOAP is not limited to HTTP.

### B. Method invocation using proxy classes

Suppose we have a C# method:

``` csharp
public Patient GetPatient(int id)
{
    // Retrieve the patient from the database
}
```

A Java application cannot directly invoke this C# method across separate
processes and servers.

With SOAP, a client can use the service's WSDL contract to generate a
client-side proxy class.

The proxy acts as an intermediary:

``` text
Java Application
       |
       v
   Proxy Class
       |
       | Converts the call into a SOAP request
       v
  C# SOAP Web Service
```

The proxy handles communication details. It can make a remote operation
look similar to a local method call in the client code.

**Important:** The proxy is not the original C# method. It is a
client-side representation that sends a request to the remote service
and handles the response.

### C. Data exchange using XML

SOAP messages use XML.

Example (simplified):

``` xml
<soap:Envelope>
    <soap:Body>
        <GetPatient>
            <id>10</id>
        </GetPatient>
    </soap:Body>
</soap:Envelope>
```

### D. Service description using WSDL

WSDL stands for **Web Services Description Language**.

It describes the service contract, including:

-   Available operations.
-   Input parameters.
-   Output messages and types.
-   Communication bindings.
-   Service endpoint information.

Client tools can use WSDL to generate proxy classes.

### SOAP limitations

-   XML messages can be verbose.
-   SOAP has additional protocol structure and complexity.
-   Client tooling and configuration can be more complicated.
-   Browser-based JavaScript clients do not naturally consume SOAP
    through generated proxy classes in the same way traditional SOAP
    clients do.

**Clarification:** AJAX did not make SOAP impossible. JavaScript can
call SOAP services using HTTP and XML, but consuming REST APIs is
generally simpler for browser applications.

## 6. RESTful Web Services

REST stands for **Representational State Transfer**.

REST is an **architectural style**, not a protocol. RESTful APIs
commonly use HTTP to allow different applications to communicate.

It addresses the same challenges:

  Challenge                 REST approach
  ------------------------- ----------------------------------------------
  Machine communication     Usually HTTP
  Resource identification   URI
  Data exchange             JSON, XML, and other representations
  Service description       OpenAPI; commonly displayed using Swagger UI

### A. Machine communication using HTTP

REST APIs commonly use these HTTP methods:

  HTTP method   Purpose
  ------------- --------------------------------------------
  GET           Retrieve data
  POST          Create a resource
  PUT           Replace a resource or update it completely
  PATCH         Partially update a resource
  DELETE        Delete a resource

### B. Resource identification using URIs

In REST, we generally identify **resources** using URIs rather than
exposing every method as a separate URL.

For example:

``` http
GET    /api/patients
GET    /api/patients/10
POST   /api/patients
PUT    /api/patients/10
PATCH  /api/patients/10
DELETE /api/patients/10
```

-   `/api/patients` identifies the patients collection.
-   `/api/patients/10` identifies a specific patient.
-   The HTTP method indicates the intended operation.

Compare:

SOAP-style operation:

``` text
GetPatient(10)
```

REST-style request:

``` http
GET /api/patients/10
```

### C. Data exchange using JSON

REST APIs commonly use JSON because it is lightweight and supported by
JavaScript, Kotlin, Java, and C#.

Example request:

``` json
{
    "name": "Ahmed",
    "age": 25,
    "phone": "01012345678"
}
```

Example response:

``` json
{
    "id": 10,
    "name": "Ahmed",
    "age": 25,
    "phone": "01012345678"
}
```

REST is not limited to JSON. It can use XML and other representations
too. BSON is a binary JSON-like format, but it is not a standard REST
requirement.

### D. Service description using OpenAPI and Swagger

**OpenAPI** is a specification for describing HTTP APIs.

**Swagger** is a collection of tools that work with OpenAPI documents.
Swagger UI provides an interactive page where developers can explore and
test API endpoints.

An API description can include:

-   Endpoint URL.
-   HTTP method.
-   Required parameters.
-   Request body schema.
-   Response schema.
-   HTTP status codes.
-   Authentication requirements.

## 7. SOAP vs REST

  -----------------------------------------------------------------------
  Feature                 SOAP                    REST
  ----------------------- ----------------------- -----------------------
  Type                    Protocol                Architectural style

  Communication           HTTP and other          Usually HTTP
                          transports              

  Data format             XML                     JSON, XML, and other
                                                  representations

  Main abstraction        Service operations      Resources identified by
                                                  URIs

  Service description     WSDL                    OpenAPI is commonly
                                                  used

  Client invocation       Often uses generated    Usually sends HTTP
                          proxies                 requests

  Complexity              More protocol structure Often simpler for web
                                                  APIs
  -----------------------------------------------------------------------

### Important comparison notes

-   REST is not necessarily faster in every situation.
-   SOAP is not obsolete.
-   SOAP can be useful in enterprise systems that rely on standardized
    contracts and WS-\* features.
-   REST is widely used for web and mobile APIs because it fits
    naturally with HTTP and common client technologies.

## 8. Practical Example Using ASP.NET Core Web API

Imagine that we want to create an API for our medical system.

A simple controller could look like this:

``` csharp
[ApiController]
[Route("api/[controller]")]
public class PatientsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetPatient(int id)
    {
        // In a real application, retrieve the patient
        // through a service and database.

        return Ok(new
        {
            Id = id,
            Name = "Ahmed"
        });
    }
}
```

The endpoint is:

``` http
GET /api/patients/10
```

Different clients can call the same endpoint.

### Website --- JavaScript

``` javascript
const response = await fetch("/api/patients/10");

if (!response.ok) {
    throw new Error("Could not retrieve the patient");
}

const patient = await response.json();

console.log(patient.name);
```

### Mobile --- Kotlin

The mobile application can call the same endpoint using an HTTP client
such as Retrofit or Ktor Client.

### Desktop --- Java

The desktop application can call the same endpoint using Java's
`HttpClient`.

All three clients communicate with the same backend and receive the same
patient information.

## 9. Important Concepts to Remember

Distinguish these three concepts:

1.  **API (Application Programming Interface):** An interface that
    defines how one software component or application can interact with
    another.
2.  **Web API:** An API exposed over a web-based communication
    mechanism, commonly HTTP.
3.  **REST API:** A Web API designed according to REST architectural
    principles.

An API does not necessarily mean REST. SOAP is also a way to build Web
Services.

## 10. Recommended Learning Order for ASP.NET Core

To understand Web APIs properly, study these topics in order:

1.  Client-server architecture.
2.  What is an API, and why do we need it?
3.  HTTP protocol.
4.  HTTP methods.
5.  REST architectural principles.
6.  JSON serialization and deserialization.
7.  HTTP status codes.
8.  Routing and controllers in ASP.NET Core.
9.  Dependency Injection and service layers.
10. Swagger and OpenAPI.
11. Authentication and authorization.
12. API versioning and error handling.

## 11. Interview Practice

Try answering this question in your own words:

**Why can't a Java application directly call a method written in C# if
both applications are running on different servers?**

Think about:

-   Different processes and runtime environments.
-   Network communication.
-   HTTP requests and responses.
-   Serialization and deserialization.
-   Remote procedure calls and client proxies.

------------------------------------------------------------------------

## Final Summary

The main reason we need APIs is to allow different applications,
platforms, and technologies to communicate through a well-defined
interface.

Instead of implementing the same business logic separately for a
website, mobile application, and desktop application, we can build:

-   Separate frontends for each platform.
-   One shared backend.
-   One shared database accessed through the backend.
-   APIs that define how clients communicate with the backend.

This architecture reduces duplicated business logic and makes
applications easier to maintain and extend.

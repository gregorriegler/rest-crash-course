
# REST

Representational State Transfer 

An architectural style to transfer State by its representation from one Machine to another in a Distributed System.
Developed 1994-2000 by Roy Fielding. 
Idea of RESTful much younger ~2014, and mostly misunderstood.

## HTTP Request / Response Model
Each Client Server interaction consists of both

- a Request which the Client sends to the Server 
- and after that a Response the Server answers back to the Client.

```
Request:
<Initializing Line> 
<Many Lines of Request Headers>

<Sometimes a Request Body e.g. JSON to represent the transfered State>
------------------------------
Response:
<Initializing Line> 
<Many Lines of Response Headers>

<Sometimes a Response Body e.g. JSON to represent transfered State>
```

A real Example:
```
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: www.google.at
User-Agent: HTTPie/1.0.3
------------------------------
HTTP/1.1 200 OK
Cache-Control: private, max-age=0
Content-Encoding: gzip
Content-Length: 5227
Content-Type: text/html; charset=ISO-8859-1
Date: Wed, 15 Jul 2020 19:27:07 GMT
Set-Cookie: 1P_JAR=2020-07-15-19; expires=Fri, 14-Aug-2020 19:27:07 GMT; path=/; domain=.google.at; Secure
Set-Cookie: NID=204=spUoQuJnONBhc7ba46s1td58Vwe9vqbbpKuU3Kk1OW01vurrueyo; expires=Thu, 14-Jan-2021 19:27:07 GMT; path=/; domain=.google.at; HttpOnly

<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" lang="de-AT"><head><meta content="text/html; charset=UTF-8" http-equiv="Content-Type"> ... html continues ...
```
## Common HTTP Status Codes
- **2xx Success**
  - 200 OK
  - 201 Created
  - 204 No Content
- **3xx Redirection**
  - 301 Moved Permanently
  - 303 See Other (Call Location with GET)
  - 304 Not Modified (Use Cache)
- **4xx Error, Client fucked up**
  - 400 Bad Request
  - 401 Unauthorized (Not Authenticated)
  - 403 Forbidden (Authenticated, but not Authorized)
  - 404 Not Found
  - 405 Method Not Allowed
  - 406 Not Acceptable
  - 418 I'm a teapot 😁
- **5xx Error, Server fucked up**
  - 500 Internal Server Error
  - 503 Service Unavailable

## Strictly Stateless
Each request must contain all information necessary to fulfill it.
The server never relies on any information from a previous request (This excludes Persistent State).
Such information is called Session State and is handled entirely on the client side.
The same is true for authentication, which should not be stored in a session.
Instead each request should come with a credential that will be validated by the server.

### Motivation behind it
No hidden State means full Transparency which helps understand a servers behaviour and reproduce bugs.
Interactions become Reliable, Repeatable and easily Testable.
Just like a pure function. Yes, the web is inherently Functional.
Enables horizontal scalability as any server can handle any request - No Session state has to be replicated.
Server never loses track of "where" a client is.
Consider how much games a Chess Master could play simultaniously, did he not have to remember the state of each game?
Removes complexity from the Server, less bugfoots (A bug seen once but never again).


## Richardson Maturity Model
Developed by Leonard Richardson

### Level 0: Call remote functions via a single HTTP Entry point
Could look similar to this: A POST request with some data to invoke a remote procedure.

```
POST http://hsp.com/api
{
    "action": "createClass",
    "content": {
        "id": "4A",
        "teacher": "Max",
        "room": 123
    }
}
------------------------------
200 OK
{
    "success": true
}
```

Even though a single entry point is disadvantageous (Single Point of Failure), it does not necessarily mean it's a bad thing. 
Many well defined Protocols are based on this idea: SOAP, RPC, GraphQL.
Some Teams just don't know any better and go on to invent their own proprietary Standards on top of this Idea, then claiming to build REST Services.
This is not recommended. You want to either take the REST Path, or choose an existing Protocol like one of the above. 
This will make things easier for you and  the users of your API.

### Level 1: Use Resources

Resources are identified by their URI (Uniform Resource Identifier). 

> URL (Uniform Resource Locator) is just a subtype of URI that includes the protocols http(s), ftp, mailto, file and others. 
> In a HTTP Context it's safe to use the terms URI and URL interchangeably.

Resources are nouns and usually named in plural as they represent a collection of entities.

E.g.: 

```
POST http://hsp.com/api/classes
{
    "action": "read",
    "content": [
        {
            "id": "4A",
            "teacher": "Max",
            "room": 123
        }
    ]
}
------------------------------
200 OK
```

It's easy to imagine Resources as if they were Folders in a File System.

We can access a single Entity by providing an id.

```
POST http://hsp.com/api/classes/4A
{
    "action": "read",
}
------------------------------
200 OK
"content": {
    "id": "4A",
    "teacher": "Max",
    "room": 123
}
```

Resources may be nested.
```
POST http://hsp.com/api/classes/4A/students
{
    "action": "read",
}
------------------------------
200 OK
["Markus", "Susan", "Peter"]
```

### Level 2: Use HTTP Methods
Instead of only doing POST, we can utilize other Methods as well.
GET, POST, PUT, PATCH, DELETE, HEAD, ...

#### GET for reading

> ☑ safe - ☑ cache-able - ☑ idempotent

Notice the empty request-body

```
GET http://hsp.com/api/classes/4A
------------------------------
200 OK
{
    "id": "4A",
    "teacher": "Max",
    "room": 123
}
```

Use Query Parameters to filter a Collection
```
GET http://hsp.com/api/classes?teacher=Max
------------------------------
200 OK
[
    {
        "id": "4A",
        "teacher": "Max",
        "room": 123
    }
]
```

#### PUT for changing state on the server

> ☐ safe - ☐ cache-able - ☑ idempotent

Stores the supplied entity under the given URI.
Used for create or update. In case of create, it's the Client who defines the Id.
Advantage over POST: It's Idempotent, meaning we can send the request multiple times while having the same effect as if we were sending it a single time.

```
PUT http://hsp.com/api/classes/4A
{
    "teacher": "Max",
    "room": 123
}
------------------------------
204 No Content
```
Notice how the Response Body is empty.
We can use 204 whenever a Body is superfluous.

> ⚠ The response-body to a GET request does not have to perfectly equal the request-body that was sent to create it. For example: A Server may add additional meta-data like who created it and when to the response.

#### POST to add a new entity to a collection

> ☐ safe - ☐ cache-able - ☐ idempotent

```
POST http://hsp.com/api/rooms
{
    "size": 40
}
------------------------------
201 Created
Location http://hsp.com/api/rooms/1
```
Executing this Request several Times will create as much rooms.
Notice how the server created the rooms id and responds with the created URI. 
The Server tells the Client where it stored the Entity.
Body is empty in this case.
Another Style would be to return a body.

```
200 Ok
{
    "id": 1
    "size": 40
}
```
#### DELETE to delete an entity

> ☐ safe - ☐ cache-able - ☑ idempotent

```
DELETE http://hsp.com/api/rooms/1
------------------------------
204 No Content
```
#### PATCH for partial updates

> ☐ safe - ☐ cache-able - ☐ idempotent

Use it to only change the given fields, and leave all others the same.

```
PATCH http://hsp.com/api/classes/4A
{
    "teacher": "Peter"
}
------------------------------
200 Ok
{
    "teacher": "Peter",
    "room": 123
}
```

PATCH is pretty popular these days.
Probably because of its flexibility, and the ability to send only the parts that are needed.

But it has 3 important disadvantages to PUT:

- It is not uniformly defined. There are many different implementations of PATCH, leaving a Client with confusion.
- Even though you may design it in an idempotent way, it's not so by definition. Intermediaries will treat it like it's not, leaving you without any advantage.
- It typically creates unnecessary complexity in the code, as you have to interact with the State field-wise.

Imo PATCH is not really RESTful and mostly premature optimization. 
Use it only when the content is huge and you cannot reasonably split it to more than one Resource, leaving you with poor PUT performance. 
Otherwise use PUT.

> PATCH was something I created for the initial HTTP/1.1 proposal because partial PUT is never RESTful. ;-)
>
> ~ Roy Fielding

Achieving Level 2 is already pretty Good. It's not yet really REST or RESTful though. It's rather RESTish or just an HTTP JSON API.

### Level 3: Hypermedia

Also called HATEOAS (**H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate).

Means Resources are not identified by their Id, but by their URIs, and we use those URIs inside Contents (Hyperlinks), to guide a Client through the API, its Resources and capabilities. The Goal is to have discover-ability of the complete API through Hyperlinks. The Client will start with a single URI pointing to a Root Resource and discover other Resources from there.

As a result the Client does not have to construct URIs anymore, decoupling it further from the Server. 
The Client does not have to change when an URI changes - it will just use the new URI that it retrieved from the Server.

This even allows us to model State Machines with discoverable capability in our API.

> ⚠️ Adds a technical challenge. ROI might not be given in a small-scale project where server and client is owned by the same organization.

```
GET http://hsp.com/api/classes
------------------------------
200 Ok
[
    {
        "teacher": "Max",
        "room": "Number 123",
        "links": {
            "class": "/api/classes/4A",
            "room": "/api/rooms/123",
            "teacher": "/api/teachers/52323"
        }
    }
]
```

Using complete URIs as links allows the server to even change the Domain without the Client having to change at all.
```
GET http://hsp.com/api/classes
------------------------------
200 Ok
[
    {
        "teacher": "Max",
        "room": 123,
        "links": {
            "class": "http://hsp.com/api/classes/4A",
            "room": "http://hsp.com/api/rooms/123",
            "teacher": "http://hsp.com/api/teachers/52323"
        }
    }
]
```

## Content Negotiation
State can be represented in many formats, JSON is just one of them.
Tree-Based Data Structures are commonly represented using JSON or XML.
CSV works nicely for simple lists.

A Client may tell the Server which MIME-Types it Accepts using the `Accept` header.
```
GET http://www.google.com
Accept: text/html, application/xhtml+xml
```

The Server may then choose one of the accepted formats and use it to send the Content.
```
200 OK
Content-Type: text/html

<html><head>Some Header</head><body> ...
```

Or, when a Server does not Support any of the Accepted MIME-Types, it may answer with a Status Code: 406 Not Acceptable.

It's not uncommon for an API to support multiple formats for a single Resource.
Even when its usually transfered as JSON, we may also create a PDF Representation for printing or XLS to enable a user to make manual calculations.

## HTTP Caching
One of HTTPs greatest strengths is its caching.

### Expiration
A Server may tell a client when a downloaded State expires by providing an `Expires` header.
```
Expires: Wed, 21 Oct 2021 07:30:00 GMT
```
The Client may now reuse this result to avoid repeated requests to the same Resource.

### If-Modified-Since
A Client may tell a Server: "Hey, i already have this Resource downloaded. So, only send it back if it hasn't changed in the meantime". 
This is done providing the `If-Modified-Since` Request Header.

```
If-Modified-Since: Wed, 21 Oct 2021 07:30:00 GMT
```
The Server will then answer with Status `304 Not Modified` and an empty body when it hasn't changed in that time.

### ETag
A Server may add an `ETag` header to a Response, usually a hash code that identifies the current State.
```
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```
The Client may then use the `If-None-Match` header to ask whether the State has changed in the meantime.

```
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```
And if it hasn't, the Server will answer with an empty body and a Status of `304 Not Modified` so that the Client uses its cached version.

## Enabling Safe API Change
First of all, you want to make the participating Machines Robust to API change by following the Robustness Principle.

> ⚠️ Be conservative in sending stuff, but liberal in receiving it.

Then there are a couple Strategies how to make Changes to an API.

### API Versioning
Every non-compatible change creates a new Version via URI (e.g. /api/V1) or Content-Type (e.g. application/vnd.v1+json).
Fits best when you have lots of clients (e.g. Google Maps API).
**Downside**: You have to maintain all those versions.

### Consumer Driven Contracts
Every Client defines their Interactions in the Form of Contracts, which are used to verify both the Server and the Client in isolation to check whether they can communicate with each other.
Works nicely in Micro Service and Macro Service Architectures as there are reasonable amounts of services developed by different teams that can be integrated this way.
Is a no-schema approach.

### Just Change It
Simplest and preferable solution. 
Fits best when you own both the Server and the Client within the same Development Team.
Can use a schema, but don't have to.
May use Parallel Change If Server and Client are deployed independently.

## Tooling
### Clients

#### Curl

The Classic Command Line Client. Not recommended. Use HTTPie instead.

#### HTTPie ([httpie.org](https://httpie.org/))

Great HTTP Command Line Client. Very Intuitive.

#### IntelliJ HTTP Scratch Files

My Personal Favourite. Can use JavaScript to store Client State like OAuth Token between Requests. Commit `*.http` files with the Code. 

#### Postman API Client ([postman.com](https://www.postman.com/))

Makes it easy to manage Collection of Requests. Allows to send Requests on behalf of a Browser that may have Logon Cookies.

#### RestTemplate

Java Class in the Spring Ecosystem to make HTTP Requests.

#### WebClient

The Little Async Brother of RestTemplate.

#### Browser and its Network Tab (F12)

### Documentation

#### Swagger UI ([swagger.io/tools/swagger-ui](https://swagger.io/tools/swagger-ui/))

Documents an API, and provies a UI to try it out.
Swagger is basically a schema.

#### SpringFox ([springfox.github.io/springfox](https://springfox.github.io/springfox/))

Automated JSON API documentation for API's built with Spring. Uses Swagger UI.

#### Spring REST Docs

https://spring.io/projects/spring-restdocs

Generate Documentation out of your MockMvc Tests.

### JSON Related

#### Browser Plugin: [JSON View](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc)

There are others as well.

#### JSONPath

XPath for JSON

https://github.com/json-path/JsonPath

https://jsonpath.com/

#### JQ

https://stedolan.github.io/jq/

A handy CLI JSON Processor to read JSON like XPath reads XML. To try it online: https://jqplay.org/

#### Online JSON Formatters & Validators

https://jsonlint.com/

https://jsonformatter.org/

#### JSON-LD ([json-ld.org](https://json-ld.org/))

A Standard for Hypermedia JSON APIs

### Web Frameworks

#### Spring

- spring-web
- spring-webflux
- spring-hateoas
- spring-data-rest

#### vertx ([vertx.io](https://vertx.io/))

A reactive Java Framework.

#### spark ([sparkjava.com](http://sparkjava.com/))

Popular Java Micro Web Framework.

https://www.baeldung.com/spark-framework-rest-api

#### javalin ([javalin.io](https://javalin.io/))

Lightweight web framework for Java and Kotlin.

### Testing

#### HTTP Test Doubles

##### WireMock ([wiremock.org](http://wiremock.org/))

Popular Java HTTP Mocking/Stubbing/Spying Tool.

##### MockServer ([mock-server.com](https://www.mock-server.com/))

Another Java HTTP Mocking Tool.

##### sinon.js ([sinonjs.org](https://sinonjs.org/how-to/))

A Test Double Tool for JavaScript. Not specifically for HTTP, but allows to stub or spy on HTTP Requests.

#### Testing HTTP Services

##### REST Assured ([rest-assured.io](http://rest-assured.io/))

Java REST Services Testing Tool that provides a fluent API. Alternative to Springs MockMvc.

#### Contract Testing

##### PACT ([pact.io](https://pact.io/))

Polyglot Contract Testing Tool. Open Source, Free To Use, De-facto Standard.

##### Spring Cloud Contract

https://spring.io/projects/spring-cloud-contract

Similar to Pact, specifically for the Spring World

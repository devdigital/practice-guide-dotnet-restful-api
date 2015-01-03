# .NET RESTful API Practice Guide

*Opinionated .NET RESTful API practice guide by [@devdigital](//twitter.com/devdigital)*

## Table of Contents

   1. [REST](#rest)
   2. [Levels of Service](#levels-of-service)
   3. [Authentication](#authentication)

## General Principles   

### REST

TODO

[Web API](http://www.asp.net/web-api) is Microsoft's recommended framework for developing RESTful services.

### URIs

A uniform resource identifier (URI) identifies a resource through a string of characters. A URI can either be a uniform resource name (URN) which identifies a resource by name in a particular namespace, or a uniform resource locator (URL) which both identifies a resource as well as specifying a means of obtaining a representation of the resource. HTTP URIs are examples of URLs.

The syntax for a URL is:

```
scheme://domain:port/path?query_string#fragment_id
```

Where:
* *scheme* - defines how a resource will be obtained. Examples include http, https, ftp, and file. Although schemes are case-insensitive, the canonical form is lowercase.
* *domain* - can be a domain name or a literal numeric IP address. An IPv6 address should be enclosed in *[]*. Case-insensitive as DNS ignores case.
* *port* - decimal port number. If omitted, the default for the scheme is used. This is port 80 for http, and port 443 for https.
* *path* - used to specify or find the resource requested. Case-sensitive although many servers may treat it as case-insensitive.
* *query string* - contains data to be passed to the server. These can be name/value pairs separated by ampersands, e.g. *?first_name=John&last_name=Smith*.
* *fragment identifier* - if present, specifies a position within the overall resource or document. The browser will scroll to the display the element with the specified identifier (id attribute).

### Levels of Service

Leonard Richardson described levels of service maturity that define how 'web friendly' a service is, based on URIs, HTTP, and hypermedia. This is referred to as the [Richardson Maturity Model](http://www.crummy.com/writing/speaking/2008-QCon/act3.html).

#### Level Zero Services

The most basic services (level zero) have a single URI and use a single HTTP method (typically POST). Most WS-* based services are level zero services.

#### Level One Services

Level one services also use a single HTTP method (typically GET), but have many URIs, each representing a logical resource. Because only a single HTTP method is used, it's possible to make changes on the server through the GET verb, which should not have side effects.

#### Level Two Services

The next level of service (level two) hosts many URI addressable resources, and uses many HTTP verbs (e.g. GET, POST, PUT, DELETE) to offer CRUD functionality and HTTP status codes to coordinate interactions. This is the minimum level of service you should aim for when creating a RESTful API, and is the most common level of service when starting with a framework such as Microsoft's Web API.

#### Level Three Services

The most web-aware level of service (level three) adds hypermedia as the engine of application state (HATEOAS). This means that the representations of resources contain URI links to other resources in which consuming clients of the service may be interested. This offers a further level of decoupling between the clients and the API server, as clients are told the location of resources by the service, rather than having to have explict knowledge of them.

[Web API](http://www.asp.net/web-api) allows us to focus on developing *level two services or above*.

### Resources vs Representations

TODO

### HTTP Methods (or Verbs)

HTTP defines methods (also known as *verbs*) to indicate the desired action on a particular resource.

[HTTP/1.0](http://www.w3.org/Protocols/HTTP/1.0/spec.html) defined the GET, POST, and HEAD methods.

[HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616.html) added the OPTIONS, PUT, DELETE, TRACE, and CONNECT methods.

[RFC 5889](http://tools.ietf.org/html/rfc5789) specified the PATCH method.

* *GET* - retreives data
* *HEAD* - identical to GET except the response body should be omitted
* *POST* - request that the server should accept the entity enclosed in the request as a new subordinate of the web resource identified by the URI
* *PUT* - requests that the enclosed entity be stored under the supplied URI, if the resource exists it is modifed, otherwise the server can create the resource with that URI
* *PATCH* - like PUT, but allows partial updates as the enclosed entity describes instructions on how a resource currently residing on the server should be modified, rather than being a complete modified replacement
* *DELETE* - deletes the specified resource
* *TRACE* - echoes back the recieved request so that a client can see what (if any) changes or additions have been made by intermediate servers
* *OPTIONS* - returns the HTTP methods that the server supports for the specified URI
* *CONNECT* - converts the request connection to a transparent TCP/IP tunnel, usually to facilitate SSL-encrypted communication

#### Safe Methods

Some methods are designed for information retrieval only, and are defined as *safe* in that they should not change the state of the server. They should not have any side effects. Safe methods can be cached, prefetched without any repercussions to the resource. Web crawlers also generally only use GET requests to index sites, so it's vital that any GET request to a RESTful API does not have any side effects on the resources accessed.

#### Idempotent Methods

In contrast, an idempotent method does have side effects on the resource, but what differentiates it from a non-safe, non-idempotent method is that if you call an idempotent methods once or call it N times, it has the same outcome. Idempotency is important in building a fault-tolerant API. The client can re-try idempotent methods knowing that the outcome will be the same no matter the number of retries.

Note that all safe methods are also idempotent, but not all idempotent methods are safe.

#### Method Summary

| HTTP Method | Idempotent? | Safe? |
|-------------|-------------|-------|
| HEAD        | Yes         | Yes   |
| OPTIONS     | Yes         | Yes   |
| GET         | Yes         | Yes   |
| POST        | No          | No    |
| PUT         | Yes         | No    |
| DELETE      | Yes         | No    |
| PATCH       | No          | No    |

Note that PATCH is defined as neither idempotent or safe. Depending on how PATCH is implemented, it *could* be idempotent, however it may not be. For example, the entity sent in a PATCH request may define *from* and *to* values for different entity fields, therefore the effect of executing the request once will be different to N times (subsequent times would result in a failure response as the *from* value no longer exists). 

#### CRUD

The HTTP verbs align well with the CRUD (Create, Read, Update, Delete) operations provided by most RESTful APIs.

| CRUD Operation | HTTP Method |
|----------------|-------------|
| Create         | POST        |
| Read           | GET         |
| Update         | PUT         |
| Delete         | DELETE      |

#### POST vs PUT

There is a misconception that the difference between POST and PUT is that POST is used for create/insert, and PUT is used for updates. However, both verbs can be used for either operation. The real difference is the fact that PUT is idempotent, and POST is not.

For example, you may use POST to insert a new user:

```
/users
POST
{
   "name": "john" 
}
```

Note that this operation is not idempotent. If we keep repeating the same request, additional users named 'john' would continue to be added.

The equivalent create using PUT would specify the URI to use for the new user:

```
/users/john
PUT
{
   "name": "john"
}
```

With PUT, if the resource doesn't exist at the specified URI, then it can be created, rather than returning an error response. If the resource does exist, then an update can occur. Note that both of these operations are idempotent.

Performing an update with POST is possible if the URI references an already existing resource:

```
/users/john
POST
{
   "name": "paul"
}
```

Here, the resource exists and will be updated. If the resource did not exist, unlike PUT, the POST request would have an error response. This particular POST operation is idempotent, but POST in the general case is not idempotent because as we've seen, performing a POST against a collection endpoint is not idempotent.

Whilst performing a POST against a collection will continue creating resources in that collection, a PUT against a collection would replace the entire collection:

```
/users
PUT
{
   "users": [ ... ]
}
```

### Cools URIs

What differentiates a URI from a cool URI, is that *a cool URI does not change*.

When developing a level three service, the URIs of associated resources are defined in the response from the server - i.e. the representation of the resource accessed by the client. However, it's still important to use URIs that are 'cool', even if they are generated by the server. It's particularly important if you are developing a level two service, as the client must reference your URIs directly, and therefore it's vital that your URIs do not change, as this would break existing clients. Therefore, the URI should be the driver for your API design.

#### Favour Nouns over Verbs

As HTTP offers HTTP methods or verbs within an HTTP request, a client already has a mechanism available for specifying the action that they wish to take on a resource. Therefore, your URIs should not contain verbs, but rather the nouns that describe the resource that the client will take action on:

**Avoid**
```
/getUser
/deleteUser
```

**Recommended**
```
/users/:username
Use the GET or DELETE HTTP verb within the request to specify the action
```

#### Favour Plurals over Singular

You should use plurals to represent a collection of resources. Clients can perform actions against a specific resource using an additional resource identifier.

**Avoid**
```
/user
/user/:username
```

**Recommended**
```
/users
/users/:username
```

#### Favour lowercase

Although paths within URLs are case-sensitive, you should favour lowercase rather than uppercase or PascalCase:

**Avoid**
```
/Users
/USERS
```

**Recommended**
```
/users
```

#### Favour Hyphens to Separate Path Words

TODO

**Avoid**
```
/orders/:orderid/order_lines
/orders/:orderid/orderLines
```

**Recommended**
```
/orders/:orderid/order-lines
```

#### Favour camelCase in JSON Responses

TODO

**Avoid**
```
{
   "id": 3,
   "order_lines": [ ... ]
}
```

**Recommended**
```
{
   "id": 3,
   "orderLines": [ ... ]
}
```

#### Keep them Clean

You should aim for clean, succient, descriptive URIs that are instinctive to any clients using your API. For example:

```
/users
Retrieve a collection of users.
```

#### Do Not Expose Your Implementation

Take a design led approach rather than an implementation led approach when determining your URIs. In other words, the URIs should be implementation agnostic, and your implementation should not leak through and be exposed in your URIs. In other words, your URIs should not be a leaky abstraction.

**Avoid**
```
/users?ForeName=john&orderby=UpdatedDate
```

Here, *ForeName* and *UpdatedDate* refer to database table columns, and the client is free to sort by any database column. Exposing implementation details like this in your URIs makes it very difficult to change your implementation (e.g. rename a table column) without breaking your clients.

**Recommended**
```
/users?forename=john&sort=update-date
```

Here, forename and sort are design led, and the range of possible domain values for the sort parameter are limited and documented in the API. The URI is implementation agnostic. It may be using a relational database, or it may be using another form of persistence, e.g. a document database. Changing the implementation won't affect the URI, thus making the URI 'cool'.

### CRUD

TODO

## Microsoft Web API

TODO

### Authentication

#### OAuth 2.0

##### Error with user credentials

If the user provides invalid credentials (username or password), the response code should be *400 (Bad Request)*. The response body should contain an *error* parameter that is set to *invalid_request* if the request is malformed, or *invalid_grant* if the credentials are not valid. 

The response body can also contain an optional *error_description* property that contains a human readable description of the error.

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache
    
{
    "error": "invalid_grant",
    "error_description": "The username or password does not exist."
}
```

See https://tools.ietf.org/html/rfc6749#page-45 for full details.
    
When using the [Microsoft OAuth OWIN middleware](https://www.nuget.org/packages/Microsoft.Owin.Security.OAuth), this response can be generated in the `OAuthAuthorizationServerProvider` `GrantResourceOwnerCredentials` method using the `SetError` method of the supplied `OAuthGrantResourceOwnerCredentialsContext`.

```csharp
public override async Task GrantResourceOwnerCredentials(OAuthGrantResourceOwnerCredentialsContext context)
{
    using (var userManager = this.userManagerFactory())
    {
       var user = await userManager.FindAsync(context.UserName, context.Password);
 
       if (user == null)
       {
          context.SetError("invalid_grant", "The username or password does not exist.");
          return;
       }
```

#### Sign out

Use a POST request for sign out.

### CRUD controllers

#### Use Plural Names for API Controllers

```csharp
public class DocumentsController : ApiController
```

## Resources

http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api
https://github.com/interagent/http-api-design

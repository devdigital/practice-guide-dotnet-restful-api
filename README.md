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

### Uris

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

### Cools URIs

What differentiates a URI from a cool URI, is that *a cool URI does not change*.

This is an incredibly important concept in the design of a RESTful API. So much so, that it should be the driver for your API design.

#### Favour Nouns over Verbs

As HTTP offers HTTP methods or verbs within an HTTP request, a client already has a mechanism available for specifying the action that they wish to take on a resource. Therefore, your URIs should not contain verbs, but rather the nouns that describe the resource that the client will take action on:

**Avoid**
```
/api/getUser
/api/deleteUser
```

**Recommended**
```
/api/users/:username
Use the GET or DELETE HTTP verb within the request to specify the action
```

#### Favour Plurals over Singular

You should use plurals to represent a collection of resources. Clients can perform actions against a specific resource using an additional resource identifier.

**Avoid**
```
/api/user
/api/user/:username
```

**Recommended**
```
/api/users
/api/users/:username
```

#### Keep them Clean

You should aim for clean, succient, descriptive URIs that are instinctive to any clients using your API. For example:

```
/api/users
Retrieve a collection of users.
```

#### Do Not Expose Your Implementation

Take a design led approach rather than an implementation led approach when determining your URIs. In other words, the URIs should be implementation agnostic, and your implementation should not leak through and be exposed in your URIs. In other words, your URIs should not be a leaky abstraction.

**Avoid**
```
/api/users?ForeName=john&orderby=UpdatedDate
Where ForeName and UpdatedDate are database table columns
```

**Recommended**
```
/api/users?forename=john&sort=update_date
Here, forename and sort are design led, and the range of possible domain values for the sort parameter are limited and documented in the API
```

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

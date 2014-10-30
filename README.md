# .NET RESTful API Practice Guide

*Opinionated .NET RESTful API practice guide by [@devdigital](//twitter.com/devdigital)*

## Table of Contents

   1. [REST](#rest)
   2. [Levels of Service](#levels-of-service)
   3. [Authentication](#authentication)
   
## REST

TODO

[Web API](http://www.asp.net/web-api) is Microsoft's recommended framework for developing RESTful services.

## Levels of Service

Leonard Richardson described levels of service maturity that define how 'web friendly' a service is, based on URIs, HTTP, and hypermedia. This is referred to as the [Richardson Maturity Model](http://www.crummy.com/writing/speaking/2008-QCon/act3.html).

### Level Zero Services

TODO

### Level One Services

TODO

### Level Two Services

TODO

### Level Three Services

[Web API](http://www.asp.net/web-api) allows us to focus on developing *level two services or above*.

## Authentication

### OAuth 2.0

#### Error with user credentials

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

### Sign out

Use a POST request for sign out.

## CRUD controllers

### Use Plural Names for API Controllers

```csharp
public class DocumentsController : ApiController
```

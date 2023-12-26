# Discover Microsoft Graph
Microsoft Graph is the gateway to data and intelligence in Microsoft 365. It provides a unified programmability model that you can use to access the tremendous amount of data in Microsoft 365, Windows 10, and Enterprise Mobility + Security.
![ms graph](https://learn.microsoft.com/en-us/training/wwl-azure/microsoft-graph/media/microsoft-graph-data-connectors.png)

In the Microsoft 365 platform, three main components facilitate the access and flow of data:

- The Microsoft Graph API offers a single endpoint, `https://graph.microsoft.com`. You can use REST APIs or SDKs to access the endpoint. Microsoft Graph also includes a powerful set of services that manage user and device identity, access, compliance, security, and help protect organizations from data leakage or loss.
    
- [Microsoft Graph connectors](https://learn.microsoft.com/en-us/microsoftsearch/connectors-overview) work in the incoming direction, **delivering data external to the Microsoft cloud into Microsoft Graph services and applications**, to enhance Microsoft 365 experiences such as Microsoft Search. Connectors exist for many commonly used data sources such as Box, Google Drive, Jira, and Salesforce.
    
- [Microsoft Graph Data Connect](https://learn.microsoft.com/en-us/graph/overview#access-microsoft-graph-data-at-scale-using-microsoft-graph-data-connect) provides a set of tools to streamline secure and scalable **delivery of Microsoft Graph data to popular Azure data stores**. The cached data serves as data sources for Azure development tools that you can use to build intelligent applications.
# Query Microsoft Graph using REST
Microsoft Graph is a RESTful web API that enables you to access Microsoft Cloud service resources. After you register your app and get authentication tokens for a user or service, you can make requests to the Microsoft Graph API.

The Microsoft Graph API defines most of its resources, methods, and enumerations in the OData namespace, `microsoft.graph`, in the [Microsoft Graph metadata](https://learn.microsoft.com/en-us/graph/traverse-the-graph#microsoft-graph-api-metadata). A few API sets are defined in their sub-namespaces, such as the [call records API](https://learn.microsoft.com/en-us/graph/api/resources/callrecords-api-overview) which defines resources like [callRecord](https://learn.microsoft.com/en-us/graph/api/resources/callrecords-callrecord) in `microsoft.graph.callRecords`.

Unless explicitly specified in the corresponding topic, assume types, methods, and enumerations are part of the `microsoft.graph` namespace.

## Calling REST API methods
To read from or write to a resource such as a user or an email message, construct a request that looks like the following:
```http
{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}
```
## Resources
A resource can be an entity or complex type, commonly defined with properties. Entities differ from complex types by always including an **id** property.

Your URL includes the resource you're interacting with in the request, such as `me`, **user**, **group**, **drive**, and **site**. Often, top-level resources also include _relationships_, which you can use to access other resources, like `me/messages` or `me/drive`. You can also interact with resources using _methods_; for example, to send an email, use `me/sendMail`.

Each resource might require different permissions to access it. You often need a higher level of permissions to create or update a resource than to read it. For details about required permissions, see the method reference topic.

## Query parameters
Query parameters can be OData system query options, or other strings that a method accepts to customize its response.

You can use optional OData system query options to include more or fewer properties than the default response, filter the response for items that match a custom query, or provide another parameters for a method.

For example, adding the following `filter` parameter restricts the messages returned to only those with the `emailAddress` property of `jon@contoso.com`.

```http
GET https://graph.microsoft.com/v1.0/me/messages?filter=emailAddress eq 'jon@contoso.com'
```

# Query  Microsoft Graph by using SDK
The Microsoft Graph SDKs are designed to simplify building high-quality, efficient, and resilient applications that access Microsoft Graph. The SDKs include two components: a service library and a core library.

The service library contains models and request builders that are generated from Microsoft Graph metadata to provide a rich, strongly typed, and discoverable experience when working with the many datasets available in Microsoft Graph.

The core library provides a set of features that enhance working with all the Microsoft Graph services. Embedded support for retry handling, secure redirects, transparent authentication, and payload compression, improve the quality of your application's interactions with Microsoft Graph, with no added complexity, while leaving you completely in control. The core library also provides support for common tasks such as paging through collections and creating batch requests.

# Apply best practice for Microsoft Graph

## Authentication
To access the data in Microsoft Graph, your application needs to acquire an OAuth 2.0 access token, and present it to Microsoft Graph in either of the following methods:

- The HTTP _Authorization_ request header, as a _Bearer_ token
- The graph client constructor, when using a Microsoft Graph client library

## Consent and Authorization
- **Use least privilege**. Only request permissions that are necessary, and only when you need them. For the APIs, your application calls check the permissions section in the method topics. For example, see [creating a user](https://learn.microsoft.com/en-us/graph/api/user-post-users) and choose the least privileged permissions.
    
- **Use the correct permission type based on scenarios**. If you're building an interactive application where a signed in user is present, your application should use _delegated_ permissions. If, however, your application runs without a signed-in user, such as a background service or daemon, your application should use application permissions.
- **Consider the end user and admin experience**. This will directly affect end user and admin experiences. For example:
    - Consider who will be consenting to your application, either end users or administrators, and configure your application to [request permissions appropriately](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-permissions-and-consent).      
    - Ensure that you understand the difference between [static, dynamic and incremental consent](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-permissions-and-consent#consent-types)

- **Consider multi-tenant applications**. Expect customers to have various application and consent controls in different states. For example:
    
    - Tenant administrators can disable the ability for end users to consent to applications. In this case, an administrator would need to consent on behalf of their users.
        
    - Tenant administrators can set custom authorization policies such as blocking users from reading other user's profiles, or limiting self-service group creation to a limited set of users. In this case, your application should expect to handle 403 error response when acting on behalf of a user.
## Handle response effectively
Depending on the requests you make to Microsoft Graph, your applications should be prepared to handle different types of responses. The following are some of the most important practices to follow to ensure that your application behaves reliably and predictably for your end users. For example:

- **Pagination**: When querying resource collections, you should expect that Microsoft Graph will return the result set in multiple pages, due to server-side page size limits. Your application should **always** handle the possibility that the responses are paged in nature, and use the `@odata.nextLink` property to obtain the next paged set of results, until all pages of the result set have been read. The final page won't contain an `@odata.nextLink` property. For more information, visit [paging](https://learn.microsoft.com/en-us/graph/paging).
    
- **Evolvable enumerations**: Adding members to existing enumerations can break applications already using these enums. Evolvable enums are a mechanism that Microsoft Graph API uses to add new members to existing enumerations without causing a breaking change for applications. By default, a GET operation returns only known members for properties of evolvable enum types and your application needs to handle only the known members. If you design your application to handle unknown members as well, you can opt in to receive those members by using an HTTP `Prefer` request header.
## Storing data locally

Your application should ideally make calls to Microsoft Graph to retrieve data in real time as necessary. You should only cache or store data locally necessary for a specific scenario, and if that use case is covered by your terms of use and privacy policy, and doesn't violate the [Microsoft APIs Terms of Use](https://learn.microsoft.com/en-us/legal/microsoft-apis/terms-of-use?context=/graph/context). Your application should also implement proper retention and deletion policies.


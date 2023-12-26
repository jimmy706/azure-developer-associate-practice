#cache #system_design
# Cache-Aside pattern
> **Reference:** https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside

Load data on demand into a cache from a data store. This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.
## How it works?
Many commercial caching systems provide read-through and write-through/write-behind operations. In these systems, an application retrieves data by referencing the cache. If the data isn't in the cache, it's retrieved from the data store and added to the cache. Any modifications to data held in the cache are automatically written back to the data store as well.
![cache-aside pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/_images/cache-aside-diagram.png)

## When to uses
- A cache doesn't provide native read-through and write-through operations.
- Resource demand is unpredictable. This pattern enables applications to load data on demand. It makes no assumptions about which data an application will require in advance.

# Static content hosting pattern
> **Reference:** https://learn.microsoft.com/en-us/azure/architecture/patterns/static-content-hosting

Deploy static content to a cloud-based storage service that can deliver them directly to the client. This can reduce the need for potentially expensive compute instances.

## How it works?
In most cloud hosting environments, you can put some of an application's resources and static pages in a storage service. The storage service can serve requests for these resources, reducing load on the compute resources that handle other web requests. The cost for cloud-hosted storage is typically much less than for compute instances.

> **Note:** When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.

![static-content hosting](https://learn.microsoft.com/en-us/azure/architecture/patterns/_images/static-content-hosting-pattern.png)

## When to uses
- Minimizing the hosting cost for websites and applications that contain some static resources.
    
- Minimizing the hosting cost for websites that consist of only static content and resources. Depending on the capabilities of the hosting provider's storage system, it might be possible to entirely host a fully static website in a storage account.
    
- Exposing static resources and content for applications running in other hosting environments or on-premises servers.
    
- Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.
    
- Monitoring costs and bandwidth usage. Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.
# Backend for Frontend pattern
> **Reference:** https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends

Create separate backend services to be consumed by specific frontend applications or interfaces. This pattern is useful when you want to avoid customizing a single backend for multiple interfaces. This pattern was first described by [Sam Newman](https://samnewman.io/patterns/architectural/bff/)

## How it works?
Create one backend per user interface. Fine-tune the behavior and performance of each backend to best match the needs of the frontend environment, without worrying about affecting other frontend experiences.

Because each backend is specific to one interface, it can be optimized for that interface. As a result, it will be smaller, less complex, and likely faster than a generic backend that tries to satisfy the requirements for all interfaces. Each interface team has autonomy to control their own backend and doesn't rely on a centralized backend development team. This gives the interface team flexibility in language selection, release cadence, prioritization of workload, and feature integration in their backend.

![backend for frontend pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/_images/backend-for-frontend-example.png)

## When to uses?
- A shared or general purpose backend service must be maintained with significant development overhead.
- You want to optimize the backend for the requirements of specific client interfaces.
- Customizations are made to a general-purpose backend to accommodate multiple interfaces.
- An alternative language is better suited for the backend of a different user interface.

# Circuit breaker pattern

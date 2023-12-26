# Explore Azure cache for Redis
Azure Cache for Redis provides an in-memory data store based on the [Redis](https://redis.io/) software. Redis improves the performance and scalability of an application that uses backend data stores heavily. It's able to process large volumes of application requests by keeping frequently accessed data in the server memory, which can be written to and read from quickly. Redis brings a critical low-latency and high-throughput data storage solution to modern applications.

Azure Cache for Redis offers both the Redis open-source (OSS Redis) and a commercial product from Redis Labs (Redis Enterprise) as a managed service. It provides secure and dedicated Redis server instances and full Redis API compatibility. The service is operated by Microsoft, hosted on Azure, and usable by any application within or outside of Azure.

## Key scenarios
Azure Cache for Redis improves application performance by supporting common application architecture patterns. Some of the most common include the following patterns:

|Pattern|Description|
|---|---|
|Data cache|Databases are often too large to load directly into a cache. It's common to use the [cache-aside](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside) pattern to load data into the cache only as needed. When the system makes changes to the data, the system can also update the cache, which is then distributed to other clients.|
|Content cache|Many web pages are generated from templates that use static content such as headers, footers, banners. These static items shouldn't change often. Using an in-memory cache provides quick access to static content compared to backend datastores.|
|Session store|This pattern is commonly used with shopping carts and other user history data that a web application might associate with user cookies. Storing too much in a cookie can have a negative effect on performance as the cookie size grows and is passed and validated with every request. A typical solution uses the cookie as a key to query the data in a database. Using an in-memory cache, like Azure Cache for Redis, to associate information with a user is faster than interacting with a full relational database.|
|Job and message queuing|Applications often add tasks to a queue when the operations associated with the request take time to execute. Longer running operations are queued to be processed in sequence, often by another server. This method of deferring work is called task queuing.|
|Distributed transactions|Applications sometimes require a series of commands against a backend data-store to execute as a single atomic operation. All commands must succeed, or all must be rolled back to the initial state. Azure Cache for Redis supports executing a batch of commands as a single [transaction](https://redis.io/topics/transactions).|

## Service tiers

|Tier|Description|
|---|---|
|Basic|An OSS Redis cache running on a single VM. This tier has no service-level agreement (SLA) and is ideal for development/test and noncritical workloads.|
|Standard|An OSS Redis cache running on two VMs in a replicated configuration.|
|Premium|High-performance OSS Redis caches. This tier offers higher throughput, lower latency, better availability, and more features. Premium caches are deployed on more powerful VMs compared to the VMs for Basic or Standard caches.|
|Enterprise|High-performance caches powered by Redis Labs' Redis Enterprise software. This tier supports Redis modules including RediSearch, RedisBloom, and RedisTimeSeries. Also, it offers even higher availability than the Premium tier.|
|Enterprise Flash|Cost-effective large caches powered by Redis Labs' Redis Enterprise software. This tier extends Redis data storage to nonvolatile memory, which is cheaper than DRAM, on a VM. It reduces the overall per-GB memory cost.|

# Configure Azure Cache for Redis

## Create and configure an Azure Cache for Redis instance

There are several parameters you need to decide in order to configure the cache properly for your purposes.

### Name
The Redis cache needs a globally unique name. The name has to be unique within Azure because it's used to generate a public-facing URL to connect and communicate with the service.

The name must be between 1 and 63 characters, composed of numbers, letters, and the '-' character. The cache name can't start or end with the '-' character, and consecutive '-' characters aren't valid.

### Location
You should always place your cache instance and your application in the same region. Connecting to a cache in a different region can significantly increase latency and reduce reliability. If you're connecting to the cache outside of Azure, then select a location close to where the application consuming the data is running.

### Cache Type
The tier determines the size, performance, and features that are available for the cache. For more information, visit [Azure Cache for Redis pricing](https://azure.microsoft.com/pricing/details/cache/).

### Clustering support
With the **Premium, Enterprise, and Enterprise Flash** tiers you can implement clustering to automatically split your dataset among multiple nodes. To implement clustering, you specify the number of shards to a maximum of 10. The cost incurred is the cost of the original node, multiplied by the number of shards.

## Access the Redis instance
Redis has a command-line tool for interacting with an Azure Cache for Redis as a client. The tool is available for Windows platforms by downloading the [Redis command-line tools for Windows](https://github.com/MSOpenTech/redis/releases/). If you want to run the command-line tool on another platform, download Azure Cache for Redis from [https://redis.io/download](https://redis.io/download).

Some common commands on Redis are:

|Command|Description|
|---|---|
|`ping`|Ping the server. Returns "PONG".|
|`set [key] [value]`|Sets a key/value in the cache. Returns "OK" on success.|
|`get [key]`|Gets a value from the cache.|
|`exists [key]`|Returns '1' if the **key** exists in the cache, '0' if it doesn't.|
|`type [key]`|Returns the type associated to the value for the given **key**.|
|`incr [key]`|Increment the given value associated with **key** by '1'. The value must be an integer or double value. This returns the new value.|
|`incrby [key] [amount]`|Increment the given value associated with **key** by the specified amount. The value must be an integer or double value. This returns the new value.|
|`del [key]`|Deletes the value associated with the **key**.|
|`flushdb`|Delete _all_ keys and values in the database.|

Some example commands are:
```bash
> set somekey somevalue
OK
> get somekey
"somevalue"
> exists somekey
(string) 1
> del somekey
(string) 1
> exists somekey
(string) 0
```
## Adding an expiration times to values
Caching is important because it allows us to store commonly used values in memory. However, we also need a way to expire values when they're stale. In Redis this is done by applying a time to live (TTL) to a key.

When the TTL elapses, the key is automatically deleted, exactly as if the DEL command were issued. Here are some notes on TTL expirations.

- Expirations can be set using seconds or milliseconds precision.
- The expire time resolution is always 1 millisecond.
- Information about expires are replicated and persisted on disk, the time virtually passes when your Redis server remains stopped (this means that Redis saves the date when a key expires).

```bash
> set counter 100
OK
> expire counter 5
(integer) 1
> get counter
100
... wait ...
> get counter
(nil)
```

## Accessing Redis Cache from client
To connect to an Azure Cache for Redis instance, you need several pieces of information. Clients need the host name, port, and an access key for the cache. You can retrieve this information in the Azure portal through the **Settings > Access Keys** page.

- The host name is the public Internet address of your cache, which was created using the name of the cache. For example, `sportsresults.redis.cache.windows.net`.
    
- The access key acts as a password for your cache. There are two keys created: primary and secondary. You can use either key, two are provided in case you need to change the primary key. You can switch all of your clients to the secondary key, and regenerate the primary key. This would block any applications using the original primary key. Microsoft recommends periodically regenerating the keys - much like you would your personal passwords.

